# tftproot for PXE

This is a simple setup for a PXE environment on a Debian 10 (buster) system.

This repository uses a Git submodule to include `localboot.lua`, so you should
probably use

    git clone --recursive https://github.com/TobiX/pxe.git

or update the submodules after the clone:

    git submodule init
    git submodule update

Most tools are just symlinked into the matching Debian packages, so you should
install at least

    syslinux-common pxelinux

## Debian Installer

To use the Debian installer entry, install `di-netboot-assistant`, configure it
to your liking and run

    di-netboot-assistant --arch=amd64 install stable stable-gtk

(or similar).

## GParted Live

Get the GParted Live ZIP file and put the `vmlinuz` and `initrd.img` into the
`gparted` directory. Put the `filesystem.squashfs` on a webserver and adjust
the URL in `pxelinux.cfg/default`.

## Using with dnsmasq

This is probably the minimal setup to use this setup with dnsmasq (assuming
this isn't the "main" DHCP server):

    dhcp-range=192.168.1.0,proxy
    dhcp-boot=pxelinux.0,192.168.1.25,192.168.1.0
    pxe-service=x86PC,"Automatic Network boot",pxelinux
    enable-tftp
    tftp-root=/srv/tftp

    # Disable DNS (if you are using a different server for DNS)
    port=0

