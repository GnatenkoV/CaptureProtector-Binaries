# Effects Studio

Effects Studio is an experimental Direct3D scene renderer. It uses more GPU resources than the standard Direct2D renderer.

## Enabling it

Open **Options → Application behavior** and enable **Use Direct3D renderer**. The renderer setting is global. It is not stored as a per-process option.

When either switch is off, CaptureProtector uses the original Direct2D renderer. Saved `DeveloperRenderer.Shaders` data remains in the per-process configuration but is ignored by the original renderer.

## Base layer and scene order

The locked **General** rule is always z-index `0` and uses the configured `BackgroundColor`. Target-window mirroring is not supported.

Text, Image, QR code, and Effects Studio items are scene layers at z-index `1+`. Drag them in the Visual rules list to set their order. A shader reads the accumulated scene below it and writes the result back before later layers are drawn.

## HLSL effect contract

An Effects Studio rule provides the body of `float4 CP_Effect(float2 uv)`. CaptureProtector owns the vertex shader, texture bindings, sampler, constants, compilation and presentation. Shader code is compiled as `ps_5_0` only after its source changes; frame animation uses `CP_Time`, not repeated compilation.

Available values:

- `CP_Scene`: the accumulated texture produced by all layers below this shader.
- `CP_Sampler`: sampler for `CP_Scene`.
- `CP_Time`: active effect time in seconds. It pauses with the protected window when the configured animation pause condition applies.
- `CP_DeltaTime`: seconds since the preceding effect frame.
- `CP_Resolution`: proxy width and height in pixels.
- `CP_SceneUvOffset` and `CP_SceneUvScale`: normalized source region for the effect layer.
- `CP_RotationRadians`: configured layer rotation in radians.
- `CP_Opacity`: configured layer opacity from `0.0` to `1.0`.
- `CP_Geometry`: `0` for Rectangle, `1` for Circle, or `2` for Triangle.
- `CP_UserParameters[0..15]`: reserved `float4` values; they currently contain zeroes.
- `CP_SampleScene(float2 uv)`: samples the accumulated scene using local effect UV coordinates.
- `CP_SampleHistory(float2 uv)`: samples the previous output of the same effect layer using local effect UV coordinates.
- `CP_BaseScene(float2 uv)`: samples the unmodified scene beneath the effect draw area. This is useful for effects that need the non-processed source after rotation padding has been applied.

Use local effect UV coordinates from `0.0` to `1.0`. The Controller validates that a source defines `float4 CP_Effect(float2 uv)`; it supplies the values and helpers above.

Example:

```hlsl
float4 CP_Effect(float2 uv)
{
    float2 mirroredUv = float2(1.0 - uv.x, uv.y);
    return CP_SampleScene(mirroredUv);
}
```

Shader code cannot declare engine resources, use external `#include` files, or replace the engine entry point. Compiler errors leave the previous valid shader active.


## Bounds and full-window placement

Each Effects Studio rule owns a `Display` block. Its `Margins`, horizontal/vertical alignment, absolute location, `Size`, `RotationAngle`, and `Transparency` apply to the effect pass. `FillToWindow: true` makes one pass cover the available proxy area while respecting margins. Effects Studio rules also support the same numeric `customAnimation` expressions as Text, Image, and QR rules: X/Y position, rotation, opacity, and scale can update using `{t}`, `{delta}`, and the documented runtime values. Inside a bounded shader use `CP_SampleScene(uv)` and `CP_SampleHistory(uv)`: both helpers remap local UV coordinates to the effect rectangle.

Each effect also has a `Geometry` value. `Rectangle` is the compatibility default and preserves the prior rectangular pass. `Circle` draws the largest true pixel circle that fits within the shader bounds, and `Triangle` draws an upward-facing triangle with its base along the lower bound. The selected shape clips only the shader result, leaving the accumulated scene unchanged outside it. `RotationAngle` and custom rotation expressions rotate the selected shape together with its effect pass.

```json
{
  "Geometry": "Circle"
}
```

### Rotated geometry bounds

Rectangle and Triangle can extend beyond their unrotated layout rectangle when rotated. The renderer expands the internal shader viewport to the exact rotated bounds and adds a small antialiasing guard margin before the shader is evaluated. The original layout rectangle remains the logical `uv` space passed to `CP_Effect`, so existing HLSL sources keep their expected `0..1` coordinate system and the existing `CP_SceneUvOffset` / `CP_SceneUvScale` names remain valid.

The extra viewport is clipped only by the protected window itself. A shape that physically reaches outside that window cannot render beyond its window boundary.

## Built-in examples

- **Burn-in phosphor persistence** uses the previous output of the same effect layer to retain a decaying phosphor ghost.
- **Old monitor (CRT)** combines gentle barrel curvature, scanlines, triad variation, vignette, and flicker in one effect.
- **TV signal distortion** adds analog line jitter, moving interference bands, RGB-channel separation, and noise.
- **TV snow distortion** mixes animated monochrome television static with the scene.
- **Chromatic aberration** offsets red and blue channels away from the scene centre.
- **Magnifying glass** starts with square 280 × 280 bounds so its circular lens and effect layer have the same footprint. It now uses a Qt-inspired radial refraction mapping with a stabilized rim for CaptureProtector's clamped sampler. Adjust `diffractionIndex`, `frameOpacity`, `glassTintOpacity`, and `highlightOpacity` directly in the HLSL source.
- **Directional shadow** adds a soft directional edge shadow. Change only `shadowAngleDegrees` from `0` to `360` in the source: `0` points right, `90` up, `180` left, and `270` down.
- **Billboard** applies a horizontal perspective tilt to the accumulated scene.
- **Toon** combines colour banding with simple edge outlines for a cartoon-style result.
