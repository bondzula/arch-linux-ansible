# Project Modules and Steps

This project is an Ansible post-install setup for an Arch Linux host named **Endor**. The main playbook is `endor.yml`; it runs locally, scrubs Nix paths from `PATH` during execution, applies roles in order, then prints the Home Manager command for user-level configuration.

## 1. Bootstrap

**Role:** `roles/bootstrap`

Steps:
1. Enable pacman parallel downloads.
2. Enable pacman colored output.
3. Enable the `[multilib]` repository.
4. Refresh pacman cache if multilib changed.
5. Install base build prerequisites: `base-devel`, `git`, `reflector`.
6. Refresh Arch mirrors with `reflector`.
7. Refresh pacman cache again.
8. Optionally run a full system upgrade with the `maintenance` tag.

## 2. CachyOS Repos

**Role:** `roles/cachyos-repos`

Steps:
1. Import and locally sign the CachyOS GPG signing key.
2. Install `cachyos-keyring`, `cachyos-mirrorlist`, and `cachyos-v3-mirrorlist` from the official CachyOS mirror.
3. Insert `[cachyos-v3]`, `[cachyos-core-v3]`, `[cachyos-extra-v3]`, and `[cachyos]` repos into `pacman.conf` *before* `[core]` so pacman prefers the x86-64-v3 optimized builds (Zen 2+ supports v3).
4. Refresh pacman cache.

## 3. System Foundation

**Role:** `roles/system-foundation`

Steps:
1. Install audio and Bluetooth stack: PipeWire, WirePlumber, RTKit, BlueZ.
2. Enable Bluetooth experimental features when `/etc/bluetooth/main.conf` exists.
3. Enable `rtkit-daemon` and `bluetooth` services.
4. Install network services: NetworkManager, Tailscale, OpenSSH, firewalld.
5. Enable `NetworkManager`, `tailscaled`, `sshd`, and `firewalld`.
6. Open LocalSend TCP/UDP port `53317` in firewalld.
7. Install and enable `tuned` with the `latency-performance` profile.
8. Drop a low-latency PipeWire config at `/etc/pipewire/pipewire.conf.d/10-lowlatency.conf` (512/48000 quantum).
9. Enable `fstrim.timer` for SSD trimming.
10. Install `ntfs-3g` for NTFS userspace support.

## 4. AMD GPU

**Role:** `roles/gpu-amd`

Steps:
1. Install AMD microcode.
2. Install Mesa and RADV Vulkan drivers.
3. Install Vulkan loader/tools.
4. Install VA-API utilities.
5. Install 32-bit Mesa/Vulkan stack for Steam and Proton.

## 5. Kernel

**Role:** `roles/kernel`

Steps:
1. Install `linux-cachyos-bore`, `linux-cachyos-bore-headers`, `scx-scheds` (scheduler binaries), and `scx-tools` (scx_loader DBus daemon + service unit). BORE is an interactive-tuned EEVDF variant; sched-ext is compiled in so scx_loader can layer `scx_lavd` on top. If scx fails to load, BORE handles scheduling.
2. Write `/etc/scx_loader.toml` selecting `scx_lavd` as the default sched-ext scheduler.
3. Enable `scx_loader.service` so it starts on next boot into the CachyOS kernel.

The stock Arch `linux` kernel is left installed as a fallback GRUB entry.

## 6. Kernel Cmdline

**Role:** `roles/kernel-cmdline`

Steps:
1. Render `kernel_cmdline_params` from `group_vars/all.yml` into `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub` (with backup).
2. Handler regenerates `/boot/grub/grub.cfg` via `grub-mkconfig` when the cmdline changes.

The default params disable CPU mitigations, enable `amd_pstate=active`, cap C-states for low latency, unlock the AMDGPU power-management surface, and remove watchdog overhead. Edit `kernel_cmdline_params` in `group_vars/all.yml` to change.

## 7. ZRAM

**Role:** `roles/zram`

Steps:
1. Install `zram-generator`.
2. Write `/etc/systemd/zram-generator.conf`: a `zram0` device sized at `ram / 2`, zstd compression, swap-priority 100.
3. Write `/etc/sysctl.d/99-zram.conf`: `vm.swappiness=180`, `vm.watermark_boost_factor=0`, `vm.watermark_scale_factor=125`, `vm.page-cluster=0` (CachyOS-recommended for zram-as-primary-swap).
4. Handlers reload the zram device and apply sysctls.

## 8. CLI Core

**Role:** `roles/cli-core`

Steps:
1. Install common command-line tools: `wget`, `curl`, `git`, `gnupg`, `zip`, `unzip`, `rsync`, `dnsutils`.
2. Install `cliphist` for clipboard history support.

## 9. AUR / yay

**Role:** `roles/yay`

Steps:
1. Create a locked system user named `aur_builder`.
2. Grant `aur_builder` passwordless access to `/usr/bin/pacman` only.
3. Install `yay-bin` using `makepkg`.
4. Install AUR packages listed in `group_vars/all.yml` under `aur_packages`.
5. Optionally upgrade all AUR packages with the `maintenance` tag.

## 10. Fonts

**Role:** `roles/fonts`

Steps:
1. Install Roboto, Inter, JetBrains Mono Nerd Font, FiraCode Nerd Font, Noto fonts, emoji fonts, and CJK fonts.
2. Rebuild the font cache when fonts are changed.

## 11. Desktop Environment

**Role:** `roles/desktop-environment`

Steps:
1. Install shared Wayland/display-manager packages when any desktop is enabled.
2. Enable SDDM without starting it immediately.
3. If enabled, install KDE Plasma, KDE applications, and KDE portal.
4. If enabled, install Niri, Xwayland Satellite, GNOME portal, and Niri portal config.
5. If enabled, install Hyprland, Hyprland portal, and companion tools like Waybar, Rofi, Dunst, Hyprpaper, Hyprlock, Hyprpicker, and Hypridle.
6. If enabled, install Noctalia runtime extras.

Desktop toggles are configured in `group_vars/all.yml`:

- `desktop_kde_enabled`
- `desktop_niri_enabled`
- `desktop_hyprland_enabled`
- `desktop_noctalia_enabled`

## 12. Applications

**Role:** `roles/apps`

Steps:
1. Install GUI applications from official Arch repositories.
2. The package list is controlled by `apps_packages` in `group_vars/all.yml`.

## 13. System Monitoring

**Role:** `roles/system-monitoring`

Steps:
1. Install `lm_sensors` and `corectrl`.
2. Check if `coolercontrol-bin` is installed from AUR.
3. Enable `coolercontrold` only when `coolercontrol-bin` is present.

## 14. Gaming

**Role:** `roles/gaming`

Steps:
1. Install Steam, Gamescope, GameMode, MangoHud, GOverlay, Wine, Winetricks, and `ananicy-cpp`.
2. Install 32-bit GameMode and MangoHud support.
3. Install `proton-cachyos` from the cachyos repo.
4. Add the target user to the `gamemode` group.
5. Enable the user-level `gamemoded.service`.
6. Enable `ananicy-cpp.service` for automatic process renicing.
7. Append `RADV_PERFTEST=gpl,sam` and `mesa_glthread=true` to `/etc/environment`.

## 15. Nix

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
