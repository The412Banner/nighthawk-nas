# nighthawk-nas

Turns a Netgear Nighthawk X4S (R7800) into a simple network drive for a home network.

It builds an [OpenWrt](https://openwrt.org) image for the R7800 that, from its first boot:

- gets its address from your existing router (plug it into your router with an Ethernet cable)
- keeps its own Wi-Fi off and runs no DHCP server or firewall of its own
- mounts any USB drive by itself under `/mnt` (NTFS, exFAT, FAT32 and ext4)
- shares `/mnt` as the network folder `usb`, open to everyone on the home network, read and write, no password

Everything is built from OpenWrt's official release with OpenWrt's own image builder, on GitHub Actions. The settings live in one first-boot script: [`files/etc/uci-defaults/99-nighthawk-nas`](files/etc/uci-defaults/99-nighthawk-nas). Extra packages are listed in [`packages.txt`](packages.txt).

## Build

Run the **Build image** workflow. It downloads OpenWrt 25.12.6's image builder, checks its sha256 against OpenWrt's list, and uploads the images as a workflow artifact:

- `…-squashfs-factory.img`: for installing from Netgear's own firmware
- `…-squashfs-sysupgrade.bin`: for updating a router that already runs OpenWrt
- `SHA256SUMS` and the package manifest

To log in over SSH later, add your public key as the repository secret `NAS_AUTHORIZED_KEY` before building. SSH accepts keys only.

## Install

1. Factory-reset the router, connect to its Wi-Fi and open `routerlogin.net`.
2. **Advanced → Administration → Firmware Update**: upload the `factory.img` file and wait for the router to restart.
3. Plug an Ethernet cable from your home router into one of the X4S's ports.
4. Plug in the USB drive. Large drives need their own power adapter.
5. Open the share from any SMB-capable file manager: `\\nighthawk-nas\usb` or `\\<its address>\usb`.

The web settings page is at the X4S's address. Its random password is in `/root/initial-password` (read it over SSH).

## Good to know

- The share is open to every device on your home network. Do not forward its ports to the internet.
- Before unplugging a drive, unmount it under **System → Mount Points** on the web page. An NTFS drive pulled out while in use gets marked as not cleanly removed; `ntfsfix` (included) can clear that.
- If something goes wrong, OpenWrt's failsafe mode and Netgear's TFTP recovery both need a wired computer or device.
- To go back to Netgear's firmware, use Netgear's TFTP recovery with their official image.

## Credits

- [OpenWrt](https://openwrt.org) (GPL-2.0) and its R7800 support
- [ksmbd](https://github.com/cifsd-team/ksmbd-tools) SMB server, packaged by OpenWrt
- [wsdd2](https://github.com/Netgear/wsdd2), so Windows sees the share in its Network view

The build files in this repository are licensed under GPL-3.0.
