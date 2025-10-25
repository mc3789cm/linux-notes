# What is Arch Linux?
**Arch Linux** (Arch) is one of the most lightweight and flexible distributions (distros) of Linux. It keeps itself
within a "Goldilocks" zone of complexity; not too complex where it's hard to understand beyond a vague introduction,
while also not being a total oversimplification. This lets Arch appeals to a wide range of users, from those who just
want a **desktop environment** (DE) and GUI applications, to power-users who take full control of every little detail
Arch has to offer.

Arch uses a **rolling release model**, meaning that the system is continuously updated - always up to date with latest
and shiniest software. No need to upgrade or reinstall to new versions. Once you get it installed, regular updates keep
the system current. 

While the initial installation will most likely be very minimal and raw - nothing much but a console prompt - this is
just one of beautiful aspects of Arch; you get to customize it however you like! Rather that's a lightweight XFCE
DE or a feature rich and customizable one like Hyprland, you get to choose the exact look and feel of your environment.

With that being said, *with great power comes great responsibility*. Arch assumes you know what you're doing or have at
least checked out the [Arch Wiki](https://wiki.archlinux.org/title/Main_page) page for whatever it is you're commiting
changes to, rather that's installing software or changing configurations. This encourages users to get a deeper
understanding of said software, configuration, etc.

In the end, it's up to YOU, the user, admin, and maintainer, to keep up and secure YOUR operating system. That's what
I'd call *freedom*.

# 0.0 How do We Install Arch?
In this guide, we'll avoid using an easy installer like `archinstaller`, and only be targeting relatively modern
computers with UEFI support. The installation steps and result may vary depending on your machine and/or preferences,
but make sure to follow them in the order as they appear. This guide assumes you have a working internet connection or 
have access to an Arch repository (this could even be your own).

## 0.1 Prerequisites
I'm not going to explain the basic intricacies of the Linux shell. That's its own topic and outside the scope of this
guide anyway. If you're unfamiliar with the Linux shell, read about it
[here](https://ubuntu.com/tutorials/command-line-for-beginners).

You're also going to need the following:
- An x86_64 (64-bit) compatible machine
- 20 GB (or more) of free disk space
- 2 GB (or more) of RAM
- A USB drive with at least 2 GB of disk space
- `coreutils`

## 1.0 Stage One: Preparing the Installation Medium
### 1.1 Download the ISO Image
The ISO is what contains the actual minimal Linux environment for installation, it's like a toolkit with everything
needed to bootstrap a real Arch install. Download the Arch ISO from the
[Arch Download Page](https://archlinux.org/download/). This can be done right in your browser, with `wget`, or `curl`.
Make sure to replace the URL with a real Arch mirror:

#### Wget:
```shell
wget https://an.arch.mirror/iso/yyyy-mm-dd/archlinux-yyyy-mm-dd-x86_64.iso
```

#### cURL:
```shell
curl -O https://an.arch.mirror/iso/yyyy-mm-dd/archlinux-yyyy-mm-dd-x86_64.iso
```

### 1.2 Flash the ISO Image
> [!CAUTION]
> Any and all data on the USB drive will be destroyed after running the `wipefs` and/or `dd` commands. Make sure to back
> up any data you'd like to keep.

Find the path to your installation medium:
```bash
lsblk
```

> [!TIP]
> USB drives show up as `/dev/sdX` and are marked with `1` in the RM (removable) column.

You should get an output like this:
```
NAME                                          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                                             8:0    1  14.4G  0 disk  /run/media/mc3789cm/WORKSPACE
zram0                                         252:0    0     4G  0 disk  [SWAP]
nvme0n1                                       259:0    0 465.8G  0 disk  
├─nvme0n1p1                                   259:1    0     2G  0 part  /boot
└─nvme0n1p2                                   259:2    0 463.8G  0 part  
  └─root                                      253:0    0 463.7G  0 crypt /home
                                                                         /var/cache/pacman/pkg
                                                                         /var/log
                                                                         /
```

In my case, I'll be using my USB drive at `/dev/sda` as my installation medium, but it may be completely different for
you.

> [!IMPORTANT]
> For any part in this guide where I reference my USB drive (`/dev/sda`), replace it with the path to your own. 

One quick thing before we execute the `dd` command. We need to wipe the entire drive of any signatures:
```bash
wipefs --all /dev/sda
```

The ISO image can be flashed to the installation medium by directly writing it to the medium with the `dd` command. Make
sure to replace the value in parameter "if" with the actual path to your ISO image:
```bash
dd oflag=sync bs=4M status=progress if=/path/to/archlinux.iso of=/dev/sda
```

> [!NOTE]
> The operation might take a few minutes depending on the write speeds of your installation medium.

After a successful write, you should get an output like this:
```
1518338048 bytes (1.5 GB, 1.4 GiB) copied, 292 s, 5.2 MB/s 
362+1 records in
362+1 records out
1519419392 bytes (1.5 GB, 1.4 GiB) copied, 291.815 s, 5.2 MB/s
```

### 1.3 Reboot Into the Live Environment
Now, we can finish this stage by *rebooting* into the **live environment**:
```bash
reboot
```

> [!IMPORTANT]
> The Arch Linux ISO image does not support Secure Boot. You will need to disable Secure Boot directly in your UEFI
> (BIOS) settings. Since this is firmware-specific, refer to your manufacturers manual on *how to disable Secure Boot*.

If you've done everything correctly, you should be met with a command prompt in the live environment:
```
root@archiso ~ #
```

## 2.0 Stage Two: Setting the Live Environment
### 2.1 Verify UEFI Bitness
UEFI firmware comes in two variants:
- 64-bit UEFI (x86_64)
- 32-bit UEFI (IA32)

Most modern systems use 64-bit UEFI, but in some older (or low-power) devices use 32-bit UEFI, even if the CPU alone is
64-bit capable. This guide only targets the 64-bit UEFI variant.

To verify the boot mode, check the UEFI bitness by getting the output of the *firmware platform size*:
```bash
cat /sys/firmware/efi/fw_platform_size
```

You should get an output of `64`, meaning we're using a 64-bit UEFI, which is what we want.

### 2.2 Set the Keyboard Layout and Font
List all the keyboard layouts:
```bash
localectl list-keymaps
```

Set the appropriate keyboard layout with one of the entries from the previous command:
```bash
loadkeys us
```

> [!TIP]
> The default console fonts are loaded in from `/usr/share/kbd/consolefonts`.

Optionally, set one of the default fonts:
```bash
setfont -v font-name
```

Optionally, view all the fonts glyphs with:
```bash
showconsolefont -v
```

### 2.3 Connecting to the Internet
An internet connection will be needed for when we [install your packages](#42-install-your-packages).

#### Wired Connection
If you're using a wired connection over ethernet, an internet connection should already be automatically established. If
this is the case, skip to [verifying connectivity](#verify-connectivity).

#### Wi-Fi Connection
To establish an internet connection via Wi-Fi, we can use the wireless control utility (`iwctl`). Open the 
`iwctl` interactive prompt:
```bash
iwctl
```

> [!TIP]
> You can use all commands in the `iwctl` interactive prompt as command line arguments without actually having to 
> enter the interactive prompt. For example, `iwctl device wlan0 show`.

Find the device you'd like to use:
```iwctl
device list
```

The output should look something like this:
```
                                    Devices                                    
--------------------------------------------------------------------------------
  Name                  Address               Powered     Adapter     Mode      
--------------------------------------------------------------------------------
  wlan0                 f4:26:79:61:0d:8c     on          phy0        station     
```

In my case, I'll be using my network device `wlan0`. Although `wlan0` is common, it may be completely different for you.

> [!IMPORTANT]
> For any part in this section where I reference my network device (`wlan0`), replace it with your own. 

Initiate a network scan and list the networks in your area:
```iwctl
station wlan0 scan

station wlan0 get-networks
```

The output should look something like this:
```
                               Available networks                              *
--------------------------------------------------------------------------------
      Network name                      Security            Signal
--------------------------------------------------------------------------------
  >   My Network                        psk                 ****
      NetworkOne                        psk                 *** 
      NetworkTwo                        psk                 **  
      NetworkThree                      psk                 *   
```

In my case, I'll be using the network `My Network`, but it may be completely different for you.

Connect to your chosen network and enter its passphrase when prompted:
```iwctl
station wlan0 connect 'My Network'
```

Finally, exit the `iwctl` interactive prompt:
```iwctl
exit
```

#### Verify Connectivity
Verify your internet connection with `ping`:
```bash
ping -c 3 ping.archlinux.org
```

If you're connected, you should be getting an output like this:
```
PING ping.archlinux.org (2a01:4f9:c010:2636::1) 56 data bytes
64 bytes from redirect.archlinux.org (2a01:4f9:c010:2636::1): icmp_seq=1 ttl=46 time=142 ms
64 bytes from redirect.archlinux.org (2a01:4f9:c010:2636::1): icmp_seq=2 ttl=46 time=166 ms
64 bytes from redirect.archlinux.org (2a01:4f9:c010:2636::1): icmp_seq=3 ttl=46 time=188 ms

--- ping.archlinux.org ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 126.949/171.336/233.469/37.356 ms
```

Let's quickly update the system clock before we move onto the next stage:
```bash
timedatectl
```

## 3.0 Stage Three: Provisioning the Disk(s)
### 3.1 Partition the Disk(s)
Run `lsblk` again to see what storage mediums you want to use. In my case, I'm just using my NVME drive at
`/dev/nvme0n1`, however yours could be completely different.

The disk(s) you partition will be the disk(s) you're going to install the final system to. Take utmost care and
precision during this stage.

Use the `fdisk` tool with the path to the drive you're going to use.

> [!TIP]
> Make sure to use the path to the entire disk (e.g. `/dev/sda`). Not just a partition on that disk (e.g. `/dev/sda1`).
```bash
fdisk /dev/nvme0n1
```

You should be met with the `fdisk` interactive prompt:
```fdisk
Welcome to fdisk (util-linux 2.41.2).                                                                                                                                                                                            
Changes will remain in memory only, until you decide to write them.                                                                                                                                                              
Be careful before using the write command.

Command (m for help):
```

You can print the whole [partition table](./partitioning.md#what-are-partitioning-schemes) at any time by entering `p`:
```fdisk
Command (m for help): p

Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: TEAM TM8FP6512G
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 634F7288-F737-4B12-9728-0AB9C569B847

Device             Start       End   Sectors  Size Type
/dev/nvme0n1p1      2048  73402367  73400320   35G Linux swap
/dev/nvme0n1p2  73402368 104859647  31457280   15G EFI System
/dev/nvme0n1p3 104859648 943720447 838860800  400G Linux filesystem
```

Create a new [GPT](./partitioning.md#gpt-guid-partition-table) partition table by entering `g`:
```fdisk
Command (m for help): g
Created a new GPT disklabel (GUID: E3E23A98-81B1-46B4-84F6-6B709924CC6E).
```

Before we create any partitions, consider what your storage needs are.
- What's your use case?
  - Desktop/laptop?
  - Server?
  - Dual-booting?
- How much swap do you need?
- Are you going to be using multiple/custom kernels?
- Is disk encryption important to you?

Basically, try to make a well-structured partition schema that's relevant for its *"why?"* case.

The process is very simple and similar each time you want to create a partition.
1. Select its partition number
2. Define the start (first sector) of the partition
3. Define the size (last sector) of the partition
4. Set the partitions type

I'll demonstrate creating a swap area.

Create a partition by entering `n`:
```fdisk
Command (m for help): n
```

Select the partition number:
```fdisk
Partition number (1-128, default 1): 2
```

Define the first sector of the partition:
```fdisk
First sector (2048-1000215182, default 2048): 2048
```

Define the size of the partition:
```fdisk
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1000215182, default 1000214527): +35G

Created a new partition 1 of type 'Linux filesystem' and of size 35 GiB.
```

Set the respective type for the partition by entering `t` and specifying the alias to the partiton type:
```fdisk
Command (m for help): t
Partition type or alias (type L to list all): 19
Changed type of partition 'Linux filesystem' to 'Linux Swap'.
```

Repeat this with all the partitions until you have your desired schema.

### 3.2 Format the Partitions
Once the partitions have been created, each partition needs to be formatted with their respective
[file system](./file-systems.md). I'll only cover two file systems and swap, but you may use other file system creation
tools like `mkfs.btrfs` or `mkfs.xfs`. The file system(s) you choose shouldn't impact any of the next steps.

Format the necessary EFI boot partition:
```bash
mkfs.fat -F 32 -n LABEL /dev/nvme0n1p1
```

Format any other partition you'll be using with your desired file system. For example with an EXT4 file system:
```bash
mkfs.ext4 -L LABEL /dev/nvme0n1p3
```

Lastly, set up a swap area:
```bash
mkswap -L LABEL /dev/nvme0n1p2
```

### 3.3 Mount the File Systems
Mount the partition you'll be using as `/` (root) **first** to `/mnt` before we mount any other partiton:
```bash
mount /dev/nvme0n1p3 /mnt
```

Mount the EFI boot partition:
```bash
mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

Enable the swap area with `swapon`:
```bash
swapon /dev/nvme0n1p2
```

And, any other additional partitions should be mounted now.

### 3.4 Generate the File System Table
The `/etc/fstab` (file system table) file is needed to mount our file systems at boot time. Generate an `fstab` file
with `genfstab`.
> [!TIP]
> Use `-U` or `-L` to define weather entries use UUID or labels, respectively. UUID's are more robust and guarantee
> uniqueness between file systems, while labels are more human-readable that can be given meaningful names with/for
> context.
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## 4.0 Stage Four: Installing to the Disk(s)
### 4.1 Consider Your Packages
Before we actually install to the disk(s) (`/mnt`), consider the essential packages you're going to need. Don't worry
about installing any higher-level applications like `steam`, `spotify`, and `firefox`. These should be installed after
we [boot into the system](#60-stage-six-booting-the-new-system). But since there isn't really anything stopping you, you
totally could install whatever you want.

#### Required:
> [!NOTE]
> This guide only covers installation for the GRUB bootloader. Check the Arch wiki for your selected bootloader's own
> install instructions.
- `base`
- A Linux kernel of your choice (Pick one)
  - `linux`
  - `linux-hardened`
  - `linux-lts`
  - `linux-rt`/`linux-rt-lts`
  - `linux-zen`
  - Or even build your own
- `linux-firmware`
- `efibootmgr`
- A bootloader (Pick one)
  - `grub`
  - `limine`
  - `lilo`
- An in-terminal text editor
  - `vi`/`vim`
  - `nano`
  - `e3`

#### Recommended:
- `sudo`
- CPU microcode updates (Pick one)
  - `amd-ucode`
  - `intel-ucode`
- Networking (Pick one)
  - `networkmanager`
  - `iwd`
  - `netctl`
- Userspace file system utilities
  - `btrfs-progs`
  - `dosfstools`
  - `exfat-utils`
  - `e2fsprogs`
  - `xfsprogs`

### 4.2 Install Your Packages
Install your considered packages to the installation target with `pacstrap`:
```bash
pacstrap -K /mnt base linux linux-firmware efibootmgr grub networkmanager sudo amd-ucode e2fsprogs nano
```

### 4.3 Change Root into the new System
To directly connect with the new system's environment, commands, and configuration as if we were booted into it,
"chroot" (change root) into the new system for the next stage:
```bash
arch-chroot /mnt
```

## 5.0 Stage Five: Configuring the Installed System
### 5.1 Set the System Time
List all available time zones:
```bash
timedatectl list-timezones
```

Set the time zone for showing the correct local time and handle Daylight Savings Time:
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime # Use your own timezone.
```

The `/etc/adjtime` file stores historical information and settings for the system's RTC. This is used to adjust for the
hardware clocks predictable drift, ensuring the system clock is correctly set on boot. Generate this file:
```bash
hwclock --systohc
```

> [!TIP]
> You can get the date or a calendar at any time with the `date` and `cal` commands.

### 5.2 Localization
Locales are used by the GNU C Library (glibc) and other locale-aware libraries for rendering numbers, dates, text, etc.
for your location and language preferences.

To use your region's respective language formatting, open the file at `/etc/locale.gen` with the text editor you
installed, and uncomment (remove the `#`) the UTF-8 locales you're going to use. Then, generate that locale:
```bash
locale-gen
```

Create the `/etc/locale.conf` file, and set the `LANG` variable accordingly to the locale you generated:
```/etc/locale.conf
LANG=en_US.UTF-8
```

Create the `/etc/vconsole.conf` file, and set the `KEYMAP` variable accordingly to the keyboard layout you set during
[setting the keyboard layout and font](#22-set-the-keyboard-layout-and-font).
```/etc/vconsole.conf
KEYMAP=us
```

### 5.3 Set the Hostname
The hostname acts as a unique identifier for the system, it's necessary for networking, logging, and administration.
Create the `/etc/hostname` file, and enter only your hostname
> [!TIP]
> A hostname must use only ASCII alphanumeric characters and the hyphen (`-`), with a minimum of 1 and maximum of 63 
> characters in the whole hostname. The hostname must not start with `-`.
```/etc/hostname
my-hostname
```

### 5.4 Create the initramfs (Initial RAM File System)
The initramfs (initial RAM file system) is an in-memory file system that the kernel uses to prepare the environment 
before accessing the real root file system. Create a new initramfs:
```bash
mkinitcpio -P
```

### 5.5 Set the Root Password
To use the `root` user for administrative actions requiring it, set the password for it, following the prompts as given:
```bash
passwd
```

### 5.6 Installing the Bootloader
The bootloader is the intermediate stage between firmware initialization and loading the Linux kernel. A very important
piece in the Linux boot puzzle. Install GRUB:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Generate the main configuration file `/boot/grub/grub.cfg`:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 6.0 Stage Six: Booting the New System
We're basically done. Congratulations, you've installed Arch. Exit the chrooted environment:
```bash
exit 0
```

Unmount all partitions:
```bash
umount -R /mnt
```

Finally, boot into your new system:
```bash
reboot
```

Make sure to remove the installation medium and login to the new system with the root account you created.

# Sources
- [Arch Wiki - Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [Arch Manual Pages - Hostname](https://man.archlinux.org/man/hostname.5)
- [Wikipedia - Arch Linux](https://en.wikipedia.org/wiki/Arch_Linux)
- [Linux Foundation - FHS 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/index.html)
- [Linux Kernel - InitRAMFS](https://www.kernel.org/doc/html/latest/filesystems/ramfs-rootfs-initramfs.html)

# Final Notes
Skip this section. This is just boring/informal/silly stuff.

I do plan on making the language, terms, and perspective used here more consistent and clear in the near future. This
really is still rough as I really just wanted to finish this and move on with something else for a bit because this 
is like 3000+ words by now and my hands are going to explode.
```
wc arch-install-guide.md -w
3215 arch-install-guide.md
```
At some point, I'll be expanding this guide into supporting more systems (i.e. 32-bit), explaining some steps 
better/further, and making dedicated pages for installing all kinds of different software, like other bootloaders 
outside of GRUB, custom kernels, etc. Expect a *Post Arch Linux Installation Guide* after I put a platinum finish on
this one.

My rationale behind using the "Unlicense" license is that I feel the access of raw facts/principles shouldn't 
have a price or owner - being freely available. I get why there's a cost in having it formatted, explained, taught, and
testing one's knowledge on it for something like a degree. But just the raw unencumbered information itself? That
shouldn't have a commercial value, or be truly "owned" by anyone. So, feel free to copy and claim as much of my works.

Don't confuse this ideal with creative works like software and fiction. Although I don't think there should be a price
on these, they *should* have credit given where it's due and be used as the creator(s) intended.

These are but mere opinions of mine. If you get offended, you should be offended. In that case, here, take some more 
offence:

"A computer is like air conditioning - it becomes useless when you open Windows" - Linus Torvalds