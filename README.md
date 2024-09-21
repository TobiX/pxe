# tftproot for PXE

This is a simple setup for a PXE environment on a Debian 10+ (currently running
and tested on Debian 12) system.

This repository uses a Git submodule to include `localboot.lua`, so you should
probably use

    git clone --recursive https://github.com/TobiX/pxe.git

or update the submodules after the clone:

    git submodule init
    git submodule update

Most tools are just symlinked into the matching Debian packages, so you should
install at least

    syslinux-common syslinux-efi pxelinux

## MultiArch support

Since 2021-05, this repository uses a bunch of [symlinks] to support BIOS and
EFI64 boot from the same config. While this seems to work for some options, the
EFI code path is much less tested then the BIOS code path. (there are probably
some options which flat-out don't work in EFI mode)

[symlinks]: https://wiki.syslinux.org/wiki/index.php?title=PXELINUX-Multi-Arch#Distinct_directory_symlink_path

## Debian Installer

To use the Debian installer entry, install `di-netboot-assistant`, configure it
to your liking and run

    di-netboot-assistant --arch=amd64 install stable stable-gtk

(or similar).

## grml

Grab the files `vmlinuz` and `initrd.img` from the grml ISO (they are somewhere
in `boot/grml64*/`) and put them into the `grml` directory. Put the
GRML SquashFS (In `live/grml64*/*.squashfs` on the ISO) on a webserver and
adjust the URL in `common.cfg`.

## Using with dnsmasq

This is probably the minimal setup to use this setup with dnsmasq (assuming
this isn't the "main" DHCP server):

    dhcp-range=192.168.1.0,proxy
    pxe-service=x86PC,"pxelinux BIOS",bios/pxelinux
    pxe-service=x86-64_EFI,"syslinux EFI64",efi64/syslinux.efi
    enable-tftp
    tftp-root=/srv/tftp

    # Disable DNS (if you are using a different server for DNS)
    port=0

## Testing with qemu

You can test some aspects of this setup using qemu. For BIOS boot use:

    qemu-system-x86_64 -accel kvm -cpu host -netdev user,id=net0,tftp=$PWD,bootfile=bios/pxelinux.0 -device virtio-net-pci,netdev=net0 -boot n

For UEFI boot use:

    qemu-system-x86_64 -accel kvm -cpu host -netdev user,id=net0,tftp=$PWD,bootfile=efi64/syslinux.efi -device virtio-net-pci,netdev=net0 -boot n -drive file=/usr/share/OVMF/OVMF_CODE_4M.fd,if=pflash,format=raw,unit=0,readonly=on

The TianoCore files are in the `ovmf` Debian package
