# Part 1: Installing Base Arch with LVM


### **Step 1: Set the keyboard layout**
Usually if you are using an English (US) keyboard you would not need to do this. But if you need to change the keyboard, you can find the available keymmaps using the following command:
```
ls /usr/share/kbd/keymaps/**/*.map.gz
```
To change the layout, append a corresponding file name to loadkeys command, omitting path and file extension. So, for e.g. to load de-latin1 keymap execute:
```
loadkeys de-latin1
```

### **Step 2: Ensure Internet access**
Check if you have internet access by executing the following command:

`ping -c 4 archlinux.org`

If the server is unreachable there are probably some issues. If you face connectivity issue or if you want to install Arch using your WiFi please follow [ArchWiki: Connect to the internet](https://wiki.archlinux.org/index.php/installation_guide#Connect_to_the_internet)

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
You can use **fdisk utility** to partition the drive, run the following command with the disk you want to partition. We will pretend that we are partitioning the drive named **nvme1n1**. 

For partitioning we would want to create 4 partitions:
- **Partition #1 (Bootloader):** This is where our bootloader will live. Bootloader is a program that loads the OS into the computer's memory. This is a specific place where your computer looks to find enteries for your OS. About `512MB` should be enough for this. 

Note: A separate EFI partition is only needed if 

- **Partition #2 (Swap):** If you run out of RAM you computer uses your non-volatile storage(SSD/HDD) as a virtual RAM. A swap partition is this allocated space on disk that is used as a virtual RAM. Size of this partition is totally up to you but I usually use 1-2 times the amount of RAM installed on the device.

- **Partition #3 (LVM):** This is where the rest of your stuff would go. The operating system, you personal files everything. We would further create virtual volumes in this  as we want to have a separate root and home partition.

We could have just create two partitions at the first place but creating volumes using LVM (virtual partitioning) makes the resizing and editing or volumes easier. 

Now that we have theory out of the way, first we will create the partitions. Enter the fdisk utility using the following command:

`fdisk /dev/nvme1n1`

Now inside the **fdisk utility** press the following keys to create the partition tables on the disk:

```
press o (only if you are running BIOS mode)
press g (only if you are running UEFI mode)
```

The EFI partition is only required if you are running UEFI. If in BIOS mode you can skip ahead to the creation of the SWAP partition.
```
--- CREATING EFI PARTITION () ---
press n (to create a new partition)
press 1 (partition number)
press enter
enter +512MB (size of partition)
press t (change type of the partition)
enter 1 (change type to EFI)
```

Now its the time for **Partition #2 (Swap):** 
```
--- CREATE SWAP PARTITION ---
press n (to create a new partition)
press 2 (partition number)
press enter
press +8GB (size of the swap)
press t (change type of the partition)
press 2 (change type of the partition #2)
press L (find the number corresponding to the type Linux Swap)
enter 19 (number corresponding to the type Linux Swap)
```
Finally lets create our **Partition #3 (LVM):**:
```
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
Now as mentioned previously, we would need to do the virtual partitioning of the LVM partition. First we need to initialize PhysicalVolume for later use by the Logical Volume Manager (LVM). For more details see: [pvcreate](https://linux.die.net/man/8/pvcreate)

```
pvcreate /dev/nvme1n1p3  #(If not running a SSD) 
pvcreate --dataalignment 1m /dev/nvme1n1p3 #(If running a SSD) 
```


```
vgcreate vg0 /dev/nvme1n1p3
# create a new volume groups to put volumes in
```
Now create logical volumes for both root and home. You can set the size of partition by using `-L 100GB` flag where 100GB would be size of partition. 

Note: `-l 100%FREE` creates a volume to fill the entire free space on the volume group.
```
lvcreate -L 200GB vg0 -n lv_root
lvcreate -l 100%FREE vg0 -n lv_home
# create logical volumes

modprobe dm_mod

vgscan
# scan all disks for volume groups and rebuild caches

vgchange -ay
# activate all known volume groups in the system
```
#### Formatting the partitions
- Format EFI partition as FAT32: `mkfs.fat -F32 /dev/nvme1n1p1`
- Format the swap partition: `mkswap /dev/nvme1n1p2`
- Format the lv_root partition: `mkfs.ext4 /dev/vg0/lv_root`
- Format the lv_home partition: `mkfs.ext4 /dev/vg0/lv_home`

### Step 5: Mounting the partitions
Mount the partitions at there respective places. `swapon` here enables the partition for swapping.

```
mount /dev/vg0/lv_root /mnt
mkdir /mnt/home
mount /dev/vg0/lv_home /mnt/home
swapon /dev/nvme1n1p2
```
### Step 6: Install Arch
Congratulations! you have finally made past partitioning. Now it is time to install essential stuff. Install linux kernel and firmware executing following:

```
pacstrap /mnt base linux linux-firmware
```


### Step 7: Generate fstab
To generate the fstab, fstab file lists all available partitions and indicates how they are to be initialized or integrated into the file system structure. So basically things like what partitions exist and where they should be mounted.

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Step 8: Chroot
As of now, we are using a place on installation media as our root. Now lets change our newly root partition to be our new root.
```
arch-chroot /mnt
```

### Step 9: Change TimeZone
You can find your region by executing: `ls /usr/share/zoneinfo` and then your city using `ls /usr/share/zoneinfo/<YOUR REGION>`.

After you have found the correct region and city for you. You can simply execute the following command to set your timezone.
```
ln -sf /usr/share/zoneinfo/America/Vancouver /etc/localtime
```

### Step 10: Set System Clock
Now we need to sync the hardware and software clock. It can be done by using following command:
```
hwclock --systohc
```
For more info on this see: [hwclock](https://linux.die.net/man/8/hwclock)

### Step 11: Set Locale
Now it is time to set the locale. This is an important step and if you skip it, you might encounter some issues later on.
```
pacman -S nano vim
nano /etc/locale.gen

#----------------
# uncomment the locale by removing # sign in front of it
# for eg. change "#en_US.UTF-8 UTF-8" -> "en_US.UTF-8 UTF-8"
# and save the file
#----------------

locale-gen
```

### Step 12: Set a hostname
Now you need to set the hostname for your machine. Create a new file `/etc/hostname/` with just the hostname in it.

```
Arch-BattleStation
```

### Step 13: Create `/etc/hosts` file with following contents

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   <your hostname here>.localdomain <hostname here>
```

### Step 14: Set a password for the root user
Set a password for the root user by executing `passwd`.

### Step 15: Create a new user
Now it is finally the time to create the users
```
useradd -m arshsekhon
passwd arshsekhon

# add user to the user groups for the sake of permissions
usermod -aG wheel,audio,video,storage,optical arshsekhon
```

### Step 16: Install `sudo`
Now in order to be able to execute commands as root user. We would need to install `sudo`. You probably would be familiar with `sudo` from other UNIX based systems, in Arch you would need to install it using the following command:
```
pacman -S sudo
```
Edit sudoers file by executing `visudo` and uncomment the following line and save the file:
```
%wheel ALL=(ALL) ALL
```

### Step 17:

Now we need to enable the Logical Volume Manager. This is a really really important step. Install `lvm2` using the following:
```
pacman -S lvm2
```

**Important:** Now edit `/etc/mkinitcpio.conf` and **add lvm2 in between block and filesystems**.

The file should now contain a line that looks like following:
```
HOOKS=(base udev autodetect modconf block lvm2 filesystems keyboard fsck)
```

Execute the following, to recreate initial ramdisk environment. This will ensure that the `lvm` kernel module is loaded after boot:

```
mkinitcpio -p linux
```

### Step 18: Setup bootloader
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

### Step 19: Finalizing Install
Now we can finalize things by installing `NetworkManager` and `git`. Finally we will enable the `NetworkManager` service.
```
pacman -S vim networkmanager git 
systemctl enable NetworkManager
```

### Step 20: Exit chroot and reboot
```
exit
umount -l /mnt
reboot
```
Now finally remove installation media.

### Step 21: Celebrate  🎉✨🎉
This is the time to give yourself a pat on the back. Now you should have a functional install of Arch Linux on your Computer. 

Things might not be as pretty as you expected as you would be welcomed just by a tty, but trust me we'll get there.