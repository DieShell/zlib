# zlib

This is a simple CMake package for the [zlib](https://zlib.net) compression library.

Note that the zlib source distribution's _contrib/_ directory is not part of zlib, and is therefore
not part of this project.

## CMake options

- `Z_ENABLE_ASSEMBLY_X86`: Enable building the x86 assembly sources.
- `Z_ENABLE_ASSEMBLY_X86_64`: Enable building the x86_64 assembly sources.
- `Z_ENABLE_INSTALL`: Install the zlib binaries on the system.
- `Z_ENABLE_INSTALL_DEVELOPMENT`: Install the zlib development files on the system.
- `Z_ENABLE_EXAMPLES`: Build the zlib library examples.
- `Z_ENABLE_SHARED`: Build zlib as a shared library. (eg. `zlib1.dll`)
- `Z_ENABLE_STATIC`: Build zlib as a static library. (eg. `libzlibstatic.a`)

By default both shared and static libraries are built, and at least one must be built by the script 
or it'll die.

## Licensing

The project as a whole is licensed under the zlib license, however, different files have different 
copyright owners.

The sources provided by the CMake wrapper are licensed as provided in the _LICENSE.txt_ file, 
second header.

The original sources are properties of Jean-loup Gailly and Mark Adler, and are licensed as
provided in the _LICENSE.txt_ file, first header.

Files included in the CMake wrapper are: _CMakeLists.txt_, _cmake/zlibConfig.in.cmake_, _examples/CMakeLists.txt_.
All other files are part of the original zlib library.
