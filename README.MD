# Arch Setup documentation 

## Important things before installation
- **Disable Secure Boot**
  
## Important things regarding the documentation
- **Words in curly brackets `{}` are placeholders**
- Comments are written after Hashtags #

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
    device {device} set-property Powered on #Turn on / Activate the adapter 
    station {device} scan #Scan for Networks
    station {device} get-networks #Print the networks
    station {device} connect SSID #Connect to a network
```

### Set Time
```bash
timedatectl set-timezone Europe/Zurich #Set timezone to Zurich
timedatectl #Verify that the time is accurate
```

### Create GPT Table and Paritions
```bash

fdisk –l #list storage devices. Similar to `lsblkb` 
#Partition table
fdisk /dev/{xyz} 
	g #Make disk GPT
	w #Write to disk 

fdisk -l #l
fdisk /dev/{xyz}
```

**Use [Calculator](https://eyer.life/sectorcalc) for calculating values, as these represent amounts of 512 byte blocks.** 
```bash
#EFI system partition
	n #New partition
	{default} 
	{default} #Begining is at the ending of the partition before
	{end of sector} #End of sector, use calc
	t #Change type ...
	uefi #... to uefi (Efi system partition, includes the bootloader and stuff)
	 
#Swap (For better power efficency on laptops + hybrid mode (saves RAM to Disk)) 
fdisk /dev/xyz 
	n #New oartition
	{default} 
	{default} #Begining is at the ending of the partition before
	{end of sector} #End of sector, use calc
	t # Change type..
	{default} #..of the last edited partion..
	swap #..to swap.
	 
#Root partition
fdisk /dev/xyz 
	n #New partition
	{default} 
	{default} #Begining is at the ending of the partition before
	{default} #Ending is the end of the Disk
	w #Write
  
#Create Filesystems
lsblk #List block devices - Check if all partitions are correct
mkfs.fat -F 32 /dev/xyz1 #Format EFI partition as fat
mkfs.ext4 /dev/xyz3 #Format Root partition as ext4
mkswap /dev/xyz2 #Mark swap partition as a swap partition
```

## Mount Partitions
```bash
mount --mkdir /dev/sda1 /mnt/boot #Mount EFI partition in /mnt/boot and create directory if it doesn't exist yet
mount --mkdir /dev/sda3 /mnt #Mount Root partition in /mnt and create directory if it doesn't exist yet
swapon /dev/sda2 #Mark swap partition as active
lsblk # Check if every partition is mounted in the correct location
```

## Install Packages and generate fstab
```bash
pacstrap -K /mnt base linux linux-firmware #Install the most basic packages for Linux to function.
genfstab -U /mnt >> /mnt/etc/fstab #Generates the fstab file, which maps the UID's of partitions, their fs, their mountpoints and more
```

## chroot 
chroot changes the root location from a live envirement to another location. This can be a newly setup installation or a already existing one. chroot is really useful, if the system can not boot throught grub anymore.

```bash
arch-chroot /mnt #chroot into the mount point of the root partition of the arch installation
mount --mkdir /dev/sda1 /mnt/boot #mount the root back to it's place 
```

## Install more Important packages
```bash

pacman -S iwd systemd udev dbus sudo fakeroot vim nano intel-ucode linux-headers dkms dhcpcd grub efibootmgr s-tui neofetch hyfetch ufw git #installs some really important and some less important packages to the installation

```

## Time and locales
```bash
ln -sf /usr/share/zoneinfo/Europe/Zurich /etc/localtime #Set the timezone

hwclock –systohc #generates some files for the hardwareclock
# timedatectl set-local-rtc 1 #use only if the installation gets dualbooted with Windows, because Windows stores the local time on the hw clock. By default, Arch stores the time in UTC format 
nano /etc/locale.gen //uncomment ```de_CH.UTF-8 UTF-8``` and ```en_US.UTF-8 UTF-8``` #Enable Swiss German and English locale files
locale-gen #Generate locale files
nano /etc/locale.conf #Define the used language for..
	LANG=de_CH.UTF-8  #..timeformat and currency..
	LC_MESSAGES=en_US.UTF-8 #.. and all userinteractions.

nano /etc/vconsole.conf #Enter the console configuration file..
	KEYMAP=de_CH-latin1 #..and set the Swiss German keyboard as the default keyboard
```

## Hostname
```bash
nano /etc/hostname #Enter hostname file.. 
	{myhostname}  #..and set hostname
```  

# Root password
```bash
passwd #Set root password 
```

## Generate new initramfs 
```bash
mkinitcpio -P
```

## Instal GRUB
```bash
grub-install --efi-directory=/boot --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg 
```

## Leaving chroot and unmounting partitions
```bash
exit
unmount -R /mnt 
reboot
```
## Remove the installation media and boot to the installation
## Login as root
## User creation
```bash
EDITOR=nano #Select texteditor for sudo-ers file. Standard is vim. 
visudo #Enter copy of the configuration file. The copy get's written to the originalfile after exiting and getting verfied by visudo
        #Uncomment this line
		%sudo ALL=(ALL:ALL) ALL 
	
groupadd sudo #Add sudo-ers group 
useradd -m -G sudo –s /bin/bash {user} #Add user and assign them to the 'sudo' group
passwd eyery #Change password of the user
su eyery #Switch to the newly created user 
```

## Internet
```bash
sudo systemctl enable dhcpcd #Enable dhcp daemon
sudo systemctl start dhcpcd #Start dhcp daemon
ping 1.1.1.1 
```

## AUR
```bash
#Install yay for easy access to the AUR 
git clone https://aur.archlinux.org/yay.git #Clone yay's files
cd yay 
makepkg –si #Build and install yay. If it failes on the first execution, just re-run 
yay -Syu #Upgrade packages. Part of the command executes `mkinitcpio -P` which prints `Missing firmware` messages.
#Look for "=> WARNING: Possibly missing firmware for module: 'xyz' 
#Search in AUR: https://aur.archlinux.org/packages and install with ..
yay –S {package}
#common ones: ast -> ast-firmware, xhci_pci -> upd72020x-fw, wd719x -> wd719x-firmware, aic94xx -> aic94xx-firmware 

yay  -S htop net-tools #Some more useful packages
```

## Install sway (**untested !!!**)
```bash
pacman -S sway #Install sway package
sudo usermod -G seat,sudo {user} #Add user to the seat group
sudo systemctl enable seatd #Enable the seat daemon
sudo systemctl start seatd #Start the sead daemon
logout 
#login as root
sway
```
