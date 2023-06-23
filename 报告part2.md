[toc]

# CS334-OS-Project

### head.s与head64.s

```asm
pg_dir:
PML4:
	.quad 0x101007	# PDPT_0
	.fill 255,8,0
	.quad 0x101007
	.fill 255,8,0
PDPT_0: # Not all processors that support IA-32e paging support 1 pages
	.quad 0x102007 # PD_0
	.rept 0x1ff
		.quad 0
	.endr
PD_0:
	.set i, 0
	.rept 64		# 64*2=128MB
		.quad (i << 21)+0x87
		.set i, (i+1)
	.endr
	.rept 512-64
		.quad 0
	.endr

.org 0x5000
.globl startup_64
startup_64:
	movl $0x10,%eax
	mov %ax,%ds
	mov %ax,%es
	mov %ax,%fs
	mov %ax,%gs
	mov %ax,%ss
	mov stack_start(%rip),%rsp

	mov turn_HHK(%rip), %rax
	pushq $0x08
	pushq %rax
	lretq

turn_HHK:
 .quad post_HHK # 0xFFFF8000XXXXXXXX after ld

post_HHK:
	movq $0, PML4(%rip)
resetup_gdt:
	lgdt gdt_descr(%rip)
	movl $0x10,%eax
	mov %ax,%ds
	mov %ax,%es
	mov %ax,%fs
	mov %ax,%gs
	mov %ax,%ss
	mov stack_start(%rip),%rsp
```



1. Introduction of new page table structures: In the old code, the page table structures used were for a 32-bit system. However, in the new code, new page table structures are introduced, namely PML4 (Page Map Level 4), PDPT_0 (Page Directory Pointer Table), and PD_0 (Page Directory). These new table structures are critical components for managing page tables in a 64-bit system. By introducing the new page table structures, support for larger physical memory spaces is enabled, allowing the system to correctly address and map memory.
2. Modification of stack setup: There are some differences in stack setup between 32-bit and 64-bit systems. In the old code, 32-bit registers and instructions were used to set up the stack. In the new code, 64-bit registers and instructions are used to set up the stack. This is because in a 64-bit system, using wider registers and instructions better accommodates stack operations in 64-bit mode.
3. Reloading the Global Descriptor Table (GDT) and segment registers: In the old code, segment registers' values were loaded based on the needs of the 32-bit system. In the new code, it is necessary to reload the Global Descriptor Table (GDT) and segment registers to ensure that after switching from 32-bit mode to 64-bit mode, the segment registers have the correct values. This ensures that the system can properly access and protect different segments and supports memory management in 64-bit mode.

These modifications are made to adapt the migration from a 32-bit system to a 64-bit system. By introducing new page table structures, modifying stack setup, and reloading the GDT and segment registers, the system can run properly in 64-bit mode and take full advantage of the larger physical memory space. These modifications ensure that the system can operate correctly on the new architecture and utilize the features and advantages of a 64-bit system effectively.



```asm
setup_idt:
	lea ignore_int(%rip),%rdx
	movl $0x00080000,%eax /* selector = 0x0008 = cs */
	movw %dx,%ax
	movw $0x8E00,%dx	/* interrupt gate - dpl=0, present */
	movq $0x00000000FFFF8000, %rbx
	lea idt(%rip), %rdi
	mov $256,%ecx
rp_sidt:
	movl %eax,(%rdi)
	movl %edx,4(%rdi)
	movq %rbx,8(%rdi)
# movl %ebx,12(%rdi)
	addq $16,%rdi
	dec %ecx
	jne rp_sidt
	lidt idt_descr(%rip)

setup_tss:
	leaq tss_table(%rip), %rdx
	xorq %rax, %rax
	xorq %rcx, %rcx
	movq $0x89, %rax # present, 64bit TSS
	shlq $40, %rax
	movl %edx, %ecx 
	shrl $24, %ecx
	shlq $56, %rcx
	addq %rcx, %rax
	xorq %rcx, %rcx
	movl %edx, %ecx
	andl $0xffffff, %ecx
	shlq $16, %rcx
	addq %rcx, %rax
	addq $0x67, %rax # limit
	leaq gdt(%rip), %rdi
	movq %rax, 24(%rdi) # gdt_idx = 3
	shrq $32, %rdx
	movq %rdx, 32(%rdi)
# load tss
	mov	$0x18, %ax # 11000b, gdt_idx = 3
	ltr	%ax

# long return to main
	lea main(%rip), %rax
	pushq	$0x08
	pushq %rax
	lretq
```

1. Modification in `setup_idt`:

   - The original code used 32-bit registers `%edx` and `%edi` to manipulate the IDT (Interrupt Descriptor Table) and load it using 32-bit instructions.
   - The modified code uses 64-bit registers `%rdx` and `%rdi` to manipulate the IDT and load it using 64-bit instructions.
   - The new code also adds setting of the attributes field for each gate in the IDT (using 64-bit register `%rbx`), providing more precise definition of the characteristics of the interrupt gates.

2. Modification in `setup_tss`:

   - The new code introduces a new function `setup_tss` for setting up the TSS (Task State Segment).
   - The modified code uses 64-bit registers `%rdx` and `%rdi` to manipulate the TSS and load it using 64-bit instructions.
   - The new code calculates and sets various fields of the TSS, including the type, segment limit, base address, etc.
   - Finally, the TSS is loaded into the task register TR using the `ltr` instruction.

   

```bash
.align 4
.word 0
idt_descr:
	.word 256*16-1		# idt contains 256 entries
	.quad idt

.align 4
.word 0
gdt_descr:
	.word 256*8-1		# so does gdt (not that that's any
	.quad gdt					# magic number, but it works for me :^)

# In 64-bit processor, an entry in idt is 16B long.
idt:	.fill 256*2,8,0		# idt is uninitialized

gdt:
	.quad	0						 # dummy
	.long 0, 0x00209a00 # code readable in long mode
	.long 0, 0x00209200 # data readable in long mode
	.fill 504,8,0			 # space for LDT's and TSS's etc (252*2=504)

tss_table:
	.fill 26,4,0 # 26 * 4 = 104 (64bit TSS)

stack_start: 
	.quad 0xFFFF800000200000
```

1. Alignment modification: Changed `".align 2"` to `".align 4"` to ensure data alignment to a 4-byte boundary. This is done to comply with the memory alignment requirements of 64-bit systems.

2. Descriptor size modification: Changed `".word 256*8-1"` to `".word 256*16-1"` to accommodate the 16-byte size of each IDT (Interrupt Descriptor Table) entry in a 64-bit system. Similarly, the descriptor size of the GDT (Global Descriptor Table) was modified accordingly.

3. IDT and GDT padding modification: Changed `".fill 256,8,0"` to `".fill 256*2,8,0"` to ensure that the padding size of the IDT matches the modified size of each entry.

4. Other padding modifications: Corresponding modifications were made to the padding size of tables such as GDT and TSS to accommodate a 64-bit system.



### lib/string.c

For `lib/string.c`, navigate to the `lib` directory and run `make` to enter the compilation process. By analyzing the error messages, we can identify the areas that require modification.

For 64-bit systems, we introduced macro definitions and corresponding instructions to ensure compatibility with the 64-bit instruction set. During the modification process, certain instructions used in 32-bit systems were replaced to meet the requirements of the 64-bit system. For example, the `movl` instruction was replaced with the `movq` instruction, and the `decl` instruction was replaced with the `movq` instruction.
