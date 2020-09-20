# Installing xorg, i3-wm and other useful softwares

### Step 1: Make sure you have a working Internet Connection

### Step 2: Installing Video Drivers

If installing on a machine with NVIDIA GPU, run the following:
```
sudo pacman -S nvidia nvidia-settings nvidia-utils
sudo nvidia-xconfig
```

Otherwise, you can find the right drivers for your GPU by following steps on this [link](https://wiki.archlinux.org/index.php/Xorg#Driver_installation) on Arch Wiki

### Step 3:

```
sudo pacman install xorg xorg-xinit firefox picom nitrogen
```

### Step 4: Enable AUR

```
sudo pacman -S base-devel
git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si
```

### Step 5: Installing the Goodies

```
sudo pacman -S dmenu i3-gaps-rounded-git
yay -S nerd-fonts-mononoki nerd-fonts-complete picom polybar rofi alacritty
```

### Step 6: Create ~/.xinintrc

```
cp /etc/X11/xinit/xinitrc ~/.xinitrc

Comment the following out from the end of the file by adding # symbol in the front of the following lines:

# twm &
# xclock -geometry 50x50-1+1 &
# xterm -geometry 80x50+494+51 &
# xterm -geometry  80x20+494+0 &
# exec xterm -geometry 80x66+0+0 -name login
```

### Step 7: Installing Display Manager

```
sudo pacman -S lightdm
yay -S lightdm-slick-greeter
```

Now modify `lightdm.config` to change the lightdm greeter by running `sudo nano /etc/lightdm/lightdm.conf`. Change the following key value pair:

```
greeter-session=lightdm-slick-greeter
```

### Step 8: Setting up audio

```
pacman -S alsa-utils
pacman -S pulseaudio pavucontrol
```

### Step 9: Setting up Bluetooth

```
sudo pacman -S bluez bluez-utils 
sudo systemctl enable bluetooth.service
```

Installing Bluetooth UI manager. After running command below you can launch the bluetooth manager by executing `blueman-manager`.
```
sudo pacman -S blueman pusleaudio-blueman
```


### Step 10: Setting up WiFi GUI

```
sudo pacman -S network-manager-applet
```




