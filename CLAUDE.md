# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains Ansible playbooks for provisioning and configuring Arch Linux systems (specifically "Endor"). The playbooks are role-based and support both full system setup and selective component installation via tags.

## Running Playbooks

**Full system setup:**
```bash
ansible-playbook endor.yml
```

**Run specific roles:**
```bash
ansible-playbook endor.yml --tags nodejs
ansible-playbook endor.yml --tags desktop,kde
ansible-playbook endor.yml --tags neovim,lsp
```

**Dry-run (check mode):**
```bash
ansible-playbook endor.yml --check
```

## Architecture

### Role Execution Order

The `endor.yml` playbook runs roles in a specific order that respects dependencies:

1. **system-foundation** - Audio/Bluetooth enhancements and network services (SSH, Avahi)
2. **cli-core** - Installs git, curl, and essential CLI tools (required by subsequent roles)
3. **yay** - Installs AUR helper (requires git from cli-core)
4. **fonts** - Font packages
5. **dev-languages** - C/C++, Zig, Go, Lua, Python, Rust, Nix toolchains
6. **nodejs** - nvm, Node.js LTS, pnpm, bun (requires curl from cli-core)
7. **neovim-setup** - LSP servers, formatters, linters, and dependencies
8. **desktop-environment** - KDE Plasma, Wayland compositors (Noctalia, Niri, Hyprland)
9. **gui-apps** - GUI applications (browsers, 1password, editors, etc.)
10. **system-monitoring** - lm_sensors, coolercontrol
11. **gaming** - Steam, gamemode, mangohud, etc.

**Critical dependency chain:** cli-core → yay → (all other roles that use AUR packages)

### Multi-file Role Structure

Several roles use `include_tasks` to split functionality across multiple files:

- **system-foundation**: `system-enhancements.yml` (audio/bluetooth) + `network-services.yml` (SSH/Avahi)
- **neovim-setup**: `lsp.yml` + `formatters.yml` + `dependencies.yml`
- **desktop-environment**: `kde.yml` + `wayland-shared.yml` + `noctalia.yml` + `niri.yml` + `hyprland.yml`
- **dev-languages**: `main.yml` + `go-tools.yml`

When modifying these roles, check all included task files for consistency.

### Package Management Patterns

**Official repos:** Most roles use the `pacman` module (shorthand) with `update_cache: yes` on the first package task. Some use `community.general.pacman` explicitly.

**AUR packages:** Currently installed via shell commands (`yay -S --noconfirm`). These are marked as non-idempotent with `changed_when: false`.

**Script-based installs:** Node tooling (nvm, pnpm, bun) uses `curl | bash` patterns. These lack version pinning and update mechanisms.

**Known issues (see REFACTORING.md):**
- Mixed usage of `pacman` vs `community.general.pacman`
- AUR installs via shell instead of `kewlfft.aur.aur` module
- No upgrade paths for AUR/npm/Go packages
- `go install` and npm global installs always report changes

## Key Configuration Patterns

### Privilege Escalation

Most package installation tasks use `become: yes`. User-specific tasks (nvm, rustup, npm, go tools) use:
```yaml
become: yes
become_user: "{{ ansible_facts.env.USER }}"
```

### Service Management

System services use `ansible.builtin.systemd_service` with handlers for restarts. Example in `system-foundation/handlers/main.yml`:
```yaml
- name: Restart Bluetooth
  become: true
  ansible.builtin.systemd_service:
    name: bluetooth
    state: restarted
```

### Conditional Execution

Roles check for file existence or system state before making changes. Example:
```yaml
- name: Check if Bluetooth main.conf exists
  ansible.builtin.stat:
    path: /etc/bluetooth/main.conf
  register: bluetooth_conf
```

## Desktop Environment Variants

The `desktop-environment` role supports multiple compositor setups:
- **KDE Plasma** (X11/Wayland traditional desktop)
- **Noctalia Shell** (custom shell, installed from GitHub releases)
- **Niri** (scrollable tiling Wayland compositor)
- **Hyprland** (dynamic tiling Wayland compositor)

Shared Wayland dependencies (pipewire, xdg-desktop-portal, etc.) are in `wayland-shared.yml`.

## Neovim Ecosystem

The `neovim-setup` role installs three categories of tooling:

**LSP servers (lsp.yml):**
- npm-based: typescript-language-server, vscode-langservers-extracted, yaml-language-server, bash-language-server
- pacman: lua-language-server, gopls, rust-analyzer, clangd, terraform-ls

**Formatters/linters (formatters.yml):**
- npm-based: prettier, @fsouza/prettierd
- Go-based: gofumpt, golangci-lint
- pacman: stylua, shfmt

**Additional dependencies (dependencies.yml):**
- npm-based: tree-sitter-cli, neovim (npm package)
- pacman: tree-sitter

All npm operations assume nvm is installed and available. Go tool installs use `go install` without version pinning.

## Post-Installation Manual Steps

Several roles require manual shell configuration (documented in README.md):

**After dev-languages role:**
```bash
export PATH="$HOME/.cargo/bin:$PATH"  # Rust tools
export PATH="$HOME/.local/bin:$PATH"  # uv/Python tools
```

**After nodejs role:**
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Consider automating these via `/etc/profile.d/` snippets in future refactors.

## Refactoring Roadmap

The REFACTORING.md file documents known technical debt and improvement priorities:

**High-priority:**
- Standardize on `community.general.pacman` with `update_cache: true`
- Replace AUR shell commands with `kewlfft.aur.aur` module
- Add explicit upgrade tasks (tagged `maintenance`)
- Pin versions for nvm/bun/pnpm installs
- Make Go/npm installs idempotent with `creates` paths

**Medium-priority:**
- Consolidate desktop portal packages into `wayland-shared.yml`
- Add checksum verification for GitHub release downloads (Noctalia, claude-code)
- Improve role ordering documentation

When making changes, align with these refactoring goals where possible.
