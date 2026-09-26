# change-manager

A single-file browser tool for writing IT change requests and reusing the ones
you write often. Ten fields (type, duration, downtime, reason, steps, test plan,
risks, security risks, backup plan, expected results), a placeholder system, and
a template library you can save, export and share.

No backend, no build step, no dependencies. Open `index.html` and go.

![screenshot](docs/screenshot.png)

## What it does

- **Write a request.** Ten fields, each with a copy button, plus copy-all for
  pasting the lot into whatever change management system you are stuck with.
- **Placeholders.** Anything written as `<Device Name>` is picked up
  automatically. Fill in a value once and replace it across every field.
- **Templates.** Save the current request under a name, load it back, filter by
  name or contents, export one or all of them as JSON, export the library as CSV.
- **Import.** Drop a JSON file in, pick one, or pull a shared library from an
  https URL. `central-templates.json` in this repo is an example library with six
  ready-made templates (application install, user offboarding, privileged account
  creation, basic user creation, firewall policy change, VM disk expansion).
- **Print.** The request prints as a readout rather than as ten half-clipped
  textareas, with the tool name and version on the page.

Everything is kept in this browser's local storage: templates, the request in
progress and the appearance you picked. Save JSON is the backup.

## Appearance

Three independent controls in the top bar, saved between visits: design
(cobalt, chamfer, console, contour), mode (dark, dusk, sepia, light) and accent
(eight presets, a colour wheel, or a hex value). Sixteen palettes plus a free
accent, and the accent is re-lit against the live surface so it never drops below
a readable contrast.

## Running it

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8080
```

It runs the same from `file://`. `_headers` (Netlify, Cloudflare Pages) and
`.htaccess` (Apache) are included for static hosting.

## Template libraries

A library is a JSON file holding a list of objects:

```json
[
  {
    "name": "Standard-Application Install",
    "data": {
      "typeOfChange": "Standard",
      "duration": "60",
      "downtime": "90",
      "reason": "- Install <Application Name> on <Device Name>",
      "steps": "...",
      "testPlan": "...",
      "risks": "...",
      "securityRisks": "...",
      "backupPlan": "...",
      "expectedResults": "..."
    }
  }
]
```

Host a copy somewhere https-reachable and the whole team can import the same set.
Importing a name that already exists overwrites it, so re-importing updates in
place rather than duplicating.

The URL field is blank by default and the tool makes no request unless you paste
one in and press the button.

## Security notes

- One HTML file. No frameworks, no vendored libraries, no CDN fallbacks, nothing
  to keep patched.
- Content Security Policy in the document and in `_headers` / `.htaccess`:
  `default-src 'none'` with `connect-src https:` so the URL import works and
  nothing else can phone home. Plain http imports are refused before the request
  is made.
- `connect-src https:` stays broad on purpose and is not narrowed to a list of
  origins. The only thing that fetches is **Import library from URL**, where the
  URL is whatever you paste in - the whole point is that it can be any host, so
  there is no fixed set of origins to name. The fetch is sent with
  `credentials: "omit"`, `cache: "no-store"` and `referrerPolicy: "no-referrer"`,
  the response is parsed as JSON and never executed, and every imported template
  is validated field by field before it is stored.
- `frame-ancestors` is set in `_headers` and `.htaccess` only. Browsers ignore
  it in a meta tag and log a warning, so it is deliberately absent from the
  document's CSP.
- Imports are checked field by field before anything is stored: a template needs
  a name and a data object, every key has to be one of the ten known fields, and
  every value has to be a string.
- Nothing is ever written to the page as markup. Every row, label and value is
  built as a node with `textContent`, which is why the old DOMPurify dependency
  is gone rather than replaced.
- No analytics, no telemetry, no third-party requests.

## History

Version 2 is a rewrite. Version 1 was React, ReactDOM, Babel Standalone and
DOMPurify vendored into `js/` (about 3.8 MB) compiling JSX in the browser on
every page load; this is one 90 KB file with no runtime dependencies, built from
the [single-file-tool](https://github.com/shanemc92) template that the rest of
these tools share.

MIT. See `LICENSE`.
