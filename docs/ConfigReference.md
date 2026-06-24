# CaptureProtector configuration reference

CaptureProtector uses two configuration layers:

| File | Responsibility |
|---|---|
| `CaptureProtector.Auto.json` | Controller process discovery, injection readiness, rule selection, and native window-selection defaults. |
| `configs\*.json` | Native per-process visual rendering, proxy geometry, toggle placement, taskbar placeholders, and performance settings. |

Complete examples are available in [ConfigurationExamples.md](ConfigurationExamples.md).

## User-data location

All mutable configuration, assets, and logs are kept under the current user profile:

```text
%LOCALAPPDATA%\CaptureProtector\
  CaptureProtector.Auto.json
  configs\*.json
  assets\*
  languages\*
  logs\*
```

The controller embeds default configuration and asset files. At startup it creates missing directories and restores only missing default files. Existing user files are not overwritten. Per-process paths must resolve inside this user-data root; paths outside it are rejected by the controller.

## Global automatic-registration configuration

`CaptureProtector.Auto.json` controls which processes the controller tracks and how the native DLL discovers windows after injection.

### Root fields

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `enabled` | boolean | `true` | Enables automatic process tracking and injection. |
| `initiallyEnabled` | boolean | `true` | Default enabled state inherited by process rules. |
| `defaultConfigPath` | string | empty | Fallback per-process rendering configuration. |
| `protectUntitledWindows` | boolean | `false` | Allows eligible windows with an empty title. |
| `includeToolWindows` | boolean | `false` | Allows `WS_EX_TOOLWINDOW` candidates. |
| `protectOwnedWindows` | boolean | `false` | Allows owned top-level windows. |
| `protectInvisibleWindows` | boolean | `false` | Allows initial attachment to hidden windows. |
| `keepHiddenWindowsAlive` | boolean | `true` | Retains an existing hidden tray-style HWND while it still matches its runtime rule. |
| `minimumWindowWidth` / `minimumWindowHeight` | integer | `120` / `80` | Minimum candidate dimensions in physical pixels. |
| `maxProtectedWindows` | integer | `8` | Process-wide maximum for managed application windows. |
| `replaceSmallerWindows` | boolean | `true` | Lets a later main window replace a smaller temporary startup window. |
| `replacementAreaGrowthPercent` | integer | `25` | Required area growth for replacement. |
| `unloadWhenNoWindows` | boolean | `true` | Allows native shutdown after no managed windows remain. |
| `emptyWindowGraceMilliseconds` | integer | `15000` | Delay before automatic native shutdown; `0..600000`. |
| `windowDiscoveryTimerMilliseconds` | integer | `1000` | Native window discovery interval; `100..60000`. |
| `useDirectPlacementHook` | boolean | `true` | Enables the direct placement hook by default. |
| `processes` | object | `{}` | Map from executable name to an ordered array of rules. |

### Process rules

Each executable key uses an ordered array:

```json
{
  "processes": {
    "example.exe": [
      {
        "name": "example-default",
        "configPath": "configs/example.exe.json",
        "protect": true
      }
    ]
  }
}
```

Put rules with specific command-line filters before a generic fallback. Pattern matching is case-insensitive. An asterisk at the beginning and/or end enables prefix, suffix, or contains matching, for example `--type=*`, `*login*`, and `Chrome_WidgetWin_*`.

| Field | Evaluated by | Meaning |
|---|---|---|
| `name` | Controller/native diagnostics | Human-readable rule name. |
| `configPath` | Controller and native DLL | Per-process rendering config. Relative paths are resolved below the user-data root. |
| `initiallyEnabled` | Native session | Initial protection state for the rule. |
| `protect` | Controller | `false` stops the rule from causing injection. |
| `requiredFlags` / `excludedFlags` | Controller before injection | Command-line argument selectors. They require readable process command-line information. |
| `requiredWindowTitles` / `excludedWindowTitles` | Native DLL | Runtime title selectors for individual HWNDs. |
| `requiredWindowClasses` / `excludedWindowClasses` | Native DLL | Runtime class selectors for individual HWNDs. |
| `unloadWhenNoWindows` | Native DLL | Per-rule override for idle DLL unload. |
| `includeToolWindows` | Native DLL | Per-rule override for tool-window candidates. |
| `protectOwnedWindows` | Native DLL | Per-rule override for owned top-level windows. |
| `useDirectPlacementHook` | Native DLL | Per-rule override for the direct placement hook. |
| `emptyWindowGraceMilliseconds` | Native DLL | Per-rule idle grace override. |
| `maxProtectedWindows` | Native DLL | Per-rule window-count override. |
| `windowDiscoveryTimerMilliseconds` | Native DLL | Per-rule discovery interval override. |
| `unloadOnControllerExit` | Controller | Determines whether a normal controller exit asks matching processes to unload. |
| `ignoreUnloadRequests` | Controller/native DLL | Refuses `FREE_LIBRARY` / `UNLOAD_LIBRARY` requests for the process. CaptureProtector.Controller uses it because the DLL is loaded directly into the Controller and stays active until the process exits. |

### Injection readiness fields

The following fields are controller-side startup gates. They are checked before the DLL is injected.

| Field | Meaning |
|---|---|
| `requiredInjectionVisibleWindow` | Requires a visible top-level window that satisfies the readiness rule. |
| `requiredInjectionWindowClasses` | Requires at least one top-level window with a matching class. |
| `requiredInjectionWindowTitles` | Requires at least one top-level window with a matching title. |
| `injectionDelayMilliseconds` | Extra delay after a readiness match, useful for applications that continue building their UI. |
| `injectionReadinessTimeoutMilliseconds` | Maximum time to wait before skipping this process instance. |

`requiredInjectionWindowTitles` and `requiredInjectionWindowClasses` do not remove protection later. For that, duplicate the criterion in `requiredWindowTitles` or `requiredWindowClasses`.

### Runtime title/class changes

The native manager listens for name-change events and re-evaluates its selected runtime title/class rule for every managed HWND. If a window no longer matches a required selector, or starts matching an excluded selector, CaptureProtector detaches the session, proxy, and toggle from that HWND immediately.

`keepHiddenWindowsAlive` does not override a title/class mismatch. It only preserves a hidden tray-style HWND that still matches the selected runtime rule. The native DLL itself then follows `unloadWhenNoWindows` and `emptyWindowGraceMilliseconds` to decide whether it can unload.

## Per-process rendering configuration

A per-process config can contain these root objects:

```json
{
  "ProtectionWindow": {},
  "ToggleButton": {},
  "Performance": {},
  "TaskbarPreviewProtection": {},
  "BackgroundColor": "rgb(22,28,38)",
  "Text": [],
  "Image": [],
  "QRCode": []
}
```

Property names accept the documented PascalCase form and compatible camelCase aliases.

## Runtime tokens

Text and QR values expand tokens at render time.

| Token | Meaning |
|---|---|
| `{USER}` | Current Windows user name. |
| `{MACHINE}` | Computer name. |
| `{PID}` | Host process ID. |
| `{PNAME}` | Host executable name, for example `example.exe`. |
| `{DATE}` | Local date. |
| `{TIME24}` | Local time in 24-hour form. |
| `{TIME12}` | Local time in 12-hour form. |

Dynamic text refreshes only when needed. A QR bitmap is regenerated only when its expanded content or error-correction level changes.

## `ProtectionWindow`

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `Margin` | integer | `0` | Uniform proxy inset or expansion in physical pixels. |
| `MarginLeft` / `MarginTop` / `MarginRight` / `MarginBottom` | integer | inherited from `Margin` | Side-specific override. |
| `Margins.Left` / `Top` / `Right` / `Bottom` | integer | inherited | Alternate object form for side-specific values. |
| `MatchCorners` | boolean | `true` | Matches custom regions and supported DWM corner preferences. |

Protection-window margins are clamped to `-256..256`. Negative values expand the proxy beyond the original window boundary; positive values inset it.

## `ToggleButton`

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `Size` | integer | `30` | Circle diameter; clamped to `18..128`. `Diameter` is accepted as an alias. |
| `AbsoluteLocation` | boolean | `false` | Uses `Location` as an absolute coordinate relative to the target. |
| `Location.X/Y` | number | `0` | Base coordinate when absolute placement is enabled. |
| `Margins.Left/Top/Right/Bottom` | number | left `10`, bottom `10` | Alignment insets; negative values are allowed. |
| `HorizontalAlignment` | string | `Left` | `Left`, `Center`, or `Right`. |
| `VerticalAlignment` | string | `Bottom` | `Top`, `Center`, or `Bottom`. |
| `Offset.X/Y` | number | `0` | Final placement adjustment. |
| `LeftInset` / `BottomInset` | number | legacy | Backward-compatible shorthand for left/bottom margins. |

Toggle margins are clamped to `-4096..4096`. The toggle is green while protection is enabled and red while it is disabled. It is hidden during native real-time preview.

## `Performance`

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `MaximumAnimationFps` | integer | `20` | GIF render cap; `1..60`. |
| `AnimationTimerMilliseconds` | integer | `0` | Replaces GIF frame delays when non-zero; `0` preserves source frame timing. |
| `DynamicContentTimerMilliseconds` | integer | `0` | Overrides automatic dynamic-token refresh when non-zero. |
| `PlacementFallbackTimerMilliseconds` | integer | `2000` | Timer-based placement recovery; `0` disables it. Direct window-position hooks remain the normal low-latency placement path. |

## `TaskbarPreviewProtection`

This feature supplies a static protected bitmap to Windows taskbar thumbnail and Peek/live-preview requests. It does not capture or copy application pixels into the placeholder.

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `Enabled` | boolean | `false` | Enables protected DWM preview handling. |
| `DisablePeek` | boolean | `false` | Disables full-sized Windows Peek instead of supplying a static placeholder. |
| `ReplaceThumbnail` | boolean | `true` | Replaces the taskbar thumbnail. |
| `ReplaceLivePreview` | boolean | `true` | Replaces the full-sized Peek/live preview. |
| `UseWindowSubclass` | boolean | `true` | Handles DWM requests after the application window procedure. |
| `Text` | string | `PROTECTED\n{PNAME}` | Text on the static preview; supports `{PNAME}` and `{PID}`. |
| `ProcessTargetDiscoveryTimerMilliseconds` | integer | `500` | Additional preview-only target discovery interval; `100..60000`. |
| `MaximumProcessTargets` | integer | `16` | Maximum additional targets; `1..128`. |
| `ProcessWindowTargets` | array | `[]` | Window selectors that receive only preview handling, with no overlay/toggle. |

A `ProcessWindowTargets` item supports required/excluded title/class patterns plus visibility, tool-window, ownership, top-level, and minimum-size switches. At least one required class or title pattern should be supplied.

## Colors

Accepted color values:

```json
"rgb(255,0,0)"
"rgba(255,0,0,128)"
"#FF0000"
"#80FF0000"
```

Eight-digit hexadecimal values use `#AARRGGBB` order.

## Common `Details.Display`

Text, image, and QR rules share the same placement model.

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `Fill` | boolean | `false` | Repeats the item as a grid. |
| `Rows` / `Columns` | integer | `1` | Grid dimensions; `1..64`. |
| `RowSpacing` / `ColumnSpacing` | number or string | `0` | Grid spacing. Strings may use `px`, `dip`, or `ch`. |
| `AbsoluteLocation` | boolean | `false` | Uses `Location` as a direct coordinate. |
| `Location.X/Y` | number | `0` | Base position or aligned-position adjustment. |
| `Margins.Left/Top/Right/Bottom` | number | `0` | Available-layout insets; `-4096..4096`. |
| `HorizontalAlignment` | string | `Center` | `Left`, `Center`, or `Right`. |
| `VerticalAlignment` | string | `Center` | `Top`, `Center`, or `Bottom`. |
| `Offset.X/Y` | number | `0` | Final placement adjustment. |
| `RotationAngle` | number | `0` | Degrees around the item center. |
| `Transparency` | number | `1` | Effective opacity from `0` to `1`. |
| `Size.Width/Height` | number | automatic | Preferred rendered size. |
| `Stretch` | string | type-specific | `None`, `Fill`, `Uniform`, or `UniformToFill`. |

When `Fill` is true, location and offset move the complete grid. When it is false, alignment is calculated inside the margin rectangle and then `Location` and `Offset` are added.

## `Text[]`

```json
{
  "Mode": "Overlay",
  "Text": "Printed on {DATE} {TIME24}",
  "Details": {
    "Display": {},
    "Style": {
      "FontName": "Segoe UI",
      "FontSize": 18,
      "FontColor": "rgb(255,0,0)",
      "IsBold": false,
      "IsItalic": false,
      "IsRTL": false
    }
  }
}
```

`Mode` is independently set to `Background` or `Overlay` for every item.

## `Image[]`

An image can use either a path relative to the configuration file or embedded Base64 data. The editor writes one source per image item: **Select image** stores `Path`, while **Select data** stores `Data` in the JSON.

```json
{
  "Mode": "Overlay",
  "Path": "../assets/shield.png",
  "Animate": false,
  "Details": {
    "Display": {
      "Size": { "Width": 96, "Height": 96 },
      "Stretch": "Uniform"
    }
  }
}
```

| Field | Type | Default | Meaning |
|---|---:|---:|---|
| `Path` | string | empty | Relative or absolute image path. Use this for **Select image**. |
| `Data` | string | empty | Base64 image data; a `data:image/...;base64,...` URI is accepted. Use this for **Select data**. |
| `Animate` | boolean | `true` | Plays a multi-frame GIF. `false` uses frame zero. |
| `AnimationSpeed` | number | `1` | Playback multiplier; `0.05..20`. |
| `customAnimation` | object | absent | Optional single-image math-expression animation. See [CustomAnimation.md](CustomAnimation.md). |

WIC decodes PNG, JPEG/JPG, GIF, BMP, TIFF, and other installed WIC codecs. The controller limits a newly selected embedded image to 64 MB. For compatibility, the native renderer still accepts manually authored items containing both fields and gives `Data` priority.

When `customAnimation.enabled` is `true`, the renderer draws one instance of the image. It uses the first normal placement as the fallback for omitted properties; `Fill`, `Rows`, and `Columns` do not duplicate the animated image.

## `QRCode[]`

```json
{
  "Name": "session-code",
  "Mode": "Overlay",
  "Text": "https://example.invalid/{PID}",
  "Details": {
    "Display": {
      "Size": { "Width": 100, "Height": 100 }
    },
    "Style": {
      "DarkColor": "rgb(0,0,0)",
      "LightColor": "rgb(255,255,255)",
      "ECCLevel": "Medium",
      "PixelsPerModule": 8,
      "QuietZoneModules": 4
    }
  }
}
```

`ECCLevel` supports `Low`, `Medium`, `Quartile`, and `High`. `PixelsPerModule` is clamped to `1..128`; `QuietZoneModules` is clamped to `0..16`.

## Rendering order

The renderer uses this stable pass order:

1. `BackgroundColor`
2. background images
3. background text
4. background QR codes
5. overlay images
6. overlay text
7. overlay QR codes

Array order is preserved within each type and pass.

## Native real-time preview

Preview does not add a separate WPF drawing layer. The controller saves a temporary config beside the edited config and sends `BEGIN_PREVIEW`, debounced `UPDATE_PREVIEW`, and `END_PREVIEW` commands to the selected connected native agent. The native session temporarily loads that configuration with the normal renderer, hides its normal toggle, and restores the saved runtime config when preview ends.

## Logs

Each controller launch writes a session folder under `%LOCALAPPDATA%\CaptureProtector\logs`. Native process logs are optional and share that session folder while native logging is enabled. The controller can switch native logging on or off for connected agents without restarting them.
