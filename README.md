# arch-linux-ansible

Post-install Ansible playbook for Endor (Arch Linux on Ryzen 7 3800X + RX 9070 XT).

This playbook handles **system-level setup only**. User-level packages (CLIs, GUI apps, dev tooling) are managed via [home-manager](https://github.com/nix-community/home-manager) using the flake at [`bondzula/nixcfg`](https://github.com/bondzula/nixcfg).

## First run on a fresh install

```bash
sudo pacman -Syu --needed git ansible
git clone <this-repo-url> ~/arch-linux-ansible
cd ~/arch-linux-ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook endor.yml --ask-become-pass
```

After the playbook finishes, apply the home-manager config:

```bash
nix run github:nix-community/home-manager -- \
  switch --flake github:bondzula/nixcfg#bondzula@endor \
  --extra-experimental-features 'nix-command flakes'
```

Subsequent updates:

```bash
home-manager switch --flake github:bondzula/nixcfg#bondzula@endor
```

## What it does

| Role | What it installs / configures |
|---|---|
| `bootstrap` | pacman.conf tweaks (parallel downloads, color, multilib), `reflector` mirrors, `base-devel`, `git` |
| `system-foundation` | PipeWire + WirePlumber, Bluez, NetworkManager, Tailscale, OpenSSH, firewalld (LocalSend ports), `fstrim.timer`, NTFS userspace tools |
| `gpu-amd` | `amd-ucode`, Mesa, RADV (Vulkan), VA-API/VDPAU, full 32-bit stack for Steam/Proton |
| `cli-core` | sudo-context basics: git, wget, curl, gnupg, zip, unzip, rsync, dnsutils |
| `yay` | Creates `aur_builder` user, bootstraps `yay-bin`, installs `aur_packages` from `group_vars` |
| `fonts` | JetBrains Mono Nerd, FiraCode Nerd, Roboto, Inter, Noto |
| `desktop-environment` | SDDM + KDE Plasma + Niri + Hyprland + Noctalia (toggle individually) |
| `apps` | GUI apps from official repos (Firefox, Ghostty, Obsidian, Discord, Signal, mpv, VLC, Okular, GIMP, Krita) |
| `system-monitoring` | lm_sensors, CoolerControl, CoreCtrl (RDNA 4 power profiles) |
| `gaming` | Steam, Gamescope, GameMode, MangoHud, vkBasalt, Wine (with lib32 variants) |
| `nix` | Nix package manager + nix-daemon, flakes enabled |

User-level CLIs, editors, browsers, dev languages, Node, neovim setup — all live in the [`bondzula/nixcfg`](https://github.com/bondzula/nixcfg) home-manager flake.

## Common runs

```bash
# Just one role
ansible-playbook endor.yml --tags desktop,hyprland

# Disable a desktop for this run
ansible-playbook endor.yml -e desktop_kde_enabled=false

# Dry run
ansible-playbook endor.yml --check
```

## Configuration

Edit `group_vars/all.yml` to:

- Toggle desktops: `desktop_kde_enabled`, `desktop_niri_enabled`, `desktop_hyprland_enabled`, `desktop_noctalia_enabled`
- Edit package lists: `aur_packages`, `apps_packages`

## Layout

```
endor.yml                main playbook
group_vars/all.yml       variables
requirements.yml         Ansible collections
roles/<name>/tasks/*.yml task files (multi-file roles use include_tasks)
```
