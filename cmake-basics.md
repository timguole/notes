# CMake Basics

## Basics

The input file name must be 'CMakeLists.txt'.

#### How to invoke

```shell
cd project-root # top-level CMakeLists.txt resides here.

# create a directory for building. This makes the project-root dir clean and intact.
# This dir can also be outside of the project-root.
mkdir build

cd build
cmake ..
cmake --build .

# several ways to install software
cmake --install .
cmake --install . --config Release
cmake --build . --target install --config Release
cmake --install . --prefix DIR
```



## Commands

#### project()

```cmake
project(name VERSION x.y)
```

The version 'x.y' can be accessed from configure-file with '@emu51_VERSION_MAJOR@' and '@emu51_VERSION_MINOR@'. For example:

```c
/* the x */
#define EMU51_VERSION_MAJOR @emu51_VERSION_MAJOR@
/* the y */
#define EMU51_VERSION_MINOR @emu51_VERSION_MINOR@
```

#### cmake_minimum_required()

```cmake
cmake_minimum_required(VERSION 3.15)
```

#### configure_file()

```cmake
configure_file(input-file output-file)
```

#### add_executable()

```cmake
add_executable(program-name source-file ...)
```

the 'program-name' is a target.

#### target_link_libraries()

```cmake
target_link_libraries(exe-or-lib-name PUBLIC lib-name)
```

This is for gcc '-l' option.

#### target_include_directories()

```cmake
target_include_directories(target-name PUBLIC dir1 dir2 ...)
```

This is for the gcc '-I' option.

#### target_compile_options()

```cmake
target_compile_options(target-name PUBLIC compiler-flags)
```

#### add_subdirectory()

```cmake
add_subdirectory(dir-name)
```

The sub-dir is another 'cmake managed project'.

#### add_library()

```cmake
add_library(lib-name SHARED|STATIC source-file ...)
```

This create a shared or static library.

#### install()

```cmake
install(TARGETS t1 t2 ... DESTNATION dir)
install(FILES f1 f2 ... DESTNATION dir)
```

#### file()

```cmake
file(GLOB name "dir/*.*")
```

#### set()

```cmake
set(name value)
```



## Generator expressions

The basic format is: $\<expr:...>. Generator expressions can be nested. The expression return true/false or the argument string.

Special expressions:

$\<1:...>: return the argument string

$\<0:...>: return false or empty string?

## Macros & variables

#### PROJECT_SOURCE_DIR

The top level project root directory

#### CMAKE_CURRENT_SOURCE_DIR

The current sub-project source dir

#### PROJECT_BINARY_DIR

The binary dir in build

#### CMAKE_INSTALL_PREFIX

Like '--prefix' in configure script.

## Usage requirement

#### INTERFACE

It means the configuration is needed by consumer, but not producer. Producer add this configuration for consumers, so consumers can use it more easily.