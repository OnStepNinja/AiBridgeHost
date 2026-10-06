# LUNA AiBridge Host

> ## ⚠️ PROTOTYPE v0.9.0 — UNDER DEVELOPMENT
>
> This is an early **prototype**, released to gather feedback. **It is still under development and may not work correctly.** It can behave unexpectedly, including in ways that **move real hardware** (your telescope mount). **Stay with the equipment, keep the STOP button within reach, and be ready to cut power.** Unattended operation is not supported. Provided "AS IS", without warranty of any kind. You use it entirely at your own risk.
>
> **Feedback is very welcome** — bug reports, rough edges, mounts and setups that do or don't work. Please open an [Issue](https://github.com/OnStepNinja/AiBridgeHost/issues).

AiBridge Host runs on your Windows PC and gives your astronomy web apps a single, common place to run and a single, common way to talk to a telescope mount. It is the PC counterpart of the [AiBridge ESP32 firmware](https://github.com/OnStepNinja/AiBridge): apps written for the shared AiBridge API, such as the star chart, run on both.

On top of that, it can let an AI assistant (claude.ai or Grok) **write and test apps for you** — "make me a panel that shows the mount position and a big STOP button" — while you keep control of anything that moves the telescope.

## Download

Get the zip from the **[Releases](https://github.com/OnStepNinja/AiBridgeHost/releases)** page (`AiBridgeHost-prototype-v0.9.0.zip`, about 7 MB). No installer: unzip it anywhere and run. This repository holds the documentation only; the software is freeware and its source code is not published.

## What it does

- **Runs web apps.** An app is a single HTML file (150 KB or less) in the `apps` folder. Open it at `http://localhost:8080/<name>.html`. Apps see the mount through one simple API, whatever the mount is.
- **Connects to your mount** in any of these ways (choose in the Host screen, or on the command line):
  - Serial (USB / RS-232C / Bluetooth virtual COM) — LX200 protocol (OnStep, NS-5000, …)
  - TCP — LX200 protocol (for example an OnStep Wi-Fi bridge)
  - **ASCOM Alpaca** telescope
  - **INDI** telescope
  - **N.I.N.A.** (through the Advanced API plugin) — the mount N.I.N.A. is connected to
  - A built-in **simulated mount**, for trying things without any hardware
- **Reaches other devices, too.** Besides the mount, apps can talk to **ASCOM Alpaca** and **INDI** devices on your network through the Host: move and stop a **focuser**, change a **filter wheel** position, and read or write any other Alpaca property or raw INDI property (any device, any driver). You choose the devices in the Host screen (*Devices for apps*).
- **Includes ready-made apps:**
  - `host_starchart` — the AiBridge star chart (English): GOTO, sync, stop, tracking, park
  - `host_alpaca_devices` / `host_indi_devices` — find and check Alpaca / INDI devices on your network
  - `sample_monitor` (position monitor) and `sample_panel` (control panel) — small, readable examples to copy and modify
  - `starchart_ja` — the star chart with a Japanese interface
- **Lets an AI build apps for you** (optional, experimental — see below).
- **Guards the door.** Web pages from other sites cannot send commands to your mount through your browser. An optional 4-digit PIN protects access from other devices when you enable LAN access.
- The Host screen is available in **English, 日本語 and 简体中文** (the Chinese translation was machine-made and has not yet been reviewed by a native reader).

## Requirements

- **Windows 10 / 11** (this is the only platform tested)
- A mount reachable by one of the methods above — or just use the simulated mount
- For AI features: a claude.ai or Grok account that can add a custom connector (check your plan), and Microsoft Edge or Google Chrome installed (the AI uses it, hidden, to test the apps it writes)

## Quick start (no AI)

1. Unzip. Keep the exe files together in one folder.
2. Double-click **`AiBridgeHost.exe`**. Your browser opens at `http://localhost:8080/`.
3. Open the connection settings (the Host screen, Mount section) and choose your mount — or **Simulated mount** for a first try.
4. Open an app from the list, for example the star chart.

To stop it, use the **Quit AiBridge Host** button on the Host screen. Closing the browser tab does not stop the program.

Command-line options, if you prefer them (`.\AiBridgeHost.exe …` in PowerShell):

| Option | Meaning |
|---|---|
| `-serial COM6 -baud 9600` | Connect over a serial port |
| `-tcp host:port` | Connect over TCP (LX200) |
| `-alpaca host:port -device 0` | Connect to an Alpaca telescope |
| `-indi host:port` | Connect to an INDI server |
| `-nina host:port` | Use the mount connected in N.I.N.A. (default API port 1888) |
| `-mock` | Simulated mount |
| `-port N` | Web server port (default 8080) |
| `-lan` | Allow other devices on your LAN (default: this PC only — use only on a network you trust, and set a PIN) |
| `-dir folder` | Apps folder (default: `apps` next to the exe) |
| `-open` | Open the browser at start |

## Quick start (with an AI assistant) — experimental

`AiBridgeHostConnect.exe` lets claude.ai or Grok connect to your Host.

1. Double-click **`AiBridgeHostConnect.exe`**. It starts the Host (if it is not already running), makes a PIN, and opens its start screen at `http://localhost:3730/`.
2. **First time only:** the start screen offers a button to download **cloudflared** (about 55 MB, from Cloudflare's official GitHub releases). It creates a temporary public address (a Cloudflare Quick Tunnel) so that claude.ai can reach your PC. Press it once.
3. Copy the **URL** and the **PIN** from the start screen. In claude.ai: *Settings → Connectors → Add custom connector*, paste the URL, connect, and enter the PIN. (Grok: add it as a custom connector / MCP server in the same way.)
4. Ask the AI to build something. Good first requests: *"Copy sample_panel and make the buttons bigger"*, or *"Make a page that shows the mount's position in large type."* Open the result at `http://localhost:8080/<name>.html`.

The URL changes every time Connect starts; remove the old connector and add the new one. The start screen's "Disconnect everyone & get a new PIN" cuts all connections at once. To quit, press **Quit** on the start screen (this also stops a Host that Connect started).

### What the AI can and cannot do

- **Apps:** it can list, read, search, write, copy, restore and delete **your own** apps, read their error logs, and **test-run** an app (`app_probe`): the Host opens the app in a hidden browser against a **built-in simulated mount only**, and returns screenshots (phone / tablet / desktop sizes), errors, and layout problems so the AI can fix them. The built-in apps (`host_*`, `sample_*`, `starchart_ja`) cannot be modified; the AI copies them (`copy_app`) and edits the copy.
- **Equipment (off until you allow it):** before the AI may touch equipment at all, the Connect start screen shows a warning that you must accept (once, until the Host stops). Then it can read mount, Alpaca and INDI status, and send commands. **Every GOTO, every sync, and any command that changes mount settings needs your approval on the Connect start screen, each time** (it expires after 60 seconds and works only once). Approval can only be given on that screen — not by the AI, and not through the tunnel. **Other actions do not ask again once you have accepted the warning:** park / unpark, tracking on / off, manual moves, and moving a focuser or a filter wheel. The AI can always send STOP.
- **Real equipment — please read:** the AI's test runs use the simulated mount only. **Nothing the AI builds has been tested on your real mount unless you test it.** First try every app on the simulated mount; then, on real equipment, stay next to the telescope with STOP within reach and try one small movement at a time.

## Safety notes

- The Host is a local web server that can command your mount. By default it accepts connections from **this PC only**. Enable LAN access only on a network you trust, and set the optional PIN (Host screen → *Use from phones and other devices*).
- Pages from other web sites are refused by the Host. Pages **you** open from the Host (apps) can, however, call it freely: **apps run outside the approval system, and an app can send GOTO and other commands, can reach the Host's admin functions, and — when you use it normally — can send data to the internet.** Read an app (or ask the AI to explain it) before using one you did not write.
- Starting the connection to a mount can reset some mount controller boards.
- Using the AI features means your conversation with the AI provider and a temporary public tunnel are involved. Don't use them on a network or account where that is not acceptable.

## Known limitations

- Verified only on the author's setup: an NS-5000 mount over serial / LX200, plus the Alpaca, INDI and N.I.N.A. **simulators**. Other mounts, other ASCOM/INDI drivers and real INDI hardware are untested; please report what you find.
- The AI features (writing and testing apps through claude.ai / Grok, approvals, screenshots) have been checked in pieces, but the whole flow has **not yet been re-verified end to end with the current build**; Grok's handling of screenshots is unconfirmed. Usage limits on free plans are unknown.
- The **mount connection** through Alpaca, INDI or N.I.N.A. offers the basics (position, GOTO, sync, stop, tracking, park); mount-specific settings stay in the mount's own driver.
- **Not supported yet:** camera images (there is no path for image data; exposures and downloads do not work through the Host), autoguiding, and — when the mount is connected through N.I.N.A. — N.I.N.A.'s own camera, focuser, guider and sequences. Focuser and filter wheel control through Alpaca / INDI has been tried with simulators only.
- Windows only. The exe is **not code-signed**, so Windows SmartScreen may show an "unknown publisher" warning (*More info → Run anyway*).
- The mount's reaction to the first command after a fresh start can differ between drivers (some simulators ignore a GOTO until they are un-parked).

## Files

| File | Purpose |
|---|---|
| `AiBridgeHost.exe` | The Host: web server, apps, mount connection |
| `AiBridgeHostConnect.exe` | AI connection: sign-in (OAuth + PIN), tunnel, approval screen. Starts the Host for you |
| `apps\` | Your apps folder (created on first start; the sample apps are placed here) |
| `aibridgehost.json` | Your settings (created automatically) |
| `aibridgehost-oauth-store.json`, `AiBridgeHostConnect.log` | Created by Connect: PIN, sign-in records, log. **Do not share these.** |

## License

**Freeware** — free to download and use, no registration or payment. Copyright (c) 2026 Nishioka Sadahiko. All rights reserved. The source code is not published, and redistribution or modification of the software requires permission. See [LICENSE](LICENSE). Third-party components: [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md).

Not affiliated with or endorsed by N.I.N.A., ASCOM, INDI, Anthropic or xAI. Related: [AiNinaBridge2](https://github.com/OnStepNinja/AiNinaBridge2) (AI control of N.I.N.A.).
