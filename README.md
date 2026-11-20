# Scratchpad — a Vicuna plugin

A notes panel in Vicuna's side bar: one scratchpad per workspace, saved automatically in Vicuna's plugin storage and themed to match the app (light, dark or a theme plugin).

**Install:** Vicuna → Settings → Plugins → `anslwy/vicuna-plugin-scratchpad` → Install. Then open it from the side panel's `+` menu (or the panel chip in Settings → Plugins).

## Use it as a template

A plugin is a GitHub repo whose latest release carries `vicuna-plugin.json` plus every file it lists (Vicuna falls back to the default branch when there is no release).

```jsonc
{
  "id": "scratchpad",                 // lowercase letters, digits, dashes
  "name": "Scratchpad",
  "version": "1.0.0",
  "description": "…",
  "panels": [
    { "id": "notes", "title": "Scratchpad", "icon": "scratchpad.svg", "entry": "scratchpad.html" }
  ],
  "themes": []                        // optional; see Vicuna's theme plugins
}
```

Panel rules (Vicuna enforces them):

- `entry` is a flat `.html` file name shipped with the plugin — no URLs, no folders. `icon` (optional) is a flat `.svg` or `.png`. Each file ≤ 512 KB, ≤ 2 MB in total, ≤ 8 panels.
- The page must be **self-contained**: inline `<script>` and `<style>` only. It runs in a sandboxed iframe with an opaque origin and a CSP of `default-src 'none'`, so it has no network, no external scripts, no cookies and no `localStorage`, and it can't touch the app.
- Its only channel is `postMessage` to `parent`:

```js
parent.postMessage({ vicuna: 1, id: 1, method: "getTheme" }, "*");
// reply:  { vicuna: 1, id: 1, result: { base: "dark", vars: { "--bg": "…", "--accent": "…", … } } }
```

| method | params | result |
|---|---|---|
| `getTheme` | — | `{ base, vars }` (the app's CSS variables) |
| `getContext` | — | `{ sessionId, title, workspace, mode }` (read-only) |
| `storage.get` | `{ key }` | the stored JSON value or `null` |
| `storage.set` | `{ key, value }` | `true` (value ≤ 64 KB; 1 MB / 256 keys per plugin) |
| `storage.remove` | `{ key }` | `true` |
| `storage.keys` | — | `["key", …]` |

Vicuna also pushes `{ vicuna: 1, event: "theme", data }` when the appearance changes and `{ vicuna: 1, event: "context", data }` when the open session or workspace changes. Storage is scoped to your plugin, kept across upgrades, and deleted on uninstall.

`scratchpad.html` contains a ~20-line client for this protocol you can copy.
