# Protection rules editor

The **Protection rules** page edits the current user's automatic-registration configuration:

```text
%LOCALAPPDATA%\CaptureProtector\CaptureProtector.Auto.json
```

The page works with an in-memory draft. Dialog changes stay local until **Save changes** validates and writes the complete JSON document atomically.

## Rule model

Every executable key maps to an ordered array of process rules.

```json
{
  "processes": {
    "example.exe": [
      {
        "name": "example-default",
        "configPath": "configs/example.exe.json",
        "initiallyEnabled": true,
        "protect": true,
        "unloadWhenNoWindows": true,
        "includeToolWindows": false,
        "protectOwnedWindows": false,
        "unloadOnControllerExit": true,
        "ignoreUnloadRequests": false,
        "requiredInjectionVisibleWindow": false,
        "emptyWindowGraceMilliseconds": 3000,
        "maxProtectedWindows": 8,
        "windowDiscoveryTimerMilliseconds": 250,
        "injectionDelayMilliseconds": 0,
        "injectionReadinessTimeoutMilliseconds": 10000
      }
    ]
  }
}
```

Rules are evaluated in array order. Put specific command-line filters before a general fallback rule for the same executable. The editor preserves ordinary characters in executable and rule names, including `+` in names such as `notepad++.exe`.


## Controller self-protection

CaptureProtector loads `CaptureProtector.Native.dll` directly into `CaptureProtector.Controller.exe` after the user configuration is available. It does not remotely inject its own process. The built-in `CaptureProtector.Controller.exe` rule uses a dedicated visual configuration and keeps the native manager alive while no Controller window is visible.

That rule also sets `ignoreUnloadRequests: true`. Both process-level and per-window native agents acknowledge unload commands as ignored, and the Controller unload service excludes this process even during forced cleanup. The DLL is naturally released by Windows only when the Controller process exits. Removing the self-protection rule is treated as an intentional opt-out: the direct native load remains harmless, but the native manager does not activate protection without an explicit Controller rule.

## Process and rule tree

Each configured executable is displayed as an expandable process node. Its child rule nodes can be added, edited, or removed independently.

The rule editor exposes:

- initial protection state and `protect` enablement;
- command-line required and excluded flags;
- required and excluded runtime titles and classes;
- injection-readiness title/class/visibility requirements;
- tool-window and owned-window handling;
- discovery, limit, grace-period, and unload behavior;
- the per-process rendering configuration path.

Every editable option has inline help. String lists support detected desktop titles/classes as well as manual entries for values that are not currently visible.

## Add protected process

**Protect process** opens a searchable process picker. It can filter by executable name, PID, executable path, or visible window title.

The draggable shield selector can be dropped on a target window. The selected process becomes the draft process entry, and the observed window class and title can be used to seed the rule. The drag picker does not activate protection immediately; save the rule draft first.

## Injection readiness and runtime selectors

The editor keeps two independent groups of window rules because they are evaluated at different times.

| Rule group | Evaluated by | Purpose |
|---|---|---|
| `requiredInjectionVisibleWindow`, `requiredInjectionWindowTitles`, `requiredInjectionWindowClasses` | Controller before injection | Delays injection until a qualifying application window is available. |
| `requiredWindowTitles`, `excludedWindowTitles`, `requiredWindowClasses`, `excludedWindowClasses` | Native manager after injection | Determines which individual HWNDs receive and retain protection. |

Use both groups when a caption must gate startup and must continue to gate active protection:

```json
{
  "requiredInjectionVisibleWindow": true,
  "requiredInjectionWindowTitles": ["*Confidential document*"],
  "requiredWindowTitles": ["*Confidential document*"]
}
```

The native manager receives Windows name-change notifications and performs a rescan. A managed window that no longer matches its runtime title/class selector is detached immediately. `keepHiddenWindowsAlive` still keeps a hidden tray-style window alive only while it continues to match its selected runtime rule.

## Owned Word file dialogs

The Windows file-open and save dialogs launched from Word are owned top-level windows. CaptureProtector normally ignores owned windows because many applications use them for transient helper UI. The bundled Word rule therefore enables `protectOwnedWindows`, and the bundled Explorer rule also enables it while remaining restricted to `CabinetWClass`, `ExploreWClass`, and `#32770`.

This covers both common hosting paths: Office-hosted dialogs and Explorer-hosted dialogs. Keep the Explorer class list narrow; enabling owned-window protection for all Explorer windows would also include unrelated shell UI.

## Per-process visual configuration

Each rule can create or link a rendering configuration beneath:

```text
%LOCALAPPDATA%\CaptureProtector\configs\<process>.json
```

The visual editor supports text, image, and QR code rules. Every visual rule has an independent placement mode:

```text
Background
Overlay
```

The editor exposes alignment, absolute positioning, negative and positive margins, offsets, spacing units (`px`, `dip`, and `ch`), rotation, opacity, installed fonts, colors, GIF animation, image stretching, and QR styling.

See [ConfigReference.md](ConfigReference.md) for the complete JSON model.

## Native real-time preview

The **Preview on target** switch uses the selected process's native protection session rather than a controller-owned WPF overlay.

1. Choose a readable entry from **Preview target**. The list contains only visible windows with a live per-window native agent session, so CaptureProtector helper proxy and toggle windows are excluded.
2. The controller saves the current unsaved visual draft into a temporary JSON file next to the regular configuration file.
3. It sends `BEGIN_PREVIEW` to the connected native process agent.
4. The native session parses that temporary config and renders it with the same `OverlayRenderer` used during normal protection.
5. Editor changes are debounced, written to the temporary JSON, and sent with `UPDATE_PREVIEW`.
6. Turning preview off, saving, or closing the editor sends `END_PREVIEW` and removes the temporary file.

Preview does not write the normal per-process configuration, does not alter the saved runtime protection state, and does not inject a process solely for preview. The selected target must already have a connected native agent.

During preview, the existing native proxy receives a one-time non-activating Z-order promotion without becoming an owned popup of the selected target. Position-only tracking preserves that Z order, and returning the selected target to the foreground requests one further controlled promotion. The ordinary native toggle is hidden until preview ends, so it cannot compete for the preview position.

## Saving and refreshing rules

**Save changes** validates the draft and writes `CaptureProtector.Auto.json`. The controller tells each already injected native agent to reload the rule file in place, then refreshes process discovery for future processes. Unrelated protected processes keep their loaded DLL and active sessions.

A process excluded by the saved configuration detaches its local protected windows. Its native helper can remain loaded briefly when its safety barrier or no-window grace period prevents immediate unloading; that state is recorded in the controller and native logs.
