# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## SkillArch Overview

SkillArch is a Linux penetration testing and cybersecurity distribution built on CachyOS (Arch-based). It provides a complete environment with offensive security tools, development tools, and a customized i3 window manager setup.

## Key Commands

### Installation and Management
- `make install` - Full SkillArch installation (requires sudo)
- `make help` - Show all available make targets
- `make update` - Update SkillArch from upstream (requires clean git state)
- `ska-update-simple` - Simple update helper (pull + install)
- `ska-update-advanced` - Advanced update helper for forked repos

### Component Installation
- `make install-base` - Install base packages and setup
- `make install-cli-tools` - Install CLI tools and development environment
- `make install-shell` - Install zsh, oh-my-zsh, and shell configuration
- `make install-docker` - Install and configure Docker
- `make install-gui` - Install i3, polybar, kitty, rofi, and GUI components
- `make install-gui-tools` - Install GUI applications
- `make install-offensive` - Install penetration testing tools
- `make install-wordlists` - Install security wordlists
- `make install-hardening` - Install security hardening tools

### Docker Commands
- `make docker-build` - Build lite Docker image
- `make docker-build-full` - Build full Docker image with GUI
- `make docker-run` - Run lite Docker container
- `make docker-run-full` - Run full Docker container with X11

### Helper Commands
- `ska-help-aliases` - Fuzzy search through available aliases
- `ska-help-bindings` - Fuzzy search through i3 key bindings
- `ska-help-packages` - Fuzzy search through installed packages
- `ska-sudo-unlock` - Unlock user after failed sudo attempts

## Architecture and Structure

### Modular Installation Pipeline
The Makefile follows a strict dependency chain ensuring proper component ordering:
```
install: install-base → install-cli-tools → install-shell → install-docker → install-gui → install-gui-tools → install-offensive → install-wordlists → install-hardening
```

**Key architectural principles:**
- Each component can be installed independently
- All targets require `sanity-check` (must run from `/opt/skillarch` with sudo access)
- Progressive complexity: base system → CLI → GUI → specialized security tools
- Cleanup performed automatically after each major installation phase

### Directory Structure
- `/opt/skillarch/` - Main installation directory
- `/opt/skillarch/config/` - Configuration files for all applications  
- `/opt/skillarch/assets/` - Images and assets
- `/opt/lists/` - Security wordlists and payloads organized by purpose
- `/opt/[tool-name]/` - Cloned security tools (chisel, phpggc, CloudFlair, etc.)

### Configuration Management System
Centralized configuration with atomic symlink operations from `/opt/skillarch/config/`:
- `config/zshrc` → `~/.zshrc`
- `config/vimrc` → `~/.vimrc`
- `config/tmux.conf` → `~/.tmux.conf`
- `config/i3/config` → `~/.config/i3/config`
- `config/hypr/hyprland.conf` → `~/.config/hypr/hyprland.conf` (Wayland alternative)
- `config/kitty/kitty.conf` → `~/.config/kitty/kitty.conf`
- `config/nvim/init.lua` → `~/.config/nvim/init.lua`
- `config/polybar/` → `~/.config/polybar/`
- `config/rofi/` → `~/.config/rofi/`

**Backup strategy**: Existing configs moved to `.skabak` files before symlinking

### Multi-Layered Package Management
**System Level** (`pacman`/`yay`):
- Core packages via pacman with parallel downloads enabled  
- AUR packages via yay with chaotic-aur repository configured
- Optimized builds in tmpfs (`/dev/shm/makepkg`) for performance

**Language-Specific** (`mise`):
- Version management for Go, Rust, Node.js, Python, PHP, Terraform
- Go security tools installed via `mise exec -- go install`
- Isolated environments prevent version conflicts

**Python Isolation** (`pipx`):
- Security tools in isolated virtual environments: sqlmap, dirsearch, exegol, semgrep
- Setuptools injected to handle dependency issues

**Security Ecosystem**:
- **PDTM**: ProjectDiscovery tools (nuclei, subfinder, httpx, dnsx)
- **Direct clones**: Custom tools in `/opt/[tool]/` for easy updates
- **Wordlists**: Curated collections (rockyou, SecLists, PayloadsAllTheThings) in `/opt/lists/`

### Docker Multi-Stage Architecture
**Two-tier build strategy**:
- **Dockerfile-lite**: Base CLI environment (cachyos → hacker user → base/cli/shell/offensive)
- **Dockerfile-full**: Extends lite with GUI components (docker/gui/gui-tools/wordlists/hardening)

**Security model**: NOPASSWD sudo only during installation, reverted to password-required afterward

## Development Environment

### Shell Environment
- **Default Shell**: zsh with oh-my-zsh framework  
- **Theme**: af-magic (not Powerlevel10k)
- **Key Plugins**: fzf (fuzzy finding), mise (language versions), docker, terraform, npm, zsh-autosuggestions, zsh-syntax-highlighting
- **Aliases**: 200+ organized aliases in `config/aliases` (sourced by zshrc)
- **Helper Commands**: `ska-*` prefixed commands for system management

### Multi-Desktop Environment Support  
**Primary**: i3-gaps ecosystem
- **Terminal**: Kitty with Catppuccin theme
- **Window Manager**: i3-gaps with AZERTY layout bindings
- **Status Bar**: Polybar with system monitoring
- **Launcher**: Rofi for applications
- **Compositor**: Picom (auto-disabled in hypervisor environments)

**Alternative**: Hyprland (Wayland)
- **Compositor**: Hyprland with GNOME coexistence
- **Status Bar**: Waybar 
- **Notifications**: Dunst
- **Background**: Hyprpaper

### Editor Configuration
- **Neovim**: LazyVim starter configuration with custom `init.lua`
- **VSCode**: Extensions auto-installed from `config/extensions.txt`
- **Vim**: Custom configuration in `config/vimrc`

## Important Notes

### Git Workflow
- Main branch is `main`
- Repository should be kept clean for updates
- Users can fork and maintain custom configurations
- Never commit secrets or sensitive information

### Docker Usage
- Lite image: CLI tools only
- Full image: Includes GUI tools and wordlists
- X11 forwarding supported for GUI applications

### Multi-Monitor Setup
- Use `arandr` for display configuration
- Save layout as `~/.screenlayout/arandr-main-layout.sh`
- Auto-apply with `~/.xprofile`

### Security Considerations
- OpenSnitch for egress firewall (opt-in)
- UFW for ingress firewall
- Docker bypasses UFW rules by default
- User added to docker group (logout required)

### VM/VirtualBox Notes
- VirtualBox guest utilities available via `ska-vbox-install-guestutils`
- Picom disabled in hypervisor environments for performance
- Recommended to use GNOME Boxes instead of VirtualBox

## File Locations

### Configuration Files
- Main aliases: `config/aliases`
- i3 configuration: `config/i3/config`
- VSCode extensions: `config/extensions.txt`
- Chrome extensions list: `config/chrome-extensions.lst`

### Important Paths
- Wordlists: `/opt/lists/`
- Custom tools: `/opt/[tool-name]/`
- Long-lived data: `/DATA/`
- Trash: `/.Trash/`

## Update and Maintenance Workflows

### Two-Tier Update Strategy
**Simple Updates** (`ska-update-simple`):
- For users wanting upstream changes without customization
- Requires clean git state
- Workflow: `cd /opt/skillarch && git pull && make install`

**Advanced Updates** (`ska-update-advanced`):  
- For users with forked repositories and custom modifications
- Merges upstream changes while preserving customizations
- Handles git conflicts and drift analysis

### Maintenance and Cleanup
- **`make clean`**: Removes caches, logs, temporary files
- **Package hygiene**: Orphan removal, cache clearing via `paccache`, `yay -Yc`
- **Docker cleanup**: System prune, image cleanup
- **Log management**: Journal vacuum, log truncation

## Testing and Validation
No specific test framework - SkillArch is primarily a system configuration and tool collection. Validation through:
- Successful modular component installation
- Docker multi-stage builds (lite/full images)
- CI/CD security scanning (Semgrep, Trivy, Gitleaks, TruffleHog)  
- Manual testing of desktop environments and tool functionality

## Security Architecture
- **OpenSnitch**: Egress firewall (opt-in, should be enabled by default)
- **UFW**: Ingress firewall (Docker bypasses by default)
- **Docker group**: Grants root-equivalent access (security consideration)
- **Secret management**: Never commit credentials; use environment variables
- **Multi-layer scanning**: Automated security checks in CI/CD pipeline