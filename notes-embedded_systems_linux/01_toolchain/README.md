# Embedded Linux Toolchains

## Introducing Toolchains

*Reference: [Mastering Embedded Linux Programming (3rd ed.)](https://www.packtpub.com/en-us/product/mastering-embedded-linux-programming-9781789530384) by Simmonds and Vasquez (Packt Publishing)*

A toolchain is a set of tools that compiles source code into executables that run on your target device. The standard GNU toolchain set includes:

1. **Binutils** - The GNU assembler and debugger

2. **GCC Compiler Collection** - C and other language compilers that feed to the backend assembler.

3. **C Runtime Libraries** - The various POSIX compliant C librries offering APIs that interface to the operating system kernel. 

To build the toolchain we need a variant of the kernel headers -- a sanitized version -- of the Linux version we intent to use in the target embedded system. Given that the kernel interface is alwasy backwards compatible, we only need the headers that is equal to or older than the target kernel.

We use the toolchain to compile code written in assembly, C and C++, the three common programming languages used in base open source packages. But what exactly do we use the toolchain for when we set out to assemble our embedded Linux development environment? 

1. bootloader

2. kernel

3. root filesystem

## Types of toolchains

### Clang or GNU

Since the early days, toolchains for Linux have been based on tools provided by the GNU project. In recent years, however, toolchains from the *Low Level Virtual Machine (LLVM)* project have become a viable alternative.  The *Clang* compiler can now compile faster, yield better compilation diagnositcs, and covers nearly all the wide range of architectures and operating systems that used to be the purview of GNU GCC. One point of note, though, is open source licensing: GNU is coverd by GPL while LLVM has a BSD license.

### Native vs Cross-Compile

When talking about toolchains we distinguish between the host system and the target embedded system. In case that these two systems are the exactly the same (operating system and computer architecture), we busy ourselves with creating a **native toolchain**. Often they two systems are different. Our build host could be a more powerful x86, for example, while the targed system is built around a resource-constrained ARM device. Here, then, we talk about preparing a **cross toolchian**.

### Variations in computer architecture

1. CPU type: MIPS, x86-64, etc.

2. Big- or Little-endian CPU

3. Floating point support that could be implemented in hardware or emulated through software

4. Application Binary Interface (ABI), the convension for passing parameters between function calls

### Toolchain nomenclature

A toolchain is identified (named) by a four-part tuple. 

```
$> gcc -dumpmachine
```

Some common examples include `x86_64-linux-gnu` or `mipsel-unknown-linux-gnu`.

1. CPU (ARM, MIPS, x86) plus endian (**el** for little-endian or **eb** for big-endian): `mipsel` for MIPS little-endian; `armeb` for big-endian ARM processor.

2. Vendor: Identifies the toolchain provider (could be `unknown`) 

3. Kernel: Often `linux`

4. Operating system: User space components that stand out, such as `gnu` or `musl`, or ABI information such as as `gnuabi` or `muslabihf` (musl C-library and hardware floating point)

## Toolchain Lab  (Beaglebone Black AM3358 ARM)

*The following procedure was noted down from the Bootlin [Embedded Systems Development](https://bootlin.com/training/embedded-linux/) course [lab notes](https://bootlin.com/doc/training/embedded-linux-bbb/embedded-linux-bbb-labs.pdf), Beaglebone Black edition.*

### Install the lab files from Bootlin

```
$> wget https://bootlin.com/doc/training/embedded-linux-bbb/embedded-linux-bbb-labs.tar.xz
$> tar xvf embedded-linux-bbb-labs.tar.xz
```

### Update your distribution

```
$> sudo apt update
$> sudo atp dist-upgrade
$> sudo apt install build-essential git autoconf bison flex texinfo help2man gawk libtool-bin \
libncurses5-dev unzip
```

### Install Crosstool-ng

```
$> git clone https://github.com/crosstool-ng/crosstool-ng
$> cd crosstool-ng
$> git checkout crosstool-ng-1.26.0
```

### Build and install Crosstool-ng

```
$> ./bootstrap
$> ./configure --enable-local
$> make
```
This will build and install Crosstools-ng in `$HOME/x-tools`.

### Confirm Crosstool-ng works

```
$> ./ct-ng help
```

### Configure the Crosstool-ng to configure an Arm toolchain

```
$> ./ct-ng list-samples
$> ./ct-ng arm-cortex_a8-linux-gnueabi
$> ./ct-ng menuconfig
```

1. **Path and misc** options:

- Enable `Try features marked as EXPERIMENTAL`
- If `wget2` is used instead of `wget`, remove `--passive-ftp` flags from the `DOWNLOAD_WGET_OPTIONS`

2. **Target** options:

- Set `Use specific FPU (ARCH_FPU) to vfpv3`
- Set `Floating point to hardware (FPU)`

3. **Toolchain** options:

- Set `Tuple's vendor string (TARGET_VENDOR) to training`
- Set `Tuple's alias (TARGET_ALIAS) to arm-linux`

4. **Operating System** options:

- Set `Version of linux` to a version older than the lastest kernel header (v6.6 as of this writing)

5. **C-library** options:

- Set `C library to musl (LIBC_MUSL)`
- Keep the `musl` default option proferred

6. **C compiler** options:

- Set `Version of gcc to 13.2.0`
- Enable `C++ (CC_LANG_CXX`

7. **Debug** options:

- Clear all the debug options since these can be installed later in the *filesystem building* phase.
  
This process will yield a configuration file for the toolchain build: [.config](https://github.com/csaatechnicalarts/beaglebone-black-workbook/blob/main/notes-embedded_systems_linux/01_toolchain/2025_0113-ct_ng-dot_config-output.txt) (in this case for the Beaglebone Black)

### Create the toolchain

```
$> ./ct-ng build
```
Here is the [build output](https://github.com/csaatechnicalarts/beaglebone-black-workbook/blob/main/notes-embedded_systems_linux/01_toolchain/ct_ng_build-output.txt) for the Beaglebone Black.

### Confirm the new toolchain works

```
$> cd ~/embedded-linux-bbb-labs/toolchain/

$> export PATH=~/x-tools/arm-training-linux-musleabihf/bin:$PATH
```
```
$> arm-linux-gcc -dumpmachine
arm-linux-gcc -dumpmachine
```
```
$> arm-linux-gcc -o hello.exe hello.c
```
```
$> file hello.exe
hello.exe: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-armhf.so.1, not stripped
```
```
$> sudo apt install qemu-user

$> qemu-arm hello.exe
qemu-arm: Could not open '/lib/ld-musl-armhf.so.1': No such file or directory
```
```
$> find ~/x-tools -name ld-musl-armhf.so.1

$> qemu-arm -L ~/x-tools/arm-training-linux-musleabihf/arm-training-linux-musleabihf/sysroot hello.exe
```
