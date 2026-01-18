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


TODO


---

<!-- GETTING STARTED -->
## 🏗️ Architecture

### Frontend
TODO

### Motion → Display On/Off (Wayland + wlr-randr)
This setup turns the display ON when Motion starts an event and OFF when Motion ends an event.
The “no motion” delay is handled by Motion’s event_gap.

**Components**

- Camera feed exposed as V4L2 loopback device: /dev/video10 => because the original Feed is not compatible with motion AND the face recognition otherwise would use the same feed
- Motion reads /dev/video10 and triggers hooks
- wlr-randr controls the Wayland output (e.g. HDMI-A-1)
- Motion runs as user motion, so we use passwordless sudo to run display scripts as user pi (Wayland session user)
- Two Scripts are created to turn the screen on and off
-- /home/pi/bin/screen-on.sh AND /home/pi/bin/screen-off.sh


**Architecture Overview**
- Camera is accessed only once via libcamera
- /dev/video10 acts as a shared virtual camera
- Motion reads /dev/video10
- Display control happens via wlr-randr
```bash
Pi Camera
   ↓
rpicam-vid (libcamera)
   ↓
ffmpeg
   ↓
/dev/video10 (v4l2loopback)
   ↓
Motion
   ├─ on_event_start → screen ON
   └─ on_event_end   → screen OFF (after event_gap)
```

**Motion Commands**
```bash
# Get to the Config
sudo nano /etc/motion/motion.conf

# Restart the service
sudo systemctl restart motion

# Motion is running as a Service in the background => can be edited here
sudo systemctl edit motion

# Check the status of motion
sudo systemctl status motion --no-pager
journalctl -u motion -f
sudo tail -f /var/log/motion/motion.log

# See the loopback Camera service file
sudo nano /etc/systemd/system/camera-loopback.service
# Enable and start it
sudo systemctl daemon-reload
sudo systemctl enable --now camera-loopback.service

# Stop User Service Magic Mirro
systemctl --user stop magicmirror

# See System Services (without GUI)
running => systemctl list-units --type=service --state=running
all => systemctl list-unit-files --type=service
# See User Services (with GUI) => MagicMirror
systemctl --user list-units --type=service

```

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
Browser    │  console.log(...)  ───────▶  │                │  console.log(...) ────▶ Terminal
Console    │  Log.log(...)      ───────▶  │                │                              │
           │                               │                │  sendSocketNotification(...) │
           │  socketNotificationReceived   │◀──── Bridge ───│  (push message to UI)        │
           │   → console.log(...)          │                │                              │
           └───────────────────────────────┘                └──────────────────────────────┘

```

<br>

## 🔧 How To Use

## Environment for development

Credentials => see .env File

1. VPC to Raspberry PI via RealVNC Viewer
2. IDE (Cursor) to PI via SSH
3. Start Mirror Application

```bash
# Start the Magic Mirror Application
node --run start
```

## How to install modules

1. Simple copy repo into /modules folder (git clone <repo-url>)


<br>

## 🤬 Hints to not cry everytime

- TODO

<br>

## 🚧 TODO

**Node Red**
- For alexa Connection

- Start it with "node-red"


- 97 LED Stripe length
- The LED is only working when using the correct venv => see documentation
- add documentation for "motion" which i use for enabel and disable PI
- Added a service that is publishing the video feed in the correct format to "motion" => sudo nano /etc/systemd/system/camera-loopback.service
-- The event gap in the settings of "motion" define how long after a motion the end of motion command should be send
- add documentation that Magic Mirror is also running as a service: 

systemctl --user stop magicmirror
systemctl --user start magicmirror
systemctl --user restart magicmirror
systemctl --user disable magicmirror
systemctl --user enable magicmirror
journalctl --user -u magicmirror -f


1. Loopback Split Service 

/etc/systemd/system/camera-loopback.service

sudo nano /etc/modprobe.d/v4l2loopback.conf => Create the devices

sudo nano /usr/local/bin/camera-loopback-split.sh => transfer the images into the devices

<br>

## 📅 Version History


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

