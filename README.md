# Arch Install Guide
An installation guide for arch
- [Part-0: Preparing and creating the installation media](#part-0-preparing-and-creating-the-installation-media)
- [Part-1: Connect to the internet](#part-1-connect-to-the-internet)
- [Part-2: Partition the disks](#part-2-partition-the-disks)
- [Part-3: Mounting the partitions and preparing the base arch install](#part-3-mounting-the-partitions-and-preparing-the-base-arch-install)
- [Part-4: Setting up important system components](#part-4-setting-up-important-system-components)
- [Part-5: Setting up users and passwords](#part-5-setting-up-users-and-passwords)
- [Part-6: Finishing the install](#part-6-finishing-the-install)
- [Part-7: Installing a desktop environment or window manager](#part-7-installing-a-desktop-environment-or-window-manager)
- [Part-8: Installing nvidia drivers](#part-8-installing-nvidia-drivers)

# Important Notices
- This guide only covers installing arch in EFI/UEFI mode (valid for most modern systems)
  - If you want to install arch in compatibility BIOS mode, refer to [this part](https://wiki.archlinux.org/title/Installation_guide#Example_layouts) in the official guide along with this one
- For a minimal install follow instructions up to and including Part-6
- You should probably look up a tutorial on vim if you don't know how to use it

# Part-0: Preparing and creating the installation media
- Download the arch iso from the [Official Arch Linux Website](https://archlinux.org/download/)
- Flash that iso to a USB drive or disk 
- The easiest way to do it is by using [Balena Etcher](https://www.balena.io/etcher/)
  - Choose the arch iso and the device you want to flash it to, and hit the `flash` button
- After you're done, boot from that device (I won't include instructions on how to do that, since getting into a computer's BIOS and choosing a bootable device differs from system to system; just search how to boot from a USB or CD/DVD on your machine)
- When booting from the installation media you will be presented with a menu. Just choose the first option, and if it doesn't work, force a reboot and choose another

# Part-1: Connect to the internet
- If you have a wired connection, skip to the last step in this part
- The arch installer has a built-in tool for using a wireless connection. To use it, run `iwctl`
- Now it's time to find what wireless modules you have in your PC/laptop. Run `station list` to print out a list of "stations" (the wifi modules in your computer)
- Now run `station STATION_NAME get-networks` to get a list of available networks nearby
- Connect to one of the networks by running `station STATION_NAME connect NETWORK_NAME`
- You can now exit from iwctl by typing `exit`, or by pressing `ctrl + d`
- To ensure that the system clock is accurate run `timedatectl set-ntp true`

# Part-2: Partition the disks
- To get a list of disks in your system run `lsblk`
- CAREFULY choose the disk you want to format
  - The disk should have 20G or more (30G is recommended)
  - This guide does not cover installing Arch alongside another OS, so you will be formating the whole disk
  - Remember that everything on that disk will be lost, so back up any important files before moving on
- Run `cfdisk /dev/DISK_NAME` to start creating the partitions
  - If you are presented with a choice between `DOS` or `GPT` choose `GPT`
- Delete any existing partitions
- First you will neet to create a new partition of 512M
  - Set its type to `EFI system partition`
  - This partition will be used for the bootloader (in our case grub), which is the thing that tells the bios where to boot from
- Choose wether you want/need swap
  - Any swap partition should at be at leas 4G
  - If you have less than 8G of RAM it's recommended (although not mandatory) to create a swap partition
  - If you have less than 1G of RAM, swap is almost mandatory if you want your system to be stable
  - If you want to use the `Hibernate` feature, the swap partition should be at least AMMOUNT_OF_RAM + 2G
- If you created a swap partition, set its type to `Linux swap`
- Use the remainder of the disk to create a `Linux filesystem` partition
- Write the changes to disk, then quit from cfdisk
- Running `lsblk` again, you should see the partitions you just created
  - If you don't see your changes, run the previous command again, and make sure you didn't forget to `Write` the changes before quiting from cfdisk
- Create a `fat32` filesystem on the `EFI` partition, by running `mkfs.fat -F32 /dev/EFI_PARTITION_NAME`
- If you made a swap partition, run the following two commands:
  - `mkswap /dev/SWAP_PARTITION_NAME`
  - `swapon /dev/SWAP_PARTITION_NAME`
- Now to choose filesystem type
  - On almost all modern system you can use `btrfs` which is generally faster and more space efficient than the alternative
  - On older systems (more than 15 years old), or in virtual machines it's recommended to use `ext4`
  - Run `mkfs.FILESYSTEM_TYPE /dev/LINUX_SYSTEM_PARTITION_NAME` (replace `FILESYSTEM_TYPE` with `btrfs` and `ext4` respectively)

# Part-3: Mounting the partitions and preparing the base arch install
- The partitions that you created need to be mounted in in order to install arch on them
- There is technically no "one good way" to do this, but the unspoken standard is to mount everything to `/mnt`
  - First, run `mount /dev/LINUX_SYSTEM_PARTITION_NAME /mnt` to mount the root partition (KEEP THE SPACE BEFORE `"/mnt"`)
  - Then, run `mount --mkdir /dev/EFI_PARTITION_NAME /mnt/boot/efi` to mount the boot partition
- To install arch, we first need to install the linux kernel (which is the thing that runs our applications) by running the following command
  - `pacstrap /mnt base base-devel linux linux-firmware vim git grub efibootmgr networkmanager`
  - You might notice that I also included `vim` and `git`, which are not part of arch, but they are used later in this guide, so leave them there
- Now you need to generate `fstab`, which Arch uses to know what partitions to mount at boot
  - If you want to add an additional partition, you should mount it now to `/mnt/mnt/NAME` (replace `NAME` with the name that you want to see in your file manager)
  - Do that by running `mount --mkdir /dev/OTHER_PARTITION_NAME /mnt/mnt/NAME`
  - Yes, `/mnt/mnt` is the corect mountpoint because `/mnt` is the temporary root partition, so `/mnt/mnt` will just be `/mnt` after you are done installing arch and you rebooted your computer
  - To generate `fstab` run `genfstab -U /mnt >> /mnt/etc/fstab`
  - You can check that `fstab` has been [generated correctly](https://wiki.archlinux.org/title/Fstab) by running `vim /mnt/etc/fstab`
- Finally, after mounting the partition, change root to the newly created arch install with the `arch-chroot /mnt` command

# Part-4: Setting up important system components
- First, choose your time zone
  - Run `ls /usr/share/zoneinfo` to get a list of regions, and locate your `REGION`
  - Run `ls /usr/share/zoneinfo/REGION` to get a list of cities in your region, and locate your `CITY`
  - Finally, run `ln -sf /usr/share/zoneinfo/REGION/CITY /etc/localtime` (AGAIN, KEEP THE SPACE BEFORE `/etc/localtime`)
  - If you found that confusing, here are some examples:
    - `ln -sf /usr/share/zoneinfo/Europe/Bucharest /etc/localtime`
    - `ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime`
- Next, sync your hardware clock with `hwclock --systohc`
- Now you need to choose your locale and generate it
  - Your locale is your language and your fonts
  - Run `vim /etc/locale.gen`
  - Uncomment the locale of your choice (for english that's `en_US.UTF-8 UTF-8`)
  - Save the file and quit from vim
  - Run `locale-gen`
- Now it's time to choose your system language out of the ones you uncommented (this step is necessary even if you only chose one locale)
  - Locales are formed of a language (for english that is `en_US.UTF-8`) and a format (for most locales that is `UTF-8`)
  - To set your system language, you only need to set the language, and not the format
  - The language and format are separated by a space, and the format looks like it's part of the language, so it's easy to accidentally include it
  - If you're unsure which part of the locale is which, just remove the last thing that is separated with a space from the rest
  - To set the system language, run `echo "LANG=YOUR_LOCALE_LANGUAGE" >> /etc/locale.conf`
  - Again, here are examples on that:
    - `echo "LANG=en_US.UTF-8" >> /etc/locale.conf` for english
    - `echo "LANG=fr_FR.UTF-8" >> /etc/locale.conf` for french
- Now to set your hostname (the name that other devices on your network see when you connect to the internet)
- To set your hostname, run `echo "HOSTNAME" >> /etc/hostname`
  - Yet again, examples:
    - `echo "john-laptop" >> /etc/hostname`
    - `echo "just-dont-include-spaces-here" >> /etc/hostname`
- Now it's recommended to create the initramfs ("initialization magic" as ubuntu calls it)
  - Technically this should have already been done by pacstrap, but it's better to do it manually anyway
  - Run `mkinitcpio -P`
- Now you need to install [microcode](https://wiki.archlinux.org/title/Microcode) for your CPU
  - If you have an amd cpu, run `pacman -S amd-ucode`
  - And if you have intel, run `pacman -S intel-ucode`
- Finally, you can install the grub bootloader, to actually be able to boot into Arch
  - First, run `grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable` to install grub
  - Then, run `grub-mkconfig -o /boot/grub/grub.cfg` to automatically configure grub
- The last step is to enable the network manager, so it starts with Arch:
  - To do this, run `systemctl enable NetworkManager`

# Part-5: Setting up users and passwords
- To set your root password run `passwd`
- Now you need to install requirements for sudo users, by running `pacman -S vi sudo`
- Add a new user by running `useradd --create-home USER_NAME` (replace `USER_NAME` with a name you can remember)
  - For example, `useradd --create-home john`
- Run `passwd USER_NAME` to set a password for your user (this should be the same as your root password)
- Now you should give that user sudo privilages (so you can install apps without having to switch to root)
  - First, run `usermod -aG wheel USER_NAME`
  - Then, run `visudo`, and uncomment the line where something similar to `%wheel ALL=(ALL) ALL` first apears

# Part-6: Finishing the install
- Exit from chroot with the `exit` command, or by pressing `ctrl + d`
- Unmount all partitions by running `umount -R /mnt`
- And shutdown you computer by typing `shutdown now`
- Remove Arch installation media (the USB or CD/DVD that you first booted from)
- Now start your PC, and in your BIOS choose to boot from your new Arch installation
- On the menu that apears now, just choose the first option
- Login with your user and password (NOT ROOT)
- To have acces to [AUR](https://wiki.archlinux.org/title/Arch_User_Repository) packages you need to install `yay`
  - First clone the `yay` repository with `git clone https://aur.archlinux.org/yay.git`
  - Next, change dir into the `yay` folder with `cd yay`
  - And install it by running `makepkg -si`
  - Verify that `yay` has installed correctly by running `yay --version`
  - And delete the source by going back with `cd ..` and delete the `yay` directory with `rm -rf /yay`
- Most popular 32-bit programs (like wine or steam) are only available in the multilib repository
  - To use multilib, you just need to turn it on in the pacman config
  - Run `sudo vim /etc/pacman.conf`
  - Uncomment the following lines (close to the bottom of the file) <br />
    "[multilib]" <br />
    "Include = /etc/pacman.d/mirrorlist"
  - Save and close the file
- Before going further you need make sure your system is up to date
  - Since Arch is a [rolling release](https://en.wikipedia.org/wiki/Rolling_release) distro, this shouldn't take too long
  - Run `sudo pacman -Syu`
- And now, for the most important part of your Arch install; the part that helps you brag to your friends about how "You use Arch by the way"
  - Run `sudo pacman -S neofetch`
  - Then, run `sudo vim /etc/bash.bashrc`
  - Add a new line at the bottom of the file, just saying `neofetch`
  - Save and close the file
- Finally, run `neofetch`
- Congrats! You have succesfully installed Arch
  - If you don't plan on doing anything that requires a desktop environment, or any GUI applications, you can stop here
  - For the people who want a graphical environment, continue to the next part

# Part-7: Installing a desktop environment or window manager
- First, it's important to understand the difference between a desktop environment and a window manager
  - Window managers do just that: they manage windows
  - Meanwhile, desktop environments take care of everything else too, like a GUI for settings, a desktop where you can place shortcuts to programs, a "start menu", where programs are better organized, and many other things
  - Usually, people install a window manager because it's very lightweight. i3wm (which is one of the more popular window managers) doesn't use more than 400M
  - Desktop environments on the other hand, like Kde Plasma (which is usually recommended for people coming from windows) end up using almost 1G
  - For the sake of impartiality, I'm going to leave my opinion on both window managers and desktop environments, but unless you are ready for a full commitment, and especially if you're used to Windows, I suggest you start with something like Plasma
  - You can change your mind later, or even install multiple window managers and desktop environments to test them out
- First we need to install a display manager like sddm (you can look at display managers as the login screen), the xorg server (a program that enables graphical applications to be drawn to the screen), and an audio driver 
  - You can do that with `sudo pacman -S sddm xorg alsa pulseaudio pulseaudio-alsa pulseaudio-bluetooth pulseaudio-equalizer pulseaudio-jack`
  - You'll be presented with choices for each of the packages above; for all of them simply press enter and the defaults will be installed
- Now we need to enable sddm to start with the computer
  - Once you do this, DO NOT restart until you finish Part 7, or you might end up getting your computer stuck in the sddm login screen with no GUI frontend to boot into
  - To enable sddm run `systemctl enable sddm`
- Now find a desktop environment or window manager that you think suits you, or choose multiple, then go to Part 7.X to install yours, and come back here when you're done
- This guide does not cover setting up window managers, since they usually require their own guide to get working properly
- If your prefered desktop environment or window manager isn't in this guide, you can look up a tutorial; I presonaly recommend:
  - [Distro Watch](https://www.youtube.com/c/DistroTube)
  - [Mental Outlaw](https://www.youtube.com/c/MentalOutlaw)
  - [Erik Dubois](https://www.youtube.com/c/ErikDubois)
  - [Tech Hut](https://www.youtube.com/c/TechHutHD)
- It's also here that you usually install a web browser, but you can do that later
  - Firefox: `sudo pacman -S firefox`
  - Brave Browser: `yay -S brave-bin`
  - Chromium: `sudo pacman -S chromium`
- Finally, restart your computer
- After the restart, you'll see in the top left a drop-down menu with all of the desktop environments and window managers you installed
- Simply choose which you want, and login

### Part-7.1: Kde Plasma
- Run `sudo pacman -S plasma konsole dolphin kwallet kwalletmanager kwallet-pam signon-kwallet-extension`, and when presented with a choice, press enter to install everything
- When prompted with a large selection of packages, just press enter to install all of them

### Part-7.2: xfce
- Run `sudo pacman -S xfce4 xfce-goodies`
- You will be prompted with a few choices
- Press enter on all of them

### Part-7.3: lxqt
- Run `sudo pacman -S lxqt breeze-icons termite`

### Part-7.4: i3wm
- Run `sudo pacman -S i3 dmenu`
- When prompted with a choice type `1 3 4 5`(keep the space between them) and press enter
- And then run `yay -S termite`

### Part-7.5: Awesome WM
- Run `sudo pacman -S awesome lua`
- Then run `yay -S termite`

# Part-8: Installing nvidia drivers
- Run the following commands:
  - `pacman -S nvidia nvidia-utils`
  - `yay -S optimus-manager`
  - `yay -S optimus-manager-qt`
- Restart your computer
- You should see a new icon pop up somewhere on your taskbar
  - Press that icon and a menu should open
  - Options on that menu are pretty self-explanatory
