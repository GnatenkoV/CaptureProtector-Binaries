# Controller UI

CaptureProtector uses a custom WPF shell with a sidebar and a single active-page presenter. General, Processes, Protection rules, Options, and About are independent views with their own view models and commands.

The controller supports dark, light, and Windows-system themes. The application logo is used in the sidebar, executable icon, and notification-area icon.

## General

The General page lists registered windows and process agents. A row provides:

- process name and PID;
- current target window title;
- resolved visual configuration file;
- native-agent connection and protection state;
- the latest native error, when one is available.

The list refreshes while the controller window is visible. Use the search field to filter by process name or window title. The clear action removes the filter.

Available actions apply to the currently shown online rows only:

- **Protect shown** enables protection.
- **Unprotect shown** disables protection.
- **Reload config** asks the native session to reload its per-process configuration.

Double-clicking an online row toggles protection for that row. Hiding the controller does not stop monitoring, injection, or native sessions.

## Processes

The Processes page shows only live processes that expose a responding CaptureProtector **process-control agent**. This is the authoritative signal that the CaptureProtector Native DLL is currently loaded in that process.

Each row provides:

- process name and PID;
- count of the process's non-helper desktop windows;
- count of live, enabled protected-window sessions;
- executable path, when Windows allows the controller to read it.

The list refreshes every two seconds while the page is visible. It excludes CaptureProtector proxy and toggle helper windows from the window count. Use the search field to filter by process name, PID, or executable path.

## Protection rules

The Protection rules page owns the global configuration draft. It provides:

- process/rule tree editing;
- searchable process selection with Process, PID, Windows, and Window title/path headers, plus drag-to-window picking;
- compact process rows: the window summary and executable path each stay on one trimmed line, with the complete value available as a tooltip;
- per-rule inline help;
- initialization, linking, and editing of per-process visual configs;
- save/reload operations that refresh the active process set.

The page-level save operation is required to persist global rule changes. The rendering editor saves its own per-process JSON only when its **Save** action is used.

For rule matching behavior and the editor workflow, see [ProtectionRules.md](ProtectionRules.md).

## Visual configuration editor

The visual configuration editor edits both shared per-window settings and text, image, and QR code rules.

The **Window and performance** section edits the root configuration objects:

- `ProtectionWindow.Margin` and `ProtectionWindow.MatchCorners`;
- `ToggleButton.Size`, `ToggleButton.LeftInset`, and `ToggleButton.BottomInset`;
- `Performance.MaximumAnimationFps`, `AnimationTimerMilliseconds`, `DynamicContentTimerMilliseconds`, and `PlacementFallbackTimerMilliseconds`.

The editor preserves the modern toggle-placement schema when a config already uses `Margins`, alignment, location, or offset fields. In that case the left and bottom inset controls update `ToggleButton.Margins.Left` and `ToggleButton.Margins.Bottom` rather than converting the config back to the legacy shorthand.

The immutable **General** rule owns the base source at z-index `0`. Every Text, Image, QR code, and Shader rule is an ordinary positive scene layer; drag-and-drop order controls its z-index. When a new visual rule is created, the editor selects it and scrolls the scene-layer list to it. Each visual rule stores placement, alignment, margins, offsets, opacity, and content-specific style. Image rules offer two explicit source actions: **Select image** writes an external `Path`, while **Select data** embeds the selected file as Base64 `Data` in the configuration. Selecting one source clears the other.

Image rules also expose a **Custom animation** panel. It contains the DVD-bounce preset, playback controls, X/Y expressions, optional appearance expressions, and a runtime-variable reference. The panel remains limited to one image instance and numeric expressions; it does not create particles or duplicate images.

The **Preview target** list shows a readable window display name rather than the underlying catalog-object representation. When **Preview on target** is enabled, the controller uses a connected native agent for the selected process window:

- it writes the draft to a temporary sibling JSON file;
- the native renderer loads the same JSON model it uses at runtime;
- changes are sent after a short debounce;
- the normal native toggle is hidden while preview is active;
- the temporary file is removed when preview ends.

The preview is intentionally transient. It does not save the main JSON configuration and it is unavailable for a target that does not yet have a connected native agent.

## Options

Options are saved automatically to:

```text
%LOCALAPPDATA%\CaptureProtector\controller-settings.json
```

Default values:

```json
{
  "language": "en",
  "theme": "dark",
  "runAtStartup": false,
  "uninjectOnExit": true,
  "nativeLoggingEnabled": false,
  "hideInTrayOnClose": true
}
```

The controller requires administrator elevation through its application manifest. `runAtStartup` creates a current-user Task Scheduler task named `CaptureProtector Controller`. The task starts at sign-in with an interactive user token, uses `HighestAvailable`, launches the controller with `--minimized`, and uses the controller installation folder as its working directory.

Creating or removing the startup task can require UAC consent. When task creation fails or consent is cancelled, the option is rolled back.

`uninjectOnExit` applies when the controller really exits. Hiding the main window does not unload native DLLs.

`hideInTrayOnClose` changes the title-bar close action into a hide-to-tray action. Use the tray menu's **Exit** command to stop the controller.

CaptureProtector uses a named per-session mutex. Launching it again activates the existing controller window instead of starting a second monitoring, tray, and injection-management instance. Installer maintenance commands bypass this interactive-instance guard so uninstall and upgrade cleanup can still run.

The Language selector always includes built-in English and discovers additional JSON files in `%LOCALAPPDATA%\CaptureProtector\languages`. Selecting another language reloads its file immediately. The refresh button beside the selector rescans that folder and reloads the active JSON content when the language still exists. Missing strings use English. See [Localization.md](Localization.md) for the file format.

## About

The About page shows the product identity, version, current installation and data locations, and a concise description of the current process-rule, scene-layer, Effects Studio, preview, theme, and localization capabilities. It also includes an explicit reminder that CaptureProtector is for applications and data the user is authorized to manage, and that Windows capture behavior depends on the exact capture workflow.

## Tray menu

The notification-area menu provides:

- Open CaptureProtector;
- General;
- Protection rules;
- Options;
- About;
- Open log folder;
- Exit.

Double-clicking the tray icon opens the controller.

## Logging

Each controller launch creates a session folder beneath:

```text
%LOCALAPPDATA%\CaptureProtector\logs\yyyyMMdd-HHmmss-pid\
```

The folder contains `controller.log` and, when native logging is enabled, one `<process>-<pid>.log` file for each injected process.

The controller log records startup, options changes, process discovery, injection/unload requests, agent connections, preview errors, per-window commands, and unexpected failures. Native logs cover startup, selected configuration, window attach/detach, rule changes, preview commands, and shutdown safety decisions.

The **Write native process logs** option controls only native DLL logging. Controller logging remains enabled. Changing the option sends the new logging state to connected agents immediately and applies it to future injections. Diagnostic text remains English regardless of the UI language.

## Row context menu

Right-click a process row for direct actions.

- Protected rows show **Unprotect**, **Open config**, and **Reload config**.
- Unprotected rows show **Protect**, **Open config**, and **Reload config**.
- **Open config** opens the resolved JSON file in the default Windows editor.
- **Reload config** asks the native session to re-read the same path.

## Shutdown progress window

On an actual controller exit, CaptureProtector displays a compact animated modal while it stops monitoring and, when enabled by **Uninject native library on controller exit**, asks active agents to unload. The modal prevents repeated user interaction while cleanup is in progress and closes automatically when the shutdown sequence completes or reaches its existing timeout.

## Input and control consistency

- Application settings use the same animated switch control as rule and overlay settings.
- Standard single-line text boxes and selection combo boxes use the same 38 px control height.
- Numeric and color values commit when the editor loses focus or when Enter is pressed; they do not update while every character is typed.
- Color fields display a live swatch after the committed RGB or hex value is valid.
