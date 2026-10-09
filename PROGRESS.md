# Local Quick Look investigation

## 2026-10-09 verified evidence

- Installed bundle: `/Applications/Syntax Highlight.app`, bundle id `org.sbarex.SourceCodeSyntaxHighlight`, version 2.1.32 (build 81).
- The installed app's main executable SHA-256 is `e81276b5f2f3414be141823ed055f06f3bd44df1db8028ac8c207cb2580fa671`; the executable in `installed-app-backup/Syntax Highlight.app` has the same hash.
- `pluginkit -mAvvv -p com.apple.quicklook.preview` lists `org.sbarex.SourceCodeSyntaxHighlight.QuickLookExtension` at the installed app path. Its `QLSupportedContentTypes` includes `com.apple.property-list`, `public.xml`, `public.data`, `public.content`, and `public.item` among its declared types.
- `qlmanage -r` and `qlmanage -r cache` completed. `mdls` identifies the plist fixture as `com.apple.property-list`, XML fixture as `public.xml`, and extensionless fixture as `public.data`.
- Invoking `qlmanage -p` on those three fixtures spawned the installed Quick Look extension and its `Syntax Highlight XPC Render` child. The preview process remained open; this confirms generator activation, not visual content or footer state.
- The bundled CLI successfully rendered plist, XML, and extensionless fixtures after restricting `PATH` to `/usr/bin:/bin:/usr/sbin:/sbin`. With `--about off`, none of the three outputs contained `buymeacoffee`, `Developed by Sbarex`, or `Syntax Highlight XPC Render`. With `--about on`, all three outputs contained the footer. The CLI's `--about` flag is a per-render override, not a persistent preference control.
- A 2 MiB text fixture rendered with `--max-data 1048576` produced a 1.1 MiB RTF and omitted its tail marker, confirming that this override trims input at 1 MiB.
- The default shell `PATH` selects Homebrew Coreutils `mktemp`, which rejects the app's macOS `mktemp -t colorize` call. Running the CLI with `PATH=/usr/bin:/bin:/usr/sbin:/sbin` uses the macOS `mktemp` and succeeds; Quick Look's own launch environment may differ.
- `qlmanage -p -o` failed with an uncaught `NSInvalidArgumentException` (`NSDictionaryM setObject:forKey: key cannot be nil`) while preparing the extension request. `qlmanage -t` produced a 1.3 KiB PNG for the plist fixture, but it was visually blank. Neither result proves a readable preview.
- `defaults write org.sbarex.SourceCodeSyntaxHighlight ...` could not write the app's sandboxed preferences domain. CUA denied direct access to Syntax Highlight, and AppleScript keyboard automation was denied by macOS (`ChatGPT is not allowed to send keystrokes`). Persistent app preferences therefore remain unverified and unchanged by this investigation.
- `/Library/Application Support` has no Syntax Highlight entry. `~/Library/Application Support/Syntax Highlight` contains only `Styles`.
- Agy now lists the local `desktop-commander` stdio MCP as enabled; its configuration is outside this repository and must be used from a new Agy session.
- The workspace app backup and source checkout already existed. The source checkout is `master` with `origin` and `upstream`; no source files were changed for the settings investigation.

## Remaining verification

- In Syntax Highlight's own Preferences, set the Advanced "Show about info" control off and set the maximum data size to 1024 KiB. Save, reload Quick Look, and confirm the rendered preview has no footer.
- Visually inspect Quick Look previews for plist, XML, and an extensionless text file, and verify a file above 1 MiB is capped or falls back as expected.
- GitHub Actions run details are recorded in the workspace-level `PROGRESS.md`, but a live recheck was unavailable in this session: `gh auth status` reported invalid stored credentials and the API request failed to connect.
