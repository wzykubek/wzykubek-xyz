+++
date = '2024-11-23T13:39:35+01:00'
draft = false
title = 'Installing Arch Linux with Btrfs, LUKS encryption and YubiKey Authentication'
layout = 'article'
+++

In this tutorial I will show you how to use your YubiKey with LUKS. In this case we will use our key to shorten encryption password. Person, who will get access to your machine but without key, will be prompted for very long, computer generated password. On the other hand -  _you_ - will be prompted only for the PIN for your attached YubiKey. Of course your PIN also should be as long and complicated as possible, but you should be able to memorize it.

## My setup

I will use following configuration:

+ Two partitions: one for EFI system partition, second one for root with Btrfs filesystem. This will allow us creating volumes for directories like `/home` and `/var` with no need to use [LVM](https://en.wikipedia.org/wiki/Logical_volume_management).

+ systemd-boot as a bootloader. This is only my preference, on my main desktop I use rEFInd, but systemd-boot is simpler to configure and is bundled with systemd and that is crucial for me. I perform all procedure on my Chromebook with 16 GB eMMC memory.

+ Encrypted root partition with LUKS which can be unlocked using YubiKey or any other hardware token which support FIDO2 HMAC Secret extension. You can add spare key as well. For critical situation we have also secure (128 characters) password for recovery.

## Prerequisites

+ USB stick with fresh [Arch Linux Live ISO](https://archlinux.org/download/).
Latest ISO should have all the tools we will use so you don't need to install anything additional.

+ Follow official [installation guide](https://wiki.archlinux.org/title/Installation_guide), but use this tutorial in important moments.

## Destroy disk

Wipe disk before encryption.

```bash
cryptsetup open --type plain --key-file /dev/urandom --sector-size 4096 /dev/sda wipeit
```

```bash
dd if=/dev/zero of=/dev/mapper/wipeit status=progress bs=1M
```

```bash
cryptsetup close wipeit
```

## Partitioning

Create only two partitions. One for ESP and one for filesystem. To simplify next steps I will use the following template.

| Partition | Target                 |
| :-------- | :--------------------- |
| /dev/sda1 | Boot partition         |
| /dev/sda2 | Filesystem partition   |

Format boot partition using following command.

```bash
mkfs -t fat -F32 /dev/sda1
```

Format second partition as a LUKS volume. This command will prompt you for passphrase. Generate it with your password manager (e.g. Bitwarden) using random characters. I recommend using 128 characters. You will not use it to unlock your drive, it will be only your recovery option in case you loose your YubiKey. This is probably the most painful step, because you need to type this passphrase few times but it is worth it to keep it secure.

```bash
cryptsetup -v luksFormat /dev/sda2
```

Decrypt and open volume (you need to enter password).

```bash
cryptsetup open /dev/sda2 root
```

## Adding YubiKey(s)

Connect only one key at once and run command bellow, enter passphrase and touch your key two times. It won't display anything after password, so you need to remember.

```bash
systemd-cryptenroll --fido2-device=auto --fido2-with-client-pin=true --fido2-with-user-presence=true /dev/sda2
```

If you have spare key, disconnect first one, connect second and run exact same command again.

You can verify if everything is addedd correctly using command bellow.

```bash
cryptsetup luksDump /dev/sda2
```

You should see 3 entries in Keyslots category and two in Tokens.

## Create filesystem on encrypted volume

Create filesystem.

```bash
mkfs -t btrfs /dev/mapper/root
```

Mount mapper.

```bash
mount /dev/mapper/root /mnt
```

Create Btrfs subvolumes.

```bash
btrfs subvolume create /mnt/@
```
```bash
btrfs subvolume create /mnt/@home
```
```bash
btrfs subvolume create /mnt/@var
```

Remount volumes.

```bash
umount /mnt
```
```bash
mount /dev/sda1 --mkdir /mnt/boot
```
```bash
mount /dev/mapper/root -o subvol=@ /mnt
```
```bash
mount /dev/mapper/root -o subvol=@home --mkdir /mnt/home
```
```bash
mount /dev/mapper/root -o subvol=@var --mkdir /mnt/var
```

## Initramfs

Follow next steps of the official guideline, enter chroot and stop at the moment. We need to configure and recreate initramfs image.

Edit `/etc/mkinitcpio.conf`. You need to configure `HOOKS` and `BINARIES` section to contain following items.

> Note: You will need to install `libfido2` on your new system.

```
BINARIES=(/usr/lib/libfido2.so.1)
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt filesystems fsck)
```

## Bootloader

In this tutorial I will use systemd-boot. We need to install bootloader on ESP and add some options to decrypt drive on boot.

Install bootloader.

```bash
bootctl install
```

Edit `/boot/loader/loader.conf` and add this line at the beggining:

```
default arch.conf
```

Now create `/boot/loader/entries/arch.conf` with following content. I use Linux Zen kernel but if you use stable one just remove `-zen` from entries.

```
title   Arch Linux
linux   /vmlinuz-linux-zen
initrd  /initramfs-linux-zen.img
options rd.luks.name=YOUR_UUID=root root=/dev/mapper/root rootflags=subvol=@ rw
```

Replace _YOUR_UUID_ with actual UUID of `/dev/sda2`. You can get it from `blkid /dev/sda2` command.

## Final thoughts

You can also setup [Plymouth](https://wiki.archlinux.org/title/Plymouth) to get nice and colorful password or PIN prompt instead of terminal window.


<!--### Additional links-->

<!--[https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LUKS_on_a_partition](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LUKS_on_a_partition)-->
<!--[https://wiki.archlinux.org/title/Dm-crypt/Drive_preparation#dm-crypt_wipe_on_an_empty_device_or_partition](https://wiki.archlinux.org/title/Dm-crypt/Drive_preparation#dm-crypt_wipe_on_an_empty_device_or_partition)-->
<!--[https://wiki.archlinux.org/title/Dm-crypt/System_configuration](https://wiki.archlinux.org/title/Dm-crypt/System_configuration)-->
<!--[https://0pointer.net/blog/unlocking-luks2-volumes-with-tpm2-fido2-pkcs11-security-hardware-on-systemd-248.html](https://0pointer.net/blog/unlocking-luks2-volumes-with-tpm2-fido2-pkcs11-security-hardware-on-systemd-248.html)-->
<!---->
