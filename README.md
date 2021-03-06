# Jojo AlterNative Patch (JANPatch)

Memory-efficient library which can apply [JojoDiff](http://jojodiff.sourceforge.net) patch files. Very useful for embedded development, for example for delta firmware updates. Written in C, and can be backed by anything that offers basic I/O primitives.

This is an implementation from scratch, done by reverse engineering the diff file format, and inspecting the diff generation code. It's not a derived work from jptch (the patch application in JojoDiff). The reason for this is that JojoDiff is licensed under GPLv3, which does not allow linking in the library in non-GPL applications. JANPatch is licensed under Apache 2.0.

For an example of using this library on an embedded system, see [binary-diff-mbedos5](https://github.com/janjongboom/binary-diff-mbedos5).

## Usage (CLI)

On POSIX systems (macOS, Linux, Cygwin under Windows):

1. Build the CLI, via:

    ```
    $ make
    ```

1. Run `./janpatch-cli` with the old file, a JojoDiff patch file, and the destination:

    ```
    $ ./janpatch-cli demo/blinky-k64f-old.bin demo/blinky-k64f.patch ./blinky-k64f-patched.bin
    ```

1. Verify that the patch was applied successfully:

    ```
    $ diff demo/blinky-k64f-new.bin blinky-k64f-patched.bin

    # should return nothing
    ```

## Usage (library)

JANPatch is implemented in a single header file, which you can copy into your project. For portability to non-POSIX platforms you need to provide the library with function pointers to basic IO operations. These are `fread`, `fwrite`, `fseek` and `ftell`. The file pointer for these functions is of type `JANPATCH_STREAM`, which you can set when building.

The functions are defined in a context, for example on POSIX systems, you define JANPatch like this:

```cpp
#define JANPATCH_STREAM     FILE        // use POSIX FILE

#include "janpatch.h"

// fread/fwrite buffers for every file, minimum size is 1 byte
// when you run on an embedded system with block size flash, set it to the size of a block for best performance
char buffer[1024];

janpatch_ctx ctx = {
    { (unsigned char*)malloc(1024), 1024 },
    { (unsigned char*)malloc(1024), 1024 },
    { (unsigned char*)malloc(1024), 1024 },

    // define functions which can perform basic IO
    // on POSIX, use:
    &fread,
    &fwrite,
    &fseek,
    &ftell
};
```

Then create three objects of type `JANPATCH_STREAM` and call `janpatch` with the context:

```cpp
JANPATCH_STREAM *source = fopen("source.bin", "rb");
JANPATCH_STREAM *patch  = fopen("patch", "rb");
JANPATCH_STREAM *target = fopen("target.bin", "wb");

janpatch(ctx, source, patch, target);
```

For a demonstration of using this library on a non-POSIX system, see [binary-diff-mbedos5#xdot](https://github.com/janjongboom/binary-diff-mbedos5/tree/xdot).

### Reporting progress

Exact progress indication is hard, as the size of the file after patching is not known, but you can get rudimentary progress (based on the pages written to the target stream) by setting the `progress` property on the context:

```
void progress(uint8_t percentage) {
    printf("Patch progress: %d%%\n", percentage);
}

ctx.progress = &progress;
```

## Generating patch files

To generate patch files you'll need to build [JojoDiff](http://jojodiff.sourceforge.net).

1. Download and extract the source code.
1. Open a terminal, and build via:

    ```
    $ cd src
    $ make
    ```

1. Generate a patch file via:

    ```
    $ ./jdiff old-file.bin new-file.bin old-to-new-file.patch
    ```

## License

Apache License version 2. See [LICENSE](LICENSE).
