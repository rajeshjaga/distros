# Ubuntu

<br/>
Firstly paritioning disk

```bash
parted -- /dev/vda mklabel gpt
```

### Boot Partition
```bash
parted -- /dev/vda mkpart ESP fat32 1MiB 512MiB
```

### We'll be create a LUKS encrypted disk with LVM, cause data is important
```bash
cryptsetup luksFormat /dev/vda2
cryptsetup open /dev/vda2 cryptlvm
```

### Create a physical volume, then Volume group on the physical volume, for our root and swap we create logical volumes.

```bash
pvcreate /dev/mapper/cryptlvm
vgcreate  cryptdisk_vg /dev/mapper/cryptlvm
lvcreate -L 3GiB -n swap cryptdisk_vg
lvcreate -l 100%FREE -n root cryptdisk_vg
```

### For Any changes in the root partition extend and reduce the partition

```bash
$ lvextend -L 10G cryptdisk_vg/root #10G says change the current size to 10Gb mention the extact partition to extend
$ lvreduce -L 4G cryptdisk_vg/root #4G says change the current size to 4Gb mention the extact partition to reduce
```

### Format the partitions or here in our lvm we partition out logical volumes.

```bash
mkfs.ext4 /dev/cryptdisk_vg/root
mkswap /dev/cryptdisk_vg/swap
mkfs.fat -F 32 -L ESP /dev/vda1
```

### Now we mount the partitions and logical units

```bash
mount /dev/cryptdisk_vg/root
mount --mkdir /dev/vda1 /mnt/boot
swapon /dev/cryptdisk_vg/swap
```

### The disk layout should like this

```bash
$ lsblk
        name                    maj:min rm    size ro type  mountpoints
        loop0                     7:0    0 1018.8m  1 loop  /run/archiso/airootfs
        sr0                      11:0    1    1.5g  1 rom   /run/archiso/bootmnt
        vda                     254:0    0     25g  0 disk
        ├─vda1                  254:1    0   1023m  0 part  /mnt/boot
        └─vda2                  254:2    0     24g  0 part
          └─cryptlvm            253:0    0     24g  0 crypt
            ├─cryptdisk_vg-swap 253:1    0      3g  0 lvm   [swap]
            └─cryptdisk_vg-root 253:2    0      9G  0 lvm   /mnt
```

### Now we use bootstrap base system, before installing base packages, we need to install debootstrap

```bash
$ pacman -Sy debootstrap ubuntu-keyring
```

### Bootstraping and base installing packages, also installing keyring to gpg check packages
<br/>

```bash
$ debootstrap --arch=amd64 --components=main,universe resolute /mnt  https://in.archive.ubuntu.com/ubuntu/
```

Mount sys, proc, dev, devpts, run for our base system to have access to internet and to have live kernel interfaces

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

Chroot in the the new system to configure system

```bash
$ chroot /mnt /bin/bash
$ export PATH=/usr/local/bin:/usr/local/sbin:/usr/sbin:/usr/bin:/bin:/sbin
$ ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
$ sed 's/# en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
$ echo 'LANG=en_US-UTF-8' >> /etc/local.conf
$ echo "ubuntu" | tee /etc/hostname
$ echo 'do_symlinks = no' >> /etc/kernel-img.conf
$ apt update && apt install linux-image-generic grub-efi-amd64 efibootmgr ubuntu-server lvm2 cryptsetup cryptsetup-initramfs linux-modules-7.0.0-14-generic
$ passwd
$ useradd -mG sudo <username>
$ passwd <username>
$ hwclock --hctosys

```

# Crypttab setup

```bash
$ echo 'add_dracutmodules+= "crypt" '|tee /etc/dracut.conf.d/crypt.conf
$ echo "luks-<uuid-partition-encrypted>  UUID=<uuid-partition-encrypted> none luks" >> /etc/crypttab #uuid of /dev/vda2
$ dracut --force --generate-all
```

# Install bootloader

```bash
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Ubuntu --recheck
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Ubuntu --removable
$ update-grub
```
