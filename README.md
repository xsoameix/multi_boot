# Install persistent Linux Mint and Windows 7 on USB

Layout:

1.  Partition 1:

    Grub and Linux Mint iso

    Type: FAT32

    Size: 2G (Linux Mint iso: 1.5G)

    Directories:

        ▾ boot/
          ▾ grub/
            ▸ fonts/
            ▸ i386-pc/
            ▸ locale/
            ▸ themes/
              grub.cfg
              grubenv
        ▾ iso/
            linuxmint-17.2-cinnamon-64bit.iso

2.  Partition 2:

    Windows 7 files

    Type: NTFS

    Size: 4G (Win7Ent_64_CHT_wSP1.ISO: 3.1G)

    Directories:

        ▸ boot/
        ▸ efi/
        ▸ sources/
        ▸ support/
        ▸ upgrade/
          autorun.inf
          bootmgr
          bootmgr.efi
          grub.exe
          setup.exe

3.  Partition 3:

    Persistent partition for Linux Mint

    Type: ext4

    Size: 0.5~4G

    Directories: (empty)

## Create partitions

Use `fdisk` to tag USB with dos label.

Create partition 1

    # fdisk /dev/sdf
    > n
    > p
    > 1
    > +2G

Change type

    > t
    > c  # W95 FAT32 (LBA)

Make it bootable

    > a
    > 1

Create partition 2

    > n
    > p
    > 2
    > +4G

Change type

    > t
    > 7  # HPFS/NTFS/exFAT

Create partition 3

    > n
    > p
    > 3
    > +1G

Change label

    # e2label /dev/sdf3 casper-rw

Replug your USB

Format

    # mkfs.vfat -F 32 /dev/sdf1
    # mkfs.ntfs -f /dev/sdf2
    # mkfs.ext4 /dev/sdf3

## Install Grub

Mount partition 1

    # mkdir /mnt/grub
    # mount /dev/sdf1 /mnt/grub

Install grub

    # grub-install --target i386-pc --boot-directory /mnt/grub --recheck /dev/sdf

Now partition 1 has new created directory

    ▾ boot/
      ▾ grub/
        ▸ fonts/
        ▸ i386-pc/
        ▸ locale/
        ▸ themes/
          grub.cfg
          grubenv

## Add Linux Mint iso to partition 1

Create a directory `iso` in partition 1 and copy linux mint iso into it

    # mkdir /mnt/grub/iso
    # cp ~/linuxmint-17.2-cinnamon-64bit.iso /mnt/grub/iso

Now partition 1 has these directories

    ▾ boot/
      ▾ grub/
        ▸ fonts/
        ▸ i386-pc/
        ▸ locale/
        ▸ themes/
          grub.cfg
          grubenv
    ▾ iso/
        linuxmint-17.2-cinnamon-64bit.iso

## Add Windows 7 to partition 2

Mount partition 2

    # mkdir /mnt/win
    # mount /dev/sdf2 /mnt/win

Download grub4dos and extract `grub.exe`

    # wget http://download.gna.org/grub4dos/grub4dos-0.4.4.zip
    # unzip grub4dos-0.4.4.zip
    # cp grub4dos-0.4.4/grub.exe /mnt/win

Mount Windows 7 iso and copy files

    # mount -r ~/Win7Ent_64_CHT_wSP1.ISO /mnt/iso
    # cp -r /mnt/iso/* /mnt/win
    # umount /mnt/iso

Now partition 2 has these directories

    ▸ boot/
    ▸ efi/
    ▸ sources/
    ▸ support/
    ▸ upgrade/
      autorun.inf
      bootmgr
      bootmgr.efi
      grub.exe
      setup.exe

## Configure Grub

The `casper-rw` label of partition 3 and `persistent` parameter in `grub.cfg` make Linux Mint persistent.

    # vim /mnt/grub/boot/grub/grub.cfg
    set timeout=10

    menuentry "Linux Mint 17.2 cinnamon 64bit" {
        set isofile='/iso/linuxmint-17.2-cinnamon-64bit.iso'
        loopback loop $isofile
        linux (loop)/casper/vmlinuz file=/cdrom/preseed/linuxmint.seed boot=casper iso-scan/filename=$isofile quiet splash persistent --
        initrd (loop)/casper/initrd.lz
    }

    menuentry "Windows 7 64-bit CHT" {
        insmod ntfs
        set root=(hd0,msdos2)
        linux /grub.exe
    }

# Open Linux Mint

Select first option of grub menu and press ENTER.

# Open Windows 7

Select second option of grub menu and press ENTER.

    grub> find --set-root /bootmgr
    grub> chainloader /bootmgr
    grub> boot

The Windows 7 installation guide interface is shown.

Press `Shift + F10` to open the terminal

    X:/sources > setup.exe
