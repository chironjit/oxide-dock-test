# Cross-Distro Playwright Setup Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Fix `make setup` so it works on non-Ubuntu Linux distros by detecting the OS and package manager before running Playwright install.

**Architecture:** Replace the single `bunx playwright install --with-deps` line in the Makefile with an inline shell block that detects the OS (`uname -s`) and on Linux checks for available package managers. Only uses `--with-deps` when `apt-get` is available; otherwise installs browsers only and warns the user.

**Tech Stack:** Make, shell scripting (`uname`, `command -v`)

---

### Task 1: Replace Playwright install line with cross-distro logic

**Files:**
- Modify: `Makefile:140`

**Step 1: Replace the Playwright install line**

Replace line 140 in the Makefile:

```makefile
	bunx playwright install --with-deps chromium firefox webkit
```

With this shell block (note: each line must be tab-indented and use `; \` continuation since Make runs each line in a separate shell):

```makefile
	@case "$$(uname -s)" in \
		Linux) \
			if command -v apt-get >/dev/null 2>&1; then \
				echo "Detected apt — installing Playwright browsers with system deps..."; \
				bunx playwright install --with-deps chromium firefox webkit; \
			else \
				echo "Non-apt Linux detected — installing Playwright browsers only."; \
				echo "You may need to install system dependencies manually."; \
				echo "See: https://playwright.dev/docs/intro#system-requirements"; \
				bunx playwright install chromium firefox webkit; \
			fi ;; \
		*) \
			bunx playwright install chromium firefox webkit ;; \
	esac
```

**Step 2: Verify the Makefile is syntactically valid**

Run: `make -n setup`

Expected: prints the commands that _would_ run without executing them, no syntax errors. You should see the `case` block in the output.

**Step 3: Test on macOS (current platform)**

Run: `make setup`

Expected: runs the `*)` branch — `bunx playwright install chromium firefox webkit` without `--with-deps`. Playwright browsers install successfully.

**Step 4: Commit**

```bash
git add Makefile
git commit -m "fix: detect distro before installing Playwright deps

On non-apt Linux distros (Fedora, Arch, etc.), `--with-deps` fails
because Playwright uses apt-get internally. Now detects the OS and
only uses --with-deps when apt-get is available. Otherwise installs
browsers only and prints a warning with a docs link.

Fixes #13"
```

---

### Task 2: Update design doc with final implementation notes

**Files:**
- Modify: `docs/plans/2026-03-03-cross-distro-playwright-setup-design.md`

**Step 1: Add implementation note at bottom of design doc**

Append a section noting the simplified approach: rather than detecting dnf/pacman/zypper individually (which Playwright doesn't support anyway), we simply check for `apt-get` and fall back to browser-only install with a warning. This is simpler and forward-compatible.

**Step 2: Commit**

```bash
git add docs/plans/
git commit -m "docs: add cross-distro playwright setup design and plan"
```
