# nighthawk-nas

Turns a Netgear Nighthawk X4S (R7800) into a simple network drive for your home network.

It is [OpenWrt](https://openwrt.org) 25.12.6 for the R7800, set up from its first boot to:

- get its address from your existing router (connect the two with an Ethernet cable)
- keep its own Wi-Fi off and run no DHCP server or firewall of its own
- mount USB drives by itself (NTFS, exFAT, FAT32 and ext4), both USB ports
- share them as the network folder `usb`, open to everyone on your home network, read and write, no password

The images are built from OpenWrt's official release with OpenWrt's own image builder, on GitHub Actions. All settings live in one first-boot script: [`files/etc/uci-defaults/99-nighthawk-nas`](files/etc/uci-defaults/99-nighthawk-nas). Extra packages are listed in [`packages.txt`](packages.txt).

> **Only for the Netgear R7800 (sold as Nighthawk X4S AC2600).** Do not flash it on any other model, including other Nighthawks.

## Download

Get the files from the [latest release](../../releases/latest):

| File | Use it when |
|---|---|
| `…-squashfs-factory.img` | the router runs Netgear's firmware, or it is stuck and you use TFTP recovery |
| `…-squashfs-sysupgrade.bin` | the router already runs OpenWrt |
| `SHA256SUMS` | to check the download |

## Install

Pick one of the three ways. Flashing takes a few minutes: never unplug the router while it runs.

### A. From Netgear's firmware (web page)

1. Connect to the router by cable or its Wi-Fi and open `http://routerlogin.net`.
2. Go to **Advanced → Administration → Firmware Update** (on some versions **Router Update**).
3. Choose the `factory.img` file, upload it and confirm any warning.
4. Wait until the router restarts. Its Wi-Fi then stays off: that is expected.

If Netgear's page refuses the file, use B.

### B. Netgear TFTP recovery (also works on a router that no longer starts)

You need a computer or other device with an Ethernet port and a TFTP client.

1. Give the computer the fixed address `192.168.1.10`, mask `255.255.255.0`, and connect it to one of the router's **black** ports.
2. Switch the router off. Hold the **Reset** button (small hole on the back), switch the router on and keep holding until the power light starts blinking, then let go.
3. Upload `factory.img` to `192.168.1.1` in binary mode:
   - Windows (turn on "TFTP Client" in Windows Features first): `tftp -i 192.168.1.1 put openwrt-…-factory.img`
   - macOS / Linux: `tftp 192.168.1.1`, then `binary`, then `put openwrt-…-factory.img`, then `quit`
4. The router writes the image and restarts by itself. Wait about five minutes.

### C. From OpenWrt

Open the OpenWrt web page, go to **System → Backup / Flash Firmware → Flash new firmware image**, choose `sysupgrade.bin`, **untick "Keep settings"** and flash.

## First start

1. Connect an Ethernet cable from your home router to one of the X4S's **black** ports (the yellow Internet port works too).
2. Plug in your USB drive. Large drives need their own power adapter.
3. Find the X4S's address: look for **nighthawk-nas** in your home router's list of connected devices.
4. Open `http://<its address>` in a browser. It has **no password yet**: set one right away under **System → Administration → Router Password**.

## Open the drive

The network folder is `usb`. Each drive is a folder inside it (`sda1`, `sdb1`, …). No user name or password.

| On | How |
|---|---|
| Windows | File Explorer address bar: `\\nighthawk-nas\usb` or `\\<its address>\usb`. It also shows under **Network**. |
| macOS | Finder → **Go → Connect to Server** → `smb://<its address>/usb` → **Guest** |
| Android | any file manager with SMB (Windows share) support: host = its address, user empty |
| Linux | file manager: `smb://<its address>/usb` |

Tip: in your home router, reserve a fixed address for nighthawk-nas so it never changes.

## Good to know

- **Home network only.** The folder is open to every device on your home network. Never forward its ports to the internet. For access from outside, use a VPN.
- **Unplugging a drive:** first unmount it under **System → Mount Points**. An NTFS drive pulled out while in use gets marked as not cleanly removed (`ntfsfix`, included, can clear that).
- **SSH** accepts keys only and has none at first. Add your public key under **System → Administration → SSH-Keys**.
- **Time zone** is UTC. Change it under **System → System**.
- **Back to Netgear's firmware:** use TFTP recovery (B) with Netgear's official image for the R7800.

### Drive tips (over SSH)

NTFS and FAT drives use OpenWrt's default `iso8859-1` character set, so names with symbols like ™ can look wrong. To use UTF-8 for a drive, and give its folder a name of your choice, use the drive's UUID from `block info`:

```
uci add fstab mount
uci set fstab.@mount[-1].uuid='<UUID>'
uci set fstab.@mount[-1].target='/mnt/MyDrive'
uci set fstab.@mount[-1].options='iocharset=utf8'
uci set fstab.@mount[-1].enabled='1'
uci commit fstab
block umount; block mount
```

exFAT drives already use UTF-8 (leave `options` out, or set only the folder name).

## Build it yourself

1. Fork this repository.
2. Optional: add your SSH public key as the repository secret `NAS_AUTHORIZED_KEY`. The image then lets that key log in, and the web page gets a random password, stored in `/root/initial-password` (read it over SSH). **Don't share images built with your key.**
3. Run the **Build image** workflow under **Actions**. The images come out as a workflow artifact, with `SHA256SUMS` and the package list.

## Credits

- [OpenWrt](https://openwrt.org) (GPL-2.0) and its R7800 support. OpenWrt source: <https://github.com/openwrt/openwrt>
- [ksmbd](https://github.com/cifsd-team/ksmbd-tools) SMB server, packaged by OpenWrt
- [wsdd2](https://github.com/Netgear/wsdd2), so Windows sees the share in its Network view

The build files in this repository are licensed under GPL-3.0.
