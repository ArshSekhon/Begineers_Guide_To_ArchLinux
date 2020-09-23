# Part 2: Installing xorg, i3-wm and other useful softwares

### Step 1: Make sure you have a working Internet Connection
Check if you have internet access by executing the following command:

`ping -c 4 archlinux.org`

If the server is unreachable there are probably some issues. If you face connectivity issue or if you want to connect to your WiFi, you would need to use `nmcli` to do so. For more info, refer to [nmcli](https://developer.gnome.org/NetworkManager/stable/nmcli.html)

### Step 2: Installing Video Drivers

First and foremost we will start by installing the video drivers. If installing on a machine with NVIDIA GPU, run the following:
```
sudo pacman -S nvidia nvidia-settings nvidia-utils
sudo nvidia-xconfig
```

Otherwise, you can find the right drivers for your GPU by following steps on [Arch Wiki](https://wiki.archlinux.org/index.php/Xorg#Driver_installation).

### Step 3: Installing xorg and other packages

We will start by installing following packages:

| Package       | Purpose                                                                                |
|---------------|----------------------------------------------------------------------------------------|
| xorg          | Display Server ([more info](https://wiki.archlinux.org/index.php/Xorg))                |
| xorg-init     | Program to start xorg server ([more info](https://wiki.archlinux.org/index.php/Xinit)) |
| firefox       | Browser                                                                                |
| nitrogen      | Wallpaper Setter                                                                       |

<br/>
You can do so by executing the following:

```
sudo pacman -S xorg xorg-xinit firefox picom nitrogen
```

### Step 4: Enable AUR

If you have been using linux for a while you probably would have heard of Arch User Repository (AUR). It is one of the best benifits of using Arch. 

For those who are not familiar, AUR is a community-driven repository for Arch-based Linux distributions. It makes installing software insanely easy.

You can enable AUR using following:
```
sudo pacman -S base-devel
git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si
```

**WORD OF CAUTION:** Be careful about what are you installing from AUR, since this is a community based repo there can be malicious packages on AUR. So it is recommended to install only well know packages from AUR. Also, you would need AUR to compeletely follow this guide.

### Step 5: Installing the Goodies

```
sudo pacman -S dmenu i3-gaps-rounded-git
yay -S nerd-fonts-mononoki nerd-fonts-complete picom polybar rofi alacritty
```

### Step 6: Create ~/.xinintrc

```
cp /etc/X11/xinit/xinitrc ~/.xinitrc

# Open this file inside the text editor of your choice (probably vim or nano)
# Comment the following out from the end of the file by adding # symbol in the front of the following lines:

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

To use bluetooth on Arch, you would need to install `bluez` and `bluez-utils`. Then finally you can enable bluetooth service.
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




