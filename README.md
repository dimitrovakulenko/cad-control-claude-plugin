# Control for AutoCAD

Control connects Claude to a **live AutoCAD session** on your own machine. Instead of
describing a drawing, Claude opens it, queries the drawing database, creates and edits
entities, manages layers and layouts, runs AutoCAD commands and AutoLISP, plots to PDF,
and captures the drawing as an image.

AutoCAD runs on Windows and macOS only, so this plugin is useful on those platforms.
Supported releases: AutoCAD 2025, 2026 and 2027.

## What you get

- **Structured drawing tools** — `entities`, `layers`, `blocks`, `layouts`, `viewports`,
  `selection`, `styles`, `annotations`, `references`, `settings`, `system_variables`.
  Each mutating call is one undo step in AutoCAD.
- **Command and script access** — the full AutoCAD command catalog, interactive
  command sessions that can answer prompts, `.scr` scripts, and AutoLISP evaluated in
  memory without AutoCAD loading a file from disk.
- **Output** — `plot` writes PDFs, `capture_drawing` returns a PNG of the drawing area.
- **Two skills** that teach Claude how to set Control up and how to drive a drawing
  safely, plus `/cad-control:doctor` to check a session end to end.

## Requirements

1. AutoCAD 2025, 2026 or 2027 installed locally (Windows x64 or Apple Silicon macOS).
2. A Control account. Control requires sign-in; see [Sign in](#sign-in).
3. Node.js 18 or newer, which is what installs and runs the server.

## Install

Enable the plugin, then restart Claude Code so it starts the MCP server. The first
launch does the rest; `/cad-control:doctor` walks you through it and verifies each step.

## What this plugin runs, installs and sends

Control is a local bridge, and everything below happens on your own machine except
where it says otherwise.

- **Installs the MCP server from a lockfile.** The plugin's own `package.json`
  declares `@dmytro-prototypes/cad-control-mcp-server` at one exact version, and
  `package-lock.json` records its registry source and integrity hash. Claude Code
  installs it into the plugin directory when you install or update the plugin, with
  lifecycle scripts disabled, and then runs the installed file directly:
  `node ${CLAUDE_PLUGIN_ROOT}/node_modules/@dmytro-prototypes/cad-control-mcp-server/bin/cli.js`.
  npm fetches that package and the one platform binary it needs
  (`...-darwin-arm64` or `...-win32-x64`) from the public npm registry; nothing is
  downloaded at any later launch.
- **Copies the server binary to a stable path**, so npm cache changes do not break a
  running client: `~/Library/Application Support/Control/bin` on macOS,
  `%LOCALAPPDATA%\Programs\Control` on Windows.
- **Installs an AutoCAD plugin.** Control talks to AutoCAD through an ObjectARX module
  that must be loaded inside AutoCAD. The launcher installs it into AutoCAD's shared
  auto-load folder — `%APPDATA%\Autodesk\ApplicationPlugins` on Windows,
  `~/Library/Application Support/Autodesk/ApplicationAddins` on macOS — and reinstalls
  it whenever the shipped version differs from what is there. **Restart AutoCAD after
  installing or updating.** Set `CONTROL_SKIP_PLUGIN_INSTALL=1` to manage that module
  yourself.
- **Writes MCP client configuration, only when you ask it to.** Running `CONTROLREGISTER`
  inside AutoCAD, or ticking clients in the plugin's setup window, writes a Control
  entry into the MCP config of the clients you selected, backing up any file it changes.
  Nothing is written unless you select it.
- **Talks to AutoCAD over a local ZeroMQ socket.** No drawing content is sent anywhere
  for a tool call.
- **Sign-in and licensing go to the vendor.** Sign-in happens in your browser; the
  resulting key is stored in the OS credential store (macOS Keychain, Windows
  Credential Manager) and exchanged for a short-lived signed lease plus an allowance of
  calls. Individual tool calls are metered locally, with no network round trip each.
- **Issue reporting is opt-in per report, and uploads off your machine.** `report_issue`
  POSTs the report and the attachments you selected over HTTPS to the vendor's feedback
  endpoint, which forwards them to the maintainer by email. You review the report and
  approve each attachment before anything leaves. This is the only tool that sends
  anything but licence traffic off the machine. To switch uploads off entirely, set the
  server's `CONTROL_FEEDBACK_URL` environment variable to an empty string.

## Sign in

Control requires an account. From AutoCAD run `CONTROLLOGIN`, or from a terminal run
`npx -y @dmytro-prototypes/cad-control-mcp-server@0.1.24 login`. Either opens your
browser, you approve the request once, and that machine is connected. `CONTROLSTATUS`
shows what you have, `CONTROLACCOUNT` opens your account page, `CONTROLLOGOUT` forgets
the key. No key is ever displayed or pasted, so your MCP client configuration holds no
secret.

Diagnostics — `commands`, `list_instances`, `console` and `cancel` — are never metered,
so a stuck session can always be inspected and cleared. The first two need no account
at all, so a failed sign-in can still be diagnosed.

## Links

- Product and documentation: <https://cad-control.dmytro-prototypes.net>
- [Privacy](https://cad-control.dmytro-prototypes.net/privacy) — everything the
  licensing service stores, why, and for how long
- [Terms of Service](https://cad-control.dmytro-prototypes.net/terms) — the account,
  the portal and billing
- Licence terms for the software itself: `LICENSE.txt` in this folder, also published
  at <https://cad-control.dmytro-prototypes.net/license>
