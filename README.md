# Meshtastic node on the Pi

Documenting initial tinkerings. Hoping to turn this into some form of meshtastic base station in the future.

## Hardware

Current kit:

- [XIAO ESP32S3 & Wio-SX1262 Kit for Meshtastic & LoRa](https://www.seeedstudio.com/Wio-SX1262-with-XIAO-ESP32S3-p-5982.html) Connected via USB cable to the Raspberry Pi
- [Getting started guide](https://wiki.seeedstudio.com/xiao_esp32s3_&_wio_SX1262_kit_for_meshtastic/)

# Setup

First, update the node's firmware using the [official flasher](https://flasher.meshtastic.org/)

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

Set the regulatory region first. The firmware generates a new keypair when the region is configured. The node may reboot and temporarily disconnect.
Using wait here to give some buffer time before disconnecting to avoid write errors. Not exactly sure why, but seems to help.

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

## Example: Adding a channel

Let's add the [Berlin Chaos Mesh](https://potatomesh.net/pages/about) channel.
You may want to update the backup after adding a new channel.

Create the channel:

```bash
meshtastic --ch-add BerlinMesh
```

Configure the channel index and public key:

```bash
meshtastic --ch-index 1 --ch-set psk \
  'base64:Nmh7EooP2Tsc+7pvPwXLcEDDuYhk+fBo2GLnbA1Y1sg='
```

To listen. Haven't found a way to filter output by channel yet.

```bash
meshtastic --listen
```

To send a message specifically to BerlinMesh:

```bash
meshtastic --ch-index 1 --sendtext "Hello BerlinMesh"
```
