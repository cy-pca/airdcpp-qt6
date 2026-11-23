# airdcpp-qt6: plan

Status: decisions made (Joe, 2026-11-23: "go with the recommended decisions"). The project is
still not committed: whether to go ahead is decided once Q0 is costed (Q-D4).

## 1. Why, and what exists

- **AirDC++'s desktop client is Windows-only.**
  - `airdcpp-windows`'s README lists Windows 10/11 as its only supported systems, and marks the
    Windows GUI "Bug fixes only" (new maintainers welcome). The core, the Web API and the Web UI
    are under active development.
  - The GUI is ATL/WTL over Win32: `windows/windows/stdafx.h` includes `atlapp.h`, `atlframe.h`,
    `atlctrls.h`, `atldlgs.h`, `atltheme.h` and `shellapi.h`. `windows/CMakeLists.txt` builds a
    `WIN32` executable against WTL, with a Common Controls 6 manifest and an `.rc` UI. None of it
    compiles elsewhere.
- **On Linux and macOS, AirDC++ is the web client:** the core plus the Web API in a daemon
  (`airdcppd`), with the Web UI in a browser.
- **What is packaged on cvcpkg (cy-pca):**
  - `airdcpp-windows` 4.30+cvc.1 (Windows), with `wtl`, `minizip` and `airdcpp-webui`;
  - `airdcpp-webclient` 2.14.0: its recipe is in cy-pca/airdcpp-webclient (Linux; macOS arm64 in
    #2). Its own publish waits on a publisher token;
  - Qt 6: Core, Gui, Widgets, Xml, Network and Concurrent. There is no QML or Quick on linux x86_64.
- **Sizes**, measured 2026-11-23 from the source trees:

  | Part | Files | Size |
  |---|---|---|
  | AirDC++ core (`airdcpp/airdcpp`) | 410 | 2.6 MB |
  | Web API (`airdcpp-webapi/api` + `web-server`) | 170 | 0.9 MB |
  | Windows GUI frames (`windows/windows/frames`) | 70 | 0.8 MB |
  | Windows GUI settings pages (`windows/windows/settings`) | 112 | 0.55 MB |
  | Windows GUI, everything (frames, settings, components, players, util, the main frame and others) | ~250 | ~2.7 MB |
  | For scale: eiskaltdcpp's Qt GUI (`eiskaltdcpp-qt/src`, transfix/eiskaltdcpp master) | 176 | 1.4 MB |

  A complete Qt client is therefore on the order of eiskaltdcpp-qt's size. That is a sizeable
  project, not a port of build settings.
- **The Windows GUI's frames**, which give the parity list (§3): hub, private chat, search, queue,
  file list, favorites, public hubs (hublist), finished, upload queue, users, recents, ADL search,
  autosearch, RSS, extensions, search spy, system log, text viewer, notepad and the CDM debug view.

## 2. Architecture options

### A. Link the core directly, as the Windows GUI does

Qt widgets subscribe to the core's C++ managers and listeners in one process.

- **For:** full access to everything the core can do, no serialization, and the Windows GUI as a
  close model of how each frame drives the core.
- **Against:** the client tracks the core's internal C++ API, which upstream changes freely. Every
  core update is a porting job. The Windows GUI shows what that costs: about 2.7 MB of glue.

### B. A client of the Web API (chosen, Q-D1)

The app talks to an `airdcppd` over the documented Web API: HTTP plus WebSocket JSON, described as
an API Blueprint in [airdcpp-apidocs](https://github.com/airdcpp-web/airdcpp-apidocs).

- By default the app starts and owns a local `airdcppd` (from the `airdcpp-webclient` package),
  bound to loopback with a generated credential, and stops it on exit unless set to keep it
  running. It can also attach to a remote `airdcppd`, so it doubles as a remote manager for a
  seedbox or server.
- **For:**
  - the Web API is the contract upstream keeps stable for its own Web UI, so core changes rarely
    reach the client;
  - one backend package serves the headless, browser and desktop uses;
  - the UI process can't take the transfer engine down with it.
- **Against:**
  - features the Web API does not expose are out of reach, or need an extension or an upstream
    API addition;
  - large lists (search results, queues, file lists) must be paged and streamed through the API's
    views. The Web UI already does this, so it is a solved problem, not a blocker;
  - two processes to package and supervise.

### C. Hybrid: the core and the Web API in-process

The client embeds the core and the Web API library and calls the API layer in-process, without
HTTP. airdcpp-windows already links `airdcpp-webapi` this way. It is less coupled than A, but the
in-process API layer is internal C++ too. It is worth revisiting only if B's local-socket latency
or packaging turns out to matter.

### Reusing eiskaltdcpp-qt

eiskaltdcpp's Qt GUI (GPL-3.0, the same licence) has DC-shaped widgets: the hub frame, search,
transfers, the settings pages and the tab frame. Both clients descend from DC++, but AirDC++'s core
has diverged a long way (async hooks, a new share, bundles), and eiskaltdcpp-qt is wired to its own
`dcpp` core. The proposal is to reuse its layout and interaction patterns, and individual widgets
where they lift cleanly, but not to fork it wholesale.

## 3. Scope and milestones (for option B)

Each milestone ends with a cvcpkg build on Linux, macOS and Windows and an offscreen smoke test
(`QT_QPA_PLATFORM=offscreen`).

- **Q0, skeleton:**
  - CMake, Qt 6 Widgets, and a cvcpkg recipe depending on `airdcpp-webclient` and Qt 6;
  - start, attach to and stop the daemon; sign in to the Web API;
  - a main window with tabs and a status bar, and the system log;
  - CI on GitHub-hosted runners.
- **Q1, hubs and chat:** the hub list and connect, hub frames (chat, users, user commands),
  private chat, the favorite hubs, and the public hub lists (including sublevels' hublist mode).
- **Q2, finding and getting files:** search (with results streamed from API views), the download
  queue and bundles, transfers, finished, and file lists (browse and download).
- **Q3, sharing and settings:** share roots and refresh, the upload queue, and the settings pages
  the Web API exposes, grouped as the Web UI groups them.
- **Q4, desktop polish:** tray and notifications, magnet and `dchub://`/`adc://`/`nmdcs://` URL
  handling, recents, users, and themes. It also checks NMDCS keyprint pinning and `$ForceMove`.
- **Q5, the rest of parity:** ADL search, autosearch, RSS, extensions, search spy, notepad and the
  text viewer.

Not planned: reimplementing the core, and protocol extensions (sublevels' NMDCpb ideas stay out).
Nicotina remains the Ariadne-native client. This is a conventional desktop client, though sublevels
would use it as one more black-box test client.

## 4. Decisions (Joe, 2026-11-23)

All taken as proposed.

| # | Question | Decided |
|---|---|---|
| Q-D1 | Architecture | B, a Web API client that owns a local daemon by default |
| Q-D2 | UI toolkit | Qt 6 Widgets, not QML (cvcpkg has no QML on linux x86_64, and Widgets suit a dense desktop UI) |
| Q-D3 | eiskaltdcpp-qt | reuse patterns and individual widgets, not a wholesale fork |
| Q-D4 | Go or no-go | decide after Q0 is costed: map each Windows GUI frame to Web API endpoints, list the gaps, and estimate Q0-Q2 |
| Q-D5 | Upstream | offer it to the AirDC++ project once Q2 works, since upstream is looking for desktop GUI maintainers |
| Q-D6 | Name | keep `airdcpp-qt6` as the repository name; the app's display name to be agreed with upstream if offered |

## 5. Risks

- Web API gaps for features the Windows GUI has. To measure in Q0: map each Windows frame to API
  endpoints, and list what is missing.
- Daemon supervision across three systems (ports, credentials, single instance, upgrades).
- Size: parity is months of work, so the milestones are ordered so that Q2 is already a useful
  client.
