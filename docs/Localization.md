# Controller localization

CaptureProtector always contains an **English** translation inside the controller executable. English is the default language and is also the fallback for every missing translation key.

Optional UI languages are loaded from the user-editable application-data location:

```text
%LOCALAPPDATA%\CaptureProtector\languages\
  languages.json
  uk.json
  pl.json
```

The installer packages all default language resources inside the Controller executable. On the first run, CaptureProtector creates missing language files in this folder without overwriting user edits. Existing installations are migrated from the former `%ProgramFiles%\CaptureProtector\languages` folder and from the previous `%LOCALAPPDATA%\CaptureProtector` root location when files are not already present in the user language folder. A language file can be added, replaced, or removed without rebuilding the controller; use the **Refresh languages** button in Options after changing language JSON files.

On startup, CaptureProtector synchronizes missing bundled keys into the Ukrainian and Polish files without replacing existing user values. The synchronized files are written as indented UTF-8 JSON with normal readable characters, rather than `\uXXXX` escape sequences. Existing language files that contain escaped Unicode are normalized on the next startup. At runtime, however, the selected language file is the only non-English translation source: a missing or empty key falls back directly to built-in English. Restart the controller or use **Refresh languages** after changing a language file.

The selected code is stored in:

```text
%LOCALAPPDATA%\CaptureProtector\controller-settings.json
```

If the saved language file is no longer available or cannot be parsed, CaptureProtector automatically uses English and saves `"language": "en"` during startup.

## Language file format

Each JSON file represents one language and must contain a stable code, a display name, a .NET/Windows culture name, and a string dictionary.

```json
{
  "code": "pl",
  "name": "Polski",
  "culture": "pl-PL",
  "strings": {
    "Navigation.General": "Ogólne",
    "Options.Language": "Język",
    "Options.RefreshLanguages": "Odśwież języki"
  }
}
```

Rules:

- `code` must be unique among files in the folder. English (`en`) is reserved for the built-in translation and cannot be overridden by a JSON file.
- `name` is shown directly in the language selector. Use the language's own name, for example `Українська` or `Polski`.
- `culture` controls formatting for values such as dates and numbers. If it is invalid, CaptureProtector uses `en-US`.
- `strings` may be incomplete. A missing or empty key resolves to the built-in English string.
- JSON files are written as readable UTF-8. Ukrainian, Polish, and other non-ASCII characters are kept as normal text rather than `\uXXXX` escape sequences.
- JSON comments and trailing commas are accepted, but an escaped-Unicode normalization or missing-key synchronization rewrites that file as standard indented JSON; comments and trailing commas are not preserved by this maintenance operation. Invalid JSON files are ignored and recorded in the controller log.
- Keep keys stable. Do not translate JSON configuration properties, pipe commands, enum identifiers, wildcard patterns, or native diagnostic values.

## Reload behavior

- Selecting a different language reloads that language file from disk immediately.
- The **Refresh languages** button rescans `%LOCALAPPDATA%\CaptureProtector\languages`, adds new files, removes missing files from the selector, and reloads the active language if it still exists.
- If the active language disappears during refresh, the controller changes to English.
- A file that changes while selected is applied after refresh without restarting the controller.

## Diagnostics

Changing the UI language affects visible controller text and standard formatting only. Controller logs, native DLL logs, exception messages, named-pipe error values, and technical diagnostics remain English so that support output is consistent across installations.
