# Migration i3 → Hyprland pour SkillArch

Ce document détaille la migration complète de i3 vers Hyprland basée sur la documentation officielle Hyprland 2025.

## 🎯 Paquets optimisés - Priorité GNOME + Fonctionnalité

### Paquets principaux (pacman)
```bash
# Core Hyprland
hyprland xdg-desktop-portal-hyprland

# Ecosystem officiel Hypr (pas d'alternatives GNOME)
hyprpaper hyprlock hypridle hyprpicker

# Interface - GNOME prioritaire
nautilus                    # GNOME file manager (avec workarounds)
waybar rofi kitty          # Universels, GNOME-compatibles

# Services (remplacements forcés par obsolescence/incompatibilité)
dunst                      # Notification daemon (X11/Wayland compatible)
hyprpolkitagent           # polkit-gnome supprimé des distros

# GNOME Apps maintenues
gnome-control-center      # Avec XDG_CURRENT_DESKTOP=GNOME

# Audio/Screenshots/Utils
pipewire pipewire-pulse wireplumber
grim slurp wl-clipboard
qt5-wayland qt6-wayland wlr-randr
```

### Paquets AUR complémentaires
```bash
yay -S hyprpolkitagent wlogout
```

## 🗑️ Paquets i3/X11 à supprimer

### À retirer complètement
```bash
sudo pacman -Rns i3-gaps i3blocks i3lock i3lock-fancy-git i3status \
                polybar picom feh arandr xorg-xhost thunar \
                i3-battery-popup-git rofi-power-menu
```

### Configurations à supprimer
- `/opt/skillarch/config/i3/`
- `/opt/skillarch/config/polybar/`
- `/opt/skillarch/config/picom.conf`
- `/opt/skillarch/config/xorg.conf.d/`

## 🔄 Table de correspondance

| **Ancien (i3/X11)** | **Nouveau (Hyprland/Wayland)** |
|---------------------|--------------------------------|
| i3-gaps | hyprland |
| polybar | waybar |
| picom | intégré dans hyprland |
| feh | hyprpaper |
| i3lock + xss-lock | hyprlock + hypridle |
| rofi | rofi (compatible Wayland) |
| flameshot | grim + slurp |
| arandr | wlr-randr ou kanshi |
| thunar | nautilus (GNOME) |
| polkit-kde-agent | hyprpolkitagent |
| VBoxClient-all | N/A (pas d'équivalent Wayland) |

## 📋 Nouveau target Makefile

### install-hyprland
```makefile
install-hyprland: sanity-check ## Replace i3 with Hyprland - GNOME Priority
	# Remove i3 ecosystem
	sudo pacman -Rns --noconfirm i3-gaps i3blocks i3lock i3lock-fancy-git i3status polybar picom feh arandr xorg-xhost thunar || true
	yay -Rns --noconfirm rofi-power-menu i3-battery-popup-git || true
	
	# Install Hyprland ecosystem - GNOME Compatible
	yes|sudo pacman -S --noconfirm --needed hyprland xdg-desktop-portal-hyprland \
		hyprpaper hyprlock hypridle hyprpicker waybar rofi dunst grim slurp \
		wl-clipboard nautilus gnome-control-center qt5-wayland qt6-wayland wlr-randr
	
	# Install AUR packages
	yay --noconfirm --needed -S hyprpolkitagent wlogout
	
	# Remove old configs
	rm -rf ~/.config/i3 ~/.config/polybar ~/.config/picom.conf || true
	sudo rm -rf /etc/X11/xorg.conf.d/30-touchpad.conf || true
	
	# Create Hyprland config directories
	mkdir -p ~/.config/hypr ~/.config/waybar ~/.config/dunst
	
	# Link Hyprland configs
	[ -f ~/.config/hypr/hyprland.conf ] && [ ! -L ~/.config/hypr/hyprland.conf ] && mv ~/.config/hypr/hyprland.conf ~/.config/hypr/hyprland.conf.skabak
	ln -sf /opt/skillarch/config/hypr/hyprland.conf ~/.config/hypr/hyprland.conf
	
	[ -f ~/.config/hypr/hyprpaper.conf ] && [ ! -L ~/.config/hypr/hyprpaper.conf ] && mv ~/.config/hypr/hyprpaper.conf ~/.config/hypr/hyprpaper.conf.skabak
	ln -sf /opt/skillarch/config/hypr/hyprpaper.conf ~/.config/hypr/hyprpaper.conf
	
	[ -f ~/.config/hypr/hyprlock.conf ] && [ ! -L ~/.config/hypr/hyprlock.conf ] && mv ~/.config/hypr/hyprlock.conf ~/.config/hypr/hyprlock.conf.skabak
	ln -sf /opt/skillarch/config/hypr/hyprlock.conf ~/.config/hypr/hyprlock.conf
	
	# Link Waybar configs
	[ -f ~/.config/waybar/config.jsonc ] && [ ! -L ~/.config/waybar/config.jsonc ] && mv ~/.config/waybar/config.jsonc ~/.config/waybar/config.jsonc.skabak
	ln -sf /opt/skillarch/config/waybar/config.jsonc ~/.config/waybar/config.jsonc
	
	[ -f ~/.config/waybar/style.css ] && [ ! -L ~/.config/waybar/style.css ] && mv ~/.config/waybar/style.css ~/.config/waybar/style.css.skabak
	ln -sf /opt/skillarch/config/waybar/style.css ~/.config/waybar/style.css
	
	# Link Dunst config
	[ -f ~/.config/dunst/dunstrc ] && [ ! -L ~/.config/dunst/dunstrc ] && mv ~/.config/dunst/dunstrc ~/.config/dunst/dunstrc.skabak
	ln -sf /opt/skillarch/config/dunst/dunstrc ~/.config/dunst/dunstrc
	
	make clean
```

## 📁 Structure de configuration nécessaire

```
/opt/skillarch/config/
├── hypr/
│   ├── hyprland.conf     # Config principale Hyprland
│   ├── hyprpaper.conf    # Configuration wallpapers
│   └── hyprlock.conf     # Configuration lock screen
├── waybar/
│   ├── config.jsonc      # Configuration waybar
│   └── style.css         # Styles CSS waybar
├── dunst/
│   └── dunstrc           # Configuration notifications
└── rofi/
    └── config.rasi       # Réutiliser config existante (compatible)
```

## ⚠️ Points d'attention

### Docker GUI
**Ancien (X11):**
```bash
-e DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix/
```

**Nouveau (Wayland):**
```bash
--network=host -e WAYLAND_DISPLAY -v $XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY
```

### VirtualBox
- Supprimer les lignes `VBoxClient-all` de la config
- Pas d'équivalent Wayland direct pour VirtualBox guest additions

### Outils de pentest
- La plupart fonctionnent via XWayland automatiquement
- Tester individuellement les outils critiques après migration

### Screenshots
**Ancien:**
```bash
flameshot gui
flameshot full -p ~/Pictures/
```

**Nouveau:**
```bash
grim -g "$(slurp)" ~/Pictures/screenshot.png  # Zone sélectionnée
grim ~/Pictures/screenshot.png               # Écran complet
```

### Keybindings AZERTY
Les raccourcis AZERTY actuels d'i3 doivent être adaptés dans `hyprland.conf` avec la syntaxe Hyprland.

## 🚀 Étapes d'implémentation

1. **Créer les fichiers de configuration Hyprland**
2. **Ajouter le target `install-hyprland` au Makefile**  
3. **Mettre à jour `install` pour utiliser `install-hyprland` au lieu de `install-gui`**
4. **Tester sur un système de développement**
5. **Documenter les changements dans README.md**

## 📝 Notes de migration

- **Session de connexion:** Hyprland apparaîtra dans le gestionnaire de connexion après installation
- **Compatibilité:** Rofi reste compatible, pas besoin de migration
- **Kitty:** Terminal existant reste compatible
- **Performance:** Hyprland est généralement plus performant que i3+picom