## MIPS Simulator / Interpreter (bi-endian)
This is a fork of [@zvrba](https://github.com/zvrba)'s MIPS simulator (no longer maintained), which was originally designed as a proof of concept for the publiciation linked below. The original project aims to simulate only the MIPS little endian ISA (with support for encrypted executables).

Here, I've added support for big endian MIPS, with the exception of the encryption technique described in the publication. The goal is to provide a general purpose big (and little!) endian MIPS simulator, which can easily be used as a library.

As originally put by Željko, the design goals of this simulator are to provide:

- **A library interface**, so that all aspects can be easily controlled from the host application.
- The ability to run in freestanding environment (i.e. an environment **without the standard C library**).

With its library oriented design, **this simulator is ideal for use as an interpreter core** for higher-level emulators (ie. a Nintendo 64 emulator).

The contents of the original README are below, but have been updated to reflect the changes introduced in the support of big endian MIPS, and will be kept in sync with future changes.

-[@kevinhartman](https://github.com/kevinhartman)


The CSPIM MIPS simulator
========================
More details about this project can be found
[here.](http://zvrba.net/articles/encrypted-execution.html) Detailed description of CSPIM
internals as well as the motivation and possible use cases can be found in the
following publication:

Vrba, Z.; Halvorsen, P.; Griwodz, C., "Program obfuscation by strong
cryptography" The International Dependability Conference, 2010. ARES
'10. International Conference on, 15-18 February, 2010.
[DOI:10.1109/ARES.2010.47](http://dx.doi.org/10.1109/ARES.2010.47)

Copies of the paper are available on explicit request by email.


COPYRIGHT NOTICE
================
All files in this distribution, unless otherwise explicitly noted, are

*  (c) 2008-2012 Zeljko Vrba <zvrba.external@zvrba.net>

and are distributed under the following license:

>  Permission is hereby granted, free of charge, to any person obtaining a copy
>  of this software and associated documentation files (the "Software"), to
>  deal in the Software without restriction, including without limitation the
>  rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
>  sell copies of the Software, and to permit persons to whom the Software is
>  furnished to do so, subject to the following conditions:
>
>  The above copyright notice and this permission notice shall be included in
>  all copies or substantial portions of the Software.
>
>  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
>  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
>  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE
>  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
>  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
>  FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
>  IN THE SOFTWARE.


Project structure
=================
The top-level directory contains the following subdirectories:

* bmips-le:   precompiled programs for little-endian MIPS I; directly runnable by CSPIM
* bmips-be:   precompiled programs for big-endian MIPS I; directly runnable by CSPIM
* hostapps:   host utilities [the interpreter itself]
* mipsapps:   source fof programs runnable by CSPIM; precompiled in both `bmips-le` and `bmips-be`
* vm:         core simulator: MIPS instruction set and ELF loader

 
How to build
============
This package has been developed and tested with Solaris cc, gcc and Visual
Studio 2008, and macOS (excluding cross-compilation of mipsapps).  Prerequisites for building:

- Working native compiler.  Solaris cc, gcc, and Visual Studio 2008 have been
  tested.
- Working cross-compiler which can generate little-endian or big-endian MIPS I
  ELF executables (for MIPS apps).  gcc 4.2.4 and binutils 2.22 have been tested.
- [CMake](http://www.cmake.org)

The interpreter is built by executing

    cmake .
    make

This will build host applications in `hostapps`. By default, MIPS applications in
`mipsapps` are not built, since they require a cross compiler. Directories `bmips-le` and `bmips-be`
contain pre-built binaries of the sources in `mipsapps`.

To build them for yourself, specify `-DSKIP_APPS=OFF` when invoking `cmake`.

### CMake options
```
MIPS_BE     Build all for big endian MIPS target.
SKIP_APPS   Skip building mips apps in mipsapps. Use this when you don't have a cross compiler.
VIRTUALIZED Build for use with load/run lvl1. Doesn't yet support big endian (MIPS_BE will be error).
SKIP_SSTEP  Skip building sstep, which uses POSIX librt. Use this on macOS.
```

Host applications
-----------------
The following applications are built in `hostapps`:

* `runtorture`: use to run the MIPS torture test
* `run`:      	use to run (possibly encrypted) MIPS executables
* `runbench`:   use to run the MIPS benchmarks
* `elfcrpyt`:   use to encrypt MIPS ELF executables

`sstep`, `hanoi-bench-native`, `mmult-bench-native` are benchmarking
applications; see the above paper for details.  The "run" programs
take as parameter a MIPS ELF executable; an example invocation is
    ./hostapps/run ./mipsapps/hanoi
`runtorture` should only be used to run the `cputorture` program.


MIPS CPU torture test
---------------------
Torture test assembly source is generated by preprocessing the `*.sm4` files
with M4:

    m4 cputorture.sm4 > cputorture.s

Endian-specific versions of loadstore.sm4 exist to support building cputorture.s
for both CPU variants.

The pregenerated source is already provided in the distribution.  The test
is run by executing

    ./hostapps/runtorture ./mipsapps/cputorture

If the test is successfully passed, the last lines of the output will be
something like (note the `(near SUCCESS)` in the last line):

    finished: exception=5, code=0x0, last_branch=0x2e74(near t_link)
    ***MIPS@0xb7c6b008***
    BASE=200000@0xb7c6b008 BRK=00003fd0 STKSZ=00004000
    R00=00000000 R01=00000000 R02=00000000 R03=0000000c
    R04=00000001 R05=00005678 R06=00005678 R07=00001234
    R08=00001234 R09=0000e7f8 R10=0000e7f8 R11=0000c5d6
    R12=0000c5d6 R13=12345678 R14=12345678 R15=c5d6e7f8
    R16=c5d6e7f8 R17=56787800 R18=56787800 R19=12345678
    R20=12345678 R21=00001234 R22=ffffe7f8 R23=00000000
    R24=90000000 R25=f0000000 R26=00003fc0 R27=00003fc4
    R28=00003fc0 R29=00003fc4 R30=00003fc8 R31=00002fa0
    HI =12345678 LO =9abcdef0
    PC =00002fb0 DS =00000000
    (near SUCCESS)
    
If the execution stops at a label name other than `SUCCESS` in the last line,
something has gone wrong and the preceding output can be determined to find
out what.


MIPS applications
-----------------
Many of the other example problems are are solved assignments (in C) from
Appendix of the third edition of Hennessy & Patterson, Computer Organization
and Design: The Hardware/Software Interface; available at.  This appendix is
also available online [here.](http://pages.cs.wisc.edu/~larus/HP_AppA.pdf)
These programs should be executed with `./hostapps/run program`.

As expected, mipsapps built for big endian will only work with hostapps built
to run big endian executables (`-DMIPS_BE`).

Encrypted execution
-------------------
All LOAD ELF segments of executables can be encrypted with RC-5 encryption
algorithm using 32-bit block size and 128-bit key.  The encryption uses ECB
mode, which should *not be used for anything but simple demos*!  Example
session:

    ./hostapps/elfcrypt mipsapps/hanoi /tmp/hanoi-crypt 0123456789ABCDEF0123456789ABCDEF
    ./hostapps/run /tmp/hanoi-crypt 0123456789ABCDEF0123456789ABCDEF

Failing to provide the key in the last command will terminate execution with
exception 3 (invalid instruction).

Big endian MIPS is currently unsupported in this mode.

Self-simulation
---------------

Compile both hosted and VIRTUALIZED build (configure this by using `ccmake`).
This must be done in two separate build directories; note that cmake supports
builds in directories other than the source directory (so-called
"out-of-source build").  This builds an extra executable,
`./mipsapps/run-lvl1`, which should be run with `./hostapps/load-lvl1`.
`./run-lvl1` can currently only run cputorture, since the nested simulation
does not yet forward system calls to the outer simulation.

NOTE: You need to remove all files previously generated by CMake for
out-of-source build to work.

Suppose that the hosted interpreter has been built in the `./HH` directory,
and the non-hosted in the `./NH` directory.  Then the CPU torture test can
be run in the nested simulation with the following command:

    ./HH/hostapps/load-lvl1 ./NH/mipsapps/run-lvl1 ./NH/mipsapps/cputorture 

Big endian MIPS is currently unsupported.

Building cross-gcc
------------------
Note that gcc 4.2.x is the highest gcc version that doesn't require GMP, MPFR
and MPC libraries, which significantly eases installation.

For a big-endian cross compiler, replace `mipsel-elf` with `mips-elf` below.

Note: building gcc 4.2.x with a modern version of gcc is tough. It helps to set
languages to C only, and to disable Werror when parameterizing GNU's `configure`
script, in addition to the below.

1. Download and build binutils (tested with version 2.22).
   Configure and build as:

   `./configure --prefix=$HOME --disable-debug --disable-nls --target=mipsel-elf`
   `make install`

2. Download and build gcc; it MUST NOT be built in the source dir (hence ../).
   Make sure that the installed binutils (mipsel-elf-as etc.) are in PATH.

    `../gcc-4.2.4/configure --prefix=$HOME --disable-debug --disable-nls --target=mipsel-elf --disable-bootstrap --disable-libssp`

    `make install`

