---
layout: post
title: Moving a Linux Installation to a New Encrypted Hard Drive [en]
description: No special tools required. Just use Linux.
tags: [linux]
comments: true
---

I've recently acquired a larger SSD to give my Linux installation a
new home with more space, and also to have plenty of space for my
Windows setup. As a Pop_OS! 21.10 user, I found no specific resource
to help me out, so I thought I'd document the steps I went through
here. With a few adjustments, this guide should also work for other
distros but I provide no guarantees.

Instead of cloning partitions with the same sizes and preserving other
characteristics, we will create _larger_ partitions and copy our
existing files over to them.

My setup has three pieces:

- EFI boot partition
- Root partition
- Home partition

Of course, that does not correspond to Pop_OS!'s defaults where root
and home are a single partition. The advantage of a dedicated home
partition is that reinstalling Linux is more convenient because I
don't need to necessarily backup and restore my user data. Even though
I rarely reinstall or switch distros, it's nice to have that
possibility if a need arises.

Also, my root and home partitions are encrypted with LUKS because
encryption is a must-have these days for any kind of professional
setup.

This guide assumes that:

- Your new SSD is installed and detected by your computer;
- You know how to boot from a pen drive;
- You have basic Linux knowledge and know how to edit files and run
  terminal commands.

I won't always be able to explain in detail how each command works,
but I encourage you to look it up if you feel curious.

## Live booting Pop_OS!

First, [download](https://pop.system76.com) and burn Pop_OS! to a pen
drive. Follow the [Bootable USB stick
instructions](https://ubuntu.com/tutorials/install-ubuntu-desktop#3-create-a-bootable-usb-stick).

After burning the ISO, reboot with your pen drive and wait until the
the live Pop_OS! UI appears.

## Creating the new partitions

The next step is to create our new partitions in our new hard drive
with [gparted](https://gparted.org/), GNOME's Partition Editor. Launch
gparted and select the new HD on the left sidebar. Assuming an empty
drive with no partitions, start by creating a new partition table:

- Go to "Device -> Create partition table"
- Select the "gpt" type.

Yes, in a modern computer we need a gpt partition table.

Next we should create our boot EFI partition:

- Click on "Partition -> New" and then select:
  - New Size (MiB): 512
  - File system: fat32

We can preserve the default values for the other fields. Now click on
"Apply All Operations", which should be a green button.

Now let's add the EFI partition flags:

- Right click on the new EFI partition and select "Manage Flags"
- Check the "boot" and "esp" flags and then apply the changes.

Next, we should create our root and home partitions. The root
partition doesn't need to be large; My choice was 150 GB, but 50 GB is
more than enough for most people. As for the home partition, make it
large enough to store your user files. I went with 850 GB.

For the root partition use the following setup:

 - New Size (MiB): 150000
 - File system: ext4
 
And for the home partition:

 - New Size (MiB): 850000
 - File system: ext4

Click again on "Apply all operations" and boom! There we have our new
larger partitions!

## Encrypting, configuring, and mounting the new partitions

Now we will encrypt our partitions with LUKS. Replace the `/dev` paths
with the actual paths to whatever partitions you've created.
Use strong passwords when asked:

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

Finally, let's mount our encrypted partitions:

```sh
sudo mkdir /mnt/{root-luks,home-luks}
sudo mount /dev/mapper/root-luks /mnt/root-luks
sudo mount /dev/mapper/home-luks /mnt/home-luks
```

## Mounting the original partitions

We will need to mount our original partitions in order to copy their
files over. Mine were encrypted with LUKS, so I used the following
commands:

```sh
# Remember: Replace /dev/... with your actual devices...

sudo cryptsetup luksOpen /dev/nvme1n1p4 root-old
sudo cryptsetup luksOpen /dev/nvme0n1p5 home-old

sudo mkdir /mnt/{root-old,home-old}
sudo mount /dev/mapper/root-old /mnt/root-old
sudo mount /dev/mapper/home-old /mnt/home-old
```

If your old partitions are not encrypted, it's all good as long as you
manage to mount them in order to make their files accessible for the
next step.

## Copying over the original files

Yes, it's really a simple step here! Just use `sudo cp` and you'll be
good to go:

```sh
sudo cp -aR /mnt/root-old/* /mnt/root-luks/
sudo cp -aR /mnt/home-old/* /mnt/home-luks/
```

I love Linux's simplicity. Of course, if you have too many bytes to
transfer this step will take a while, so be patient.

## Updating partition UUID references

There are two places where we'll need to update UUID references. First
let's edit `/mnt/root-luks/etc/crypttab` with vim or whatever editor you
fancy:

```sh
sudo vim /mnt/root-luks/etc/crypttab
```

My final `crypttab` looks like this, so use it as a reference:

```
root-luks       UUID=b6061e44-15ad-4ae3-be69-285d09857211       none    luks
home-luks       UUID=bf0a7b89-13a5-41ef-bc7b-1154fce37c82       none    luks
```

The first column corresponds to the names of our LUKS partitions,
`root-luks` and `home-luks`. Make sure that it matches whatever names
you have on your side. Now, how did we get to these UUIDs? That's the
tricky part. We can get the UUID of the root partition with `blkid`:

```sh
$ sudo blkid /dev/nvme0n1p2
/dev/nvme0n1p2: UUID="b6061e44-15ad-4ae3-be69-285d09857211" TYPE="crypto_LUKS" PARTUUID="13df1c97-1464-4275-b8ed-575fc3234eae"
```

And similarly for the home partition:

```sh
$ sudo blkid /dev/nvme0n1p3
/dev/nvme0n1p3: UUID="bf0a7b89-13a5-41ef-bc7b-1154fce37c82" TYPE="crypto_LUKS" PARTUUID="ae5339cb-103d-4bde-93c6-e25919cfc46f"
```

Now that we are done with `crypttab`, let's edit
`/mnt/root-luks/etc/fstab`. For reference, my final `fstab` looks like
this:

```sh
PARTUUID=81a3ca7f-2365-4fa9-9db9-c000ebfd5f28   /boot/efi       vfat    umask=0077      0       0
UUID=fdcf2cbe-6859-44f6-b3e9-08d8c391d321       /       ext4    noatime,errors=remount-ro       0       0
UUID=43670a9a-c14a-4bb7-b569-276bf713b840       /home   ext4    noatime,errors=remount-ro       0       0
```

Above we have three lines, corresponding respectively to the EFI,
root, and home partitions. The only setting we'll need to change from
our original `fstab` are the UUIDs.

We can get `PARTUUID` for `/boot/efi` with:

```sh
$ sudo blkid /dev/nvme0n1p1
/dev/nvme0n1p1: UUID="C70C-7EE1" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="81a3ca7f-2365-4fa9-9db9-c000ebfd5f28"
```

For `/` and `/home` we'll need the LUKS UUIDS instead of the partition
UUIDs (`PARTUUID`). For EFI we used `PARTUUID` because it is not an
encrypted partition, so the actual partition reference is enough to
solve the problem. Thankfully, we can get the LUKS UUIDs with `lsblk`.

For `/` the LUKS UUID is:

```sh
$ sudo lsblk -o +uuid /dev/mapper/root-luks
NAME      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT UUID
root-luks 253:0    0 146.5G  0 crypt /          fdcf2cbe-6859-44f6-b3e9-08d8c391d321
```

And for `/home`:

```sh
$ sudo lsblk -o +uuid /dev/mapper/home-luks
NAME      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT UUID
home-luks 253:1    0 830.1G  0 crypt /home      43670a9a-c14a-4bb7-b569-276bf713b840
```

Done! Our "cloned" partitions are _almost_ ready to boot!

## Installing the boot loader

For this section, I will steal the relevant steps from Pop_OS!'s
[Repair the
Boot-loader](https://support.system76.com/articles/bootloader/) guide.
Actually, I will adapt them.

First let's mount our EFI partition:

```sh
sudo mount /dev/nvme0n1p1 /mnt/root-luks/boot/efi
```

Now let's use the following commands to configure systemd-boot.
Pop_OS! doesn't use grub, but if you do you can [follow their official
guide]((https://support.system76.com/articles/bootloader/#grub)) to
make it work .

```sh
for i in dev dev/pts proc sys run; do sudo mount -B /$i /mnt/root-luks/$i; done
sudo cp -n /etc/resolv.conf /mnt/root-luks/etc/
sudo chroot /mnt/root-luks
apt install --reinstall linux-image-generic linux-headers-generic
update-initramfs -c -k all
exit
sudo bootctl --path=/mnt/boot/efi install
```

While running `update-initramfs`, watch out for any warnings. If the
UUIDs happen to be wrong, warnings will be printed out, which is a
sign we should go back, fix the UUIDs, and rerun `update-initramfs`.

**EXTRA**: If you use the `amdgpu` drive this will be relevant for you.
In my case, I also got a dozen `amdgpu` warnings with `update-initramfs`. I
recommend fixing those specific warnings if you happen to see them,
otherwise the graphical screen to input your password at boot time
won't work properly (even though you'll still be able to blindly type
your password or press one of the arrow keys to reveal a pure text UI
where you'll be able to type your password). Here's what I did:

```sh
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
mv /lib/firmware/amdgpu /lib/firmware/amdgpu-backup
cp -R linux-firmware/amdgpu /lib/firmware/
update-initramfs -c -k all
rm -rf ./linux-firmware /lib/firmware/amdgpu
mv /lib/firmware/amdgpu-backup /lib/firmware/amdgpu
```

I still got three `amdgpu` warnings after that but they were
harmless - in other words, that was enough to get the password UI
working! If anyone knows a better solution, I'd be more than
interested in hearing about it :)

## Booting your system on your new hard drive

That's pretty easy, just reboot your computer! The Linux Boot Manager
should already be the default boot partition. If you're not happy with
that choice, go to your BIOS boot manager and change the default boot
partition (`F8` in my machine).

If you face any errors while booting Linux, you may have gotten some
step wrong. On the other hand, if your system looks exactly as it was
before, that's a win, congratulations! If you're keeping your old hard
drive, now you can wipe it off and enjoy even more space than you had
before!

P.S.: If you are wondering how I dual-boot between Linux and Windows,
I respect Pop_OS! defaults and don't use grub at all. Rather, I press
`F8` at boot time to launch my BIOS boot manager!
