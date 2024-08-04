# Arch Setup documentation 

## Important things before installation
- **Disable Secure Boot**
  
## Important things regarding the documentation
- **Words in `<>` are placeholders**
- Comments are written after Hashtags #

## Typical Installation
### Basic Setup for zsh (shell)
```bash
loadkeys de_CH-latin1 #Set keybord layout
setfont ter-132b #Make font bigger
```

### Check UEFI bitness (Boot Mode)
```bash
cat /sys/firmware/efi/fw_platform_size
```
Returns:
- 64 -> 64 Bit UEFI
- 32 -> 32 Bit UEFI
- File doesn't exist -> BIOS / Legacy boot on UEFI


### Internet
```bash
ip link #Basic Information about Network interfaces
```

### WLan 
```bash
iwctl #iwctl client. iwd is the daemon and iwmon is the wifi monitoring tool
    device list #List WLan adapters
    device <device> set-property Powered on #Turn on / Activate the adapter
    station <device> scan #Scan for Networks
    station <device> get-networks #Print the networks
    station <device> connect SSID #Connect to a network
```

### Create GPT Table and Paritions
```bash

fdisk –l #list storage devices. Similar to `lsblk`
#Partition table
fdisk /dev/<disk>
	g #Make disk GPT
	w #Write to disk

fdisk -l #l
fdisk /dev/<disk>

#EFI system partition
	n #New partition
	<default> #Partition type is primary
	<default> #Partition number
	<default> #Begining is at the ending of the partition before
	<end of sector> #End of sector, I recommand using relative values, like '+2g' or '-2g'.
	t #Change type ...
	uefi #... to uefi (Efi system partition, includes the bootloader and stuff)
	 
#Swap (For better power efficency on laptops + hybrid mode (saves RAM to Disk))
	n #New opartition
	<default> #Partition type is primary
	<default> #Partition number
	<default> #Begining is at the ending of the partition before
	<end of sector> #End of sector, I recommand using relative values, like '+2g' or '-2g'.
	t # Change type..
	<default>
	 #..of the last edited partion..
	swap #..to swap.
	 
#Root partition
	n #New partition
	<default> #Partition type is primary
	<default> #Partition number
	<default> #Begining is at the ending of the partition before
	<default> #Ending is the end of the Disk
	w #Write
  
#Create Filesystems
lsblk #List block devices - Check if all partitions are correct
mkfs.fat -F 32 /dev/<part1> #Format EFI partition as fat
mkfs.ext4 /dev/<part3> #Format Root partition as ext4
mkswap /dev/<part2> #Mark swap partition as a swap partition
```

### Mount Partitions
```bash
# BE SURE THE MOUNT THE PARTITIONS IN THE RIGHT ORDER
mount /dev/<part3> /mnt #Mount Root partition in /mnt
mount --mkdir /dev/<part1> /mnt/boot #Mount EFI partition in /mnt/boot and create directory if it doesn't exist yet
swapon /dev/<part2> #Mark swap partition as active
lsblk # Check if every partition is mounted in the correct location
```

### Install Packages and generate fstab
```bash
pacstrap -K /mnt base linux linux-firmware #Install the most basic packages for Linux to function.
genfstab -U /mnt >> /mnt/etc/fstab #Generates the fstab file, which maps the UID's of partitions, their fs, their mountpoints and more
```

### chroot 
chroot changes the root location from a live envirement to another location. This can be a new installation or an already existing one. chroot is really useful, if the system can not boot through grub anymore.

```bash
arch-chroot /mnt #chroot into the mount point of the root partition of the arch installation
```

### Install more Important packages
```bash
pacman -S nano neovim
nano /etc/pacman.conf #Uncomment `ParallelDownloads = 5`and possibly set it to a higher value, for example 10 or 20. I also bekommen enable the `extra` and `multilib` gepos by uncommenting the matching lines
	
pacman -S iwd systemd udev dbus sudo fakeroot intel-ucode linux-headers dkms networkmanager grub efibootmgr bash-completion s-tui fastfetch hyfetch btop openssh ufw git base-devel go #installs some really important and some less important packages to the installation
```

### Time and locales
```bash
ln -sf /usr/share/zoneinfo/Europe/Zurich /etc/localtime #Set the timezone

hwclock --systohc #generates some files for the hardwareclock
# timedatectl set-local-rtc 1 #use only if the installation gets dualbooted with Windows, because Windows stores the local time on the hw clock. By default, Arch stores the time in UTC format 
nano /etc/locale.gen #uncomment `de_CH.UTF-8 UTF-8` and `en_US.UTF-8 UTF-8` #Enable Swiss German and English locale files
locale-gen #Generate locale files
nano /etc/locale.conf #Define the used language for..
	LANG=de_CH.UTF-8 #..timeformat and currency..
	LC_MESSAGES=en_US.UTF-8 #.. and all userinteractions.

nano /etc/vconsole.conf #Enter the console configuration file..
	KEYMAP=de_CH-latin1 #..and set the Swiss German keyboard as the default keyboard
```

### Hostname
```bash
nano /etc/hostname #Enter hostname file.. 
	<myhostname> #..and set hostname
```

### Root password
```bash
passwd #Set root password 
```

### Generate new initramfs 
```bash
mkinitcpio -P
```

### Install GRUB
```bash
grub-install --efi-directory=/boot --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg 
```

### Leaving chroot and unmounting partitions
```bash
exit
umount -R /mnt 
reboot
```
### Remove the installation media and boot to the installation
### Login as root
### User creation
```bash
export EDITOR=nano #Select texteditor for sudo-ers file. Standard is vim. 
visudo #Enter copy of the configuration file. The copy get's written to the originalfile after exiting and getting verfied by visudo
        #Uncomment this line
		%sudo ALL=(ALL:ALL) ALL 
	
groupadd sudo #Add sudo-ers group 
useradd -m -G sudo -s /bin/bash <user> #Add user and assign them to the 'sudo' group
passwd <user> #Change password of the user
su <user> #Switch to the newly created user
```

### Internet
```bash
sudo systemctl enable --now NetworkManager #Enable and starting the NetworkManager daemon
nmcli device wifi list #List WIFI networks
nmcli device wifi connect <SSID> password <password> #Connect to network

ping 1.1.1.1 #Be patient! Getting the ip address can take some time..
```

### Create default user directories
```
sudo pacman -Syu xdg-user-dir
xdg-user-dirs-update
```

### AUR
```bash
#Install yay for easy access to the AUR 
cd
git clone https://aur.archlinux.org/yay.git #Clone yay's files
cd yay 
makepkg –si #Build and install yay. If it failes on the first execution, just re-run 
yay -Syu #Upgrade packages. Part of the command executes `mkinitcpio -P` which prints `Missing firmware` messages.
#Look for "=> WARNING: Possibly missing firmware for module: 'xyz' 
#Search in AUR: https://aur.archlinux.org/packages and install with ..
yay –S <package>
#common ones: ast -> ast-firmware, xhci_pci -> upd72020x-fw, wd719x -> wd719x-firmware, aic94xx -> aic94xx-firmware 

yay -S htop net-tools #Some more useful packages
```

## Installation over SSH 
```bash
loadkeys de_CH-latin1 #Set keybord layout
passwd #Set ssh password
nano /etc/ssh/sshd_config #set `PermitRootLogin` to `yes`
#systemctl reload sshd #only needed it `PermitRootLogin` was not set to `yes
```
Now continue with the normal installation. <br>
Don't forget to install `sshd` when with pacman!
