# Install persistent Linux Mint and Windows 7 on USB

Layout:

1.  Partition 1:

    Grub and Linux Mint iso

    Type: FAT32

    Size: 5G (Linux Mint iso: 1.5G + Win7Ent_64_CHT_wSP1.ISO: 3.1G)

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
            Win7Ent_64_CHT_wSP1.ISO

2.  Partition 2:

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
    > 3
    > +1G

Change label

    # e2label /dev/sdf2 casper-rw

Replug your USB

Format

    # mkfs.vfat -F 32 /dev/sdf1
    # mkfs.ext4 /dev/sdf2

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

## Linux Mint

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

## Windows 7

Download grub4dos and extract `grub.exe`

    # wget http://download.gna.org/grub4dos/grub4dos-0.4.4.zip
    # unzip grub4dos-0.4.4.zip
    # cp grub4dos-0.4.4/grub.exe /mnt/grub

Copy Windows 7 iso into it

    # cp ~/Win7Ent_64_CHT_wSP1.ISO /mnt/grub/iso

Download `Contig.exe`

    # wget https://download.sysinternals.com/files/Contig.zip
    # unzip Contig.zip
    # cp Contig.exe /mnt/grub/iso

Use `Contig.exe` to defrag the iso

    # E is the label of partition 1
    E:\iso> Contig.exe -a -v -s Win7Ent_64_CHT_wSP1.ISO*

Now partition 2 has these directories

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
        Win7Ent_64_CHT_wSP1.ISO
        Contig.exe
      grub.exe

## Setup imdisk

    # wget http://www.ltr-data.se/files/imdiskinst.exe
    # sudo apt-get install p7zip-rar
    # 7z e -y imdiskinst.exe
    # mkdir /mnt/grub/imdisk
    # cp * /mnt/grub/imdisk

Create installing imdisk script

    # vim /mnt/grub/imdisk/SetupImDisk.CMD
    rundll32.exe setupapi.dll,InstallHinfSection DefaultInstall 132 .\imdisk.inf

Create loading `Win7Ent_64_CHT_wSP1.ISO` script

    # vim /mnt/grub/imdisk/SetupCDROM.CMD
    Set fullname=%~1
    imdisk -a -f "%fullname%" -m #:

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
        set root=(hd0,msdos1)
        linux /grub.exe
    }

# Install Linux Mint

Select first option of grub menu and press ENTER.

# Install Windows 7

Select second option of grub menu and press ENTER.

    grub> map (hd0) (hd1)
    grub> map (hd1) (hd0)
    grub> find --set-root /iso/Win7Ent_64_CHT_wSP1.ISO
    grub> map /iso/Win7Ent_64_CHT_wSP1.ISO (hd32)
    grub> map --hook
    grub> chainloader (hd32)
    grub> boot

The Windows 7 installation guide interface is shown.

Press `Shift + F10` to open the terminal

    X:\sources> pushd E:\imdisk
    E:\imdisk> SetupImDisk.CMD
    E:\imdisk> SetupCDROM.CMD E:\iso\Win7Ent_64_CHT_wSP1.ISO

Make NTFS partition to store Virtual Hard Disk files

    E:\imdisk> diskpart
    DISKPART> list disk
    DISKPART> select disk 0
    DISKPART> list partition
    DISKPART> select partition 1  # partition 1 is NTFS partition
    DISKPART> format fs=ntfs label="vhds" quick
    DISKPART> assign letter=c
    DISKPART> active

Create Virtual Hard Disk (win7.vhd: 20G)

    DISKPART> create vdisk file=c:\win7.vhd maximum=20000 type=expandable
    DISKPART> attach vdisk file=c:\win7.vhd
    DISKPART> exit
    E:\imdisk> exit

Proceed with installation and choose the partition just created (20G).
When the computer reboot, plug out the USB and continue the installation.

# Reference

[利用 GRUB2 掛載 ISO 檔安裝 OS](http://jw1903.blogspot.tw/2013/08/grub2-iso-os.html)

[Grub4Dos 系統安裝碟](http://fireball-catcher.blogspot.tw/2012/07/grub4dos.html)

[Installing a Fresh Windows OS to a New Bootable VHD with no Host OS for Boot to VHD](http://www.johnpapa.net/bootoffmetal/)
