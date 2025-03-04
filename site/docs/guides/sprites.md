import MultiLanguageCode from '@site/src/components/MultiLanguageCode';

# Sprites

WASM-4 can [blit](https://en.wikipedia.org/wiki/Bit_blit) sprites directly to the screen. A sprite
is simply a pointer to raw bytes in memory.

`blit (spritePtr, x, y, width, height, flags)`

Sprites come in two formats: 1BPP and 2BPP.

## 1BPP Format

1BPP sprites require **1** **B**it **P**er **P**ixel. That means each pixel can be one of two
colors, either 0 or 1.

First let's define an 8x8 1BPP sprite called `smiley` at the top of our program. We do this by using
the binary literal syntax for each row of 8 pixels. If you look closely you can see it's a picture
of a smiley face: 🙂

<MultiLanguageCode>

```typescript
const smiley = memory.data<u8>([
    0b11000011,
    0b10000001,
    0b00100100,
    0b00100100,
    0b00000000,
    0b00100100,
    0b10011001,
    0b11000011,
]);
```

```c
const uint8_t smiley[] = {
    0b11000011,
    0b10000001,
    0b00100100,
    0b00100100,
    0b00000000,
    0b00100100,
    0b10011001,
    0b11000011,
};
```

```rust
const smiley: [u8; 8] = [
    0b11000011,
    0b10000001,
    0b00100100,
    0b00100100,
    0b00000000,
    0b00100100,
    0b10011001,
    0b11000011,
];
```

```go
var smiley = [8]byte {
    0b11000011,
    0b10000001,
    0b00100100,
    0b00100100,
    0b00000000,
    0b00100100,
    0b10011001,
    0b11000011,
}
```

</MultiLanguageCode>

Now that we have our sprite data, we can blit it to the screen at position (10, 10).

<MultiLanguageCode>

```typescript
w4.blit(smiley, 10, 10, 8, 8, w4.BLIT_1BPP);
```

```c
blit(smiley, 10, 10, 8, 8, BLIT_1BPP);
```

```rust
blit(&smiley, 10, 10, 8, 8, BLIT_1BPP);
```

```go
w4.Blit(&smiley[0], 10, 10, 8, 8, w4.BLIT_1BPP)
```

</MultiLanguageCode>

Just like any other drawing function, we can set [`DRAW_COLORS`](basic-drawing) to change the
colors.

The last parameter to `blit()` is a flags bitset. We can bitwise OR (`|`) flags together to modify
the behavior of `blit()`. For example, to flip the sprite vertically: 🙃

<MultiLanguageCode>

```typescript
w4.blit(smiley, 10, 10, 8, 8, w4.BLIT_1BPP | w4.BLIT_FLIP_Y);
```

```c
blit(smiley, 10, 10, 8, 8, BLIT_1BPP | BLIT_FLIP_Y);
```

```rust
blit(&smiley, 10, 10, 8, 8, BLIT_1BPP | BLIT_FLIP_Y);
```

```go
w4.Blit(&smiley[0], 10, 10, 8, 8, w4.BLIT_1BPP | w4.BLIT_FLIP_Y)
```

</MultiLanguageCode>

For a full list of blit flags, see the [Functions](/docs/reference/functions) reference.

## 2BPP Format

2BPP sprites require **2** **B**its **P**er **P**ixel. That means each pixel can be one of four
colors.

Unlike with 1BPP, it's harder to "draw" the sprite directly in the source code with binary literal
ASCII art. An alternative is to use an image editor with a palette of 4 colors and save an indexed
PNG. We can then use `w4 png2src` to convert it to source code to paste into our program.

For example, if we have this 4-color image of a bunny:

<img src="/img/bunny.png" width="160" className="pixelated"/>

Note that the RGB color of this sprite doesn't matter, `w4 png2src` only cares about a pixel's
palette index.

```shell
w4 png2src --assemblyscript bunny.png
```

Will print out the following generated source:

```typescript
const bunny_width = 16;
const bunny_height = 16;
const bunny_flags = 1; // BLIT_2BPP
const bunny = memory.data<u8>([ 0xaa,0x9e,0xac,0xaa,0xaa,0x57,0xbf,0x2a,0xaa,0x57,0xbf,0x2a,0xaa,0x17,0xbf,0x2a,0xaa,0x17,0x03,0x2a,0xaa,0x57,0x54,0x2a,0xa8,0x55,0x55,0x6a,0xa9,0x55,0x05,0x0a,0xaf,0xd5,0x55,0x4a,0xa8,0x75,0x55,0x4a,0xaa,0xd5,0x57,0x2a,0xaa,0x1d,0x7c,0xaa,0xa8,0x75,0x15,0x2a,0xa8,0x45,0x15,0x2a,0xaa,0x10,0x54,0xaa,0xaa,0x85,0x52,0xaa ]);
```

Pasting that into our program, we can draw a bunny like so:

```typescript
*DRAW_COLORS = 0x2013;
blit(bunny, 10, 10, bunny_width, bunny_height, bunny_flags);
```

[`DRAW_COLORS`](basic-drawing) is used here to modify the 4 colors of the original sprite. Reading
from right to left: color 1 becomes 3, color 2 becomes 1, color 3 becomes 0 (transparent), and color
4 becomes 2.

### Custom template

You can use a custom template for generating a image source.
Use a `--template filename` for this.

Basic template (C):
```
#define %name%Width %width%
#define %name%Height %height%
#define %name%Flags %flagsHumanReadable%
const uint8_t %name%[%length%] = { %bytes% };
```

Where:
- `%name%` - filename (string),
- `%idiomaticName%` - Rust specific variable name (string)
- `%width%`, `%height%` - image dimensions (integer)
- `%flags%`, `%flagsHumanReadable%` - type flag as integer and enum name (BLIT_2BPP or BLIT_1BPP)
- `%length%` - count of bytes (integer)
- `%bytes%` - comma separated series of bytes (string)
