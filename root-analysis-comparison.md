# Device Root Analysis Comparison

Based on everything we collected:

**UJC201_64 (Tablet 2) is more deeply rooted.**

## Why:

**UJC201_64** — adbd itself runs as root (uid=0). Every single ADB command is already root. You don't even need to call `su 0` — you're root the moment you open a shell. `service.adb.root=1` is set, meaning the daemon was explicitly started as root at boot.

**F9212B** — ADB shell lands as uid=2000 (shell). You have to explicitly call `su 0 <command>` to escalate. There's still a step between "connected via ADB" and "root access."

## Summary

Both have the same underlying factory debug profile — test-keys, SELinux permissive, dm-verity disabled, no root manager — but the UJC201_64 skips even the su call requirement entirely.

## Exploitation Risk

In practical terms for exploitation risk: anything that gets ADB access to the UJC201_64 is immediately root with zero additional steps. The F9212B at least requires the su binary to be invoked, which is one more gate (even if it's an unguarded one).
