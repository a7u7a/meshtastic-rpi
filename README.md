# Meshtastic on the pi

## To-do

- Update firmware
- Rotate node security keys
- Disable bluetooth
- Backup configuration
- Give the node a recognizable name, instead of Meshtastic 8944
- Disable wifi (for now) (done)
- Add the local BerlinMesh coordination channel described here: https://codeberg.org/berlinmesh/meshwiki/wiki/Empfohlene-Einstellungen

## Hardware

Current kit:
- https://www.seeedstudio.com/Wio-SX1262-with-XIAO-ESP32S3-p-5982.html
- Getting started guide: https://wiki.seeedstudio.com/xiao_esp32s3_&_wio_SX1262_kit_for_meshtastic/

## Python CLI installation

Installation using virtual environment

```bash
python3 --version
python3 -m venv ~/.venvs/meshtastic
source ~/.venvs/meshtastic/bin/activate
python -m pip install --upgrade pip
python -m pip install --upgrade "meshtastic[cli]"
```

Find the serial port:

```bash
python -m serial.tools.list_ports -v
```

Output:

```
(meshtastic) ayu@ramiel:~ $ python -m serial.tools.list_ports -v
/dev/ttyACM0        
    desc: seeed-xiao-s3 - TinyUSB CDC
    hwid: USB VID:PID=2886:0059 SER=1CDBD4A88944 LOCATION=1-1:1.0
/dev/ttyAMA10       
    desc: n/a
    hwid: n/a
2 ports found
```

Test:

```bash
meshtastic --info
```

Test with specific port:

```bash
meshtastic --port /dev/ttyACM0 --info
```
For a persistent setup, check for a stable device name:

```bash
ls -l /dev/serial/by-id/
```

Then use that full path:

```bash
meshtastic --port "/dev/serial/by-id/<device-name>" --info
```

A successful result downloads the node database and prints your node information, radio configuration and channels.

```bash

```

```bash

```

## Back up config

```bash
meshtastic --export-config > meshtastic-backup.yaml
```

## Name the node

```bash
meshtastic --set-owner "Your descriptive name"
meshtastic --set-owner-short "ABCD"
```

## Full erase and firmware installation

Stop the CLI and disconnect the node from the Pi.
Connect it to your Mac.
Open the official Meshtastic Web Flasher in Chrome or Edge.
Select:
Device: Seeed XIAO S3
Firmware: latest stable/beta release
Full Erase and Install
Flash it, then reconnect it to ramiel.


## Ideas

Cool things to try. From easy to advanced:

- **Send a message on the primary channel** — with 60 nodes visible, someone may reply
- **Enable telemetry** — my node can broadcast battery voltage, and if you add sensors later, temperature/humidity
- **Traceroute** — in the web client, pick any node and run a traceroute to see the exact hops your packet takes to reach it. Fascinating way to understand the mesh topology
- Create a private channel using custom PSK
- **Set up MQTT + WiFi** on your node so it becomes a gateway for your local area
- **Range test module** — walks you through automatic range testing, logs RSSI/SNR at each point
- Run a Store & Forward server on the XIAO ESP32S3
- **Python CLI** (`pip install meshtastic`) — lets you script interactions, send messages programmatically, read telemetry from Raspberry Pi.
- **ePaper display** — the project you already have planned
- **Custom sensor node** — attach a temperature sensor and have it broadcast readings into the mesh as telemetry
- **PlatformIO custom module** — as discussed