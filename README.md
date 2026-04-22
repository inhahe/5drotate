# 5D Pixel Rotator

Load an image and rotate it in five dimensions. Every pixel becomes a point in
5D space — two spatial coordinates (X, Y) and three color coordinates (R, G, B).
Twenty sliders let you rotate any axis toward any other axis, blending position
and color in ways that don't exist in normal image editing.

Open `5d-voxel-rotator.html` in a browser. No build step, no dependencies.

## How it works

A pixel at column 80, row 40 with color rgb(200, 100, 50) becomes the 5D point:

    (80, 40, 200, 100, 50)

All five coordinates are centered and normalized so they carry roughly equal
weight during rotation. A 5x5 rotation matrix is built from however many of the
20 sliders are non-zero, then applied to every pixel each frame. The output X
and Y become the pixel's new screen position; the output R, G, B become its new
color.

This means a single rotation can:

- **Move pixels based on their color** — bright red pixels drift one way, dark
  blue pixels drift another.
- **Recolor pixels based on their position** — the left side of the image turns
  blue while the right side turns red.
- **Do both at once** when you stack multiple sliders.

## The 20 axes

In 5D there are 5 x 4 = 20 directed rotation axes, organized by source
dimension:

| Group   | Sliders              |
|---------|----------------------|
| **X**   | X→Y  X→R  X→G  X→B  |
| **Y**   | Y→X  Y→R  Y→G  Y→B  |
| **Red** | R→X  R→Y  R→G  R→B  |
| **Green** | G→X  G→Y  G→R  G→B |
| **Blue**  | B→X  B→Y  B→R  B→G |

"X→R" means "rotate the X axis toward the Red axis." At +90 degrees, what was
purely spatial-X becomes purely Red. Pixels that were far right become bright
red; pixels that were far left become dark (or go out of gamut).

X→Y and Y→X share the same geometric plane but push in opposite directions.
Cranking both the same way partially cancels; cranking them opposite ways
compounds.

## Out-of-gamut clipping

When a rotation pushes a pixel's R, G, or B outside 0–255, it has left the
visible color cube. The **OOB color** toggle controls what happens:

- **Magenta** (default) — the pixel renders as #FF00FF, making clipped regions
  obvious.
- **Black** — clipped pixels vanish into the background.
- **Clamp** — each channel is clamped to [0, 255] independently, which
  preserves some color information but distorts hue.

The info bar at the bottom shows what percentage of pixels are currently
clipped. A faint dashed rectangle on the canvas marks the original image
bounds so you can see which pixels have drifted outside.

## Controls

### Sidebar

| Control       | What it does |
|---------------|-------------|
| Drop zone     | Load a JPEG, PNG, or GIF. Images are processed at full native resolution. |
| Dot scale     | Multiplier on the auto-computed splat size. 1.0 = seamless tiling. < 1 = gaps (pointillist). > 1 = overlap (blobby). |
| Zoom          | Scale the point cloud on the canvas. Also controllable via mouse wheel. |
| OOB color     | Magenta / Black / Clamp toggle for out-of-gamut pixels. |
| 20 sliders    | -180 to +180 degrees each. Double-click a slider to zero it. |
| Reset All     | Zero every slider. |
| Animate       | Randomize rotation speeds and spin continuously. |
| Randomize     | Jump to random angles on all 20 axes. |
| Export PNG    | Save the current canvas as a PNG. |

### Keyboard

| Key     | Action |
|---------|--------|
| Space   | Toggle animation |
| R       | Reset all rotations |
| E       | Export PNG |

### Mouse

Scroll wheel over the canvas to zoom in/out.

## Performance

Every pixel of the loaded image is transformed each frame — there's no
downsampling. The per-frame cost scales linearly with pixel count:

- ~100K pixels (300x300): 60 fps
- ~1M pixels (1000x1000): 30–60 fps
- ~4M pixels (2000x2000): 10–20 fps

If frame rate drops, zoom the browser tab or resize the window (smaller canvas =
less pixel-write work). The FPS counter in the bottom-right corner of the
sidebar shows the actual rate.

## Technical notes

**Normalization.** Spatial coordinates are divided by `max(width, height)` to
preserve aspect ratio. Color coordinates are divided by 256. All dimensions end
up in roughly the [-0.5, 0.5] range so rotations mix them equally.

**Matrix composition.** The 20 rotation matrices are multiplied left-to-right in
a fixed order (X→Y, X→R, ... B→G). Since rotations in different planes don't
commute, the order matters — the same slider values always produce the same
result, but reordering the sliders in code would change the output.

**Rendering.** An ImageData buffer is written directly — no Canvas 2D draw calls
per pixel. The splat size is `ceil(displayScale / max(imgW, imgH) * dotScale)`
so that at identity rotation with dot scale 1.0, adjacent pixels tile with no
gaps.

## License

Public domain. Do whatever you want with it.
