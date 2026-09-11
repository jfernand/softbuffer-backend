# softbuffer-backend

Glyph rasterization and a [`ratatui`](https://docs.rs/ratatui) `Backend` that
renders directly into a [`winit`](https://docs.rs/winit) window via
[`softbuffer`](https://docs.rs/softbuffer) — no terminal emulator, PTY, or
ANSI escape sequences involved.

This crate is the reusable rendering layer: it owns glyph rasterization and
the `Backend` implementation, but not an event loop or any application UI.

## Modules

- **`glyph`** — rasterizes a fixed, known set of characters up front
  (printable ASCII, Box Drawing, and Block Elements from a primary monospace
  font, plus Braille Patterns from a fallback font) into a monospace cell
  grid, so drawing a frame never touches the font rasterizer at runtime.
  Characters outside those ranges fall back to `?`.
- **`backend`** — a `ratatui::backend::Backend` (`WinitBackend`) that
  translates each cell into a filled background rectangle plus a rasterized
  glyph, and presents the result into a `softbuffer` surface bound to a
  `winit` window.

## Usage

Add the crate and create a `WinitBackend` around a `winit::window::Window`,
then drive a `ratatui::Terminal` as usual — the application (event loop,
input handling, window setup) is the caller's responsibility.

```rust
use ratatui::Terminal;
use softbuffer_backend::backend::WinitBackend;
use softbuffer_backend::glyph::GlyphCache;

// `window: Rc<winit::window::Window>` created and owned by the app.
let glyph_cache = GlyphCache::new(16.0); // font size in pixels
let backend = WinitBackend::new(window, glyph_cache)?;
let mut terminal = Terminal::new(backend)?;

terminal.draw(|frame| {
    // draw ratatui widgets as usual
})?;
```

## License

Licensed under either of [Apache License, Version 2.0](LICENSE-APACHE) or
[MIT license](LICENSE-MIT) at your option.

Bundled fonts (`assets/fonts`) are Bitstream Vera / DejaVu fonts; see
`assets/fonts/DejaVu-LICENSE.txt`.
