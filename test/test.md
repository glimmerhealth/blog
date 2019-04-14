# WASM Port of cmark-gfm

A WASM module implementing MD --> HTML with github extensions. 



## Release

As of this writing, this is based off of version [0.28.3.gfm.20](https://github.com/github/cmark-gfm/releases/tag/0.28.3.gfm.20)



## PC Build

```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=RELEASE ..

# For Binary
make -j8 cmake-gfm

# For libraries
make -j8 libcmark-gfm-extensions_static
make -j8 libcmark-gfm_static

# Example use of binary
src/cmark-gfm -e table -e footnotes -e strikethrough -e autolink -e tagfilter -e tasklist -t html ../../README.md > out.html
```



## WASM Build

WASM build consists of 2 parts: 

### Regular: 

1. A codeblocks project used for PC based unit testing was used to generate a shared library (.a). 
2. cbp2mak was used to generate a Makefile with debug and release targets
3. Compiler was switched to `emcc, em++, emar, ...`
4. Compilation was accelerated with 8 threads `make -j8 release`
5. A bld.sh is used for actual WASM generation. Specifically the `emcc -O3 -s WASM=1 -s EXPORTED_FUNCTIONS="[ '_malloc', '_free']" $TARG` generated the .wasm and .js

### ByteCode

1. A `wasm-bytecode` target was added to the codeblocks project mentioned above. It mimics the `release` config in every way
2. cbp2make is used to generate a Makefile. The following hand edits are applied:
   1. `OUT_WASM_BYTECODE = bin/wasm/libcmark-gfm-wasm.a` is changed to `OUT_WASM_BYTECODE = bin/wasm/libcmark-gfm-wasm.bc`
   2. `$(AR) rcs $(OUT_WASM_BYTECODE) $(OBJ_WASM_BYTECODE)` is changed to `emcc -O3 $(OBJ_WASM_BYTECODE) -o $(OUT_WASM_BYTECODE)` 
3. With the above changes, and general compiler toolchain switch to emcc..., we now have a target that spits out bytecode like this: 
   1. `make -j8 wasm-bytecode` 
4. Additionally, the previously written bld.sh remains in place for generating a .wasm and .js wrapper for standalone builds, also useful for unit testing

## Updating This Code

Update consists of the following steps:

1. Download a release .zip from cmark-gfm repo
2. Build regular desktop binary and check that this file renders fine
3. Replace cmark-gfm-X* folder with newly downloaded
4. Diff for any changes to the main function in `src/main.c`. Incorporate the changes
5. Test with new app that this file renders fine
6. Update wasm/lib



## Test Beyond Here

The rest of this document is for testing cmark-gfm

## Reason for existence

1. WYSIWYG editor for markdown on the web doesn’t exist
2. 1 Is even worse on mobile devices
3. As a learning experience building a full-fledged web application
4. Opportunity to showcase ZenLibs

## Features

1. Automatic save to local storage
2. Ability to save directly to GitHub
3. Markdown to HTML5 in native

## TODO

- [x] Convert to PWA
  - [x] Add SW from traversy code
  - [x] Add app-manifest, and check that things work as a pwa
- [x] Switch between edit and view mode
- [ ] Use BrowserFS for: 
  - [x] Local file storage
  - [x] Sync with dropbox
  - [ ] Tie-in with UI for clean UX
- [ ] Connect file-drawer & local storage for ~~meaningful~~ navigation
- [ ] Add shortcuts for: 
  - [ ] new file
  - [ ] new folder
  - [ ] delete file/folder

## FootNotes Test

Here's a simple footnote,[^1] and here's a longer one.[^bignote]


## Table Test

| Some    | Dummy  | Table            |
| ------- | ------ | ---------------- |
| Stuuf   | To     | grease           |
| the     | wheels | of               |
| fortune | which  | favors the brave |

# Code

```C
#define MAX_NUM (0xFFFFF)
int main(int argc char *argv[])
{
    return 0;
}
```

## Footnotes
[^1]: This is the first footnote.

[^bignote]: Here's one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.
    
    `{ my code }`
    
    Add as many paragraphs as you like.