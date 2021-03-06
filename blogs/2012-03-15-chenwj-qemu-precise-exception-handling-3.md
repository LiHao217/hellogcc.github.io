转换时间： 2021-01-31

# QEMU Internal – Precise Exception Handling 3/5
Posted on 2012/03/15 by chenwj

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

首先我們來看 `__stb_mmu`。請看 `${SRC}/softmmu_template.h` 和 `${BUILD}/i386-softmmu/op_helper.i``。SOFTMMU` 相關的 helper function 是透過 `softmmu_*` 檔案內的巨集加以合成。這裡只挑部分加以描述。

SUFFIX 可以是 `b (byte, 8)`、`w (word, 16)`、`l (long word, 32)` 和 `q (quad word, 64)`，代表資料大小。MMUSUFFIX 可以是 `cmmu` 或是 `mmu`，分別代表欲讀取的是 code 或是 data。mmu_idx 代表索引的是內核態亦或是用戶態的 TLB。addr 代表 guest virtual address。
```
// ${SRC}/softmmu_template.h
void REGPARM glue(glue(__st, SUFFIX), MMUSUFFIX)(target_ulong addr,
                                                 DATA_TYPE val,
                                                 int mmu_idx)
{
   ...
}
```
接著看展開巨集後的函式體。
```
// ${BUILD}/i386-softmmu/op_helper.i
void __stb_mmu(target_ulong addr, uint8_t val, int mmu_idx)
{
 redo:
    // 查找 TLB
    tlb_addr = env-&gt;tlb_table[mmu_idx][index].addr_write;
    if (...) {

        // TLB 命中

    } else {

        // TLB 不命中

        /* the page is not in the TLB : fill it */
        // retaddr = GETPC();
        retaddr = ((void *)((unsigned long)__builtin_return_address(0) - 1));

        // 試圖填入 TLB entry。
        tlb_fill(addr, 1, mmu_idx, retaddr);
        goto redo;
    }
}
```
這裡 QEMU 利用 GCC 的 `__builtin_return_address` 擴展 [1] 來判定 tlb_fill 是從一般 C 函式或是 code cache 中被呼叫。retaddr 若為 0，表前者，retaddr 若不為 0，表後者。之後，我們會透過 GDB 更加清楚前面所述所代表的意思。我們關注 retaddr 不為 0，也就是從 code cache 中呼叫 tlb_fill 的情況。

在看 tlb_fill 之前，我們先偷看 cpu_x86_handle_mmu_fault (target-i386/helper.c) 的註解。我們關注返回值為 1，也就是頁缺失的情況。
```
/* return value:
   -1 = cannot handle fault
   0  = nothing more to do
   1  = generate PF fault
*/
int cpu_x86_handle_mmu_fault(CPUX86State *env, target_ulong addr, ...)
{
  ...
}

我們來看 tlb_fill。

void tlb_fill(target_ulong addr, int is_write, int mmu_idx, void *retaddr)
{
    ret = cpu_x86_handle_mmu_fault(env, addr, is_write, mmu_idx, 1);
    if (ret) {
        if (retaddr) {

            // 當客戶發生頁缺失 (ret == 1) 且 tlb_fill 是從 code cache 中被
            // 呼叫 (retaddr != 0)，我們會在這裡。

            /* now we have a real cpu fault */
            pc = (unsigned long)retaddr;
            tb = tb_find_pc(pc);
            if (tb) {
                /* the PC is inside the translated code. It means that we have
                   a virtual CPU fault */
                cpu_restore_state(tb, env, pc, NULL);
            }
        }
        raise_exception_err(env-&gt;exception_index, env-&gt;error_code);
    }
    env = saved_env;
}
```
請注意! 如果 retaddr != 0，其值代表的 (幾乎) 是發生例外的 host binary 所在位址。QEMU 利用它來查找是哪一個 TranslationBlock 中的 host binary 發生例外。tb_find_pc (exec.c) 利用該 host binary pc 進行查找，取得 tb。
```
TranslationBlock *tb_find_pc(unsigned long tc_ptr)
{
    // tbs 是 TranslationBlock * 數組。每一個在 code cache 中 (已翻譯好的)
    // basic block 都有相對應的 TranslationBlock 存放其相關資訊。

    /* binary search (cf Knuth) */
    m_min = 0;
    m_max = nb_tbs - 1;
    while (m_min &gt; 1;
        tb = &amp;tbs[m];
        // tc_ptr 代表 host binary 在 code cache 的起始位址。
        v = (unsigned long)tb-&gt;tc_ptr;
        if (v == tc_ptr)
            return tb;
        else if (tc_ptr &lt; v) {
            m_max = m - 1;
        } else {
            m_min = m + 1;
        }
    }
    return &amp;tbs[m_max];
}
```
一但找到該負責的 tb，QEMU 就會回復 guest CPUState 以便 guest exception handler 處理 guest 的頁缺失例外。
```
    if (tb) {
        /* the PC is inside the translated code. It means that we have
           a virtual CPU fault */
        cpu_restore_state(tb, env, pc, NULL);
    }
```
接著我們看 cpu_restore_state (translate-all.c)。

[1] http://gcc.gnu.org/onlinedocs/gcc/Return-Address.html
