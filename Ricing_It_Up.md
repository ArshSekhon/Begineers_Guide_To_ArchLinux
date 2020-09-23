# ✨💫💅 Ricing it up 💅💫✨
### Prep: Installing icons and themes

```
sudo pacman -S lxappearance materia-gtk-theme
yay -S plata-theme paper-icon-theme-git flattr-icon-theme
```

Now you can pick and apply your favorite theme to the distro by launching `lxappearance` from the terminal.

For my install I like using `Materia - Dark` for the widgets and `Paper` for the Icons.

### Step 1: Setting up picom 

```
yay -S picom-git

# copy the config files

# test picom by executing
picom --experimental-backends --config ~/.config/picom.conf
```

### Step 2: Configuring polybar

```
sudo pacman -S bc jq
yay -S ttf-weather-icons zscroll-git

# copy the polybar config

# test polybar by executing:
~/.config/polybar/launch.sh
```

### Step 3: Configuring i3

Copy the dot files
```
yay -S i3-gaps-rounded-git
```

### Step 4: Configuring rofi
```
yay -S rofi
```

### Step 5: Configuring alacritty
```
use alacritty.yml

yay -S noto-fonts-emoji
```

### Step 6: Configuring vim and neovim

### Step 7: Configuring dunst


### Step 8: Configuring VSCode

### Step 9: Configuring fish
```
sudo pacman -S fish
curl -L https://get.oh-my.fish | fish

```

```
sudo pacman -S zsh
 
#install ohmyzsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

#edit .zshrc and 
```

### Step 9: Customizing zsh with custom prompt

### Step 10: Setting up a lockscreen
I would use light-dm as locker

```
dm-tool lock
```

### Step 11: Customizing lightdm

### Step 12: Configuring tmux  