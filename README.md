# CaptureProtector

> Configurable visual privacy surfaces for selected Windows application windows.

CaptureProtector is a Windows desktop application for applying configurable capture-protection behavior and privacy-focused visual presentation to application windows you own or are authorized to manage.

It combines a WPF controller with a native rendering component to manage selected processes, protect matching windows, and present configurable text, image, GIF, and QR-code content where appropriate.

## Highlights

- **Per-process protection rules**
  - Choose which application processes are managed.
  - Match windows by title, class, visibility, ownership, and other rule conditions.
  - Keep separate configuration for different applications.

- **Flexible visual rules**
  - Add text, images, animated GIFs, and QR codes.
  - Configure every visual item independently as a **Background** or **Overlay**.
  - Control alignment, spacing, margins, size, opacity, rotation, and stretching.
  - Use dynamic values such as `{USER}`, `{MACHINE}`, `{PNAME}`, `{PID}`, `{DATE}`, and `{TIME24}`.

- **Custom animation for text, images, and QR codes**
  - Animate position, rotation, opacity, and scale with a small numeric-expression engine.
  - Includes ready-to-use presets such as DVD bounce, orbit, figure eight, wave drift, pendulum, pulse, float, and beacon.
  - Uses a constrained expression format rather than a general scripting engine.
  - Animation pauses while a protected window is unavailable, hidden, minimized, or otherwise inactive.

- **Live configuration preview**
  - Preview visual changes using the same native renderer used during normal operation.
  - Tune a selected application configuration without maintaining a separate WPF-only preview implementation.

- **Taskbar-aware presentation**
  - Supports optional protected taskbar thumbnail and live-preview presentation for managed windows.

- **Desktop-friendly controller**
  - Tray-capable WPF interface.
  - Dark/light theme support and localization.
  - Per-user configuration, visual assets, and logs stored under `%LOCALAPPDATA%\CaptureProtector`.

## Design boundaries

CaptureProtector is designed for applications that you explicitly choose to manage. It does not modify or inject into conferencing, chat, or screen-capture applications, and it does not create a virtual camera or virtual display.

Windows capture behavior depends on the capture API, Windows version, application rendering technology, security context, and the capture tool being used. CaptureProtector is not a guarantee against every possible screen-capture method. Use it only for applications, systems, and information that you are authorized to manage.

## Architecture overview

```text
CaptureProtector.Controller
        │
        ├── manages rules, configuration, preview, logs, themes, and localization
        ├── discovers selected application processes and matching windows
        └── communicates with native agents through named pipes

CaptureProtector.Native
        │
        ├── manages per-window protection sessions
        ├── renders configurable visual surfaces
        ├── evaluates dynamic content and custom-animation expressions
        └── maintains optional taskbar preview presentation
```

## Documentation

- [Configuration reference](docs/ConfigReference.md)
- [Configuration examples](docs/ConfigurationExamples.md)
- [Protection rules editor](docs/ProtectionRules.md)
- [Controller UI](docs/ControllerUI.md)
- [Custom animation](docs/CustomAnimation.md)
- [Localization](docs/Localization.md)

## License

See [LICENSE.md](LICENSE.md).

Third-party acknowledgements are listed in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
