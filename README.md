## How to install Arch Linux
![banner](https://raw.githubusercontent.com/GoDzM4TT3O/Arch/master/banner.png)

These instructions were written by [me](https://github.com/GoDzM4TT3O), following the [Arch Wiki](https://wiki.archlinux.org), to install Arch in a VM &/or real machine. These instructions include both the installation process and the post-install process, as well as some [additions](#additions)

NOTE: if you don't like using `vim`, please replace `vim` with `nano` or an editor of your choice, like `emacs`. Keep this in mind!

I suggest also checking out [the arch wiki](https://wiki.archlinux.org/index.php/Installation_Guide). If you haven't already, download [the Arch ISO](https://www.archlinux.org/download).

### If you use a VM:
+ Open [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

+ Click "New"

+ Click "Expert Mode"
	- Name: Arch (or whatever you want to put)
	- RAM: At least 512 MB (though >1024MB is recommended if you want to use a DE like GNOME or KDE)
	- Hard disk: Create a virtual hard disk now
	
+ Click "Create"
	- File Size: At least 5 GB (though >10GB is recommended)
	- Hard disk file type: VDI
	- Storage on physical HDD: Dynamically allocated
	
+ Click "Create"
+ Select the new VM
+ Click "Settings"

+ Go into the "Storage" tab
	- Click on the CD/DVD icon
	- On the right, select the location of the Arch iso
	- Select "Live CD/DVD"
	
+ Go into the "Network " tab
	- Click on "NAT"
	- Select "Bridged adapter"
	- Choose the network interface you want to use for the Virtual Machine (for example wlp4s0 and enp3s0 are two network interfaces)
	
+ Click "Ok"
+ Start the VM

NOTE: this will use `MBR`, meaning you'll use BIOS, not UEFI! If you want to use UEFI, follow these instructions: https://www.virtualbox.org/manual/ch03.html#efi

### If you use a real machine
+ Get a hold of a DVD/USB with at least 1GB of storage
	- USB: Using a tool like [Etcher](https://balena.io), flash the ISO onto the USB
	- DVD: using a tool like [Brasero](https://wiki.gnome.org/Apps/Brasero/Documentation#Installing) on Linux or [ImgBurn](https://imgburn.com/index.php?act=download) on Windows, flash the ISO onto the DVD.
+ Put the DVD/USB in your machine
+ Open the BIOS/UEFI and [modify the boot order](https://www.pcsteps.com/1508-change-the-boot-order-usb-dvd-bios-uefi/), choosing the DVD/USB as the first boot device.

***

After starting the VM/Computer, choose "Arch Linux", then hit [ENTER]

## [Set the keyboard layout](https://wiki.archlinux.org/index.php/Installation_Guide#Set_the_keyboard_layout)

To find out available keyboard layouts, run: `ls /usr/share/kbd/keymaps/**/*.map.gz`

Once you find your keyboard layout, run: `loadkeys <keyboard-layout>`: For example, I ran `loadkeys it` because my keyboard has an italian keyboard layout.

## [Verify the boot mode](https://wiki.archlinux.org/index.php/Installation_guide#Verify_the_boot_mode)

Run this command: `ls /sys/firmware/efi/efivars`.

If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS (or CSM) mode. If the system did not boot in the mode you desired, refer to your motherboard's manual. 

## 1) [Connect to the internet](https://wiki.archlinux.org/index.php/Installation_Guide#Connect_to_the_internet)
Note: if you want to use Wi-Fi, run `wifi-menu` to connect to a wireless network.
If, for some reason, `wifi-menu` fails, run these two commands:

(If the wireless network interface isn't `wlan0`, use the one you have)

```
wpa_passphrase "My Network" "My passphrase" > wpa.conf
wpa_supplicant -i wlan0 -c wpa.conf
```

- 1.1) Check the network interfaces: `ip link`
- 1.2) If the above looks good, check if internet works: `ping archlinux.org`
+ 1.3) If you can't connect to `archlinux.org`, make sure you run: `dhcpcd <interface>`
  - 1.3.1) if you connect wirelessly, use `wlp4s0`, `wlan0` or any other wireless interface
  - 1.3.2) if you connect via ethernet, use `enp2s0`, `eth0`, or any other wired interface
- 1.4) If the above looks good, sync the clock: `timedatectl set-ntp true`

## 2) [Partition the disk](https://wiki.archlinux.org/index.php/Installation_Guide#Partition_the_disks)

Disk partitioning will be different if you use MBR or GPT:
+ On MBR systems, you'll use the BIOS, meaning you won't need the EFI System Partition.
+ On GPT systems, you'll use UEFI, meaning you will need the EFI System Partition.

NOTE: If you already have partitions (from Window$ or another Operating System), we need to delete them and convert the partition table to MBR/GPT.

First of all, remove all pre-existing file systems from SATA HDD/SSD: `wipefs -a /dev/sda` (If you use an NVMe drive, it is going to be called something like `/dev/nvme0n1`, check available disks with `fdisk -l`)

NOTE: Say you have a 250GB HDD and 4GB RAM. If you are going to be using your system graphically, it is recommended the swap space needs to be 2\*RAM (in this case 2\*4GB is 8GB SWAP), otherwise, if you're going to be using it console-only, then a swap partition the same size as the RAM (1\*RAM, 1\*4GB is 4GB SWAP) is fine. That said, we are going to be using our system graphically, so the swap will be 8GB.

If you want to use other filesystems instead of Ext4, please see https://wiki.archlinux.org/index.php/File_systems#Types_of_file_systems

+ **MBR**: Run `fdisk /dev/sda`, then type `o`, then `w`. Run `cfdisk /dev/sda`, then select `dos`. Now we need to create two partitions: the first partition will be the biggest partition, used for storage, the second partition will be the smallest partition, used for the swap.
	- First partition (Ext4): DISK SPACE - SWAP = 250GB - 8GB = 242GB
	- Second partition (SWAP): SWAP = 8GB.

+ How to partition the disk:
	- Press `New`, Partition size `242G`, Choose `primary`;
	- Select `Bootable`;
	- Press the down-arrow key, press `New`;
	- Partition size `8G`, Choose `primary`;
	- Go to `Type`, Select `Linux swap / Solaris`.
	- Press `Write`, then `y` to save changes to disk.
	
+ **GPT**:  Run `fdisk /dev/sda`, then type `g`, then `w`. Run `cfdisk /dev/sda`, then select `gpt`. Now we need to create three partitions: the first partition will be the smallest partition, used for EFI, the second partition will be the biggest partition, used for storage, the third partition will be used for the swap.
	- First partition (EFI System partition): EFI = 512M
	- Second partition (Ext4): DISK SPACE - SWAP - EFI = 250G - 8G - 512M = 242G - 512M = 242 - 0.5G = 241.5G = 241G (remove any decimal places by rounding down)
	- Third partition (SWAP): SWAP = 8G

+ How to partition the disk:
	- Press `New`, Partition size `512M`, Choose `primary`;
	- Go to type, Select `EFI System`;
	- Press the down-arrow key, press `New`;
	- Partition size `241G`, Choose `primary`;
	- Go to type, Select `Linux root (x86-64)`;
	- Press the down-arrow key, press `New`;
	- Partition size `8G`, Choose `primary`;
	- Go to type, Select `Linux swap`;
	- Press `Write`, then `y` to save changes to disk.
	
## 3) [Create the file system](https://wiki.archlinux.org/index.php/Installation_Guide#Format_the_partitions)

+ **MBR**:
	- Ext4: `mkfs.ext4 /dev/sda1`
	- SWAP: `mkswap /dev/sda2`
	
+ **GPT**:
	- EFI: `mkfs.fat -F32 /dev/sda1`
	- Ext4: `mkfs.ext4 /dev/sda2`
	- SWAP: `mkswap /dev/sda3`

## 4) [Mount the partitions](https://wiki.archlinux.org/index.php/Installation_Guide#Mount_the_file_systems)

+ **MBR**:
	- Mount the Ext4 partition: `mount /dev/sda1 /mnt`
	- Mount the SWAP partition: `swapon /dev/sda2`
	
+ **GPT**:
	- Mount the EFI System partition: `mount /dev/sda1 /mnt/boot/efi`
	- Mount the Ext4 partition: `mount /dev/sda2 /mnt`
	- Mount the SWAP partition: `swapon /dev/sda3`

## 5) [Installation: modifying the mirror list](https://wiki.archlinux.org/index.php/Installation_Guide#Installation)

+ Update `/etc/pacman.d/mirrorlist`:
	- 5.1) `vim /etc/pacman.d/mirrorlist`
	- 5.2) Select a server closest to you, for example Italy, and delete all of the lines, keeping only the server you chose. Alternatively, use the [Pacman Mirrorlist Generator](https://www.archlinux.org/mirrorlist/)
	- 5.3) Save and quit

## 6) [Install essential packages](https://wiki.archlinux.org/index.php/Installation_Guide#Install_essential_packages)

+ 6.1) `pacstrap /mnt base base-devel linux linux-firmware linux-headers man-db man-pages bash-completion`

## 7) [Configuring the system: Fstab](https://wiki.archlinux.org/index.php/Installation_Guide#Fstab)

+ 7.1) Create fstab: `genfstab -U /mnt >> /mnt/etc/fstab`

## 8) [Configuring the system: Chroot](https://wiki.archlinux.org/index.php/Installation_Guide#Chroot)

+ 8.1) Change root into the new system: `arch-chroot /mnt`
+ 8.2) Install vim to modify files: `pacman -S vim`

## 9) [Localization](https://wiki.archlinux.org/index.php/Installation_Guide#Localization)

I will be using the Italian time; to see available timezones, run `ls /usr/share/zoneinfo` and select accordingly.

+ 9.1) Timezone: `ln -sf /usr/share/zoneinfo/Europe/Rome /etc/localtime`
+ 9.2) Sync the clock: `hwclock --systohc`

+ 9.3) Set the language(s):
	- 9.3.1) Edit `/etc/locale.gen`: `vim /etc/locale.gen`
	- 9.3.2) Uncomment `en_US.UTF-8 UTF-8`, then save and quit
	- 9.3.3) Edit `/etc/locale.conf`: `vim /etc/locale.conf`
	- 9.3.4) Add `LANG=en_US.UTF-8`, then save and quit
	- 9.3.5) Generate the locale: `locale-gen`
+ 9.4) Set the keyboard layout:
	- 9.4.1) Edit `/etc/vconsole.conf`: `vim /etc/vconsole.conf`
	- 9.4.2) Write `KEYMAP=YOUR-KEYMAP`:
	
Here after `KEYMAP` you need to type your keyboard's layout, you chose this before when you ran `loadkeys <code>`. I chose `it` because I use a keyboard with an Italian keyboard layout, meaning I wrote `KEYMAP=it`. See more here: https://jlk.fjfi.cvut.cz/arch/manpages/man/vconsole.conf.5

After you're done, save and quit.

## 10) [Network Configuration: hostname](https://wiki.archlinux.org/index.php/Installation_Guide#Network_configuration)
+ 10.1) Modify `/etc/hostname`: `vim /etc/hostname`
+ 10.2) Write an hostname of your choice, like `arch`

## 11) [Network Configuration: hosts](https://wiki.archlinux.org/index.php/Installation_Guide#Network_configuration)
+ 11.1) Modify `/etc/hosts`: `vim /etc/hosts`
+ 11.2) Add the following lines:

```
# IPv4
127.0.0.1	localhost
127.0.1.1	<your hostname>.localdomain <your hostname>

# IPv6
::1		localhost
```

Where `<your hostname>` is the hostname you chose in `/etc/hostname`.

11.3) Save the file and exit

## 12) [Set the root password](https://wiki.archlinux.org/index.php/Installation_Guide#Root_password)
+ 12.1) `passwd`

## 13) [Add a new user](https://wiki.archlinux.org/index.php/Users_and_groups#Example_adding_a_user):

For example I chose `matteo` so I will replace `<user>` with `matteo`.

+ 13.1) `useradd -m <user>`
+ 13.2) Add a password to the user: `passwd <user>`
+ 13.3) [Add the user to necessary groups](https://wiki.archlinux.org/index.php/Users_and_groups#Group_list): `usermod -aG wheel,audio,video,optical,storage <user>`
+ 13.4) Check user is part of the groups we added above: `groups <user>`
+ 13.5) Add user to sudoers: `visudo`
	- 13.6) Search `# %wheel ALL=(ALL) ALL`, remove the `#` at the beginning

## 14) [Install the bootloader](https://wiki.archlinux.org/index.php/Installation_Guide#Boot_loader)

+ **MBR**:
	+ **GRUB**: `pacman -S grub os-prober freetype2 dosfstools`
		- Install GRUB to disk: `grub-install /dev/sda`
		- Configure GRUB: `grub-mkconfig -o /boot/grub/grub.cfg`
		
+ **GPT**:
	+ **GRUB**: `pacman -S grub efibootmgr os-prober freetype2 dosfstools`
		- Install GRUB to disk: `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`
		- Configure GRUB: `grub-mkconfig -o /boot/grub/grub.cfg`
	+ **rEFInd**: `pacman -S refind efibootmgr os-prober freetype2 dosfstools`
		- Install rEFInd to disk: `refind-install`
		- NOTE: I also recommend using a custom theme; I use this one: https://github.com/munlik/refind-theme-regular

For other bootloaders, see https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader

## 15) Install the network tools

+ 15.1) `pacman -S dhcpcd net-tools netctl dialog wpa_supplicant networkmanager nm-connection-editor inetutils ifplugd`
+ 15.2) Exit out of chroot: `exit`
+ 15.3) After the above is done, enable `dhcpcd`, `NetworkManager` and `ifplugd`: `systemctl enable {dhcpcd,NetworkManager,ifplugd}`

## 16) [Reboot](https://wiki.archlinux.org/index.php/Installation_Guide#Reboot)

+ 16.1) Poweroff the machine: `poweroff`
+ 16.2) Remove the Arch Live CD/DVD/US/ISO from the VM/Computer
+ 16.3) Start the VM/Computer

## 17) [Post-installation](https://wiki.archlinux.org/index.php/Installation_Guide#Post-installation)

+ 17.1) To login, type the `<user>`'s name
+ 17.2) Type the password you chose for `<user>` (it will be invisible)

If you don't want to use Arch graphically, go to section [number 20 (install necessary drivers)](#20-install-necessary-drivers)

## 18) [Install X.Org](https://wiki.archlinux.org/index.php/Xorg#Installation)

+ 18.1) Install X.Org: `sudo pacman -S xorg xorg-xinit`
+ 18.2) Select all packages by pressing [ENTER]

## 19) [Install a Desktop Environment](https://wiki.archlinux.org/index.php/Desktop_environment#List_of_desktop_environments)

NOTE: Instead of using a Desktop Environment, you can directly install a Window Manager. I use [i3](https://wiki.archlinux.org/index.php/i3). If you would like to use i3, follow this guide I wrote [here](https://godzm4tt3o.js.org/dotfiles/).

Please choose one of the following Desktop Environments:

- [Budgie](https://wiki.archlinux.org/index.php/Budgie): `sudo pacman -S gnome budgie-desktop`
- [Cinnamon (please read)](https://wiki.archlinux.org/index.php/Cinnamon#Installation): `sudo pacman -S cinnamon lightdm`
- [Deepin (please read)](https://wiki.archlinux.org/index.php/Deepin#Installation): `sudo pacman -S deepin deepin-extra networkmanager lightdm`
- [GNOME](https://wiki.archlinux.org/index.php/GNOME): `sudo pacman -S gnome gnome-extra networkmanager`
- [KDE Plasma](https://wiki.archlinux.org/index.php/KDE_Plasma): `sudo pacman -S plasma kde-applications sddm`
- [LXDE](https://wiki.archlinux.org/index.php/LXDE): `sudo pacman -S lxde lxdm`
- [LXQt](https://wiki.archlinux.org/index.php/LXQt): `sudo pacman -S lxqt breeze-icons sddm`
- [MATE](https://wiki.archlinux.org/index.php/MATE): `sudo pacman -S mate mate-extra lightdm`
- [XFCE](https://wiki.archlinux.org/index.php/Xfce): `sudo pacman -S xfce4 xfce4-goodies lightdm`

After installing Xorg, a Desktop Environment and its dependencies, enable the Display Manager (DM)

NOTE: If a Display Manager does not load, please refer to [this page](https://wiki.archlinux.org/index.php/Display_manager#Loading_the_display_manager)

  - GDM (Budgie, GNOME): `sudo systemctl enable gdm`
  - LightDM (Cinnamon, Deepin, MATE, XFCE): `sudo systemctl enable lightdm`
  - LXDM (LXDE): `sudo systemctl enable lxdm`
  - SDDM (KDE Plasma, LXQt): `sudo systemctl enable sddm`

## 20) [Install necessary drivers](https://wiki.archlinux.org/index.php/Xorg#Driver_installation)

+ AMD: `sudo pacman -S mesa xf86-input-libinput xf86-video-ati xf86-video-amdgpu vulkan-radeon amd-ucode`
+ Intel: `sudo pacman -S mesa xf86-input-libinput xf86-video-intel vulkan-intel intel-ucode`
+ NVIDIA: `sudo pacman -S mesa xf86-input-libinput nvidia xf86-video-nouveau nvidia-utils`

(NOTE: other video drivers can be found in the `xorg-drivers` group)

## 21) [Install audio sound system](https://wiki.archlinux.org/index.php/Sound_system)

`sudo pacman -S pulseaudio pulseaudio-alsa alsa-utils`

You might also want to install some codecs (audio, image & video):

`sudo pacman -S flac faac wavpack libmad opus libvorbis openjpeg libwebp x265 libde265 x264 libmpeg2 libvpx`

## Additions

The sections below contain various optional additions:
- Touchpad tap to click;
- Pacman easter egg;
- Command not found;
- Automatically enter a directory (without writing `cd`).

## 22) [Enable touchpad tap to click](https://wiki.archlinux.org/index.php/Libinput)

Since we installed `xf86-input-libinput` in [section 20](#20-install-necessary-drivers), we need to tweak the configuration.

+ 22.1) Copy the default configuration: `sudo cp /usr/share/X11/xorg.conf.d/40-libinput.conf /etc/X11/xorg.conf.d`
+ 22.2) Enable touchpad tap to click: `sudo vim /etc/X11/xorg.conf.d/40-libinput.conf`
	- 22.2.1) Find `Identifier "libinput touchpad catchall"`
	- 22.2.2) Under `Driver "libinput"` add this line: `Option "Tapping" "on"`
	- 22.2.3) Save and quit
	The touchpad config should look like this:
	```
	Section "InputClass"
		Identifier "libinput touchpad catchall"
		MatchIsTouchpad "on"
		MatchDevicePath "/dev/input/event*"
		Driver "libinput"
		Option "Tapping" "on"
	EndSection
	```

## 23) [Pacman easter egg](https://eeggs.com/items/59538.html)
+ 23.1) Modify `/etc/pacman.conf`: `sudo vim /etc/pacman.conf`
+ 23.2) Find the line that says `# Misc options`
+ 23.3) Remove the `#` before `Color`
+ 23.4) Below `#VerbosePkgLists` add this line: `ILoveCandy`

Each time you will install a new package you will see a little pacman in the progress bar.
The relevant `/etc/pacman.conf` should look like this:

```
# Misc options
#UseSysLog
Color
#TotalDownload
CheckSpace
#VerbosePkgLists
ILoveCandy
```

## 24) [Command-not-found](https://wiki.archlinux.org/index.php/Pkgfile#Installation)

On Ubuntu there's a package called `command-not-found`: its job is to tell the user the name of package to install in order to run a specific command. On Arch Linux, we can use `pkgfile`.

+ 24.1) Install pkgfile: `sudo pacman -S pkgfile`
+ 24.2) Update the pkgfile database (if necessary, run with `sudo`): `pkgfile -u`
+ 24.3) If you use `bash`, [append the line below to `.bashrc`](https://wiki.archlinux.org/index.php/Bash#Command_not_found): `vim ~/.bashrc`

`source /usr/share/doc/pkgfile/command-not-found.bash`

+ 24.4) Save and quit. Done!

## 25) [Automatically enter a directory](https://wiki.archlinux.org/index.php/Bash#Auto_%22cd%22_when_entering_just_a_path)
If you enter only the name of a directory (for example, `/etc`), cd into it.

+ 25.1) Append the line below to `.bashrc`: `vim ~/.bashrc`
(Append the line below)

`shopt -s autocd`

+ 25.2) Save and quit. Done!

***

How to manage packages on Arch Linux:
+ To upgrade Arch Linux, run: `sudo pacman -Syu`
+ To install a package, run: `sudo pacman -S <package>`
+ To remove a package, run: `sudo pacman -Rns <package>`
+ To search a package in the official repositories, run: `pacman -Ss package-name`
+ To search a package you installed, run: `pacman -Qs package-name`
+ To clear the package cache, run: `sudo pacman -Sc`

For other commands, see https://wiki.archlinux.org/index.php/Pacman/Rosetta

***

Now you installed Arch Linux, enjoy!

You can improve these instructions by [creating a new pull request](https://github.com/GoDzM4TT3O/Arch/compare), or by creating [a new issue](https://github.com/GoDzM4TT3O/Arch/issues/new/choose).
