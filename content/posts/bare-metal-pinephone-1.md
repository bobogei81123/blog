---
title: "Bare Metal Pinephone 1"
date: 2022-01-20T00:37:18-08:00
draft: true
---

I got a [Pinephone](https://www.pine64.org/pinephone/) and decided to write a
hobby mobile OS for fun. The first step is to see if I can write a bare-metal
program for it. It turned out to be not very difficult, and I will show you the
steps here.

## Preparation

We need a couple of things:
1. A Pinephone, of course.
2. A USB to serial audio jack connector. You can buy one from [Pine64
   store](https://pine64.com/product/pinebook-pinephone-pinetab-serial-console/)
   or get a similar cable (I was able to succeed using a TTL-232R-3V3-AJ
   cable).
3. An SD card.

## Building a bare-metal Arm64 program

First, let’s try to build a bare-metal program for Arm64 (AArch64), which is
the CPU architecture of Pinephone. My computer's CPU architecture is x86-64
(which is probably the one most people are using), and we will have to set up a
cross compiler to compile for another architecture different than our PC.

### Installing the cross compiler

GCC provides multiple cross compiler for different architecture. For bare-metal
programming in Arm64, the one we need is `aarch64-none-elf-gcc`.

1. Check your Linux distro's package manager. I use Arch Linux, and some kind
   people have already made a [build
   script](https://aur.archlinux.org/packages/aarch64-none-elf-gcc-bin) to
   install the toolchain.
2. Or you can download it from Arm’s website
   [here](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads).
   Find “AArch64 ELF bare-metal target” under “x86_64 Linux hosted cross
   compilers”. Extract the tar file, and for your convenience, add the bin
   directory into the `$PATH` environment variable.

You can check if you have successfully install the compiler by running the
following command:

```bash
$ aarch64-none-elf-gcc --version
aarch64-none-elf-gcc (GNU Toolchain for the A-profile Architecture ...
```

### Writing an Arm64 assembly

Now, let's write a simple Arm64 program in assembly. There are several Arm64
assembly tutorials out there, and I went through an awesome guide ["Embedded
Programming with the GNU Toolchain"] (Though the tutorial is for Arm32).

Create a new file `add.s` in a directory. Add the following lines:

```armasm
.text
start:
mov     x0, #1
mov     x1, #2
add     x2, x0, x1
stop:
b       stop
```

The program above is fairly simple. We load constant 1 and 2 into registers
`x0` and `x1`, and store their sum 3 into `x2`. Now, let’s compile it:

```bash
$ aarch64-none-elf-as -o add.o add.s
$ aarch64-none-elf-ld -o add.elf add.o
$ aarch64-none-elf-objcopy -O binary add.elf add.bin
```

In the first two steps, we compile add.s into ELF, and then use `objcopy` to turn
it into raw binary format.

## Run the program on QEMU

Let’s try to run the simple bare-metal program we just wrote on QEMU, a machine
emulator. Use your Linux distro’s package manager to install QEMU. Notice that
we need a specialized one `qemu-system-aarch64` to emulate Arm64 machines.
For Arch Linux, the package name is `qemu-arch-extra`.

Run the following command:

```bash
$ qemu-system-aarch64 -M virt -cpu cortex-a53 -kernel add.bin \
                      -nographic -serial /dev/null
```

This will open a QEMU monitor:

```
QEMU 6.2.0 monitor - type 'help' for more information
(qemu)
```

Type `info registers` to show register values:

```
(qemu) info registers
 PC=000000004008000c X00=0000000000000001 X01=0000000000000002
X02=0000000000000003 X03=0000000000000000 X04=0000000040080000
```

Notice the value of X00, X01, and X02 are 1, 2, 3, respectively, which are the
values we will expect after running the program. This shows that our code
have successfully run on Arm64!

In the next post, I will walk through how to set up a bootloader on pinephone
for loading bare-metal programs.
