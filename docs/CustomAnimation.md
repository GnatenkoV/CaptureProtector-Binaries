# Custom animation

CaptureProtector supports `customAnimation` for **one text, image, or QR-code item**. It is intentionally numeric-only: expressions can position, rotate, fade, or scale the item, but they cannot call Windows APIs, read files, or run scripts.

`customAnimation` belongs directly on a `Text[]`, `Image[]`, or `QRCode[]` item. Use lower-camel JSON property names when saving from the Controller. The native loader also accepts compatible PascalCase aliases for manually authored files.

```json
{
    "Path": "../assets/logo.png",
    "customAnimation": {
        "enabled": true,
        "pauseWhenWindowHidden": true,
        "updateIntervalMilliseconds": 16,
        "xExpression": "{left} + tri({t} / 7.0) * {availableWidth}",
        "yExpression": "{top} + tri({t} / 4.8 + 0.35) * {availableHeight}",
        "rotationExpression": "0",
        "opacityExpression": "1",
        "scaleExpression": "1"
    },
    "Details": {
        "Display": {
            "Size": { "Width": 96, "Height": 96 },
            "Stretch": "Uniform"
        }
    }
}
```

The example is the **DVD bounce** preset available in the visual-rule editor. Different periods and a phase offset prevent the path from repeating as a single diagonal too quickly.

## Controller setup

Select a **Text**, **Image**, or **QR code** rule in the visual configuration editor. The **Custom animation** block remains compact until its toggle is enabled; the setup then expands with a short transition. Use the dedicated controls to:

- choose a preset: **DVD bounce**, **Horizontal sweep**, **Vertical sweep**, **Orbit**, **Figure eight**, **Wave drift**, **Pendulum**, **Float**, **Pulse**, **Spin + pulse**, or **Beacon**;
- choose the requested update interval;
- pause the active animation clock while the target window is hidden;
- edit X/Y position and optional rotation, opacity, and scale expressions;
- view the runtime variables that are refreshed as the protected window changes size.

The bundled Explorer configuration includes an **Explorer animated shield** image rule using the DVD bounce preset. Existing bundled Explorer layouts are upgraded when the Controller starts. The known 1.1.0 starter-rule opacity issue is repaired automatically for that named bundled rule; unrelated custom image rules are left unchanged.

### Included presets

- **DVD bounce** — independent triangle-wave movement across both axes.
- **Horizontal sweep** / **Vertical sweep** — one-axis traversal while preserving the other static placement.
- **Orbit** — an elliptical loop around the proxy-client center.
- **Figure eight** — a Lissajous path that crosses through the center.
- **Wave drift** — a left-to-right sweep with a vertical sine-wave offset.
- **Pendulum** — horizontal sine movement paired with gentle rotation.
- **Float** — small vertical drift, subtle rotation, and gentle scale/opacity variation.
- **Pulse** / **Spin + pulse** — static placement with animated appearance only.
- **Beacon** — repeated opacity/scale flashes without motion.

## Fields

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `enabled` | boolean | `false` | Enables compiled custom animation for this visual item. At least one expression is required. |
| `pauseWhenWindowHidden` | boolean | `true` | Freezes the active animation clock when the protected window is hidden, minimized, cloaked, disabled, or detached. Resuming does not jump through elapsed hidden time. |
| `updateIntervalMilliseconds` | integer | `16` | Requested update interval; clamped to `3..1000`. The native timer also shares work with GIF animation where applicable. |
| `xExpression` | string | empty | Top-left X coordinate in proxy-client pixels. Empty uses the image's normal placement. |
| `yExpression` | string | empty | Top-left Y coordinate in proxy-client pixels. Empty uses the image's normal placement. |
| `rotationExpression` | string | empty | Degrees added to `Details.Display.RotationAngle`. |
| `opacityExpression` | string | empty | Multiplier applied to `Details.Display.Transparency`; the final value is clamped to `0..1`. |
| `scaleExpression` | string | empty | Multiplier applied to the normally resolved image width and height; the final value is clamped to `0.01..10`. |

## Runtime variables

Runtime values always use braces. They are re-evaluated on each animation update, so changing the protected-window size changes the path immediately.

| Variable | Meaning |
|---|---|
| `{t}` | Active animation time in seconds. Paused time is excluded when `pauseWhenWindowHidden` is enabled. |
| `{dt}` | Seconds since the previous active animation update. It is reset after pause/resume and capped internally to avoid large movement jumps. |
| `{windowWidth}` / `{windowHeight}` | Current proxy-client size in pixels. |
| `{imageWidth}` / `{imageHeight}` | Resolved visual-item width and height after normal display sizing, before `scaleExpression`. The legacy variable names are used for all item kinds. |
| `{availableWidth}` / `{availableHeight}` | Remaining travel distance after image dimensions and non-negative display margins. |
| `{left}` / `{top}` / `{right}` / `{bottom}` | The corresponding `Details.Display.Margins` values. |
| `{margin}` | The smallest non-negative display margin. Useful when all sides use a shared margin. |
| `{centerX}` / `{centerY}` | Center of the proxy-client area. |

## Expression language

Expressions are compiled once when the configuration is loaded and evaluated many times. Invalid expressions reject the configuration with an error that identifies the text, image, or QR-code item and property. The Controller also validates them before saving.

Supported syntax:

```text
Numbers:     1, -10, 0.25, 1e-3
Operators:   +  -  *  /  %  ^
Grouping:    ( )
Variables:   {t}, {windowWidth}, {imageHeight}, ...
Constants:   pi, e
Functions:   sin, cos, tan, abs, min, max, clamp, floor, ceil, round, frac, tri, lerp
```

Function argument counts are fixed: `min(a,b)`, `max(a,b)`, `clamp(value,min,max)`, and `lerp(from,to,amount)`. All other functions take one argument. `tri(value)` returns a triangle wave in the inclusive range `0..1`; it is useful for bounce motion.

Examples:

```text
# Horizontal wave around the center
{centerX} + sin({t} * 1.4) * 220 - {imageWidth} / 2

# Vertical orbit component
{centerY} + cos({t} * 1.1) * 130 - {imageHeight} / 2

# Gentle rotation and pulse
sin({t} * 2.0) * 12
0.70 + sin({t} * 1.7) * 0.20
1.0 + sin({t} * 2.2) * 0.08
```

Use no expression, or leave individual expression fields blank, to preserve that property’s normal static behavior. Custom animation does not create duplicate items or particle effects.
