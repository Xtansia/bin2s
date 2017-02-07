# bin2s
Convert binary files to GCC assembly modules.

A C++ port of devkitPro's [bin2s](https://github.com/devkitPro/general-tools/blob/master/bin2s.c).

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

  unsigned int identifier_size = ...
  unsigned char identifier[identifier_size] = { ... }
  unsigned char identifier_end[] = identifier + identifier_size

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
