# NixOS Config

Personal NixOS configuration based on [nixferatu](https://github.com/eduardofuncao/nixferatu) — Niri + Stylix + Home Manager.

## What's Inside

- **Niri** — scrollable-tiling Wayland compositor
- **Neovim** — editor
- **Yazi** — file manager
- **Kitty** — terminal
- **Fish** — shell
- **Stylix** — system-wide theming
- **Waybar** — status bar

## Repo Structure
```
.
├── flake.nix                 # Flake entrypoint
├── flake.lock                # Pinned dependencies
├── nixos/
│   ├── configuration.nix     # System-level NixOS config
│   └── hardware-configuration.nix  # Hardware-specific (machine-dependent)
├── home-manager/
│   └── home.nix              # User-level home-manager config
├── modules/                  # Modular configs (nixos + home-manager)
└── dotfiles/                 # Manually managed dotfiles (~/.config)
    ├── niri/
    ├── nvim/
    ├── fish/
    ├── ripgrep/
    ├── scripts/
    ├── swayidle/
    ├── wallpapers/
    └── kanata.kbd
```

## Fresh Install

### 1. Install base NixOS

Boot from NixOS minimal ISO, then:
```sh
# Partition (adjust /dev/nvme0n1 to your disk)
parted /dev/nvme0n1 -- mklabel gpt
parted /dev/nvme0n1 -- mkpart ESP fat32 1MiB 1GiB
parted /dev/nvme0n1 -- set 1 esp on
parted /dev/nvme0n1 -- mkpart root ext4 1GiB 100%

# Format
mkfs.fat -F 32 -n BOOT /dev/nvme0n1p1
mkfs.ext4 -L nixos /dev/nvme0n1p2

# Mount
mount /dev/disk/by-label/nixos /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-label/BOOT /mnt/boot

# Generate hardware config
nixos-generate-config --root /mnt
```

### 2. Clone this repo
```sh
nix-shell -p git
git clone https://github.com/YOUR_USERNAME/nixos-config /mnt/etc/nixos-config
```

### 3. Set up /etc/nixos
```sh
cp /mnt/etc/nixos/hardware-configuration.nix /tmp/
rm -rf /mnt/etc/nixos/*
cp -rT /mnt/etc/nixos-config /mnt/etc/nixos
mv /tmp/hardware-configuration.nix /mnt/etc/nixos/nixos/
```

> **Note:** `hardware-configuration.nix` is machine-specific. Always use the freshly generated one for new machines.

### 4. Install
```sh
NIX_CONFIG="experimental-features = nix-command flakes" nixos-install --flake /mnt/etc/nixos#nixos --no-root-passwd
reboot
```

### 5. Post-install: apply home-manager and dotfiles
```sh
# Apply home-manager config
cd /etc/nixos
nix-shell -p home-manager
home-manager switch --flake .#swad@nixos

# Copy dotfiles
cp -r /etc/nixos/dotfiles/niri ~/.config/
cp -r /etc/nixos/dotfiles/nvim ~/.config/
cp -r /etc/nixos/dotfiles/fish ~/.config/
cp -r /etc/nixos/dotfiles/ripgrep ~/.config/
cp -r /etc/nixos/dotfiles/scripts ~/.config/
cp -r /etc/nixos/dotfiles/swayidle ~/.config/
cp -r /etc/nixos/dotfiles/wallpapers ~/.config/
sudo cp /etc/nixos/dotfiles/kanata.kbd /etc/

# Reboot
sudo reboot
```

## Day-to-Day Usage

### Rebuild after editing NixOS config
```sh
sudo nixos-rebuild switch --flake /etc/nixos#nixos
```

### Rebuild after editing home-manager config
```sh
home-manager switch --flake /etc/nixos#swad@nixos
```

### Install a new system package

Edit `/etc/nixos/nixos/configuration.nix` (or the relevant module in `modules/`), add the package to `environment.systemPackages`, then rebuild:
```sh
# Example: add htop
# In configuration.nix, find environment.systemPackages and add `pkgs.htop`
sudo nixos-rebuild switch --flake /etc/nixos#nixos
```

### Install a user-level package via home-manager

Edit `/etc/nixos/home-manager/home.nix`, add to `home.packages`, then:
```sh
home-manager switch --flake /etc/nixos#swad@nixos
```

### Temporarily use a package without installing
```sh
nix-shell -p <package-name>
# or with flakes:
nix shell nixpkgs#<package-name>
```

### Search for packages
```sh
nix search nixpkgs <query>
```

### Update all flake inputs
```sh
cd /etc/nixos
sudo nix flake update
sudo nixos-rebuild switch --flake .#nixos
home-manager switch --flake .#swad@nixos
```

### Rollback to a previous generation
```sh
# List generations
sudo nix-env --list-generations --profile /nix/var/nix/profiles/system

# Rollback one generation
sudo nixos-rebuild switch --rollback

# Or boot into a previous generation from the bootloader menu
```

### Garbage collect old generations
```sh
sudo nix-collect-garbage -d
nix-collect-garbage -d
```
