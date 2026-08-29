# Meshtastic node on the Pi

Initial tinkerings and pokings. Hoping to turn this into some form of meshtastic base station in the future.

## To-do

- Update the node's firmware using the [official flasher](https://flasher.meshtastic.org/) (done)
  - Rotate security keys by choosing full erase
- Install Meshtastic Python CLI and verify it works (done)
- Rotate node security keys (done)
- Disable node's bluetooth (done)
- Give the node a recognizable name (done)
- Disable wifi (for now) (done)
- Backup configuration (done)
- Add the local BerlinMesh coordination channel described [here](https://codeberg.org/berlinmesh/meshwiki/wiki/Empfohlene-Einstellungen)

## Hardware

Current kit:

- [XIAO ESP32S3 & Wio-SX1262 Kit for Meshtastic & LoRa](https://www.seeedstudio.com/Wio-SX1262-with-XIAO-ESP32S3-p-5982.html) Connected via USB cable to the Raspberry Pi
- [Getting started guide](https://wiki.seeedstudio.com/xiao_esp32s3_&_wio_SX1262_kit_for_meshtastic/)

## Python CLI installation

Install using virtual environment. SSH into the Pi and run:

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
/dev/ttyACM0
    desc: seeed-xiao-s3 - TinyUSB CDC
    hwid: USB VID:PID=2886:0059 SER=1CDBD4A88944 LOCATION=1-1:1.0
/dev/ttyAMA10
    desc: n/a
    hwid: n/a
2 ports found
```

Test. A successful result downloads the node database and prints your node information, radio configuration and channels.

```bash
meshtastic --info
```

Test with specific port:

```bash
meshtastic --port /dev/ttyACM0 --info
```

## Configure the node

Set the regulatory region first. Modern firmware generates its new keypair when the region is configured. The node may reboot and temporarily disconnect. Using wait here to give some buffer time before disconnecting to avoid write errors. Not exactly sure why, but seems to help.

```bash
meshtastic --set lora.region EU_868 --wait-to-disconnect 20
```

Essential Berlin settings:

```bash
meshtastic --set lora.modem_preset MEDIUM_FAST --wait-to-disconnect 20
meshtastic --set device.role CLIENT --wait-to-disconnect 20
```

Set meaningful names:

```bash
meshtastic --set-owner "some-name" --wait-to-disconnect 20
meshtastic --set-owner-short "smnm" --wait-to-disconnect 20
```

### Verify the config

```bash
meshtastic --get lora.region # expected: 3, for EU_868
meshtastic --get lora.use_preset # expected: True
meshtastic --get lora.modem_preset # expected: 4, for MEDIUM_FAST
meshtastic --get lora.channel_num
meshtastic --get lora.tx_enabled
meshtastic --qr
```

### Create a backup of the node's configuration

Save the backup in a secure place. This snippet copies the config from the Pi into the clipboard. It runs the Meshtastic CLI installed inside the virtual environment, reads the connected radio’s configuration and prints it as YAML.

```bash
ssh your@pi.local '~/.venvs/meshtastic/bin/meshtastic --export-config' | pbcopy
```

## Bluetooth

Set a non-default Bluetooth PIN:

```bash
meshtastic --set bluetooth.mode FIXED_PIN
meshtastic --set bluetooth.fixed_pin 123456
```

Or, disable Bluetooth over USB with:

```bash
meshtastic --set bluetooth.enabled false
```

Verify afterward:

```bash
meshtastic --get bluetooth.enabled
```

## Misc commands

To watch incoming traffic continuously, run:

```bash
meshtastic
```

Logs:

```bash
meshtastic --noproto
```

Show nodes:

```bash
meshtastic --nodes
```

Show QR code of the primary-channel URL:

```bash
meshtastic --qr
```

Retrieve public key:

```bash
meshtastic --get security.public_key
```

## Ideas

Cool things to try. From easy to advanced:

- Send a message on the primary channel — someone may reply
- Enable telemetry — my node can broadcast battery voltage, and if you add sensors later, temperature/humidity
- Traceroute — in the web client, pick any node and run a traceroute to see the exact hops your packet takes to reach it. Fascinating way to understand the mesh topology
- Create a private channel using custom PSK
- Set up MQTT + WiFi on your node so it becomes a gateway for your local area
- Range test module — walks you through automatic range testing, logs RSSI/SNR at each point
- Run a Store & Forward server on the XIAO ESP32S3
- Python CLI (`pip install meshtastic`) — lets you script interactions, send messages programmatically, read telemetry from Raspberry Pi.
- ePaper display — the project you already have planned
- Custom sensor node — attach a temperature sensor and have it broadcast readings into the mesh as telemetry
- PlatformIO custom module — as discussed
