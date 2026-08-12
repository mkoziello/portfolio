# Adding a project

Everything the desktop shows comes from the JSON in `data/`. To add a game, demo
or experiment you edit one JSON file and drop your HTML somewhere in the repo.
You never touch `index.html`.

## Repo layout

```
index.html          the desktop (this page)
editor.html         the VertexVoodoo 3D editor
cv.html
harmony.html        a project page, sitting at the root
games/
  starfall/
    index.html      or give each project its own folder
data/
  profile.json
  shipped.json
  personal.json     ← new experiments go here
  toolchain.json
  output.json
  trash.json        optional
points/             editor only
```

## 1. Add the entry

Open `data/personal.json` and add an object to `items`:

```json
{
  "id": "starfall",
  "name": "Starfall",
  "tag": "Arcade",
  "stack": "Three.js, Web Audio",
  "color": "#c98a3a",
  "desc": "Twin-stick shooter built in a weekend to test a particle system.",
  "links": [
    { "label": "Play", "url": "games/starfall/", "play": true }
  ]
}
```

| Field | Required | What it does |
|---|---|---|
| `id` | yes | Unique, lowercase, no spaces. Used for the window id — reusing one breaks window management. |
| `name` | yes | Shown on the row, the window title and in search. |
| `tag` / `stack` | no | The mono subtitle under the name. |
| `color` | no | Hex. Becomes the row dot, the titlebar swatch and the search icon. |
| `desc` | no | The Notes paragraph inside the window. |
| `links` | no | Buttons. See below. |

`shipped.json` works the same way, except its items use `studio`, `genre`, `year`
and a `points` array (the bullet list) instead of `tag` / `stack` / `desc`.

## 2. Add the page

Put the HTML anywhere in the repo and point `url` at it. Relative to the repo
root: `harmony.html`, `games/starfall/`, `demos/gaia/index.html`.

## Links

Each entry in `links` is `{ "label", "url", "play", "mode" }`.

- **`label`** — the button text. "Play", "Steam", "Watch", "Source", anything.
- **`play`** — `true` paints the button green with a play triangle. Use it for
  things you can actually run.
- **`mode`** — where it opens:
  - `"window"` opens the page inside a desktop window, in an iframe. It gets its
    own taskbar button and can be dragged, maximised and closed like any window.
  - `"tab"` opens a new browser tab.
  - **Leave it out** and the right one is picked automatically: your own pages
    open in a window, `http(s)://` addresses open in a tab. That default is
    almost always what you want — external sites like Steam and PlayCanvas send
    `X-Frame-Options` headers and refuse to load in an iframe anyway.

Several links on one project is fine. The first one becomes the button on the
row in the list; all of them appear in the project's window, alongside the
"Inspect in the 3D editor" button, which is always there.

```json
"links": [
  { "label": "Play",   "url": "games/starfall/" },
  { "label": "Source", "url": "https://github.com/you/starfall" }
]
```

## Making a page behave inside a window

A page opened in `"window"` mode runs in an iframe sized about 940×660. To make
that comfortable:

- Let it fill its frame — `html,body{margin:0;height:100%}` — rather than
  assuming the full viewport.
- Keyboard input reaches the iframe only once it has focus, so a click-to-start
  screen works better than listening for keys straight away.
- Autoplaying audio is blocked until the user interacts with the page. A start
  button solves this too.
- If it really needs the whole screen, set `"mode": "tab"`.

## The override table

`index.html` has an `EXTRA_LINKS` block near the top holding the Steam, Harmony
and PlayCanvas links, keyed by project name. It exists because those links
weren't in the JSON yet. Anything you put in a `links` array wins over it, so
once you've moved a link into the JSON you can delete its line from the table.

## Checklist

1. `id` is unique across `shipped.json` and `personal.json`.
2. The JSON parses — a trailing comma will fail the whole load and you'll get
   the boot error screen.
3. The `url` path is right relative to the repo root, and the case matches
   (GitHub Pages is case-sensitive; your local machine may not be).
4. Test before pushing:
   ```
   python3 -m http.server 8000
   ```
   then open `http://localhost:8000/`. Opening `index.html` by double-clicking
   won't work — browsers block `fetch()` on `file://` URLs.
