# sdl3-image

SDL_image 3 for sysl — image files decoded into a surface or straight into a texture, and surfaces
written back out.

```
dependencies {
  sdl3       { git = "github.com/sysl-lang/sdl3",       version = "0.2.0" }
  sdl3-image { git = "github.com/sysl-lang/sdl3-image", version = "0.2.0" }
}
```

```sysl
import sh.sysl.sdl3.*
import sh.sysl.sdl3_image.*
import sh.sysl.sdl3.c.INIT_VIDEO

main()
    init(INIT_VIDEO)

    val window = create_window("picture", 800, 600, 0).expect("a window")
    val renderer = window.create_renderer().expect("a renderer")
    val picture = load_texture(renderer, "photo.png").expect("the image decoded")

    renderer.clear_to(BLACK)
    renderer.copy(picture)
    renderer.present()
    delay(2000)
```

## Installing

```
brew install sdl3_image                 # pulls sdl3 with it
sysl run prog.sysl
```

No flags — see [`sdl3`](https://github.com/sysl-lang/sdl3)'s README, which also says why this is a
separate package rather than a module inside that one.

**This package transcribes no constants at all**, so it asks the C compiler for nothing and reads no
header: SDL_image's whole surface is functions. `sh.sysl.sdl3_image.c` is ten `extern`s and an import
of `sh.sysl.sdl3.c`'s handles, and that is the entire C layer. What it needs is the **library**, at
link time — and until 0.2.1 this file said nothing about that at all, so a machine without SDL_image
found out from the linker. It names it now, and pkg-config supplies the link line.

**Needs sysl 0.0.56.**

## Handles own themselves

A surface or a texture from here is a `&T` with an `impl Drop`, exactly as one from `sdl3` is, so
there is no `destroy` anywhere in this API — a decoded image goes when the last reference to it
does.

## There is no init

SDL_image 3 loads its decoders on demand, so unlike the text and mixer bindings there is nothing to
bring up: `load`, `load_texture` and the savers are the whole of it.

Which formats are available depends on what the installed SDL3_image was built with. PNG, JPEG, BMP,
GIF, TGA, PNM, XPM and QOI are always there; AVIF, JXL, TIFF and WEBP are build options.

## Loading sniffs the bytes; saving reads the name

**`load` decides the format from the file's content**, so a PNG called `photo.jpg` decodes as a PNG.
That is SDL_image's behaviour and it is the right one — a program that trusted the extension would
be wrong about every file a user has renamed. The tests pin it by writing a PNG to a `.jpg` path and
asserting it comes back losslessly.

**`save` decides from the extension**, which is the one place a name decides, because there is no
content to sniff yet. `save_png`, `save_jpg`, `save_bmp`, `save_gif` and `save_tga` name the encoder
outright where that is clearer.

## Tests

```
sysl test .
```

Eight tests, headless. **They make their own input** — a test needing a PNG checked into the
repository would be testing that git still holds a file, so a surface is drawn here, written out and
read back, and the encoder and decoder check each other.

The picture is a flat colour on purpose: JPEG transforms 8×8 blocks, so a single coloured pixel
comes back smeared across its whole block and no tolerance worth writing would pass. A flat field
survives any quantizer, which leaves the round trip testing the codec rather than testing what JPEG
does to a spike.

## License

ISC — see [LICENSE](LICENSE).
