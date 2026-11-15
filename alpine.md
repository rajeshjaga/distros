# Alpine Linux

### Disk partition
```bash
parted -- /dev/vda mklabel gpt
parted -- /dev/vda mkpart ESP fat32 1M 1G
parted -- /dev/vda mkpart '""' 1G 100%
```

### LVM partition
```bash
pvcreate /dev/vda2
vgcreate suse /dev/vda2
lvcreate -n swap -L 2G <distro>
lvcreate -n root -l 100%FREE <distro>
```

### Mounting drives in the right places
```bash
mkfs.ext4 /dev/<distro>/root
mount /dev/<distro>/root /mnt
mkswap /dev/<distro>/swap
swapon /dev/<distro>/swap
mkfs.fat -F 32 /dev/vda1
mount -m /dev/vda1 /mnt/boot
```

### Install base system

```bash
pacman -S apk-tools alpine-keyring
apk --root /mnt --initdb --arch x86_64 -U \
--keys-dir /usr/share/apk/keys \
-X https://dl-cdn.alpinelinux.org/alpine/latest-stable/main \
-X https://dl-cdn.alpinelinux.org/alpine/latest-stable/community \
add alpine-base
genfstab -U /mnt >> /mnt/etc/fstab
mount --rbind /dev /mnt/dev
mount --rbind /sys /mnt/sys
mount -t proc proc /mnt/proc
mount --bind /run /mnt/run
mount --make-rslave /mnt/dev
mount --make-rslave /mnt/sys
mount -t efivarfs efivarfs /mnt/sys/firmware/efi/efivars
arch-chroot /mnt 
```

### Chroot into alpine base system

```bash
export PATH=/usr/sbin:/usr/bin:/bin:/sbin
cat  << EOF > /etc/resolv.conf
nameserver 8.8.8.8
nameserver 9.9.9.9
EOF
cat << EOF > /etc/network/interfaces
auto lo
iface lo net loopback
auto eth0
iface eth0 inet dhcp
EOF
 apk add linux-lts linux-firmware mkinitfs lvm2 cryptsetup grub grub-efi efibootmgr build-base cmake
 sed -i 's/ext4/ext4 lvm cryptsetup virtio/' etc/mkinitfs/mkinitfs.conf
```

### OPenrc services of new system
```bash
rc-update add devfs sysinit
rc-update add dmesg sysinit
rc-update add mdev sysinit
rc-update add hwdrivers sysinit
rc-update add modloop sysinit
rc-update add hostname boot
rc-update add bootmisc boot
rc-update add hwclock boot
rc-update add modules boot
rc-update add sysctl boot
rc-update add swap boot
rc-update add localmount boot
rc-update add networking boot
rc-update add syslog boot
rc-update add mount-ro shutdown
rc-update add killprocs shutdown
rc-update add savecache shutdown
```

### Rebuild linux image
```bash
/ $ mkinitfs -o /boot/initramfs-lts $(ls /lib/modules)
```

### Now GRUB
```bash
/ $ blkid /dev/vda2
/ $ cat 'GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4 rootfstype=ext4 cryptroot=UUID=<vda2-uuid> cryptdm=alpinecrypt root=/dev/mapper/alpine_vg-root"' >> /etc/deault/grub
/ $ grub-install --target=x86_64-efi --efi-directory=/boot --boot-directory=/boot --bootloader-id=alpine --removable --recheck 
/ $ grub-mkconfig -o /boot/grub/grub.cfg
```
