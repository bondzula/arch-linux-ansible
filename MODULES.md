# Project Modules and Steps

This project is an Ansible post-install setup for an Arch Linux host named **Endor**. The main playbook is `endor.yml`; it runs locally, scrubs Nix paths from `PATH` during execution, applies roles in order, then prints the Home Manager command for user-level configuration.

## 1. Bootstrap

**Role:** `roles/bootstrap`

Steps:
1. Enable pacman parallel downloads.
2. Enable pacman colored output.
3. Enable the `[multilib]` repository.
4. Remove any old Ansible-managed multilib block.
5. Refresh pacman cache if multilib changed.
6. Install base build prerequisites: `base-devel`, `git`, `reflector`.
7. Refresh Arch mirrors with `reflector`.
8. Refresh pacman cache again.
9. Optionally run a full system upgrade with the `maintenance` tag.

## 2. System Foundation

**Role:** `roles/system-foundation`

Steps:
1. Install audio and Bluetooth stack: PipeWire, WirePlumber, RTKit, BlueZ.
2. Enable Bluetooth experimental features when `/etc/bluetooth/main.conf` exists.
3. Enable `rtkit-daemon` and `bluetooth` services.
4. Install network services: NetworkManager, Tailscale, OpenSSH, firewalld.
5. Enable `NetworkManager`, `tailscaled`, `sshd`, and `firewalld`.
6. Open LocalSend TCP/UDP port `53317` in firewalld.
7. Enable `fstrim.timer` for SSD trimming.
8. Install `ntfs-3g` for NTFS userspace support.

## 3. AMD GPU

**Role:** `roles/gpu-amd`

Steps:
1. Install AMD microcode.
2. Install Mesa and RADV Vulkan drivers.
3. Install Vulkan loader/tools.
4. Install VA-API utilities.
5. Install 32-bit Mesa/Vulkan stack for Steam and Proton.

## 4. CLI Core

**Role:** `roles/cli-core`

Steps:
1. Install common command-line tools: `wget`, `curl`, `git`, `gnupg`, `zip`, `unzip`, `rsync`, `dnsutils`.
2. Install `cliphist` for clipboard history support.

## 5. AUR / yay

**Role:** `roles/yay`

Steps:
1. Create a locked system user named `aur_builder`.
2. Grant `aur_builder` passwordless access to `/usr/bin/pacman` only.
3. Install `yay-bin` using `makepkg`.
4. Install AUR packages listed in `group_vars/all.yml` under `aur_packages`.
5. Optionally upgrade all AUR packages with the `maintenance` tag.

## 6. Fonts

**Role:** `roles/fonts`

Steps:
1. Install Roboto, Inter, JetBrains Mono Nerd Font, FiraCode Nerd Font, Noto fonts, emoji fonts, and CJK fonts.
2. Rebuild the font cache when fonts are changed.

## 7. Desktop Environment

**Role:** `roles/desktop-environment`

Steps:
1. Install shared Wayland/display-manager packages when any desktop is enabled.
2. Enable SDDM without starting it immediately.
3. If enabled, install KDE Plasma, KDE applications, and KDE portal.
4. If enabled, install Niri, Xwayland Satellite, GNOME portal, and Niri portal config.
5. If enabled, install Hyprland, Hyprland portal, and companion tools like Waybar, Rofi, Dunst, Hyprpaper, Hyprlock, Hyprpicker, and Hypridle.
6. If enabled, install Noctalia runtime extras and remove old local Noctalia config.

Desktop toggles are configured in `group_vars/all.yml`:

- `desktop_kde_enabled`
- `desktop_niri_enabled`
- `desktop_hyprland_enabled`
- `desktop_noctalia_enabled`

## 8. Applications

**Role:** `roles/apps`

Steps:
1. Install GUI applications from official Arch repositories.
2. The package list is controlled by `apps_packages` in `group_vars/all.yml`.

## 9. System Monitoring

**Role:** `roles/system-monitoring`

Steps:
1. Install `lm_sensors` and `corectrl`.
2. Check if `coolercontrol-bin` is installed from AUR.
3. Enable `coolercontrold` only when `coolercontrol-bin` is present.

## 10. Gaming

**Role:** `roles/gaming`

Steps:
1. Install Steam, Gamescope, GameMode, MangoHud, GOverlay, Wine, and Winetricks.
2. Install 32-bit GameMode and MangoHud support.
3. Add the target user to the `gamemode` group.
4. Enable the user-level `gamemoded.service`.

## 11. Nix

**Role:** `roles/nix`

Steps:
1. Install Nix from official Arch repositories.
2. Run `systemd-sysusers` to create Nix build users/groups.
3. Ensure `/etc/nix` exists.
4. Write `/etc/nix/nix.conf` with flakes and `nix-command` enabled.
5. Enable and start `nix-daemon.service`.
6. Restart `nix-daemon` when Nix configuration changes.

## Configuration Files

- `endor.yml` — main local playbook and role order.
- `group_vars/all.yml` — user, desktop toggles, AUR packages, and app packages.
- `requirements.yml` — required Ansible collections.
- `roles/*/tasks/*.yml` — implementation for each module.

## Common Commands

```bash
# Install required collections
ansible-galaxy collection install -r requirements.yml

# Run the full playbook
ansible-playbook endor.yml --ask-become-pass

# Run selected desktop tasks
ansible-playbook endor.yml --tags desktop,hyprland

# Disable one desktop for a run
ansible-playbook endor.yml -e desktop_kde_enabled=false

# Dry run
ansible-playbook endor.yml --check

# Maintenance upgrade tasks
ansible-playbook endor.yml --tags maintenance --ask-become-pass
```
