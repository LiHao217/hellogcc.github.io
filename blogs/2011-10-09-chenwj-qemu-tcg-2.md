原始博客链接：

转换时间：
2021-01-31

注意：由于2011年HelloGCC博客遭遇了一次VPS掉电丢失，所有2011年之前（含）的博客都是从 Google Reader 等RSS中恢复的。损失了大部分的内容，非常的遗憾。如果你的电脑中保存过HelloGCC2012年之前的报告，欢迎发送给我们，非常感谢。请转发给 hellogcc.workgroup@gmail.com

# QEMU Internal – Tiny Code Generator (TCG) 2/2
Posted on 2011/10/09 by chenwj

Copyright (c) 2011 陳韋任 (Chen Wei-Ren)

1.2 TCG Flow

介紹完一些資料結構之後，我開始介紹 cpu_exec 的流程。底下複習一下 process mode 的流程:

```
cpu_loop (linux-user/main.c) -> cpu_x86_exec/cpu_exec (cpu-exec.c)
```

`cpu_exec` 有兩層 `for` 迴圈。我們先看內層:

```
next_tb = 0; /* force lookup of first TB */
for(;;) {

    tb = tb_find_fast();

    tc_ptr = tb->tc_ptr;

    next_tb = tcg_qemu_tb_exec(tc_ptr);
}
```

- tb_find_fast 會先試圖查看目前 pc (guest virtual address) 是否已有翻譯過的 host binary 存放在 code cache。
```
    // pc = eip + cs_base
    cpu_get_tb_cpu_state(env, &pc, &cs_base, &flags);

    // CPUState 中的 tb_jmp_cache 即是做此用途。
    tb = env->tb_jmp_cache[tb_jmp_cache_hash_func(pc)];

    // 檢查該 tb 是否合法。這是因為不同的 eip + cs_base 可能會得到相同的 pc。
    if (unlikely(!tb || tb->pc != pc || tb->cs_base != cs_base ||
                   tb->flags != flags)) {
            tb = tb_find_slow(pc, cs_base, flags);
    }

    // code cache 已有翻譯過的 host binary，返回 TranslationBlock。
    return tb;
```
- tb_find_slow 以 pc 對映的物理位址 (guest physcal address) 查找 TB。如果成功，則將該 TB 寫入 env->tb_jmp_cache; 若否，則進行翻譯。
```
    phys_pc = get_page_addr_code(env, pc);
    phys_page1 = phys_pc & TARGET_PAGE_MASK;

    // 除了 env->tb_jmp_cache 這個以 guest virtual address 為索引的緩存之外，
    // QEMU 還維護了一個 tb_phys_hash，這個是以 guest physical address 為索引。
    h = tb_phys_hash_func(phys_pc);
    ptb1 = &tb_phys_hash[h];
    for (;;) {

      not_found:

      found:

      // TranslationBlock 中的 phys_hash_next 用在這裡。
      // 如果 phys_pc 索引到同一個 tb_phys_hash 欄位，用 phys_hash_next 串接起來。
      ptb1 = &tb->phys_hash_next;
    }

    not_found:
      tb = tb_gen_code(env, pc, cs_base, flags, 0);

    found:
      env->tb_jmp_cache[tb_jmp_cache_hash_func(pc)] = tb;
      return tb;
```
- 這裡小結一下流程。
```
    cpu_exec (cpu-exec.c) -> tb_find_fast (cpu-exec.c)
      -> tb_find_slow (cpu-exec.c)
```

QEMU 先以 guest virtual address (GVA) 查找是否已有翻譯過的 TB，再以 guest physical address (GPA) 查找是否已有翻譯過的 TB。

如果沒有翻譯過的 TB，開始進行 guest binary -> TCG IR -> host binary 的翻譯。大致流程如下:
```
    tb_gen_code (exec.c) -> cpu_gen_code (translate-all.c)
      -> gen_intermediate_code (target-i386/translate.c)
      -> tcg_gen_code (tcg/tcg.c) -> tcg_gen_code_common (tcg/tcg.c)
```
- tb_gen_code 配置內存給 TB (TranslationBlock)，再交由 cpu_gen_code。
```
  // 注意! 這裡會將 GVA 轉成 GPA。phys_pc 將交給之後的 tb_link_page 使用。
  phys_pc = get_page_addr_code(env, pc);
  tb = tb_alloc(pc);
  if (!tb) {
    // 清空 code cache
  }

  // 初始 tb

  // 開始 guest binary -> TCG IR -> host binary 的翻譯。
  cpu_gen_code(env, tb, &code_gen_size);

  // 將 tb 加入 tb_phys_hash 和二級頁表 l1_map。
  // phys_pc 和 phys_page2 分別代表 tb (guest pc) 對映的 GPA 和所屬的第二個
  // 頁面 (如果 tb 代表的 guest binary 跨頁面的話)。
  tb_link_page(tb, phys_pc, phys_page2);
  return tb;
```
我底下分別針對 cpu_gen_code 和 tb_link_page 稍微深入的介紹一下。

- cpu_gen_code 負責 guest binary -> TCG IR -> host binary 的翻譯。
```
        // 初始 TCGContext 的 gen_opc_ptr 和 gen_opparam_ptr，使其分別指向
        // gen_opc_buf 和 gen_opparam_buf。gen_opc_buf 和 gen_opparam_buf
        // 分別存放 TCGOpcode 和 operand。
        tcg_func_start(s);

        // 呼叫 gen_intermediate_code_internal 產生 TCG IR
        gen_intermediate_code(env, tb);

        // TCG IR -> host binary
        gen_code_size = tcg_gen_code(s, gen_code_buf);
```
- gen_intermediate_code_internal (`target-*/translate.c`) 初始化並呼叫 `disas_insn` 反組譯 guest binary 成 TCG `IR。disas_insn` 呼叫 `tcg_gen_xxx` (`tcg/tcg-op.h)` 產生 TCG IR。分別將 opcode 寫入 `gen_opc_ptr` 指向的緩衝區 (`translate-all.c` 裡的 `gen_opc_buf`); operand 寫入 `gen_opparam_ptr` 指向的緩衝區 `(translate-all.c` 裡的 `gen_opparam_buf`)。
- `tcg_gen_code` (`tcg/tcg.c`) 呼叫 `tcg_gen_code_common` `(tcg/tcg.c`) 將 TCG IR 轉成 host binary。
```
        tcg_reg_alloc_start(s);

        s->code_buf = gen_code_buf;
        // host binary 會寫入 TCGContext s 的 code_ptr 所指向的緩衝區。
        s->code_ptr = gen_code_buf;
```
至此，guest binary -> TCG IR -> host binary 算是完成了。剩下把 `TranslationBlock` (TB) 納入 QEMU 的管理，這是 tb_link_page 做的事。
- tb_link_page (exec.c) 把新的 TB 加進 tb_phys_hash 和 l1_map 二級頁表。 tb_find_slow 會用 pc 對映的 GPA 的哈希值索引 tb_phys_hash。
```
        /* 把新的 TB 加進 tb_phys_hash */
        h = tb_phys_hash_func(phys_pc);
        ptb = &tb_phys_hash[h];
        // 如果兩個以上的 TB 其 phys_pc 的哈希值相同，則做 chaining。
        tb->phys_hash_next = *ptb;
        *ptb = tb; // 新加入的 TB 放至 chaining 的開頭。

        // 在 l1_map 中配置 PageDesc 給 TB，並設置 TB 的 page_addr 和 page_next。
        tb_alloc_page(tb, 0, phys_pc & TARGET_PAGE_MASK);
        if (phys_page2 != -1) // TB 對應的 guest binary 跨頁
            tb_alloc_page(tb, 1, phys_page2);
        else
            tb->page_addr[1] = -1;

        // 以下和 block chaining 有關，留待下次再講，這邊暫且不提。
        tb->jmp_first = (TranslationBlock *)((long)tb | 2);
```
- tb_alloc_page (exec.c) 設置 TB 的 page_addr 和 page_next，並在 l1_map 中配置 PageDesc 給 TB。
```
        static inline void tb_alloc_page(TranslationBlock *tb,
                                         unsigned int n, tb_page_addr_t page_addr)
        {
          // 代表 tb (guest binary) 所屬 guest page。
          tb->page_addr[n] = page_addr;
          // 在 l1_map 中配置一個 PageDesc，返回該 PageDesc。
          p = page_find_alloc(page_addr >> TARGET_PAGE_BITS, 1);
          // 將該頁面目前第一個 TB 串接到此 TB。將來有需要將某頁面所屬所有 TB 清空。
          tb->page_next[n] = p->first_tb;
          // n 為 1 代表 tb 對應的 guest binary 跨 page。
          p->first_tb = (TranslationBlock *)((long)tb | n);
          // PageDesc 會維護一個 bitmap，這是給 SMC 之用。這裡不提。
          invalidate_page_bitmap(p);
        }
```
這裡先回顧一下，QEMU 查看當前 env->pc 是否已翻譯過。若否，則進行翻譯。
```
tb_find_fast (cpu-exec.c) -> tb_find_slow (cpu-exec.c)
  -> tb_gen_code (exec.c)
```
tb_gen_code 講到這裡，guest binary -> host binary 已翻譯完成，相關資料結構也已設置完畢。返回 TB (`TranslationBlock *`) 給 tb_find_fast。
```
tb = tb_find_fast();

tc_ptr = tb->tc_ptr; // tc_ptr 指向 code cache (host binary)

next_tb = tcg_qemu_tb_exec(tc_ptr);
```
很好，我們準備從 QEMU 跳入 code cache 開始執行了。:-) `tcg_qemu_tb_exec` 被定義在 tcg/tcg.h。
```
#define tcg_qemu_tb_exec(tb_ptr) ((long REGPARM (*)(void *))code_gen_prologue)(tb_ptr)
```
`(long REGPARM (*)(void *))` 將 `code_gen_prologue` 轉型成函式指針，`void *` 為該函式的參數，返回值為 long。REGPARM 指示 GCC 此函式透過暫存器而非棧傳遞參數。至此，`(long REGPARM (*)(void *))` 將數組指針 `code_gen_prologue` `轉型成函式指針。tb_ptr` `為該函式指針的參數。綜合以上所述，code_gen_prologue` 被視為一函式，其參數為 `tb_ptr，返回當前` TB (`tc_ptr` 代表的 TB，等講到 block chaining 會比較清楚)`。code_gen_prologue` 所做的事為一般函式呼叫前的 prologue，之後將控制交由 `tc_ptr` 指向的 host binary 並開始執行。
