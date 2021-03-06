# nZDC-Compiler
nZDC is a compiler transformation for soft error detection. This repository contain a LLVM-based compiler with nZDC transfromation. 

## Prerequisites:
There are several prerequisite software packages required to build and run LLVM compiler infrastructure ([http://releases.llvm.org/3.7.1/docs/GettingStarted.html]) and gem5 simulator ([http://gem5.org/Compiling_M5]). In this document we assume that all the required packages has been installed by the user and we focus on compiling benchmarks with nZDC soft error protection transformation and running them on gem5 simulator. We will be using three tools: 

1) LLVM-nZDC compiler to compile a program 
2) GCC cross-compiler (gcc-aarch64-linux-gnu) to generate static binary of the compiled programs
3) Gem5 to run programs

## Build and Install: 

A) Download llvm-nZDC compiler from here:
	[https://github.com/cmlasu/nZDC-Compiler/]
	or by executing following comand from your terminal:
```
git clone git@github.com:cmlasu/nZDC-Compiler.git
```
B) Building nZDC-llvm compiler
```
cd ./LLVM3.7

mv ./clang llvm/tools/

mkdir  ./build

cd ./build

cmake ../llvm

make -j2
```

NOTE: At this point, the build directory should contain a "bin" directory which contains llvm tools including **clang** and **llc**.

C) GCC cross-compiler for ARM V8
```
sudo apt-get install gcc-aarch64-linux-gnu
```
D) Install gem5 SE mode

    http://www.gem5.org/Download
    
    https://www.youtube.com/watch?v=SW63HJ0nW90

## Compiling benchmarks

### Compiling C/C++ source codes by clang and generating nZDC-protected assembely file
```
./LLVM3.7/build/bin/clang -I /usr/arm-linux-gnueabi/include/  -O3 -static -emit-llvm -target armv8-none-eabi  -S  ./programs/mm.c -o ./programs/mm.ll

./LLVM3.7/build/bin/llc -O3 -reserveRegs=true -enable-nZDC=true -march=aarch64 ./programs/mm.ll  -o ./programs/mmopt-nZDC.s
```
NOTE: At this point the assembely file (.s) should contain duplicated and checking instructions. Since nZDC an error detection scheme, you should add your recovery scheme. The recovery blocks are inserted and only contain one instruction which is "sub	 x25, x25, x25". The simpleset recovery strategy is to terminate the program. It can be done by simply replacing  "sub	 x25, x25, x25" by "bl exit" instructions.

### Creating executable file from assembely file
```
aarch64-linux-gnu-gcc -static -O3 ./programs/mmopt-nZDC.s -o ./programs/mmopt-nZDC
```
### Running nZDC-protected executable on Gem5 Simulator
```
gem5$: build/ARM/gem5.opt configs/example/se.py -c ./programs/mmopt-nZDC
```
## nZDC instruction duplication and soft error detection implementation

You can find the implementation of nZDC transfromation is the follwoing .cpp file:
/LLVM3.7/llvm/lib/Target/AArch64/ZDC-R.cpp

## Aditional Resources
- [Soft error challenge](http://aviral.lab.asu.edu/soft-error-resilience/)
- [nZDC error detection paper](https://dl.acm.org/citation.cfm?id=2898054)
- [Multithreaded nZDC](https://ieeexplore.ieee.org/document/8289351/)

## Contact Us

For any questions or comments on nZDC-Compiler, please contact us at cmlasu@gmail.com

CML's nZDC Webpage - http://aviral.lab.asu.edu/nzdc/
