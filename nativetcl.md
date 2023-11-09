# Modern native Tcl development

This document presents the review of recent state of [Tcl](https://www.tcl.tk) development including portable code packaging with [Starkit](https://wiki.tcl-lang.org/page/Starkit) and creation of the [Starpack](https://wiki.tcl-lang.org/page/Starpack) platform-specific single executables with [Tclkit](https://wiki.tcl-lang.org/page/Tclkit).

## Prerequisites

- Development environment for Linux
- [MSYS2](https://www.msys2.org/) development environment for Windows
- Native Tcl

## [KitCreator](https://kitcreator.rkeene.org) setup

Clone a fork of the [original KitCreator repository](https://github.com/rkeene/KitCreator) with [Tcl/Tk 8.6.13](https://www.tcl.tk/software/tcltk/8.6.html) updates

```shell
git clone https://github.com/okhlybov/KitCreator.git
```

_The original repository is slightly dated yet still usable as a fallback._

## [Tclkit](https://wiki.tcl-lang.org/page/Tclkit) bootstrap

Bootstrap the native Tcl single executable

```shell
build/pre.sh
KC_TCL_STATICPKGS=1 KITCREATOR_PKGS="mk4tcl" ./kitcreator --enable-kit-storage=cvfs --disable-threads 
```

This yields a minimal platform-specific self-contained Tcl runtime (the Tclkit itself) `tclkit-$version` (`tclkit-$version.exe` on Windows).

As of writing time the version is `8.6.13` thich is used from now on.

## [SDX](https://wiki.tcl-lang.org/page/sdx) bootstrap

Get a latest SDX package

```shell
wget https://chiselapp.com/user/aspect/repository/sdx/uv/sdx-20110317.kit
cp sdx-20110317.kit sdx.kit
```

Make a temporary copy of the Tcl runtime

```shell
cp tclkit-8.6.13{.exe} ./tclkit{.exe}
```

Make the native SDX executable

```shell
./tclkit sdx.kit unwrap sdx.kit
./tclkit sdx.kit wrap sdx -runtime tclkit-8.6.13{.exe}
```

_Due to the Windows executable file locking the runtime being assembled into a Starpack must be different from the one running sdx.kit_

## More options

Precanned Tcl wrapper [Freewrap](https://freewrap.dengensys.com/).

[Online KitCreator](http://kitcreator.rkeene.org/kitcreator) service.
