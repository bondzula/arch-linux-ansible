# arch-linux-ansible

Post-install Ansible playbook for Endor (Arch Linux on Ryzen 7 3800X + RX 9070 XT).

## First run on a fresh install

```bash
sudo pacman -Syu --needed git ansible
git clone <this-repo-url> ~/arch-linux-ansible
cd ~/arch-linux-ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook endor.yml --ask-become-pass
```

> **Sudo note:** AUR installs (via `yay`) shell out to `sudo pacman` internally. Either keep the terminal interactive to type your sudo password when yay prompts, or add `NOPASSWD: /usr/bin/pacman` for your user in `/etc/sudoers.d/`.

## What it does

| Role | What it installs / configures |
|---|---|
| `bootstrap` | pacman.conf tweaks (parallel downloads, color, multilib), `reflector` mirrors, `base-devel`, `git` |
| `system-foundation` | PipeWire + WirePlumber, Bluez, NetworkManager, Tailscale, OpenSSH, firewalld (LocalSend ports), `fstrim.timer` |
| `gpu-amd` | `amd-ucode`, Mesa, RADV (Vulkan), VA-API/VDPAU, full 32-bit stack for Steam/Proton |
| `cli-core` | git, gh, ripgrep, fd, bat, btop, eza, jq, fzf, direnv, neovim, tmux, zoxide, aws-cli, terraform |
| `yay` | Bootstraps `yay-bin` AUR helper |
| `fonts` | JetBrains Mono Nerd, FiraCode Nerd, Roboto, Inter, Noto |
| `dev-languages` | C/C++, Zig, Go, Lua, Python (uv), Rust (rustup), PHP, Nix; drops `/etc/profile.d/lang-paths.sh` |
| `nodejs` | Pinned nvm, Node LTS, pnpm (repo), bun-bin (AUR); drops `/etc/profile.d/nvm.sh` |
| `neovim-setup` | LSPs, formatters, linters, tree-sitter, lazygit |
| `desktop-environment` | SDDM + KDE Plasma + Niri + Hyprland + Noctalia (toggle individually) |
| `gui-apps` | Ghostty, Firefox, Chromium, Discord, OBS, Zed, 1Password, VSCodium, Claude Code, Codex |
| `system-monitoring` | lm_sensors, CoolerControl, CoreCtrl (RDNA 4 power profiles) |
| `gaming` | Steam, Gamescope, GameMode, MangoHud, vkBasalt, Wine, Heroic, ProtonUp-Qt (with lib32 variants) |

## Common runs

```bash
# Just one role
ansible-playbook endor.yml --tags nodejs
ansible-playbook endor.yml --tags desktop,hyprland

# Update everything (pacman + AUR + npm + Go tools)
ansible-playbook endor.yml --tags maintenance

# Disable a desktop for this run
ansible-playbook endor.yml -e desktop_kde_enabled=false

# Dry run
ansible-playbook endor.yml --check
```

## Configuration

Edit `group_vars/all.yml` to:

- Toggle desktops: `desktop_kde_enabled`, `desktop_niri_enabled`, `desktop_hyprland_enabled`, `desktop_noctalia_enabled`
- Pin versions: `nvm_version`, `node_install_version`, `noctalia_version`
- Add/remove Go tools (idempotent install via `creates`; upgrade with `--tags maintenance`)

## Layout

```
endor.yml                main playbook
group_vars/all.yml       variables
requirements.yml         Ansible collections
roles/<name>/tasks/*.yml task files (multi-file roles use include_tasks)
```
