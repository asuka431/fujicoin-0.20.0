WINDOWS BUILD NOTES
====================

Below are some notes on how to build Baricoin Core for Windows.

The options known to work for building Baricoin Core on Windows are:

* On Linux, using the [Mingw-w64](https://mingw-w64.org/doku.php) cross compiler tool chain. Ubuntu Bionic 18.04 is required
and is the platform used to build the Baricoin Core Windows release binaries.
* On Windows, using [Windows
Subsystem for Linux (WSL)](https://docs.microsoft.com/windows/wsl/about) and the Mingw-w64 cross compiler tool chain.
* On Windows, using a native compiler tool chain such as [Visual Studio](https://www.visualstudio.com).

Other options which may work, but which have not been extensively tested are (please contribute instructions):

* On Windows, using a POSIX compatibility layer application such as [cygwin](https://www.cygwin.com/) or [msys2](https://www.msys2.org/).

Installing Windows Subsystem for Linux
---------------------------------------

With Windows 10, Microsoft has released a new feature named the [Windows
Subsystem for Linux (WSL)](https://docs.microsoft.com/windows/wsl/about). This
feature allows you to run a bash shell directly on Windows in an Ubuntu-based
environment. Within this environment you can cross compile for Windows without
the need for a separate Linux VM or server. Note that while WSL can be installed with
other Linux variants, such as OpenSUSE, the following instructions have only been
tested with Ubuntu.

This feature is not supported in versions of Windows prior to Windows 10 or on
Windows Server SKUs. In addition, it is available [only for 64-bit versions of
Windows](https://docs.microsoft.com/windows/wsl/install-win10).

Full instructions to install WSL are available on the above link.
To install WSL on Windows 10 with Fall Creators Update installed (version >= 16215.0) do the following:

1. Enable the Windows Subsystem for Linux feature
  * Open the Windows Features dialog (`OptionalFeatures.exe`)
  * Enable 'Windows Subsystem for Linux'
  * Click 'OK' and restart if necessary
2. Install Ubuntu
  * Open Microsoft Store and search for "Ubuntu 18.04" or use [this link](https://www.microsoft.com/store/productId/9N9TNGVNDL3Q)
  * Click Install
3. Complete Installation
  * Open a cmd prompt and type "Ubuntu1804"
  * Create a new UNIX user account (this is a separate account from your Windows account)

After the bash shell is active, you can follow the instructions below, starting
with the "Cross-compilation" section. Compiling the 64-bit version is
recommended, but it is possible to compile the 32-bit version.

Cross-compilation for Ubuntu and Windows Subsystem for Linux
------------------------------------------------------------

The steps below can be performed on Ubuntu (including in a VM) or WSL. The depends system
will also work on other Linux distributions, however the commands for
installing the toolchain will be different.

First, install the general dependencies:

    sudo apt update
    sudo apt upgrade
    sudo apt install build-essential libtool autotools-dev automake pkg-config bsdmainutils curl git

A host toolchain (`build-essential`) is necessary because some dependency
packages need to build host utilities that are used in the build process.

See [dependencies.md](dependencies.md) for a complete overview.

If you want to build the windows installer with `make deploy` you need [NSIS](https://nsis.sourceforge.io/Main_Page):

    sudo apt install nsis

Acquire the source in the usual way:

    git clone https://github.com/asuka431/baricoin.git
    cd baricoin

## Building for 64-bit Windows

The first step is to install the mingw-w64 cross-compilation tool chain:

    sudo apt install g++-mingw-w64-x86-64

Ubuntu Bionic 18.04 <sup>[1](#footnote1)</sup>:

    sudo update-alternatives --config x86_64-w64-mingw32-g++ # Set the default mingw32 g++ compiler option to posix.

Once the toolchain is installed the build steps are common:

Note that for WSL the Baricoin Core source path MUST be somewhere in the default mount file system, for
example /usr/src/baricoin, AND not under /mnt/d/. If this is not the case the dependency autoconf scripts will fail.
This means you cannot use a directory that is located directly on the host Windows file system to perform the build.

Acquire the source in the usual way:

    git clone https://github.com/asuka431/baricoin.git

Once the source code is ready the build steps are below:

Do not compile tests and bench.

    PATH=$(echo "$PATH" | sed -e 's/:\/mnt.*//g') # strip out problematic Windows %PATH% imported var, not required for Ubuntu
    cd depends
    make HOST=x86_64-w64-mingw32
    cd ..
    mkdir dist
    ./autogen.sh
    CONFIG_SITE=$PWD/depends/x86_64-w64-mingw32/share/config.site ./configure --prefix=$PWD/dist --disable-tests --disable-bench --disable-dependency-tracking
    make install
    cd dist/bin
    strip baricoin-cli.exe baricoind.exe baricoin-qt.exe baricoin-tx.exe


## Building for 32-bit Windows

To build executables for Windows 32-bit, install the following dependencies:

    sudo apt install g++-mingw-w64-i686 mingw-w64-i686-dev

For Ubuntu Bionic 18.04 and Windows Subsystem for Linux <sup>[1](#footnote1)</sup>:

    sudo update-alternatives --config i686-w64-mingw32-g++  # Set the default mingw32 g++ compiler option to posix.

Note that for WSL the Baricoin Core source path MUST be somewhere in the default mount file system, for
example /usr/src/baricoin, AND not under /mnt/d/. If this is not the case the dependency autoconf scripts will fail.
This means you cannot use a directory that located directly on the host Windows file system to perform the build.

Acquire the source in the usual way:

    git clone https://github.com/asuka431/baricoin.git

Then build using:

Do not compile tests and bench.

    PATH=$(echo "$PATH" | sed -e 's/:\/mnt.*//g') # strip out problematic Windows %PATH% imported var, not required for Ubuntu
    cd depends
    make HOST=i686-w64-mingw32
    cd ..
    mkdir dist
    ./autogen.sh
    CONFIG_SITE=$PWD/depends/i686-w64-mingw32/share/config.site ./configure --prefix=$PWD/dist --disable-tests --disable-bench --disable-dependency-tracking
    strip src/baricoin-cli.exe src/baricoin-tx.exe src/baricoind.exe src/qt/baricoin-qt.exe
    make install
    cd dist/bin
    strip baricoin-cli.exe baricoind.exe baricoin-qt.exe baricoin-tx.exe


## Depends system

For further documentation on the depends system see [README.md](../depends/README.md) in the depends directory.

Installation
-------------

After building using the Windows subsystem it can be useful to copy the compiled
executables to a directory on the Windows drive in the same directory structure
as they appear in the release `.zip` archive. This can be done in the following
way. This will install to `c:\workspace\baricoin`, for example:

    make install DESTDIR=/mnt/c/workspace/baricoin

You can also create an installer using:

    make deploy

Footnotes
---------

<a name="footnote1">1</a>: Starting from Ubuntu Xenial 16.04, both the 32 and 64 bit Mingw-w64 packages install two different
compiler options to allow a choice between either posix or win32 threads. The default option is win32 threads which is the more
efficient since it will result in binary code that links directly with the Windows kernel32.lib. Unfortunately, the headers
required to support win32 threads conflict with some of the classes in the C++11 standard library, in particular std::mutex.
It's not possible to build the Baricoin Core code using the win32 version of the Mingw-w64 cross compilers (at least not without
modifying headers in the Baricoin Core source code).
