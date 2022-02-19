---
layout: post
title: Moving an Existing Linux Installation to a New Encrypted Drive [en]
description: Moving an existing Linux installation to a new encrypted drive
tags: [linux]
comments: true
---

I've recently acquired larger SSD and wanted to transfer my Linux
installation over to it. As a Pop_OS! 21.10 user, I found no
reasonable guide to help me with that so I thought I'd document what I
did to solve the problem. This guide will probably work with distros
whose setup is similar than mine.

We will not clone any partitions; instead, we will create new larger
partitions and transfer our existing files over to them.

My setup has three partitions:

- EFI boot partition
- Root partition
- Home partition

That is _not_ Pop_OS!'s default partition scheme where root and home
are a single partition. The advantage of a dedicated home partition is
that reinstalling Linux (or another distro) is more convenient because
I don't need to backup or restore my user data. Even though I rarely
reinstall my OS or switch distros, it's nice to have that kind of
flexibility.

My root and home partitions are encrypted with LUKS because encryption
is a must-have these days for kind of any professional setup.

This guide assumes that:

- Your new SSD is already installed and has already been detected by
your computer;
- You have basic computer knowledge and know how to boot from a pen
  drive;
- You have basic Linux knowledge and know how to edit files or run
  terminal commands.

I won't always be able to explain how each command works in detail,
but I encourage you to look that up if you feel curious.

## Live booting Pop_OS!

First, [download](https://pop.system76.com) and burn Pop_OS! to a USB
pen drive. Follow the Bootable USB stick instructions
[here](https://ubuntu.com/tutorials/install-ubuntu-desktop#3-create-a-bootable-usb-stick)

After burning the ISO, reboot with your pen drive and wait until the
the live Pop_OS! UI shows up.

## Creating the new partitions

We will create our partitions with [gparted](https://gparted.org/),
GNOME Partition Editor. Open it up and locate your new hard drive.
We're assuming an empty drive with no partitions.

We'll start by creating a new partition table:

- Go to "Device -> Create partition table"
- Select the "gpt" type.

Yes, we need a gpt partition table.

The next step is to create our boot EFI partition:

- Click on "Partition -> New" and then select:
  - New Size (MiB): 512
  - File system: fat32

You can preserve the default values for the other fields. Now click on
"Apply All Operations" (should be a green button).

Now let's add the EFI partition flags:

- Right click on the new EFI partition and select "Manage Flags"
- Check the "boot" and "esp" flags and then apply the changes.

Next, we should create our root and home partitions. The root
partition doesn't need to be large; My choice was 150 GB, but 50 GB is
more than enough for most people. As for the home partition, make it
large enough to store your user files. I went with 850 GB there.

For the root partition use the following setup:

 - New Size (MiB): 150000
 - File system: ext4
 
And for the home partition:

 - New Size (MiB): 850000
 - File system: ext4

Click again on "Apply all operations" and boom! There we have our new
larger partitions.

## Encrypting and configuring the new partitions

Now we will encrypt our partitions with LUKS, which means protecting
them with passwords. Replace the `/dev` paths with the actual paths to
whatever partitions you have created. Use strong passwords when asked:

```
# Root
sudo cryptsetup -v -y -c aes-xts-plain64 -s 512 -h sha512 --use-random luksFormat /dev/nvme0n1p2

# Home
sudo cryptsetup -v -y -c aes-xts-plain64 -s 512 -h sha512 --use-random luksFormat /dev/nvme0n1p3
```

Now let's open our encrypted partitions:

```sh
sudo cryptsetup luksOpen /dev/nvme0n1p2 root-luks
sudo cryptsetup luksOpen /dev/nvme0n1p3 home-luks
```

And create our filesystems:

```sh
sudo mkfs.ext4 /dev/mapper/root-luks
sudo mkfs.ext4 /dev/mapper/home-luks
```

Finally, let's _mount_ our encrypted partitions:

```sh
sudo mkdir /mnt/{root-luks,home-luks}
sudo mount /dev/mapper/root-luks /mnt/root-luks
sudo mount /dev/mapper/home-luks /mnt/home-luks
```

## Mounting the original partitions

We will need to mount our old partitions in order to copy their files
over. My old partitions were also LUKS, so I used the following
commands:

```sh
# Remember: Replace /dev/... with your actual devices...

sudo cryptsetup luksOpen /dev/nvme1n1p4 root-old
sudo cryptsetup luksOpen /dev/nvme0n1p5 home-old

sudo mkdir /mnt/{root-old,home-old}
sudo mount /dev/mapper/root-old /mnt/root-old
sudo mount /dev/mapper/home-old /mnt/home-old
```

If you did not use LUKS for your old partitions, it's all good as long
as you manage to mount them!

## Copying over the original files

Yes, it's a really simple step here! Just use `sudo cp` for both
partitions and you'll be good to go:

```sh
sudo cp -aR /mnt/root-old/* /mnt/root-luks/
sudo cp -aR /mnt/home-old/* /mnt/home-luks/
```

I love Linux's simplicity. Of course, if you have too many files this
step will take some time, so be patient.

## Updating UUID references

There are two places where we'll need to update UUID references. First
let's edit `/mnt/root-luks/etc/crypttab`. With the vim editor, we
would use the following command:

```sh
sudo vim /mnt/root-luks/etc/crypttab
```

My final `crypttab` looks like this, which you can use as a reference:

```
root-luks       UUID=b6061e44-15ad-4ae3-be69-285d09857211       none    luks
home-luks       UUID=bf0a7b89-13a5-41ef-bc7b-1154fce37c82       none    luks
```

The first column corresponds to the names of our LUKS partitions,
`root-luks` and `home-luks` - make sure they match with whatever names
you have. Now, how to get their UUIDs? That's the tricky part. Let's
get the UUID for the root drive with `blkid`:

```sh
$ sudo blkid /dev/nvme0n1p2
/dev/nvme0n1p2: UUID="b6061e44-15ad-4ae3-be69-285d09857211" TYPE="crypto_LUKS" PARTUUID="13df1c97-1464-4275-b8ed-575fc3234eae"
```

And now the UUID of the home drive:

```sh
$ sudo blkid /dev/nvme0n1p3
/dev/nvme0n1p3: UUID="bf0a7b89-13a5-41ef-bc7b-1154fce37c82" TYPE="crypto_LUKS" PARTUUID="ae5339cb-103d-4bde-93c6-e25919cfc46f"
```

Now that we are done with `crypttab`, let's edit
`/mnt/root-luks/etc/fstab`. Mine looks like this:

```sh
PARTUUID=81a3ca7f-2365-4fa9-9db9-c000ebfd5f28   /boot/efi       vfat    umask=0077      0       0
UUID=fdcf2cbe-6859-44f6-b3e9-08d8c391d321       /       ext4    noatime,errors=remount-ro       0       0
UUID=43670a9a-c14a-4bb7-b569-276bf713b840       /home   ext4    noatime,errors=remount-ro       0       0
```

We have three lines, one for the EFI partition, another for the root
partition, and another for the home partition. The only setting we'll
need to change from our original file are the UUIDs.

We can get `PARTUUID` for `/boot/efi` with:

```sh
$ sudo blkid /dev/nvme0n1p1
/dev/nvme0n1p1: UUID="C70C-7EE1" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="81a3ca7f-2365-4fa9-9db9-c000ebfd5f28"
```

For `/` and `/home` we'll need the LUKS UUIDS instead of the partition
UUIDs, right? We can figure that out with `lsblk`. For `/` the UUID
is:

```sh
$ sudo lsblk -o +uuid /dev/mapper/root-luks
NAME      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT UUID
root-luks 253:0    0 146.5G  0 crypt /          fdcf2cbe-6859-44f6-b3e9-08d8c391d321
```

Finally, let's get the UUID for `/home`:

```sh
$ sudo lsblk -o +uuid /dev/mapper/home-luks
NAME      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT UUID
home-luks 253:1    0 830.1G  0 crypt /home      43670a9a-c14a-4bb7-b569-276bf713b840
```

And we're done! Our "cloned" partitions are ready to boot!

## Installing the boot loader

For this section, I will steal the relevant steps from Pop_OS!'s
[Repair the
Boot-loader](https://support.system76.com/articles/bootloader/) guide.
Actually, I will _adapt_ them.

First let's mount our EFI partition:

```sh
sudo mount /dev/nvme0n1p1 /mnt/root-luks/boot/efi
```

Now let's use the following commands to configure systemd-boot
(Pop_OS! doesn't use grub):

```sh
for i in dev dev/pts proc sys run; do sudo mount -B /$i /mnt/root-luks/$i; done
sudo cp -n /etc/resolv.conf /mnt/root-luks/etc/
sudo chroot /mnt/root-luks
apt install --reinstall linux-image-generic linux-headers-generic
update-initramfs -c -k all
exit
sudo bootctl --path=/mnt/boot/efi install
```

When running `update-initramfs`, watch out for any warnings. If the
drive UUIDs are wrong, for example, warnings will be printed out,
which means we should go back, fix the UUIDs, and rerun
`update-initramfs`.

**EXTRA**: In my case, I also got a dozen of `amdgpu` warnings. I
recommend fixing these specific warnings if you happen to see them,
otherwise the graphical screen to input your password at boot time
won't work properly (but you would still be able to blindly type your
password, of course). Here's what I did:

```sh
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
mv /lib/firmware/amdgpu /lib/firmware/amdgpu-backup
cp -R linux-firmware/amdgpu /lib/firmware/
update-initramfs -c -k all
rm -rf ./linux-firmware /lib/firmware/amdgpu
mv /lib/firmware/amdgpu-backup /lib/firmware/amdgpu
```

That fixed the graphical screen, even though I still got three
`amdgpu` warnings! If anyone has a better solution, I'd be more than
interested in hearing about it :)

## Booting your system on your new drive

That's pretty easy, just reboot your computer! The Linux Boot Manager
should already be the default boot option. If your system looks
exactly as it was before, that's a win, congratulations!


