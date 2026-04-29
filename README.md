# 5D Pixel Rotator

Load an image and rotate it in five dimensions. Every pixel becomes a point in
5D space — two spatial coordinates (X, Y) and three color coordinates. Choose
between **RGB** (Red, Green, Blue) and **HSV** (Hue, Saturation, Value) color
spaces — the sliders and rendering adapt accordingly. Ten sliders let you rotate
any axis toward any other axis, blending position and color in ways that don't
exist in normal image editing.

**[Live demo](http://inhahe.com/5d-voxel-rotator.html)** — or open
`5d-voxel-rotator.html` locally in a browser. No build step, no dependencies.

## How it works

A pixel at column 80, row 40 with color rgb(200, 100, 50) becomes the 5D point:

    RGB mode: (80, 40, 200, 100, 50)
    HSV mode: (80, 40, 14°, 0.75, 0.78)

All five coordinates are centered and normalized so they carry roughly equal
weight during rotation. A 5x5 rotation matrix is built from however many of the
10 sliders are non-zero, then applied to every pixel each frame. The output X
and Y become the pixel's new screen position; the output color channels become
its new color (converted back to RGB for display).

This means a single rotation can:

- **Move pixels based on their color** — bright red pixels drift one way, dark
  blue pixels drift another.
- **Recolor pixels based on their position** — the left side of the image turns
  blue while the right side turns red.
- **Do both at once** when you stack multiple sliders.

In **HSV mode**, the same idea applies but the color axes are perceptually
meaningful: rotating X↔H moves pixels left/right based on their hue, while X↔V
separates bright from dark spatially. HSV rotations tend to produce smoother
color transitions since hue is inherently circular.

## The 10 rotation planes

In 5D there are 5×4/2 = 10 unique rotation planes — every way to pick two axes
out of five:

    RGB mode:             HSV mode:
    X↔Y  X↔R  X↔G  X↔B  X↔Y  X↔H  X↔S  X↔V
         Y↔R  Y↔G  Y↔B       Y↔H  Y↔S  Y↔V
              R↔G  R↔B            H↔S  H↔V
                   G↔B                 S↔V

Each slider goes from -180° to +180°. Positive values rotate the first axis
toward the second; negative values rotate the other way. Double-click a slider
to zero it.

"X↔R" at +90° means what was purely spatial-X becomes purely Red. Pixels that
were far right become bright red; pixels that were far left become dark (or go
out of gamut). In HSV mode, "X↔H" at +90° rotates spatial position into hue —
pixels on the right become red/orange, pixels on the left become blue/violet.

## Out-of-gamut clipping

When a rotation pushes a pixel's color outside the valid range, it has left the
visible gamut. In RGB mode this means any channel outside 0–255; in HSV mode it
means S or V outside [0, 1] (hue wraps freely since it's circular). The
**OOB color** toggle controls what happens:

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
| Color wt      | Ratio of color depth to spatial extent. At 1.0 color and position carry equal weight. Higher values make color axes "longer" — rotations mix color more aggressively. Lower values keep pixels spatially anchored. |
| Color space   | RGB / HSV toggle. Changes which color axes are used for the 5D embedding. Slider labels update to match. |
| OOB color     | Magenta / Black / Clamp toggle for out-of-gamut pixels. |
| 10 sliders    | -180 to +180 degrees each. Double-click a slider to zero it. |
| Reset All     | Zero every slider. |
| Animate       | Randomize rotation speeds and spin continuously. |
| Randomize     | Jump to random angles on all 10 axes. |
| Export PNG    | Save the current canvas as a PNG. |

### Keyboard

| Key     | Action |
|---------|--------|
| Space   | Toggle animation |
| R       | Reset all rotations |
| E       | Export PNG |
| C       | Toggle color space (RGB ↔ HSV) |

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
preserve aspect ratio. In RGB mode, color channels are divided by 256. In HSV
mode, hue is divided by 360 and centered at 180°; S and V are centered at 0.5.
All dimensions end up in roughly the [-0.5, 0.5] range. The **Color weight**
slider then scales the color dimensions relative to spatial — at 1.0 they carry
equal weight; at 2.0 color is twice as "wide" as position, so rotations that
mix color and space move pixels further.

**Matrix composition.** The 10 rotation matrices are multiplied left-to-right in
a fixed order (X↔Y, X↔R, ... G↔B). Since rotations in different planes don't
commute, the order matters — the same slider values always produce the same
result, but reordering the planes in code would change the output.

**Rendering.** An ImageData buffer is written directly — no Canvas 2D draw calls
per pixel. The splat size is `ceil(displayScale / max(imgW, imgH) * dotScale)`
so that at identity rotation with dot scale 1.0, adjacent pixels tile with no
gaps.

## License

Public domain. Do whatever you want with it.
