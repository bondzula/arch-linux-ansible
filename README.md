# Endor System Setup

Ansible playbooks for setting up Endor (Arch Linux).

## Usage

Run the main playbook:
```bash
ansible-playbook endor.yml
```

Or run specific roles:
```bash
ansible-playbook endor.yml --tags nodejs
```

## Roles

### cli-core
Installs essential CLI tools: wget, curl, git, gnupg, ripgrep, fd, zip, bat, btop, eza, rsync, tree, jq, httpie, tealdeer, procs, fastfetch, wakeonlan, neovim, fzf, direnv, aws-cli-v2, aws-vault, terraform.

### yay
Installs yay AUR helper for managing AUR packages.

### fonts
Installs fonts: ttf-roboto, inter-font, ttf-jetbrains-mono-nerd, ttf-firacode-nerd.

### dev-languages
Installs programming language tooling:
- **C/C++**: gcc, clang, cmake
- **Zig**: zig compiler
- **Go**: go compiler and tooling
- **Lua**: lua, luarocks, luajit
- **Python**: python, uv (version + package manager)
- **Rust**: rustup (toolchain manager)
- **Nix**: nix package manager

**Post-installation:** Add to your `~/.zshrc`:
```bash
export PATH="$HOME/.cargo/bin:$PATH"  # Rust tools
export PATH="$HOME/.local/bin:$PATH"  # uv/Python tools
```

### gui-apps
Installs GUI applications:
- **Official repos**: ghostty, firefox, chromium, discord, obs-studio, zed
- **AUR**: 1password, localsend-bin, vscodium-bin
- **Native installers**: claude-code, openai-codex

### system-monitoring
Installs system monitoring tools:
- **Official repos**: lm_sensors
- **AUR**: coolercontrol-bin

Runs sensors-detect and enables the coolercontrold daemon.

### nodejs
Sets up nvm (Node Version Manager) with Node.js LTS.

**Post-installation:** Add the following to your `~/.zshrc`:
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Then reload: `source ~/.zshrc`

### gaming
Installs gaming tools: steam, gamescope, gamemode, mangohud, goverlay, heroic-games-launcher, protonup-qt.
