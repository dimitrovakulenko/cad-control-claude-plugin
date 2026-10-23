---
name: autocad-drawing
description: Work in a live AutoCAD drawing through Control — picking the right tool, creating and editing entities, layers, blocks, layouts and annotations, running AutoCAD commands and AutoLISP, plotting to PDF, and capturing a view. Use whenever the task touches an AutoCAD drawing, a DWG open in AutoCAD, or an AutoCAD command, and for any mention of AutoLISP or plotting from AutoCAD.
---

# Work in a live AutoCAD drawing

Control acts on the drawing that is **MDI-active** in AutoCAD, and every mutating call
is one undo step, so the user can always back out a single operation.

## Pick the structured tool first

Reach for raw commands last. The structured tools return typed data and fail with a
clear message instead of parking AutoCAD at a prompt:

- `list_instances` → the running AutoCAD. Omit `instanceId` when only one is live.
- `documents` → open, activate, close. Most tools need a `documentId` from it.
- `entities` → read, query, create, edit, transform, copy, mirror, array. This is the
  main tool; prefer it over typing geometry commands.
- `layers`, `blocks`, `layouts`, `viewports`, `selection`, `styles`, `annotations`,
  `references` (DWG xrefs), `settings`, `system_variables`.
- `view` → zoom and read what is in view. `capture_drawing` → a PNG of the drawing area,
  never of AutoCAD's window chrome.
- `plot` → PDF output.
- `commands` → the AutoCAD command catalog. It needs no running AutoCAD, so it is the
  cheap way to check that a command exists and what it is called.

Query before you write. Read the entities, layer or style you are about to change, so
the edit is based on what is in the drawing rather than on an assumption about it.

## Closing a drawing

When closing through `documents`, **always** pass `unsavedChanges`: `save`, or `discard`
only when the user has authorised losing that work. Never leave AutoCAD sitting at a
save prompt — it blocks every later call, and no tool can dismiss that dialog.

## Raw commands, when you need them

- `send_command` runs one command line. It cannot cancel itself; use `cancel` for that.
- `command_session` is the right tool for an interactive command with branches: start
  the command, read the prompt it returns, then submit one response with the exact
  `sessionId` and `promptId` it gave you. Do not guess a prompt's wording.
- `run_script` runs an existing `.scr`. If it returns `awaitingInput`, drive it with
  `script_control` (`action=input`, then `action=status`), or abort with `cancel`.
- `console` shows the AutoCAD command line, which is how you find out that something is
  parked at a prompt.

## AutoLISP

`lisp` takes inline source, or the path of an existing `.lsp`. File source is read by
the server and evaluated in memory, so AutoCAD never loads a file from disk.

AutoLISP is also the robust way to pass tokens that contain spaces — plotter device and
paper names are the usual case. Write one `(command "._-PLOT" "_Yes" …)` expression in
which every quoted string is one atomic token and `""` means "accept the prompt's
default".

AutoCAD for Mac has core AutoLISP but **not** ActiveX (`vla-*`) or VBA, and it cannot
return a value from LISP to the caller. On macOS, read state back with `settings`,
`system_variables`, `entities` or `view` instead of expecting a LISP return value.

## Reporting a problem

`report_issue` emails a sanitised bug report to the maintainer and works without AutoCAD
or sign-in. Use it only when the user asks for it: show them the report text, get an
explicit yes, and get a separate yes for each attachment before sending.
