# CaptureProtector

CaptureProtector is a Windows desktop application for managing configurable privacy-oriented presentation and capture-protection behavior for selected application windows.

It lets you create per-process rules, apply configurable background or overlay content, and manage protected windows from one controller. Visual configuration supports text, images, animated GIFs, QR codes, dynamic values, custom animation, and an optional Direct3D Effects Studio.

## What it is for

Use CaptureProtector with Windows applications and information that you own or are authorized to manage. Typical workflows include preparing a selected application window for controlled demonstrations, presentations, screenshots, or supported capture workflows without changing the capture application itself.

CaptureProtector does not inject into conferencing, chat, or screen-capture applications. It does not install a virtual camera or virtual display.

## Key capabilities

- Per-process protection rules with title, class, visibility, ownership, and command-line matching.
- Independent protected-window controls and a small on-window protection toggle.
- Configurable Text, Image, GIF, QR code, Background, and Overlay scene layers.
- Live preview in the configuration editor using the same native renderer used at runtime.
- Optional Direct3D Effects Studio with Rectangle, Circle, and Triangle geometry.
- Dark, light, and system themes; English, Ukrainian, and Polish UI language packs.
- User-editable configuration, assets, languages, and logs under `%LOCALAPPDATA%\CaptureProtector`.

## Requirements

- 64-bit Windows 10 or Windows 11.
- Administrator approval may be required when the controller manages an elevated target application.

## Install and start

1. Download the current CaptureProtector installer or release archive from this repository.
2. Install or extract the release package.
3. Start **CaptureProtector Controller**.
4. Open **Protection rules**, choose **Protect process**, and select the application process you want to manage.
5. Open the linked visual configuration, adjust scene layers, and save the rule.
6. Use **General** to confirm and control the protected windows.

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
- [Effects Studio](docs/EffectsStudio.md)

## Support files

The application stores its user-editable data in:

```text
%LOCALAPPDATA%\CaptureProtector\
```

This includes rules, per-process configurations, saved assets, language packs, settings, and session logs.

## License

See [LICENSE.md](LICENSE.md). Third-party notices are in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
