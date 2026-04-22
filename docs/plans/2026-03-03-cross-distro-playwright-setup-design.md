# Cross-Distro Playwright Setup

**Issue:** [#13 — Can't install packages when utilising non-Ubuntu-based linux distros](https://github.com/fridzema/oxide-dock/issues/13)
**Date:** 2026-03-03

## Problem

`make setup` runs `bunx playwright install --with-deps chromium firefox webkit` (Makefile:140). The `--with-deps` flag tells Playwright to install OS-level browser dependencies via `apt-get`, which only exists on Debian/Ubuntu. This fails on Fedora, Arch, openSUSE, and other non-apt distros with:

```
sh: line 1: apt-get: command not found
```

## Solution

Replace the single Playwright install line with inline shell logic in the Makefile `setup` target that:

1. Detects the OS via `uname -s`
2. On macOS: installs browsers without `--with-deps` (browsers are self-contained)
3. On Linux: detects the available package manager and installs with `--with-deps` only when `apt-get` is available; otherwise installs browsers only and prints a warning about manually installing system deps
4. Falls back gracefully for unknown environments

### Package manager detection

| Package manager | Distro family       | Behavior                                   |
|-----------------|---------------------|--------------------------------------------|
| `apt-get`       | Debian/Ubuntu       | `--with-deps` (current behavior, preserved)|
| `dnf`           | Fedora/RHEL         | browsers only + warning                    |
| `pacman`        | Arch                | browsers only + warning                    |
| `zypper`        | openSUSE            | browsers only + warning                    |
| Unknown         | Other               | browsers only + warning                    |

Playwright's `--with-deps` only supports apt-based distros. For non-apt distros, users must install system dependencies manually. The warning message will point users to Playwright's docs for their distro.

## Scope

- **Makefile** `setup` target: replace line 140 with distro-aware shell block
- **CI unchanged**: CI always runs on Ubuntu, `--with-deps` is correct there
- **No new files**
- **Backwards-compatible**: Debian/Ubuntu users get identical behavior
