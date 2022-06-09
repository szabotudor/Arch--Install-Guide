# Arch Install Guide
An installation guide for arch
- [Preparation and creating the installation media](#part-0-preparation-and-creating-the-installation-media)
- [Connect to the internet](#part-1-connect-to-the-internet)
- [Partition the disks](#part-2-partition-the-disks)

# Importan Notices
- This guide only covers installing arch in EFI/UEFI mode (valid for most modern systems)
  - If you want to install arch in compatibility BIOS mode, refer to [this part](https://wiki.archlinux.org/title/Installation_guide#Example_layouts) in the official guide along with this one
- For a minimal install follow instructions up to and including Part-6
- You should probably look up a tutorial on vim if you don't know how to use it

# Part-0: Preparation and creating the installation media
- Download the arch iso from the [Official Arch Linux Website](https://archlinux.org/download/)
- Flash that iso to a USB drive or disk 
- The easiest way to do it is by using [Balena Etcher](https://www.balena.io/etcher/)
  - Choose the arch iso and the device you want to flash it to, and hit the `flash` button
- After you're done, boot from that device (I won't include instructions on that, since this may vary from system to system, just search how to boot from a USB or CD/DVD on your machine
- When booting from the installation media you will be presented with a menu. Just choose the first option, and if it doesn't work, force reboot and choose another

# Part-1: Connect to the internet
- If you have a wired connection, skip to the last step in this part
- The arch installer has a built-in tool for using a wireless connection. To use it, run `iwctl`
- Now it's time to find what wireless modules you have in your PC/laptop. Run `station list` to print out a list of "stations" (or modules)
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
- After creating the swap partition, set its type to `Linux swap`
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
