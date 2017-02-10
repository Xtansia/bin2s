# bin2s
Convert binary files to GCC assembly modules.

A C++11 port of devkitPro's [bin2s](https://github.com/devkitPro/general-tools/blob/master/bin2s.c).

```
bin2s {OPTIONS} [FILES...]

  Convert binary files to GCC assembly modules.

OPTIONS:

    -h, --help                        Show this help message and exit
    -a [ALIGNMENT],
    --alignment=[ALIGNMENT]           Boundary alignment, in bytes [default:
                                      4]
    -l [LINE_LENGTH],
    --line-length=[LINE_LENGTH]       Number of bytes to output per line of
                                      ASM [default: 16]
    -o [OUTPUT], --output=[OUTPUT]    Output file, "-" represents stdout
                                      [default: -]
    FILES...                          Binary file to convert to GCC assembly
    "--" can be used to terminate flag options and force all following
    arguments to be treated as positional options


For each input file it will output assembly defining:

  * {identifier}:
      An array of bytes containing the data.
  * {identifier}_end:
      Will be at the location directly after the end of the data.
  * {identifier}_size:
      An unsigned int containing the length of the data in bytes.

Roughly equivalent to this pseudocode:

  const unsigned int identifier_size = ...
  const unsigned char identifier[identifier_size] = { ... }
  const unsigned char identifier_end[] = identifier + identifier_size

Where {identifier} is the input file's name,
sanitized to produce a legal C identifier, by doing the following:

  * Stripping all character that are not ASCII letters, digits or one of _-./
  * Replacing all of -./ with _
  * Prepending _ if the remaining identifier begins with a digit.

e.g. for gfx/foo.bin {identifier} will be foo_bin,
     and for 4bit.chr it will be _4bit_chr.
```

## Example

Given input file `hello_world.txt`:

```
Hello World
```

It will produce the following assembly:

```
  .section .rodata
  .balign 4
  .global hello_world_txt
  .global hello_world_txt_end
  .global hello_world_txt_size

hello_world_txt:
  .byte  72,101,108,108,111, 32, 87,111,114,108,100

hello_world_txt_end:

  .align
hello_world_txt_size: .int 11
```

You can then use it from your program by for example creating a header like so:

```c
extern const unsigned char hello_world_txt[];
extern const unsigned char hello_world_txt_end[];
extern const unsigned int hello_world_txt_size;
```

## CMake

Here's an example CMake snippet to automatically download & build bin2s, 
and add a function for creating a library target from binary files.

```cmake
include(ExternalProject)

ExternalProject_Add(bin2s_git
  PREFIX vendor/
  GIT_REPOSITORY https://github.com/Xtansia/bin2s
  GIT_TAG master
  GIT_SUBMODULES
  UPDATE_COMMAND ""
  PATCH_COMMAND ""
  BUILD_COMMAND ""
  CMAKE_ARGS 
    "-DCMAKE_BUILD_TYPE=Release" 
    "-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>"
  INSTALL_COMMAND
    "${CMAKE_COMMAND}"
    --build .
    --target install
    --config Release)

add_executable(bin2s IMPORTED GLOBAL)
set_target_properties(bin2s PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/vendor/bin/bin2s)
add_dependencies(bin2s bin2s_git)

function(add_binfile_library target_name)
  if (NOT ${ARGC} GREATER 1)
    message(FATAL_ERROR "add_binfile_library : Argument error (no input files)")
  endif()
  
  get_cmake_property(_enabled_languages ENABLED_LANGUAGES)
  if (NOT _enabled_languages MATCHES ".*ASM.*")
    message(FATAL_ERROR "add_binfile_library : ASM language needs to be enabled")
  endif()

  set(_output_dir ${CMAKE_CURRENT_BINARY_DIR}/binfile_asm)
  set(_output_file ${_output_dir}/${target_name}.s)
  
  file(MAKE_DIRECTORY ${_output_dir})

  add_custom_command(OUTPUT ${_output_file}
                     COMMAND bin2s -o "${_output_file}" ${ARGN}
                     DEPENDS ${ARGN}
                     WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
  
  add_library(${target_name} ${_output_file})
endfunction()
```

Which you could then use like so:

```cmake
add_binfile_library(resources res/some_picture.png)

add_executable(myprogram src/main.cpp)
target_link_libraries(myprogram resources)
```
