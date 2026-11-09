# adb-qr-pair

Pair an Android device for wireless debugging from the terminal: show a QR code that the phone scans, or type in its 6-digit pairing code. It does the same job as Android Studio's "Pair devices using Wi-Fi" dialog, but runs as a shell script.

## Requirements

- `adb` from Android SDK platform-tools 30 or newer, with mDNS support (see [Troubleshooting](#troubleshooting))
- `qrencode`, for QR mode only
- Android 11 or newer, with **Developer options → Wireless debugging** turned on
- The computer and the phone on the same network, with mDNS not blocked

## Install

```bash
git clone git@github.com:rodolfovillaruz/adb-qr-pair.git
cd adb-qr-pair
chmod +x adb-qr-pair
# optional: put it on your PATH
ln -s "$PWD/adb-qr-pair" ~/.local/bin/adb-qr-pair
```

## Usage

```bash
adb-qr-pair          # show a QR code
adb-qr-pair -c       # use a 6-digit pairing code instead
adb-qr-pair -t 300   # wait up to 300 s for the device (default: 120)
adb-qr-pair -n       # pair only, don't connect afterwards
adb-qr-pair -h       # help
```

### QR code

1. Run `adb-qr-pair`.
2. On the phone, open **Developer options → Wireless debugging → Pair device with QR code**.
3. Scan the QR code shown in the terminal.

### Pairing code

1. On the phone, open **Developer options → Wireless debugging → Pair device with pairing code**.
2. Run `adb-qr-pair -c`.
3. When the script finds the phone, type the 6-digit code shown on the phone.

## How it works

The QR code holds `WIFI:T:ADB;S:<name>;P:<password>;;`, where the name and password are random for each run. After scanning it, the phone announces an `_adb-tls-pairing._tcp` service with that name on the local network over mDNS. The script polls `adb mdns services` until that service appears, then runs `adb pair <ip:port> <password>`.

After pairing, the script checks whether the phone has already connected. If not, it looks for the phone's `_adb-tls-connect._tcp` service and runs `adb connect`.

## Troubleshooting

**`the running adb server has no mDNS support`**: the running adb server was started by an adb build without mDNS, often a distro `android-tools` package. Restart the server so that the SDK's adb starts it:

```bash
adb kill-server && adb-qr-pair
```

Restarting the server disconnects every device connected at that moment.

**Timed out waiting for the device**: the phone and the computer must be on the same network. Some guest, office and VPN networks block mDNS. Check that `adb mdns services` lists the phone while its pairing screen is open.

**Paired, but couldn't find the connect port**: run `adb connect <phone-ip>:<port>` yourself, using the IP address and port shown on the **Wireless debugging** screen.
