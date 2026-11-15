# FEDORA

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
$ pacman -S dnf distribution-gpg-keys
$ rpm --root=/mnt --import /usr/share/distribution-gpg-keys/fedora/RPM-GPG-PRIMARY-fedora-45-key
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
$ genfstab -U /mnt >> /mnt/etc/fstab
$ chroot /mnt /bin/bash
$ export PATH=/usr/local/bin:/usr/local/sbin:/usr/sbin:/usr/bin:/bin:/sbin
$ ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
$ dnf install kernel kernel-headers grub2-efi-x64 efibootmgr cryptsetup lvm2 grub2-pc shim vim  -y
```

### Prep the storage, lvm and cryptsetup before installing base system
```bash
cat << EOF > /etc/crypttab
luks-<UUID of or the partition> UUID=<UUID-Partition2> none luks
EOF
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
echo 'do_symlinks = no' >> /etc/kernel-img.conf
echo 'add_dracutmodules+=" crypt lvm "' | tee /etc/dracut.conf.d/
```

### Install Minimal system
```bash
dnf install kernel kernel-headers kernel-modules-extra grub2-efi-x64 shim efibootmgr cryptsetup lvm2 glibc-langpack-en vim git -y
dracut --regenerate-all --force
```

### Verify LVM and crypstsetup is in initramfs
```bash
lsinitrd /boot/initramfs-*.img | grep -E "lvm|luks"
```

### Setup the system for basic user, selinux on root and root password
```bash
ls -Z /usr/lib/systemd/systemd # if the first character of this command output is ? then do below command to fix system
setfiles -F -v /etc/selinux/targeted/contexts/files/file_contexts /
passwd
useradd -mG wheel <username>
passwd <username>
```

### Installing grub (the dnf command run the scriptlet for grub but fails to register in BIOS
```bash
dnf install grub2-efi-x64-modules grub2-tools grubby grub2-tools-efi grub-tools-extra -y
grub2-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Fedora --recheck --force
grub2-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Fedora --recheck --removable --force
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Gracefully unmount the system
```bash
exit
umount -R /mnt
swapoff -a
cryptsetup close <crypsetup-name>
```
