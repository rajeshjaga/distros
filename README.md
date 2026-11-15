# DISTRO ALL-in-ONE
Installation of linux distributions via ArchLinux ISO
</br>
Yes, You can almost install all the linux distros such as debian and debian based, fedora and fedora based, any source based distro(gentoo, LFS), Nix OS, obviously Arch Linux

## Index
- [Alpine](./alpine.md)
- [Debian](./debian.md)
- [Fedora](./fedora.md)
- [Rockylinux](./rockylinux.md.md)
- [Ubuntu](./ubuntu.md)

# Create a latest Arch installation medium, follow below instructions:

Use a ArchLinux medium to create this iso, either use a premade from [ArchLinux ISO](https://archlinux.org/download/) boot into vm or on a machine then copy to other drive or usb

## Now after booting up the iso or on a arch machine, follow from here:
#### Prerequistes
```bash
pacman -S archiso
mkdir -pv archiso/{tmp,iso,profile}
cp -rv /usr/share/archiso/configs/releng/* archiso/profile/
```

Here in the archiso directory we have a profile folder to customize the final iso with packages and more stuff

#### Building the iso
```bash
printf 'distribution-gpg-keys \nrpm \ndebootstrap \napk-tools \nzypper \ndnf' | tee -a ./archiso/profile/packages.x86_64
mkarchiso -v -r -w /tmp/archiso-tmp -o ./archiso ./archiso/profile && rm -rf ./tmp/* # it is adviced to remove this tmp data if the build is complete or not this directory cannot be used again
```


That's it you have a ISO that can bootstrap debian, alpine, fedora based distros.</br>
For more customization with archiso refer https://wiki.archlinux.org/title/Archiso
