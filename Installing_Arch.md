# Installing Arch with LVM


### **Step 1: Set the keyboard layout**

### **Step 2: Ensure Internet access**
Check if you have internet access by executing the following command:

`ping -c 4 archlinux.org`

If the server is unreachable there are probably some issues 

### **Step 3: Ensure the Time and date is correct**
Before proceeding with the installation we need to make sure that the date and time on our system is correct. This is important as we'll need to download stuff from the internet for the install and wrong date/time can cause issues with the same.

Since we have the internet enabled we can enable Network Time Protocol Service to set the correct time on our machine.

`timedatectl set-ntp true`

Now make sure `ntp` has been activated by executing the following command:

`timedatectl status`

If the NTP Service is set to enabled, you're ready to move forward.


### **Step 4: Partitioning the disk**
#### **Finding the disk you want to install Arch Linux on**
Partitioning the disk is one of the most important step in the Arch installation process.

List all the disks installed on your system by running the following command:

`lsblk`

This command will print out all of the installed disks on your system. Find the name of the disk you want to install the Arch Linux. It would be something similar to **sda, sdb or sdc etc.** or if you have a NVMe drive installed it would be something along the lines of **nvme0n1 or nvme1n1** etc.

#### **Partitioning the drive**
You can use **fdisk utility** to partition the drive, run the following command with the disk you want to partition. We will pretend that we are partitioning the drive named **nvme1n1**. First we will create the EFI, enter the fdisk utility using the following command:

`fdisk /dev/nvme1n1`

Now inside the **fdisk utility** press the following keys to create the partitions on the disk:

```
press o (only if you are running BIOS mode)
press g (only if you are running UEFI mode)

--- CREATING EFI PARTITION ---
press n (to create a new partition)
press 1 (partition number)
press enter
enter +512MB (size of partition)
press t (change type of the partition)
enter 1 (change type to EFI)

--- CREATE SWAP PARTITION ---
press n (to create a new partition)
press 2 (partition number)
press enter
press +8GB (size of the swap)
press t (change type of the partition)
press 2 (change type of the partition #2)
press L (find the number corresponding to the type Linux Swap)
enter 19 (number corresponding to the type Linux Swap)

--- CREATE LVM PARTITION ---
press n (to create a new partition)
press 3 (partition number)
press enter
press enter (size of the LVM partition to fill rest of the disk)
press t (change type of the partition)
press 3 (change type of the partition #3)
press L (find the number corresponding to the type Linux LVM)
enter 39 (number corresponding to the type Linux LVM)
w (write to write changes)
```

#### **Setting up the LVM**
- Non-SSD: # pvcreate /dev/nvme1n1p3
- SSD: # pvcreate --dataalignment 1m /dev/nvme1n1p3
- \# vgcreate vg0 /dev/nvme1n1p3
- \# lvcreate -L 200GB vg0 -n lv_root
- \# lvcreate -l 100%FREE vg0 -n lv_home
- \# modprobe dm_mod
- \# vgscan
- \# vgchange -ay

#### Formatting the partitions
- Format EFI partition as FAT32: `mkfs.fat -F32 /dev/nvme1n1p1`
- Format the swap partition: `mkswap /dev/nvme1n1p2`
- Format the lv_root partition: `mkfs.ext4 /dev/vg0/lv_root`
- Format the lv_home partition: `mkfs.ext4 /dev/vg0/lv_home`

### Step 5: Mounting the partitions
Mount the partitions.

```
mount /dev/vg0/lv_root /mnt
mkdir /mnt/home
mount /dev/vg0/lv_home /mnt/home
swapon /dev/nvme1n1p2
```
### Step 6: Generate fstab
To generate the fstab

`genfstab -U /mnt >> /mnt/etc/fstab`

### Step 7: Chroot
`arch-chroot /mnt`

### Step 8: Change TimeZone
`ln -sf /usr/share/zoneinfo/America/Vancouver /etc/localtime`

### Step 9: Set System Clock
`hwclock --systohc`

### Step 10: Set Locale
```
pacman -S nano vim
nano /etc/locale.gen

----------------
uncomment the locale by removing # sign in front of it
for eg. change "#en_US.UTF-8 UTF-8" -> "en_US.UTF-8 UTF-8"
and save the file
----------------

locale-gen
```

### Step 11: Set a hostname
Create a new file `/etc/hostname/` with just the hostname in it.

```
Arch-BattleStation
```

### Step 12: Create /etc/hosts file

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   <your hostname here>.localdomain <hostname here>
```

### Step 13: Set a password for the root user
Set a password for the root user by executing `passwd`.

### Step 14: Create a new user
```
useradd -m arshsekhon
passwd arshsekhon
usermod -aG wheel,audio,video,storage,optical
```

### Step 15: Install `sudo`
```
pacman -S sudo
```
Edit sudoers file by executing `visudo` and uncomment the following line and save the file:
```
%wheel ALL=(ALL) ALL
```

### Step 16:
Edit /etc/mkinitcpio.conf and add lvm2 in between block and filesystems and execute
```
pacman -S lvm2
mkienitcpio -p linux
```

### Step 17: Setup bootloader
```
pacman -S grub
pacman -S efibootmgr dosfstools os-prober mtools
mkdir /boot/EFI
mount /dev/nvme1n1p1 /boot/EFI

--- IF RUNNING IN UEFI MODE ---
grub-install --target=x86_64-efi  --bootloader-id=grub_uefi --recheck
--- END ---

--- IF RUNNING IN BIOS MODE ---
grub-install --target=i386-pc /dev/nvme1n1
--- END ---


cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
grub-mkconfig -o /boot/grub/grub.cfg
```

### Step 18: Finalizing Install

```
pacman -S vim networkmanager git 
systemctl enable NetworkManager
```

### Step 19: Exit chroot and reboot
```
exit
umount -l /mnt
reboot
```

Now you should have a functional install of Arch Linux on your Computer.