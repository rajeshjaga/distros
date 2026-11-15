# Instasll suse based systems

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
lvcreate -n swap -L 2G suse
lvcreate -n root -l 100%FREE suse
```

### Mounting drives in the right places
```bash
mkfs.ext4 /dev/suse/root
mount /dev/suse/root /mnt
mkswap /dev/suse/swap
swapon /dev/suse/swap
mkfs.fat -F 32 /dev/vda1
mount -m /dev/vda1 /mnt/boot
```

### Initializing the basestrap
```bash
pacman -Sy distribution-gpg-keys zypper
zypper --root /mnt ar https://download.opensuse.org/tumbleweed/repo/oss tumbleweed
zypper --root /mnt refresh
zypper --root /mnt search patterns-base
rpm --root /mnt --import /usr/share/distribution-gpg-keys/opensuse/RPM-GPG-KEY-openSUSE-tumbleweed
zypper --root /mnt install patterns-base-base

```
