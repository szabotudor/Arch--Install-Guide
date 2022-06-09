# Arch Install Guide
An installation guide for arch
- [Preparing and creating the installation media](#part-0-preparing-and-creating-the-installation-media)
- [Connect to the internet](#part-1-connect-to-the-internet)
- [Partition the disks](#part-2-partition-the-disks)
- [Mounting the partitions and preparing the base arch install](#part-3-mounting-the-partitions-and-preparing-the-base-arch-install)
- [Setting up important system components](#part-4-setting-up-important-system-components)

# Importan Notices
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
- Now it's time to find what wireless modules you have in your PC/laptop. Run `station list` to print out a list of "stations" (the wifi chips in your computer)
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
  - `swapop /dev/SWAP_PARTITION_NAME`
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
