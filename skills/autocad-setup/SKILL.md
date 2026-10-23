---
name: autocad-setup
description: Get Control talking to a live AutoCAD session, and fix it when it is not. Use when the AutoCAD tools report "no instance found", "not signed in", "quota exhausted" or "revoked", when a tool call hangs or returns "busy" or "not_quiescent", when the user has just installed this plugin, or when they ask how to connect Claude to AutoCAD.
---

# Set up and repair the AutoCAD connection

Control has three moving parts. Almost every failure is one of them being absent:

1. **The MCP server** — installed into the plugin's own `node_modules` when the plugin
   is installed or updated, and run from there, so it is running whenever the AutoCAD
   tools are listed at all.
2. **The ObjectARX plugin inside AutoCAD** — installed by the server's launcher, but
   loaded only when AutoCAD **starts**. A fresh install needs an AutoCAD restart.
3. **A signed-in account** — one browser approval per machine, shared by every MCP
   client on it.

## First run

Work through this in order and report what you find; do not ask the user to do a step
you can verify yourself.

1. Call `list_instances`. If it names a running AutoCAD, the bridge is up — go to step 4.
2. If it reports no instance, ask the user whether AutoCAD is running. Do not launch
   AutoCAD yourself unless they ask: on a licensed seat a launch is not free.
3. If AutoCAD is running but no instance is found, the ObjectARX plugin is not loaded.
   It is installed on every server launch, so the usual cause is that AutoCAD has not
   been restarted since the install. Ask the user to restart AutoCAD, then retry
   `list_instances`.
4. Call `account` to check sign-in. On `not signed in`, tell the user to run
   `CONTROLLOGIN` at the AutoCAD command line, or
   `npx -y @dmytro-prototypes/cad-control-mcp-server@0.1.24 login` in a terminal. Both
   open a browser for a one-click approval. You cannot sign in for them.
5. Call `documents` to list open drawings. Most other tools need a `documentId` from it.

## What each refusal means

| Message | Cause | Fix |
| :- | :- | :- |
| `no instance found` | The ObjectARX plugin is not loaded in any running AutoCAD | Start AutoCAD, or restart it so the plugin loads |
| `not signed in` | No key in the OS credential store | `CONTROLLOGIN` in AutoCAD, or the `login` subcommand |
| `quota exhausted` | The period's metered calls are spent | The message says when it resets; the plan changes at the account portal |
| `revoked` | That machine's key was revoked | Sign in again |
| `feedback_unavailable` | Issue uploads are switched off by `CONTROL_FEEDBACK_URL=""` | Nothing to fix; other tools still work |

Diagnostics (`commands`, `list_instances`, `console`, `cancel`) are never metered, so
they keep working when a quota refusal blocks everything else. `commands` and
`list_instances` go further and need no account at all — they never reach AutoCAD, so
they still answer "is the plugin installed, is AutoCAD running" while sign-in is the
thing that is broken.

## When a call hangs or returns busy

`timeout` means Control stopped waiting, **not** that AutoCAD cancelled the command.
The drawing may well have changed. Verify the drawing's state with `entities`,
`settings` or `view` before you retry anything.

`busy` or `not_quiescent` means AutoCAD is still executing, or is blocked by its own UI.
Do not retry blindly:

1. Read `console` to see whether a command is parked at a prompt — an unfinished
   `SPLINE` or `PLINE` waiting for the next point is the common case.
2. If something is parked, call `cancel`. It sends the equivalent of Escape in AutoCAD's
   own command context; `send_command` cannot cancel itself.
3. `busy: timed-out command recovered; retry sendCommand` is safe to retry once.
4. A modal dialog or palette cannot be dismissed by `cancel` on macOS — ask the user to
   close it. On Windows, `cancel` also tries to close a visible modal popup owned by
   AutoCAD, and never touches unrelated windows.

## Registering other MCP clients

Control can write its own entry into the MCP configuration of other clients on the
machine. The user does that from AutoCAD with `CONTROLREGISTER` — on Windows also via
**Add-ins → Control → Register MCP**, on macOS via the **Control** panel in the 2D
Drafting Tool Sets palette (Cmd-1). It is idempotent, backs up any file it changes, and
writes nothing for clients the user did not tick. The client must be restarted
afterwards. This is the user's action, not yours.

## Updating

The server reinstalls the AutoCAD plugin whenever the shipped module differs from what
is installed, so the two never drift apart. On Windows, AutoCAD keeps `Control.arx`
locked while it is open: the update is skipped with a note on stderr and applied at the
next launch after AutoCAD closes. Either way, **AutoCAD must restart** for a new module
to load.
