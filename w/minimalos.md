# a small os template

Most tutorials use `gcc` cross compilation, which requieres compiling it. The `clang` compiler can be used as a cross compiler without the extra step (it produces architecture independent code in `llvm` which can target multiple architectures).

## tools

Use the `multiboot` standard so it can be booted by using standard tools such as `grub2`. To test if a file is compliant with `multiboot`:

	grub-file --is-x86-multiboot file

or, for `multiboot2`:

	grub-file --is-x86-multiboot2 file

A `multiboot` compliant image has a header containing three consecutive `32 bit` values: a magic number, flags and a checksum. This values have to be `4 bytes` aligned in the fist 8k in the file.

`clang` as the crosscompiler to target another platform. A platform is something like:

	<processor, os, binary format, ~other things~>

such as

	<x86, none, elf, ...>

The assembler and linker are invoked using the `clang` command.

The compiler will create an `elf` binary.

## code to call a kernel

A bit of assembler to define some sections in the kernel image and call the kernel entry point. First some constants:

	.set ALIGN,    1<<0             /* alignment */
	.set MEMINFO,  1<<1
	.set FLAGS,    ALIGN | MEMINFO
	.set MAGIC,    0x1badb002
	.set CHECKSUM, -(MAGIC + FLAGS) /* checksum */

Now the `multiboot` section:

	.section .multiboot
	.align 4
	.long MAGIC
	.long FLAGS
	.long CHECKSUM

Later, use the linker to put the sections together to get it in the first `8k` of the image.

The kernel needs a stack if it's written in `c` (calling functions, etc.). Use the `.bss` section. Remember that in `x86` the stack grows from higher memory addresses to lower.

	.section .bss
	.align 16     # apparently, the stack on x86 must be 16-byte aligned in the System V ABI
	stack_bottom: # stack "begins"
	.skip 16384   # 16 KiB of stack
	stack_top:    # stack "ends"

Define the `.text` section containing the executable code. Call the kernel entry point.

	.section .text
	.global _start
	.type _start, @function
	_start:
		mov $stack_top, %esp
		call kernel_main
		cli
	1:	hlt
		jmp 1b


## kernel code

The kernel needs to implement a `kernel_main` function, which will be called in the previous code.

Assuming the system uses `bios`, the following kernel just prints `kernel running` in the video memory.

	#define VIDEO_MEM 0xb8000
	
	void print_msg(void) {
	    char *video = (char*) VIDEO_MEM;
	    const char *msg = "kernel running";
	
	    for(int i = 0; msg[i]; i++) {
	        video[2*i] = msg[i];
	        video[2*i + 1] = 0x07;
	    }
	}
	
	void clear_screen() {
	    int i;
	    char *video = (char*) VIDEO_MEM;
	
	    for(i = 0; i < 25*80; i++) {
	        video[2*i] = 0;
	    }
	}
	
	void kernel_main(void) {
	    clear_screen();
	    print_msg();
	}

## linker script

A linker script defines the resulting binary. For example, the sections to include, their memory address, the entry symbol, etc.

	ENTRY(_start)
	
	SECTIONS
	{
		. = 0x100000; // load at 1M
	
		.text : {          // code
			*(multiboot)   // section for multiboot (within the 8k)
			*(.text)
		}
		.data ALIGN(4096) : {
			*(.data)
		}
		.rodata ALIGN(4096) : {
			*(.rodata)     // read-only data
		}
		.bss ALIGN(4096) : {
			*(.bss)        // variables, stack
		}
	}

## build script

To build the project:

	#!/bin/sh
	
	# kernel.c
	clang --target=i386-pc-none-elf -ffreestanding -o kernel.o -c kernel.c
	
	# bootloader
	clang --target=i386-pc-none-elf -o boot.o -c boot.S

	# link
	clang --target=i386-pc-none-elf -ffreestanding -nostdlib -Wl,--no-dynamic-linker -Wl,-Tlinker.ld boot.o kernel.o -o kernel.bin

# testing

The easiest way to test a kernel is to use a virtual machine. In particular, `qemu` can boot a kernel directly using `multiboot`, without the need to create a bootable media (an `.iso` for example):

	qemu-system-i386 -kernel kernel.bin


## references

[osdev](https://wiki.osdev.org/Main_Page) for everything, in particular:

- [using clang](https://wiki.osdev.org/LLVM_Cross-Compiler)
- [template](https://wiki.osdev.org/Bare_Bones)
- another [template](https://wiki.osdev.org/Bare_Bones_with_NASM) but using the `nasm` assembler, as an alternative to `gas`
- how to [debug](https://wiki.osdev.org/Kernel_Debugging) a kernel
- info on the [multiboot](https://wiki.osdev.org/Multiboot) standard

Not in osdev:

- [elf](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) in Wikipedia
- [elf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f00/docs/elf.pdf)
- some code and build instructions for using `clang` [x86-Toy-OS](https://github.com/Henje/x86-Toy-OS)