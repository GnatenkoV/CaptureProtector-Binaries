# CaptureProtector configuration examples

These examples use paths relative to:

```text
%LOCALAPPDATA%\CaptureProtector\
```

`CaptureProtector.Auto.json` controls process discovery, injection readiness, and runtime window matching. Files under `configs\` control the visual rendering for those selected windows.

## 1. Simple process rule

Protect all eligible windows created by one executable.

```json
{
  "enabled": true,
  "initiallyEnabled": true,
  "defaultConfigPath": "configs/default.json",
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

## 2. Specific command-line profile before a fallback

Rules are ordered. Put the specific process mode first and the general fallback second.

```json
{
  "processes": {
    "browser.exe": [
      {
        "name": "browser-main-ui",
        "requiredFlags": ["--type=browser"],
        "configPath": "configs/browser-main.json",
        "protect": true
      },
      {
        "name": "browser-fallback",
        "excludedFlags": ["--type=*"],
        "configPath": "configs/browser-default.json",
        "protect": true
      }
    ]
  }
}
```

Use command-line filters only where Windows exposes the target process command line reliably. A rule with flag filters does not match when the command line cannot be read.

## 3. Wait for a real application window before injection

This pattern is useful when a process starts helper windows before its main UI.

```json
{
  "processes": {
    "editor.exe": [
      {
        "name": "editor-main-window",
        "configPath": "configs/editor.exe.json",
        "requiredInjectionVisibleWindow": true,
        "requiredInjectionWindowClasses": ["EditorMainWindow"],
        "injectionDelayMilliseconds": 500,
        "injectionReadinessTimeoutMilliseconds": 15000,
        "protect": true
      }
    ]
  }
}
```

## 4. Detach protection when a document title changes

Injection readiness alone does not control an already injected window. Add a runtime selector when the title must continue to control protection.

```json
{
  "processes": {
    "documenthost.exe": [
      {
        "name": "confidential-document-only",
        "configPath": "configs/documenthost.exe.json",
        "requiredInjectionVisibleWindow": true,
        "requiredInjectionWindowTitles": ["*Confidential document*"],
        "requiredWindowTitles": ["*Confidential document*"],
        "excludedWindowTitles": ["*Print preview*"],
        "protect": true,
        "unloadWhenNoWindows": true,
        "emptyWindowGraceMilliseconds": 3000
      }
    ]
  }
}
```

When the caption changes so that it no longer matches `requiredWindowTitles`, or matches `excludedWindowTitles`, the native manager detaches that HWND on its next name-change rescan. The DLL uses its normal no-window unload policy afterward.

## 5. File-manager style class filtering

Use class selectors when a single process hosts both desired application windows and infrastructure windows.

```json
{
  "processes": {
    "explorer.exe": [
      {
        "name": "folder-windows-only",
        "configPath": "configs/explorer.exe.json",
        "requiredWindowClasses": ["CabinetWClass", "ExploreWClass"],
        "excludedWindowClasses": [
          "Shell_TrayWnd",
          "Shell_SecondaryTrayWnd",
          "WorkerW",
          "Progman",
          "NotifyIconOverflowWindow"
        ],
        "maxProtectedWindows": 16,
        "unloadWhenNoWindows": false,
        "protect": true
      }
    ]
  }
}
```

## 6. Visual rule configuration with dynamic text

This is a complete small per-process rendering config.

```json
{
  "ProtectionWindow": {
    "Margin": 0,
    "MatchCorners": true
  },
  "ToggleButton": {
    "Size": 30,
    "HorizontalAlignment": "Left",
    "VerticalAlignment": "Bottom",
    "Margins": {
      "Left": 10,
      "Bottom": 10
    }
  },
  "BackgroundColor": "rgb(22,28,38)",
  "Text": [
    {
      "Mode": "Overlay",
      "Text": "PROTECTED {PNAME}\n{USER} · {MACHINE}\n{DATE} {TIME24}",
      "Details": {
        "Display": {
          "HorizontalAlignment": "Center",
          "VerticalAlignment": "Center",
          "RotationAngle": -25,
          "Transparency": 0.75
        },
        "Style": {
          "FontName": "Segoe UI Semibold",
          "FontSize": 26,
          "FontColor": "rgba(255,255,255,190)",
          "IsBold": true
        }
      }
    }
  ],
  "Image": [],
  "QRCode": []
}
```

## 7. Negative margins and a flexible toggle position

Negative margins let a proxy, visual item, or toggle extend outside the target window bounds. Protection-window margins allow `-256..256`; visual-rule and toggle margins allow `-4096..4096`.

```json
{
  "ProtectionWindow": {
    "MarginLeft": -10,
    "MarginTop": -10,
    "MarginRight": -10,
    "MarginBottom": -10
  },
  "ToggleButton": {
    "Size": 32,
    "HorizontalAlignment": "Right",
    "VerticalAlignment": "Top",
    "Margins": {
      "Right": -12,
      "Top": -12
    },
    "Offset": {
      "X": 4,
      "Y": 0
    }
  }
}
```

## 8. Background image, foreground text, and QR code

Each item selects its own rendering pass.

```json
{
  "BackgroundColor": "rgb(14,20,30)",
  "Image": [
    {
      "Mode": "Background",
      "Path": "../assets/protection-grid.png",
      "Animate": false,
      "Details": {
        "Display": {
          "Stretch": "UniformToFill",
          "Transparency": 0.35
        }
      }
    }
  ],
  "Text": [
    {
      "Mode": "Overlay",
      "Text": "Internal use only",
      "Details": {
        "Display": {
          "HorizontalAlignment": "Center",
          "VerticalAlignment": "Center",
          "Fill": true,
          "Rows": 3,
          "Columns": 2,
          "RowSpacing": "5ch",
          "ColumnSpacing": "8ch",
          "RotationAngle": -22,
          "Transparency": 0.68
        },
        "Style": {
          "FontName": "Segoe UI",
          "FontSize": 28,
          "FontColor": "rgb(255,255,255)"
        }
      }
    }
  ],
  "QRCode": [
    {
      "Name": "session-code",
      "Mode": "Overlay",
      "Text": "https://example.invalid/session/{PID}",
      "Details": {
        "Display": {
          "HorizontalAlignment": "Right",
          "VerticalAlignment": "Bottom",
          "Margins": {
            "Right": 18,
            "Bottom": 18
          },
          "Size": {
            "Width": 120,
            "Height": 120
          }
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
  ]
}
```

## 9. Taskbar thumbnail and Peek placeholder

This provides static protected artwork for Windows-generated taskbar thumbnail and Peek requests. It never copies live application pixels into the placeholder.

```json
{
  "BackgroundColor": "rgb(22,28,38)",
  "TaskbarPreviewProtection": {
    "Enabled": true,
    "DisablePeek": false,
    "ReplaceThumbnail": true,
    "ReplaceLivePreview": true,
    "UseWindowSubclass": true,
    "Text": "PROTECTED\n{PNAME}"
  }
}
```

For detailed field descriptions and accepted aliases, see [ConfigReference.md](ConfigReference.md).
