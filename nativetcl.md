# Modern native Tcl development

This document presents the review of recent state of [Tcl](https://www.tcl.tk) development including portable code packaging with [Starkit](https://wiki.tcl-lang.org/page/Starkit) and creation of the [Starpack](https://wiki.tcl-lang.org/page/Starpack) platform-specific single executables with [Tclkit](https://wiki.tcl-lang.org/page/Tclkit).

_This document captures the state of things as of late 2023._

## Prerequisites

- Development environment for Linux
- Native Tcl

### Ubuntu 22.04 (WSL)

Install required packages

```shell
sudo apt install wget git gcc g++ autoconf make xorg-dev gcc-mingw-w64-x86-64-win32 g++-mingw-w64-x86-64-win32
```

## Set up [KitCreator](https://kitcreator.rkeene.org)

Clone a fork of the [original KitCreator repository](https://github.com/rkeene/KitCreator) with [Tcl/Tk 8.6.13](https://www.tcl.tk/software/tcltk/8.6.html) updates

```shell
git clone https://github.com/okhlybov/KitCreator.git
cd KitCreator
```

Prepare the repository

```shell
./build/pre.sh
```

_The original repository is slightly dated yet still usable as a fallback._

## Bootstrap Tclkit

Bootstrap the Linux-native Tclkit

```shell
./kitcreator clean
KITCREATOR_PKGS=" mk4tcl " ./kitcreator --enable-kit-storage=mk4
mv tclkit-* tclkit0
```

## Build native Linux Tclkit

```shell
./kitcreator clean
KC_TCL_STATICPKGS=1 KITCREATOR_PKGS=" mk4tcl " ./kitcreator --enable-kit-storage=mk4
mv tclkit-* tclkit
```

## Build native Windows Tclkit with system-hosted MinGW cross-complier

_Passing -static flag forces static linking of the winpthreads library (when using the POSIX-threaded MinGW toolchain) which is otherwise linked as DLL even if --disable-shared is specified._

```shell
./kitcreator clean
TCLKIT=$(pwd)/tclkit0 KC_TCL_STATICPKGS=1 KC_KITSH_LDFLAGS=-static KITCREATOR_PKGS=" mk4tcl " build/make-kit-win64 --enable-kit-storage=mk4
mv tclkit-* tclkit.exe
```

_More packages from this repository which are to be included into the Tclkit should be put into **KITCREATOR_PKGS**._

## Build [SDX](https://wiki.tcl-lang.org/page/sdx)

_SDX is a tool for managing Starkits/Starpacks._

Get a latest SDX package

```shell
wget https://chiselapp.com/user/aspect/repository/sdx/uv/sdx-20110317.kit
cp sdx-20110317.kit sdx.kit
```

Unwrap the kit

```shell
rm -rf sdx.vfs
./tclkit0 sdx.kit unwrap sdx.kit
```

_Due to the Windows executable file locking the runtime executable being assembled into a Starpack must be different from the one currently running sdx.kit_

### Build Linux-native SDX executable

```shell
./tclkit0 sdx.kit wrap sdx -runtime tclkit
```

### Build windows-native SDX executable

```shell
./tclkit0 sdx.kit wrap sdx.exe -runtime tclkit.exe
```

## Build native Linux Wishkit

```shell
./kitcreator clean
KC_TCL_STATICPKGS=1 STATICTK=1 KITCREATOR_PKGS=" mk4tcl tk " ./kitcreator --enable-kit-storage=mk4
mv tclkit-* wishkit
```

## Build native Windows Wishkit

```shell
./kitcreator clean
TCLKIT=$(pwd)/tclkit0 KC_TCL_STATICPKGS=1 KC_KITSH_LDFLAGS=-static STATICTK=1 KITCREATOR_PKGS=" mk4tcl tk " build/make-kit-win64 --enable-kit-storage=mk4
mv tclkit-* wishkit.exe
```

## More options

Precanned Tcl wrapper [Freewrap](https://freewrap.dengensys.com/).

[Online KitCreator](http://kitcreator.rkeene.org/kitcreator) service.
