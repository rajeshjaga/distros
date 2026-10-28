# FEDORA

### PREPARE DISK
```bash
cryptsetup luksFormat /dev/vda2
$ cryptsetup open /dev/vda2 fedora
$ pvcreate /dev/mapper/fedora
$ vgcreate fedora_vg /dev/mapper/fedora
$ lvcreate -L 2G fedora_vg -n swap
$ lvcreate -l 100%FREE  fedora_vg -n root
$ mount /dev/fedora_vg/root /mnt
$ mkfs.ext4 /dev/fedora_vg/root
$ mount /dev/fedora_vg/root /mnt
$ mkswap /dev/fedora_vg/swap
$ swapon /dev/mapper/fedora_vg-swap
$ mkfs.fat -F 32 /dev/vda1
$ mount -m /dev/vda1 /mnt/boot
```

### Install required tools for the base install of the distro(fedora)
```bash
# For the version you want that is in beta
$ export ver=45
$ export BASEURL="https://dl.fedoraproject.org/pub/fedora/linux/releases/test/$(echo $VER)_Beta/Everything/x86_64/os/"
$ export UPDATEURL=https://dl.fedoraproject.org/pub/fedora/linux/updates/$ver/Everything/x86_64/

# For the latest stable version
$ export BASEURL=https://dl.fedoraproject.org/pub/fedora/linux/releases/44/Everything/x86_64/os/
$ export UPDATEURL=https://dl.fedoraproject.org/pub/fedora/linux/updates/44/Everything/x86_64/

# for testing
$ export BASEURL=https://dl.fedoraproject.org/pub/fedora/linux/releases/test/45_Beta/Everything/x86_64/os/
$ export UPDATESURL=https://dl.fedoraproject.org/pub/fedora/linux/updates/testing/45/Everything/x86_64
```

### Base installation
```bash
$ dnf --installroot=/mnt --repofrompath=fedora,$BASEURL --releasever=45 group install -y "Core"
```

### Mount sys, proc, dev, devpts, run for our base system to have access to internet and to have live kernel interfaces

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
$ mount --rbind /dev /mnt/dev
$ mount --rbind /sys /mnt/sys
$ mount -t proc proc /mnt/proc
$ mount --bind /run /mnt/run
$ mount --make-rslave /mnt/dev
$ mount --make-rslave /mnt/sys
$ mount -t efivarfs efivarfs /mnt/sys/firmware/efi/efivars
```

### Chroot in the the new system to configure system

```bash
$ chroot /mnt /bin/bash
$ export PATH=/usr/local/bin:/usr/local/sbin:/usr/sbin:/usr/bin:/bin:/sbin
$ ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
$ dnf install kernel kernel-headers grub2-efi-x64 efibootmgr cryptsetup lvm2 grub2-pc shim vim  -y
```

```bash
dnf install kernel kernel-headers  grub2-efi-x64 grub2-pc shim vim  -y
dnf install efibootmgr cryptsetup lvm2  -y
cat << EOF > /etc/crypttab
luks-<UUID of or the partition> UUID=<UUID-Partition2> none luks
EOF
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
echo 'do_symlinks = no' >> /etc/kernel-img.conf
dracut --regenerate-all --force
vim /etc/dracut.conf.d/
dracut --regenerate-all --force
lsinitrd /boot/initramfs-*.img | grep -E "lvm|luks"
lsinitrd /boot/initramfs-*.img | grep -E "lvm"
lsinitrd /boot/initramfs-7.2.12-300.fc45.x86_64.img | grep -E "lvm"
lsinitrd /boot/initramfs-7.2.12-300.fc45.x86_64.img | grep -E "lvm|luks"
lsinitrd /boot/initramfs-7.2.12-300.fc45.x86_64.img | grep -E "luks"
dnf install kernel-modules-extra -y
dnf install --reinstall grub2-efi-x64 efibootmgr
dnf install grub2-efi-x64 efibootmgr --reinstall
dnf reinstall grub2-efi-x64 efibootmgr -y
passwd
dnf install glibc-langpack-en
localectl set-locale LANG=en_US.UTF-8
localectl -a
ls -Z /usr/lib/systemd/systemd
sestatus
useradd -mG wheel <username>
passwd <username>
EDITOR=vim visudo
```
