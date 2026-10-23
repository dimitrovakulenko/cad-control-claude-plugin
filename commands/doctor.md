---
description: Check the AutoCAD connection end to end and report what is wrong
---

Check the Control bridge to AutoCAD and report the result in a few lines. Follow the
`cad-control:autocad-setup` skill for what each failure means.

1. Call `list_instances` to find a running AutoCAD and the loaded Control plugin.
2. Call `account` to check sign-in and remaining quota.
3. If an instance was found, call `documents` to list the open drawings.

Then say, in one short block: whether AutoCAD is reachable, whether the user is signed
in, which drawings are open, and the single next action if anything is missing. Do not
launch AutoCAD, and do not try to sign in — both are the user's to do.
