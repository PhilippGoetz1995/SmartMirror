<h1 align="center">
  <br>
  <a href="https://p-goetz.de/"><img src="https://p-goetz.de/wp-content/uploads/2025/04/20250404_P-Goetz_DEV_logo.png" alt="P-Goetz" width="200"></a>
</h1>

<h4 align="center">📦 P-Goetz SmartMirror</h4>

<p align="center">
  <a href="https://p-goetz.de/"><img src="https://img.shields.io/badge/Version-1.0.0-blue"></a>
  <a href="https://p-goetz.de/"><img src="https://img.shields.io/badge/Author-Philipp_Goetz-yellow"></a>
  <a href="https://p-goetz.de/"><img src="https://img.shields.io/badge/uptime-100%25-brightgreen"></a>

</p>

<p align="center">
  <a href="#-Architecture">Architecture</a> •
  <a href="#-how-to-use">How To Use</a> •
  <a href="#-hints-to-not-cry-everytime">Hints</a>
</p>

<!-- Screenshot is optional -->
<!-- ![screenshot](https://raw.githubusercontent.com/amitmerchant1990/electron-markdownify/master/app/img/markdownify.gif) -->

---


The **P-Goetz SmartMirror** project is an interactive, modular smart mirror platform designed to enhance your everyday living space with timely information and useful features accessible at a glance. Whether you're interested in checking your calendar, the latest weather update, news headlines, or your daily schedule, SmartMirror puts essential data front and center, all beautifully displayed on a reflective interface.



---

<!-- GETTING STARTED -->
## 🏗️ Architecture


<p align="center">
  <img src="https://github.com/PhilippGoetz1995/SmartMirror/blob/main/SmartMirror_Architecture_v1.drawio.png" alt="Smart Mirror Architecture" width="700">
</p>



<br>

## 🔧 How To Use

## 🧑‍💻 Environment for development

### 1. Prerequisites

- Credentials => see `.env` file in Cloud Drive
- Raspberry Pi reachable in the local network (`raspberrypi.local`)

### 2. Connect to the Raspberry Pi

1. VPC to Raspberry Pi via RealVNC Viewer
2. IDE (Cursor) to Pi via SSH

### 3. Start / Stop the Mirror

The mirror runs as a **user** systemd service (`magicmirror.service`).
Stop the service before starting the mirror manually, otherwise two instances compete for the same port and display.

```bash
# Production: run via systemd
systemctl --user start magicmirror.service
systemctl --user stop magicmirror.service
systemctl --user restart magicmirror.service
systemctl --user status magicmirror.service --no-pager

# Development: stop the service, then start manually from the MagicMirror folder
systemctl --user stop magicmirror.service
cd ~/MagicMirror
node --run start
# Stop the manual run with Ctrl+C
```

### 4. Logs & Debugging

| What | Where / Command |
|------|-----------------|
| Mirror application log (Node.js / systemd) | `journalctl --user -u magicmirror.service -f` (live) |
| Last 200 lines | `journalctl --user -u magicmirror.service -n 200 --no-pager` |
| Since last boot | `journalctl --user -u magicmirror.service -b --no-pager` |
| Manual run (`node --run start`) | Printed directly in the terminal |
| Module frontend log (`Log.log`, `console.log` in `MMM-*.js`) | Browser console (Chrome DevTools, `F12` / `Ctrl+Shift+I`) |
| Module backend log (`node_helper.js`) | Same as the application log (journal / terminal) |
| Notification stream (`MMM-ViewNotifications`) | `MagicMirror/modules/MMM-ViewNotifications/MMM-ViewNotifications-notifications.jsonl` |
| Motion | `sudo tail -f /var/log/motion/motion.log` or `journalctl -u motion -f` |
| Camera loopback | `journalctl -u camera-loopback.service -f` |

```bash
# Follow the notification debug file live
tail -f ~/MagicMirror/modules/MMM-ViewNotifications/MMM-ViewNotifications-notifications.jsonl
```

> Note: MagicMirror itself does not write a log file; it logs to stdout, which systemd stores in the journal.
> The log verbosity is controlled by `logLevel` in `config/config.js` (e.g. `["INFO", "LOG", "WARN", "ERROR", "DEBUG"]`).
> Ensure the VIF notification module comments are enabled so notification entries are captured and visible in the debug file.

### 5. Useful systemd commands

```bash
# --- System services (e.g. camera-loopback, motion) ---
systemctl list-units --type=service --state=running   # Only running ones
systemctl list-units --type=service                   # All

sudo systemctl start camera-loopback.service
sudo systemctl stop camera-loopback.service

sudo systemctl enable camera-loopback.service         # Start on boot
sudo systemctl disable camera-loopback.service

sudo nano /etc/systemd/system/camera-loopback.service # Edit the config
sudo systemctl daemon-reload                          # After editing a unit file

# --- User services (e.g. magicmirror) ---
systemctl --user list-units --type=service --state=running   # Only running ones
systemctl --user list-unit-files --type=service              # All

systemctl --user enable magicmirror.service           # Start on boot
systemctl --user disable magicmirror.service

nano ~/.config/systemd/user/magicmirror.service       # Edit the config
systemctl --user daemon-reload                        # After editing a unit file
```

## 📄 Documentation | Motion Service

The motion.service is started via systemd but the configuration of the service is in other file

**Motion Commands**
```bash
# Get to the Config
sudo nano /etc/motion/motion.conf

# Check the status of motion
sudo systemctl status motion --no-pager
journalctl -u motion -f
sudo tail -f /var/log/motion/motion.log

```



## 📄 Documentation | Camera Loopback Splitter (Raspberry Pi)

This service captures the Raspberry Pi camera **once** (via `rpicam-vid`) and **duplicates** the raw video stream into **two V4L2 loopback devices**.  
This is useful when you want **two independent consumers** (e.g., Motion detection + MagicMirror Face Detection) without fighting over the physical camera device.

---

**What it does**

- Captures from the Pi Camera using `rpicam-vid` (libcamera stack)
- Streams raw video (`yuv420p`) to stdout
- Uses `tee` to duplicate the stream
- Feeds each stream into `ffmpeg`, writing to:
  - **/dev/video10** (consumer A, e.g. Motion)
  - **/dev/video11** (consumer B, e.g. MagicMirror Face Detection)

So your apps read from `/dev/video10` and `/dev/video11` instead of directly from the Pi camera.

sudo nano /etc/modprobe.d/v4l2loopback.conf => Create the devices

---


## 📄 Documentation MagicMirror Module

**How to install modules**

1. Simple copy repo into /modules folder (git clone <repo-url>)

**Logging Mechanism**

There are two logging mechanism:
1. Node.js Proccess => will be shown in the terminal
2. Frontend Log => will be shown in Chrome Console

```bash

                 (Renderer / UI process)                          (Node.js process)
           ┌───────────────────────────────┐                ┌──────────────────────────────┐
           │  MMM-YourModule.js            │                │  node_helper.js              │
           │  (runs in Chromium/Electron)  │                │  (runs in Node)              │
           │                               │                │                              │
Browser    │  console.log(...)  ───────▶   │                │  console.log(..) ──▶ Terminal│
Console    │  Log.log(...)      ───────▶   │                │                              │
           │                               │                │  sendSocketNotification(...) │
           │  socketNotificationReceived   │◀──── Bridge ───│  (push message to UI)        │
           │   → console.log(...)          │                │                              │
           └───────────────────────────────┘                └──────────────────────────────┘

```

**Design in config**

Layout ⇒ [https://deepwiki.com/MagicMirrorOrg/MagicMirror/2.4-ui-and-css-framework](https://deepwiki.com/MagicMirrorOrg/MagicMirror/2.4-ui-and-css-framework)

General
|        | left          | center          | right         |
|--------|---------------|-----------------|---------------|
| Top    | top_left      | top_center      | top_right     |
| Center | upper_third   | middle_center   | lower_left    |
| Bottom | bottom_left   | bottom_center   | bottom_right  |


Page 1

|        | left          | center          | right         |
|--------|---------------|-----------------|---------------|
| Top    | Time & Date   | top_center      | Wetter        |
| Center |               |                 |               |
| Bottom | Spotify       |                 | Trello Task   |



Page 2
|        | left | center          | right |
|--------|------|-----------------|-------|
| Top    |      |                 |       |
| Center |      | Google Calender |       |
| Bottom |      |                 |       |


**Use Cases**

1. **Sleep if no motion**
  - Mirror goes to sleep if no motion is detected for more than 10 minutes.
  - Asas
  - as

2. **Multi Page**  
  - Mirror layout is split into multiple pages; each page shows a different set of modules.

3. **Swipe to change page**  
  - Swipe left or right on the screen to switch between pages.

4. **Face Detection**  
  - Detects a face and triggers an Alexa sound and shows the Trello module.

5. **Trello**  
  - Displays Trello tasks from a specific list on the mirror.

6. **Alexa Connection**  
  - Connects to Alexa to play notification sounds.

7. **Spotify**  
  - Allows playing and controlling Spotify; if the mirror is not the active device, a device selection button is shown.

<br>

## 📄 Documentation LED Stripe

- The LED is only working when using the correct venv => see documentation

<br>


## ☁️ Backup: Raspberry Pi Code to Google Drive

To ensure that the code running on your Raspberry Pi is safe and recoverable, the project is regularly backed up to Google Drive. This protects you from data loss due to SD card corruption or hardware failure.

ssh pi@raspberrypi.local "sudo dd if=/dev/mmcblk0 bs=4M status=progress" \
| gzip > raspberrypi-full-backup.img.gz

<br>

## 🤬 Hints to not cry everytime

- TODO

<br>

## 🚧 NEXT STEPS / OPEN TODO

- Trello Tasks ⇒ https://github.com/benjaminflessner/MMM-Trello
- Indoor & Outdoor Thermometer
- Strava ⇒ https://github.com/tylerstambaugh/MMM-StravaWeekInBike
- Tesla Charging State ⇒ https://github.com/denverquane/MMM-Teslamate
- Schritt anzeige ⇒ https://github.com/fry0815/MMM-GoogleFit
- Show Internet Speed ⇒ https://github.com/irsheep/MMM-SnmpIntSpeed
- Campfire
- SpaceX Flights ⇒ https://github.com/koxm/MMM-SpaceX
- Schweiz Uhrzeit ⇒ https://github.com/splattner/MMM-bernwordclock


<br>

## 📅 Version History

<details>
<summary><strong>v1.0.1</strong> – 02.08.2026</summary>

- Added local notification logging for `MMM-ViewNotifications`.
- Added debugging methodology for the notification JSONL file.
- Enabled VIF notification module comment support for notification visibility.

</details>

<details>
<summary><strong>v1.0.0</strong> – 18.01.2026</summary>

- 🔧 Updated the readme and added a Architecture Drawing

</details>


<details>
<summary><strong>v0.0.1</strong> – 10.01.2026</summary>

- 🔧 First Version

</details>



<!--
========================
README MARKDOWN CHEAT SHEET (GitHub)
========================

HEADINGS
# H1
## H2
### H3
#### H4

TEXT
**bold**
*italic*
***bold & italic***
~~strikethrough~~
`inline code`

LISTS
- Unordered
  - Nested
1. Ordered
- [ ] Task
- [x] Task done

LINKS & IMAGES
[Link text](https://example.com)
![Alt text](path/to/image.png)

CODE BLOCKS
```js
console.log("Hello World");
```
 
BLOCKQUOTE
> This is a quote

TABLE
| Column | Column | Column |
|--------|--------|--------|
| Value  | Value  | Value  |

HORIZONTAL LINE
---

EMOJIS
🚀 ✨ ✅ ❌ ⚠️
:rocket: :sparkles: :white_check_mark:

COLLAPSIBLE (GitHub)
<details>
<summary>Click to expand</summary>

Hidden content
</details>

INTERNAL LINKS
[Go to Section](#section-name)

COMMON README STRUCTURE
# Project Name
## Features
## Installation
## Usage
## Configuration
## Contributing
## License

========================
-->

