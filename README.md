Vinabox X6 Pro Armbian Core Image

This is a core-only release.

Included:
- U-Boot SPL for eMMC boot
- Patched DTB:
  - model = Vinabox X6 Pro
  - Ethernet internal PHY MII
  - LEDs PL10 red, PA15 blue
- Hostname: x6pro
- MOTD: Vinabox X6 Pro
- Board name patched in /etc/armbian-release

Not included:
- Desktop GUI
- XFCE
- Firefox
- Extra packages

Users can install their own GUI later, for example:
  sudo apt update
  sudo apt install xfce4 lightdm firefox-esr

If image is split:
  cat IMAGE.img.xz.part-* > IMAGE.img.xz

Stream split image directly:
  cat IMAGE.img.xz.part-* | xzcat | ncat -l 9000 --send-only

Stream normal image:
  xzcat IMAGE.img.xz | ncat -l 9000 --send-only

Rescue shell:
  busybox nc 10.66.0.1 9000 | dd of=/dev/mmcblk1 bs=4M conv=fsync status=progress
  sync

After first boot, expand rootfs:
  sudo parted -s /dev/mmcblk1 resizepart 1 100%
  sudo resize2fs /dev/mmcblk1p1

Build generated: 2026-06-19T22:19:20+07:00
