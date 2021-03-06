转换时间： 2021-01-31

# QEMU Internal – Precise Exception Handling 5/5
Posted on 2012/03/15 by chenwj

Copyright (c) 2012 陳韋任 (Chen Wei-Ren)

最後，如同我之前所承諾的。我們來看 tlb_fill 從一般 C 函式和 code cache 被呼叫是什麼意思。

我們可以看到 retaddr == 0 時，tlb_fill 是從一般 C 函式被呼叫。

```
(gdb) b tlb_fill
(gdb) r -boot a -fda Image -hda hdc-0.11-new.img -vnc 0.0.0.0:1 -d in_asm,op,out_asm
Breakpoint 1, tlb_fill (addr=4294967280, is_write=2, mmu_idx=0, retaddr=0x0) at
/tmp/chenwj/qemu-0.13.0/target-i386/op_helper.c:4816
4816    {
(gdb) bt
#0  tlb_fill (addr=4294967280, is_write=2, mmu_idx=0, retaddr=0x0) at /tmp/chenwj/qemu-0.13.0/target-i386/op_helper.c:4816
#1  0x000000000050ee86 in __ldb_cmmu (addr=4294967280, mmu_idx=0) at /tmp/chenwj/qemu-0.13.0/softmmu_template.h:134
#2  0x000000000051045e in ldub_code (ptr=4294967280) at /tmp/chenwj/qemu-0.13.0/softmmu_header.h:87
#3  0x000000000051054b in get_page_addr_code (env1=0x110e390, addr=4294967280) at /tmp/chenwj/qemu-0.13.0/exec-all.h:325
#4  0x0000000000510986 in tb_find_slow (pc=4294967280, cs_base=4294901760, flags=68) at /tmp/chenwj/qemu-0.13.0/cpu-exec.c:139
#5  0x0000000000510b9d in tb_find_fast () at /tmp/chenwj/qemu-0.13.0/cpu-exec.c:188
#6  0x00000000005112db in cpu_x86_exec (env1=0x110e390) at /tmp/chenwj/qemu-0.13.0/cpu-exec.c:575
#7  0x000000000040aabd in qemu_cpu_exec (env=0x110e390) at /tmp/chenwj/qemu-0.13.0/cpus.c:767
#8  0x000000000040abc4 in cpu_exec_all () at /tmp/chenwj/qemu-0.13.0/cpus.c:795
#9  0x000000000056e417 in main_loop () at /tmp/chenwj/qemu-0.13.0/vl.c:1329
#10 0x00000000005721cc in main (argc=11, argv=0x7fffffffe1a8, envp=0x7fffffffe208) at /tmp/chenwj/qemu-0.13.0/vl.c:2992
```
我們可以看到 retaddr != 0 時，tlb_fill 是從 code cache 中被呼叫。

Br```eakpoint 1, tlb_fill (addr=28668, is_write=1, mmu_idx=0, retaddr=0x4000020c)
at /tmp/chenwj/qemu-0.13.0/target-i386/op_helper.c:4816
4816    {
(gdb) bt
#0  tlb_fill (addr=28668, is_write=1, mmu_idx=0, retaddr=0x4000020c) at /tmp/chenwj/qemu-0.13.0/target-i386/op_helper.c:4816
#1  0x000000000054e511 in __stl_mmu (addr=28668, val=982583, mmu_idx=0) at /tmp/chenwj/qemu-0.13.0/softmmu_template.h:272
#2  0x000000004000020d in ?? ()  <--- 我們在 code cache 裡!
```
我們來看一下 qemu.log 驗證一下我們對 QEMU 的了解。;) 既然 retaddr = 發生例外的 host binary 下一條指令位址減去 1，我們定位到 0x4000020d。
```
0x40000208:  callq  0x54e3a0
0x4000020d:  movzwl %bx,%ebp
```
瞧瞧 `__stl_mmu` 的位址，果然是 0x54e3a0。這代表我們在 code cache 呼叫 `__stl_mmu`。`__stl_mmu` 再去呼叫 `tlb_fill` 的時候發生例外。
```
(gdb) p __stl_mmu
$1 = {void (target_ulong, uint32_t, int)} 0x54e3a0
```
這裡我們可以看到 SOFTMMU 相關的 helper function 在各個地方都會被用到，不論是 QEMU 本身的函式 (一般 C 函式) 或是 code cache 都會調用 `__{ld,st}{b,w,l,q}_{cmmu,mmu}`。這些 helper function 又會調用 `tlb_fill`。`tlb_fill` 就是透過 retaddr 來判定是否需要回復 guest CPUState。
