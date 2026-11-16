# PhoneDeck — Specification

Turn a spare Android phone, iPhone or tablet into a Stream Deck-style control
panel for a Windows 11 PC.

**Status:** requirements agreed, nothing built yet. The owner wants to give an
explicit go-ahead before any build work starts — confirm with them first.

This file records the decisions made during requirements gathering so that a
new session can start from it without re-asking.

## Reference

Inspired by [GLAUCOMA82/OldPhoneDeck](https://github.com/GLAUCOMA82/OldPhoneDeck)
(MIT), which does the same thing for a Mac: a menu bar app serves a web page
that the phone opens in its browser; the page shows pages of keys and widgets;
pairing is by QR code plus an access code.

That project is macOS-specific (Swift, AppleScript, Dock badges, Focus), so
this is a fresh implementation of the same idea for Windows, not a port. Take
the concept, not the code or the Apple-style visuals.

## Target environment

| | |
|---|---|
| PC | Windows 11, connected by Wi-Fi |
| Phones / tablets | Xiaomi MIX Fold 3 (HyperOS 3), iPhone 17 Pro (iOS 27.2), OPPO Find N2 Flip (ColorOS 16), OPPO Pad Air (ColorOS 13) |
| Network | All devices on the same Wi-Fi network |
| Connection | Wi-Fi only. No USB mode. |
| Devices at once | Usually one, but all four will be tried to see which fits the desk best, so several may be paired |

All devices have modern browsers, so no legacy (ES5 / old iOS) support is
needed.

Screen usage:

- MIX Fold 3: both the cover screen and the inner screen
- Find N2 Flip: main (inner) screen only
- Orientation: follow however the device is placed (portrait and landscape)

The deck layout must therefore adapt live to rotation and to folding or
unfolding, across very different aspect ratios (tall phone, near-square
unfolded foldable, tablet).

## Decisions

| Topic | Decision |
|---|---|
| Phone side | Web page in the phone's browser. No native app for now. |
| Fullscreen | Yes. iPhone via "Add to Home Screen"; Android via a tap-to-fullscreen control. |
| Screen stays on | Yes. See "HTTPS and its consequences" below. |
| Pairing / security | QR code plus access code (below). |
| HTTPS | Not in the core. Add as an opt-in switch once the core works. |
| PC app launch | Started manually by double-click. No auto-start with Windows for now. |
| Layout editing | A config file first; a drag-and-drop editor later. |
| Tech stack | Owner has no preference and is new to programming; recommendation below was accepted. |
| Updates | Must be simple for the owner: download a ready-built `.exe`, no dev tools required to use it. |
| Audience | Personal use first; may be published on GitHub for others if it works well. |
| Visual style | Stream Deck / car-display look: dark, large glowing tiles. Explicitly not Apple-styled. |
| Scope | A small working core first, then extend in rounds. |
| Name | PhoneDeck |

### Pairing: QR code plus access code

The PC app generates a long random access code and shows a QR code containing
the PC's address plus that code. The phone scans it once with its camera, the
deck opens, and the phone remembers the code. The PC rejects any request that
does not carry the code, so other devices on the Wi-Fi cannot trigger keys.
Nothing is typed by hand. Regenerating the code unpairs every device.

### HTTPS and its consequences

The owner does not need HTTPS for security on a home network. Without it,
browsers withhold some features, and these workarounds were accepted:

- **Keep screen awake:** the Wake Lock API is unavailable on plain HTTP. Use
  phone settings instead — Auto-Lock "Never" on the iPhone, "Stay awake while
  charging" (developer options) on Android.
- **Fullscreen on Android:** no installable app-like icon without HTTPS; the
  user taps once to enter fullscreen each time.
- **Voice input:** in-page microphone capture is unavailable; the dictation
  button on the phone's keyboard is the substitute.

When HTTPS is added later it should be opt-in: the PC app creates its own
certificate authority and each phone installs and trusts it once. With HTTPS
on, use the Wake Lock API and a proper web app manifest.

## Recommended tech stack

- **PC side: C# / .NET** (current LTS), a tray application. Chosen for direct
  access to the Windows APIs this project needs.
- **Phone side: plain HTML, CSS and JavaScript**, no build step or framework,
  served by the PC app.
- **Layout: one human-readable JSON config file**, later edited by the
  drag-and-drop editor.
- **Releases: GitHub Actions** builds a self-contained single-file `.exe` on a
  Windows runner whenever the code changes.

Implementation pointers (suggestions, not decisions):

- Built-in web server listening on the LAN; a WebSocket (or similar) to push
  live state such as now-playing and volume to the phone.
- Media info and controls via the Windows media session API
  (`GlobalSystemMediaTransportControlsSessionManager`), which covers Spotify
  with no Spotify login.
- Keyboard shortcuts via `SendInput`.
- Volume and microphone mute via the Core Audio APIs.
- App launching must handle both ordinary `.exe` paths and packaged (Store /
  MSIX) apps, which launch by app ID rather than by file path.
- Windows Firewall will prompt on first run; the Wi-Fi network needs to be
  set to "Private" for phones to reach the PC.
- Show the PC's `.local` name or IP address alongside the QR code as a
  fallback.

## Core (first build)

1. Tray app, QR pairing with access code, and a deck page that adapts to all
   four devices, both orientations, and fold state
2. App launcher keys, pre-configured for: Claude Desktop, Docker Desktop,
   SteelSeries GG, Microsoft Teams
3. Keyboard shortcut keys (ship a couple of editable examples), plus keys that
   open a URL, open a folder, or run a script
4. Media controls and now-playing with artwork — title, artist, artwork,
   play/pause, next, previous. This basic set is enough for Spotify; no
   Spotify Web API integration in the core.
5. Volume and microphone mute
6. Clock widget
7. The Stream Deck / car-display visual style

## Later rounds (wanted, not in the core)

- HTTPS opt-in (first thing after the core is confirmed working)
- Macros (multi-step key sequences)
- More system controls: brightness, dark mode, sleep / lock, screenshot
- Window snapping
- Monitor input switching (DDC/CI; monitor model not yet known)
- More widgets: weather, calendar, CPU / RAM / GPU, network speed, battery,
  notes, focus timer, Windows notifications
- OBS, Discord and other app-specific controls (specific actions not yet
  defined)
- Claude keys: open a chat by voice, open Claude Code in a folder
- Phone as a small second monitor
- Drag-and-drop layout editor
- Deeper Spotify control (like, shuffle, playlists) — needs a Spotify
  developer app and Premium; only if wanted later

## Not wanted

- Phone as webcam or microphone
- USB cable mode
- Native phone apps (for now)
- Auto-start with Windows (for now)

## Development notes

- Build and test on the owner's Windows 11 PC, since nearly all of the core
  is Windows-specific. The PC needs the .NET SDK installed.
- The owner is not yet familiar with programming languages: keep setup steps
  explicit, keep the config file readable, and document how to add or change
  a key.
