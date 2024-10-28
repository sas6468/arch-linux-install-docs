# Documentation of Steps involved in Installing Arch Linux
The first step is to aquire the iso file from an HTTP mirror and then verify the signature to be sure that there are no malicous interceptions. Next we create a new virtual machine using this iso file and we allocate 40 GB space. 
The installation medium's boot loader menu appears and then we select arch linux installation medium to enter the installation environment. I am now logged in on the virtual console as the root user and presented with a zsh shell prompt. 
The next thing I did was to verify if my system booted in a bios mode. Since my system is in bios mode we will next create partitions which will include Root and Swap only. 
I was stuck here as I made mistakes in partitioning the disk. My mistakes were two-fold. First, I created boot also and created issues while mounting both boot and Root. This corrupted the partition table so I had to delete from VM and reinstall. Next install, I made a mistake of allocating just 8 GB which became a problem while installing LXQT and SDDM. 

## Partitioning the Disk
1. Run `fdisk /dev/sda` (assuming `/dev/sda` is your primary disk).
   - **Note:** Use `fdisk` to manage disk partitions.
2. Create a Root partition and a Swap partition. Example:
   - Root partition: `/dev/sda1` (35 GB).
   - Swap partition: `/dev/sda2` (4 GB).
   

## Format Partitions
mkfs.ext4 /dev/sda1  # Format root partition with ext4 filesystem
mkdir /mnt  # Create a mount point for the root partition
mount /dev/sda1 /mnt  # Mount the root partition
swapon /dev/sda2

## Next install essential packages using pacstrap
pacstrap /mnt base linux linux-firmware  # Install base system packages

# Next we will configure the system

## Generate fStab file
genfstab -U /mnt >> /mnt/etc/fstab  # Generate fstab file for automatic mounting

# Change Root into the new system before we start configuration 
arch-chroot /mnt  # Change root into the new system

# Set Time-zone
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime  # Link the correct timezone
hwclock --systohc  # Set hardware clock to current time

# Localization
locale-gen  # Generate locale files
echo "LANG=en_US.UTF-8" > /etc/locale.conf  # Set default system language

# Create the hostname file
echo "arch-vm" > /etc/hostname  # Set the hostname for the system

# Configure DHCP client daemon
pacman -S dhcpcd  # Install DHCP client daemon
systemctl enable dhcpcd  # Enable DHCP service to start on boot

# Set Root password
passwd  # Set the root user password (follow prompts)

# Install Grub package boot loader
pacman -S Grub 
grub-install --target=i386-pc /dev/sda1 # Install Grub for BIOS system
grub-mkconfig -o /boot/grub/grub.cfg. # Generate Grub configuration file

# Install a desktop environment
pacman -S lxqt xorg xorg-server sddm  # Install LXQt desktop environment and Xorg server
systemctl enable sddm  # Enables the Simple Desktop Display Manager

# Create user accounts with sudo privileges
useradd -m -G wheel justin  # Create user 'justin' with a home directory and wheel group
passwd  # Set password for 'justin'
useradd -m -G wheel codi  # Create user 'codi'
passwd  # Set password for 'codi'
chage -d 0 justin  # Force password change on next login for 'justin'
chage -d 0 codi  # Force password change on next login for 'codi'
visudo  # Visudo command did not work for me so I used nano to edit /etc/sudoers to give wheel group sudo permissions by uncommenting the following line:
%wheel ALL=(ALL) ALL  # Allow users in the 'wheel' group to use sudo 

# Install an alternative shell
pacman -S zsh  # Install Zsh shell 
chsh -s /bin/zsh justin  # Change shell for 'justin'
chsh -s /bin/zsh codi  # Change shell for 'codi'

# Install SSH
pacman -S openssh  # Install OpenSSH server
systemctl enable sshd  # Enable SSH daemon to start on boot
systemctl start sshd  # Start the SSH daemon
ssh localhost  # Check if SSH is working locally

# Add color coding to terminal
pacman -S grml-zsh-config  # Install color configuration for Zsh
Modify the .zshrc file for color prompts by adding the following line:
export PS1="%F{green}%n@%m %F{blue}%1~ %# %F{reset}"  # Set colorful prompt

# Set the system to boot into GUI
systemctl enable sddm  # For LXQt

# Add aliases to shell configuration
Edit the .zshrc and add the following useful aliases
alias ll='ls -lah'  # List files with detailed information
alias update='sudo pacman -Syu'  # Update the system
alias rm='rm -i'  # Prompt before removing files
alias c='clear'  # Clear the terminal screen

# Install a web browser
pacman -S firefox  # Install Firefox browser

# Reboot the system through GUI
exit # Exit the chroot environment
umount -R /mnt # Unmount all the partitions
reboot 

# The system now boots through the grub bootloader into the GUI using SDDM service and asks to select either Codi or Justin. Selecting either one of these accounts prompts for password. Once the password "GraceHopper1906" is entered, the system prompts for the password to be changed. This is per design. Now we are in the system and then I started FireFox and browsed few websites successfully. After that, I noticed that the date time was off so I went and corrected it by selecting automatic sync and mapping it to US/Chicago time zone. 

## THE LINUX INSTALL IS READY FOR PRIMETIME!!!




