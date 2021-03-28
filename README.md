# zlib

This is a simple CMake package for the [zlib](https://zlib.net) compression library.

Note that the zlib source distribution's _contrib/_ directory is not part of zlib, and is therefore
not part of this project.

## CMake options

- `Z_ENABLE_ASSEMBLY_X86`: Enable building the x86 assembly sources.
- `Z_ENABLE_ASSEMBLY_X86_64`: Enable building the x86_64 assembly sources.
- `Z_ENABLE_INSTALL`: Install the zlib binaries on the system.
- `Z_ENABLE_INSTALL_DEVELOPMENT`: Install the zlib development files on the system.
- `Z_ENABLE_TESTS`: Build the zlib test suite.
- `Z_ENABLE_SHARED`: Build zlib as a shared library. (eg. `zlib.dll`)
- `Z_ENABLE_STATIC`: Build zlib as a static library. (eg. `libzlibstatic.a`)

By default both shared and static libraries are built, and at least one must be built by the script 
or it'll die.

## Licensing

The sources provided by this project are licensed under the ISC (or OpenBSD) license. For this
license see the _LICENSE.txt_ file, second header.

These file are: _CMakeLists.txt_.

The original sources are properties of Jean-loup Gailly and Mark Adler, and are licensed under the 
zlib license. For this license see the _LICENSE.txt_ file, first header.

