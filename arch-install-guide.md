# What is Arch Linux?
**Arch Linux** (Arch) is one of the most lightweight and flexible distributions (distros) of Linux. It keeps itself
within a Goldilocks zone of complexity; not too complex where it's hard to understand beyond a vague understanding,
while also not being a total oversimplification. This lets Arch atone to a large range of users, from those who just
want a**desktop environment** (DE) and GUI applications to power-users who take full control of every little detail
Arch has to offer.

Arch uses a **rolling release model**, meaning that the system is continuously updated - always up to date with latest
and shiniest software. No need to upgrade or reinstall to new versions. Once you get it installed, regular updates keep
the system current.

While the initial installation will most likely be very minimal and raw - nothing much but a console prompt, this is
just one of beautiful aspects of Arch; you get to customize it however you like! Rather that's a lightweight XFCE
DE or a feature rich and customizable one like Wayland.

With that being said, *with great power comes great responsibility*. Arch assumes you know what you're doing or have at
least checked out the [Arch Wiki](https://wiki.archlinux.org/title/Main_page) page for whatever it is you're commiting
changes to, rather that's installing software or changing configurations. This encourages users to get a deeper
understanding of said software, configuration, ect.

In the end, it's up to YOU, the user, admin, and maintainer, to keep up and secure YOUR operating system.

# How do We Install Arch?
In this guide, we'll avoid using an easy installer like `archinstaller`, and only be targeting a relatively modern PC 
with UEFI support. The installation steps and result may vairy depending on your machine and/or preferences. This guide
assumes you have a working internet connection or have access to an Arch repository (this could even be your own).

## Prerequisites
### Required:
- An x86_64 (64-bit) compatible machine
- A USB drive with at least 2 GB of disk space
- 512 MB of RAM
- 2 GB of free disk space
- `coreutils`

### Recommended:
- 2 GB of RAM
- 20 GB of free disk space
- A basic understanding of how to use the Linux shell

## Stage One: Preparing the Installation Medium
### Download the ISO Image
Download the Arch ISO from the [Arch Download Page](https://archlinux.org/download/). This can be done right in your
browser or with one of the following methods. Make sure to replace the URL with a real Arch mirror:

#### Method One:
```shell
wget https://an.arch.mirror/iso/yyyy-mm-dd/archlinux-yyyy-mm-dd-x86_64.iso
```

#### Method Two:
```shell
curl -O https://an.arch.mirror/iso/yyyy-mm-dd/archlinux-yyyy-mm-dd-x86_64.iso
```

### Flash the ISO Image
> [!CAUTION]
> Any and all data on the USB drive will be destroyed after running the `wipefs` and `dd` commands. Make sure to back up
> any data you'd like to keep.

Find the path to your installation medium with `lsblk`:
```bash
lsblk
```

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
> For any part in this guide where I reference my USB drive (`/dev/sda`), replace it with the path to your own. This 
> includes partitions on the drive as well.

One quick thing before we execute the `dd` command, we need to wipe the entire drive of any signatures:
```bash
wipefs --all /dev/sda
```

The ISO image can be flashed to the installation medium by directly writing it to the medium with the `dd` command. Make
sure to replace the value in parameter "if" with the path to your ISO image:
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

### Reboot Into the Live Environment
Now, we can finish this stage by rebooting into the **live environment**.
```bash
reboot
```

> [!IMPORTANT]
> The Arch Linux ISO image does not support Secure Boot. You will need to disable Secure Boot directly in your
> UEFI (BIOS) settings. Since this is firmware-specific, refer to your manufacturers manual on *how to disable Secure
> Boot*.

If you've done everything correctly, you should be met with a command prompt.
```
root@archiso ~ #
```

## Stage Two: Setting the Live Environment
### Verify UEFI Bitness
To verify the boot mode, check the UEFI bitness by getting the output of the *firmware platform size*:
```bash
cat /sys/firmware/efi/fw_platform_size
```

You should get an output of `64`, meaning we're using a 64-bit UEFI, which is what we want.

### Set the Keyboard Layout and Font
List all the keyboard layouts with `localetl`:
```bash
localectl list-keymaps
```

Set the appropriate keyboard layout with one of the entries from the previous command with `loadkeys`:
```bash
loadkeys us
```

> [!TIP]
> The default console fonts are loaded in from `/usr/share/kbd/*/`.

Set one of the default fonts:
```bash
setfont -v ter
```

Optionally, you can view all the fonts glyphs with `showconsolefont`:
```bash
showconsolefont -v
```

### Connecting to the Internet
#### Wired Connection
If you're using a wired connection over ethernet, an internet connection should already be automatically established. If
this is the case, verify your connection with the `ping` command:
```bash
ping ping.archlinux.org
```

#### Wi-Fi Connection
To establish an internet connection via Wi-Fi, we can use the wireless control utility (`iwctl`). Open the 
`iwctl` interactive prompt:

> [!TIP]
> You can use all commands in the `iwctl` interactive prompt as command line arguments without actually having to 
> enter the interactive prompt. For example, `iwctl device wlan0 show`.

```bash
iwctl
```

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

In my case, I'll be using my network device `wlan0`, but it may be completely different for you.

> [!IMPORTANT]
> For any part in this section where I reference my network device (`wlano`), replace it with your own. 

Initiate a network scan and list the networks in your area:
```iwctl
station wlan0 scan

station wlan0 get-networks
```

The output should look something like this:
```
                               Available networks                             *
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

## Stage Three: Provisioning the Disks
Unfinished...

## Stage Four: Installation
Unfinished...

## Stage Five: Configuration
Unfinished...
