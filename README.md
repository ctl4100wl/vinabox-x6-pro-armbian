# Vinabox X6 Pro Armbian Core Image
###  This is a core-only release.
## Included:
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

# Installation guide
## Load images to ram and boot to rescue u-boot first
```
cd "your-location/vinabox-x6pro-working-kit/rescue"

KERNEL="zImage"
INITRD="initramfs.cpio.gz"
UBOOT="../bootloader/u-boot-sunxi-with-spl.bin"

KERNEL_ADDR="0x42000000"
INITRD_ADDR="0x43400000"

INITRD_SIZE_DEC="$(stat -c '%s' "$INITRD")"
INITRD_SIZE_HEX="$(printf '0x%x' "$INITRD_SIZE_DEC")"

echo "INITRD_SIZE_HEX=$INITRD_SIZE_HEX"

sudo sunxi-fel ver

sudo sunxi-fel -p write "$KERNEL_ADDR" "$KERNEL"
sudo sunxi-fel -p write "$INITRD_ADDR" "$INITRD"
sudo sunxi-fel -p uboot "$UBOOT" 
```
## UART: U-Boot setup to boot the kernel
```
setenv kernel_addr_r 0x42000000
setenv ramdisk_addr_r 0x43400000
setenv fdt_addr_r 0x43000000
setenv initrd_size 0x104332

fdt addr -c
echo ${fdtcontroladdr}

cp.b ${fdtcontroladdr} ${fdt_addr_r} 0x40000
fdt addr ${fdt_addr_r}
fdt resize 0x20000

fdt set /soc/mmc@1c10000 status disabled
fdt set /soc/mmc@1c11000 status okay
fdt set /soc/mmc@1c11000 pinctrl-names default
fdt set /soc/mmc@1c11000 pinctrl-0 <0x0000004b>
fdt set /soc/mmc@1c11000 vmmc-supply <0x00000009>
fdt set /soc/mmc@1c11000 bus-width <0x00000008>
fdt set /soc/mmc@1c11000 non-removable
fdt set /soc/mmc@1c11000 cap-mmc-highspeed

fdt set /soc/usb@1c19000 status okay
fdt set /soc/usb@1c19000 dr_mode peripheral
fdt set /soc/phy@1c19400 status okay

setenv bootargs 'console=ttyS0,115200 earlyprintk rdinit=/init rw loglevel=8'

bootz ${kernel_addr_r} ${ramdisk_addr_r}:${initrd_size} ${fdt_addr_r}
```
### In rescue shell, confirm eMMC exist
```
cat /proc/partitions
ls -l /dev/mmcblk*
fdisk -l /dev/mmcblk1
```
## Stream xz image from release over netcat (on your PC)
```
xzcat Vinabox-X6-Pro-Armbian_26.08_Trixie_6.18.35_Core_eMMC.img.xz | \
  ncat -l 9000 --send-only
```
## Rescue shell: Recieve then flash to eMMC then reboot
```
busybox nc 10.66.0.1 9000 | dd of=/dev/mmcblk1 bs=4M conv=fsync status=progress
sync
reboot -f
```
# If you have successfully booted into armbian, do this first
```
sudo parted -s /dev/mmcblk1 resizepart 1 100%
sudo resize2fs /dev/mmcblk1p1
df -h /
```
### Check filesystems
```
cat /proc/device-tree/model
hostname
hostnamectl
cat /etc/armbian-release
lsblk
df -h /
ip -br addr
networkctl status end0
ping -c 3 1.1.1.1
ping -c 3 deb.debian.org
ls /sys/class/leds
cat /sys/class/leds/x6pro:red:pwr/trigger
cat /sys/class/leds/x6pro:blue:status/trigger
```
### Test LED if you want to light the LEDs
```
echo default-on | sudo tee /sys/class/leds/x6pro:red:pwr/trigger
echo heartbeat | sudo tee /sys/class/leds/x6pro:blue:status/trigger
```
