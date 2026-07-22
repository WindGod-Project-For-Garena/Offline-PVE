---
name: fc-lint-fix
description: Trigger after any .fcg or .fcc edit, or when the user asks to compile, validate, lint, check, or fix FC compiler errors.
---

# FC Lint Fix

Use this skill after every `.fcg` / `.fcc` edit. Also use it when the user asks to compile, validate, lint, check, or fix FC errors. This is the highest-priority FC skill.

## Completion Rule

Pure FC text edits are not complete until full `fccompile.exe -i Assets` validation has run. `-m` is only a fast metadata-only precheck and must not be treated as final delivery validation. If the task changed editor-owned assets, UI, scene entities, or script attachments through Craftland Studio MCP, final validation must use `game-build` + `game-console-get-logs`. If validation cannot run, the final response must state which required file, path, or MCP tool is missing.

## Locate Project Config

Read `Temp/UGCLanguage/fcconfig.json` under the project root when available. Use these fields:

- `CompilerExecutable`
- `CorePath`
- `EditorGenLibPath`
- `MinimumExtensionVersion`

The editor symbol table is normally located at:

```powershell
<ProjectRoot>\Temp\UGCLanguage\editorGen\EditorGenSymbol.json
```

## Locate Compiler

Use the first existing compiler path in this order:

1. `Temp/UGCLanguage/fcconfig.json.CompilerExecutable`
2. `<CorePath>\fccompile_external.exe`
3. `<CorePath>\fccompile.exe`
4. `%USERPROFILE%\.vscode\extensions\craftlandstudio.ffugclanguage-*\library\release\fccompile.exe`
5. `%USERPROFILE%\.cursor\extensions\craftlandstudio.ffugclanguage-*\library\release\fccompile.exe`

If multiple plugin versions match, use the latest directory unless project config provides an explicit compiler path.

## Locate Standard And Generated Libraries

Prefer `CorePath` from `fcconfig.json`. If missing, read editor setting `ffugclanguage.editor.CorePath` when available, then fall back to plugin `library/release/`.

The compiler's `-e` option expects the editorGen directory, not the `EditorGenLib.fcc` file. By default it uses:

```powershell
<ProjectRoot>\Temp\UGCLanguage\editorGen
```

## Run Validation

Run from the project root. Match the VS Code extension's compiler invocation shape: pass source roots through `-i` and rely on compiler defaults for editorGen and symbol paths.

Final local compiler validation:

```powershell
<Compiler> -i Assets
```

Optional fast metadata-only precheck:

```powershell
<Compiler> -i Assets -m
```

`-m` means compile to metadata only. In the compiler, it still runs preprocessing and precompiler validation, but skips the final `Compiler()` phase. It can catch many syntax, import, type, and symbol issues quickly, but it is not a full compile and must not be the final validation for delivered `.fcg` / `.fcc` edits.

If the project has multiple source roots, join them with `;`, matching `fcconfig.json` / extension source roots:

```powershell
<Compiler> -i "Assets;Other/AssetsFolder"
```

Do not pass `-e` or `-s` unless you must override compiler defaults. If overriding:

- `-e` must be the editorGen directory, for example `Temp\UGCLanguage\editorGen`.
- `-s` is joined with the current working directory by the compiler, so use a project-relative path such as `Temp\UGCLanguage\editorGen\EditorGenSymbol.json`.

`fccompile.exe -i Assets` matches the VS Code extension full compiler invocation. Use it as the default final validation for pure `.fcg` / `.fcc` edits.

Use Craftland Studio MCP `game-build` + `game-console-get-logs` as the final validation when the task touches UI assets, scene entities, script attachments, `button.clickHandlers`, or other editor-owned data. Without an editor process, local full compile (`<Compiler> -i Assets`) is only a fallback and the final response must state that full editor validation was not run.

## Fix Loop

1. Run validation.
2. Parse diagnostics such as `./Assets/Script.fcg:10:5` or an equivalent `path:line:col` location.
3. Open the failing file and inspect nearby code.
4. Apply the smallest confirmed fix.
5. Run validation again.
6. Repeat up to 3 rounds.

If errors remain after 3 rounds, stop and report:

- Remaining diagnostics.
- Fixes already attempted.
- Which source or user action is needed next.

## Common Fix Checks

- Missing import: confirm the owning `.fcc` through `fc-symbol-lookup` before adding it.
- Unknown FC symbol: confirm it through `fc-symbol-lookup` before using it.
- Unknown assetId or entityId: confirm it through `fc-asset-check` before using it.
- Graph field declaration: use `isJumping bool`.
- Current attached entity: use `thisEntity<Transform>.Position`.
- Component access: use `entity<Component>.Field`.
- Vector construction: use `Vector3{x, y, z}`.
- Vector component access: use `pos.X`, `pos.Y`, `pos.Z`.
- HUD button callback: use `func OnClick(button entity<UIWidgetButton>, player entity<Player>)`.
- Custom UI asset id: cast strings with `"asset-id" as CustomUIAssetID`.
- Namespaced call: use the import alias confirmed by the source `.fcc`.
- Platform mismatch: verify `[platform_client]` or `[platform_server]`.
- Client/server event mismatch: put the platform decorator on the `graph`, not on the event listener inside the graph body.

## Final Response Requirements

Must include:

- Whether full compiler validation ran.
- The compiler path used.
- Whether full editor build validation ran when MCP assets/entities/UI were changed.
- Whether validation passed.
- If validation did not run, which config, path, or MCP tool is missing.
- If validation failed, a short summary of remaining errors.
