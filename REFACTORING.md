# Refactoring Plan

This repository is in good shape but a few adjustments will make the playbooks more predictable, upgrade-friendly, and flexible. The items below are ordered by impact.

## High-priority improvements
- **Standardize `pacman` usage**: Several roles mix `pacman` and `community.general.pacman` (e.g., `roles/cli-core/tasks/main.yml:4`, `roles/gui-apps/tasks/main.yml:4`, `roles/system-monitoring/tasks/main.yml:4`). Pick one (recommended: `community.general.pacman`) and always enable `update_cache: true` for the first package task in each role to avoid stale indexes.
- **Use an AUR module instead of shelling out**: AUR installs currently rely on `yay -S` shell commands (`roles/gui-apps/tasks/main.yml:15`, `roles/system-monitoring/tasks/main.yml:16`, `roles/gaming/tasks/main.yml:17`). Switching to `kewlfft.aur.aur` or `community.general.pacman` with `state: latest` and `use: yay` makes installs/upgrades idempotent, honors check mode, and centralizes retry/backoff.
- **Add explicit upgrade paths**: Pacman roles install packages but never upgrade them. Add optional `upgrade` tasks (tagged `maintenance`) that run `community.general.pacman: upgrade: true` and `aur` module upgrades so non-`pacman` packages (AUR, npm, Go tools) also get updated in one pass.
- **Harden Node tooling installs**: `curl | bash` installs for nvm/bun/pnpm (`roles/nodejs/tasks/main.yml:8,30,43`) lack version control and update flows. Prefer packaged installs (`pnpm` from the official repo; `bun-bin` via AUR) and expose `node_version`/`bun_version` variables so updates are controlled and reproducible. Keep the `nodejs` role after `cli-core` to ensure `curl`/`git` exist.
- **Make Go installs idempotent**: `go install` tasks always report changes (`roles/dev-languages/tasks/go-tools.yml:3`, `roles/neovim-setup/tasks/formatters.yml:22`) and don’t pin versions. Add a `go_tools` var list with `name`/`version` and use `creates` pointing at `{{ ansible_env.HOME }}/go/bin/<binary>` so re-runs are clean and upgrades are explicit.
- **Guard npm-based updates**: NPM update steps (`roles/neovim-setup/tasks/lsp.yml:14`, `roles/neovim-setup/tasks/formatters.yml:12`, `roles/neovim-setup/tasks/dependencies.yml:9`) assume nvm is present. Gate them with a fact/check on `~/.nvm/nvm.sh`, and consider `community.general.npm` with `executable: "{{ ansible_env.HOME }}/.nvm/versions/node/<version>/bin/npm"` for idempotent `state: latest`.

## Medium-priority improvements
- **Consolidate desktop portal packages**: `xdg-desktop-portal-*` repeats across Wayland/KDE/Hyprland tasks. Move shared portal deps into `wayland-shared.yml` and select compositor-specific portal packages via vars to avoid accidental conflicts.
- **Versioned asset downloads**: Noctalia and claude-code installs (`roles/desktop-environment/tasks/noctalia.yml:13`, `roles/gui-apps/tasks/main.yml:28`) pull “latest” tarballs/scripts with no checksum or update plan. Introduce `noctalia_version`/`claude_code_version`, pin URLs, add checksum verification, and re-run when the version changes.
- **Role ordering and prerequisites**: Keep `cli-core` (for `git`/`curl`) before `yay` and `nodejs`. Consider a small `bootstrap` task that installs `base-devel` and `git` (needed by `yay`) and refreshes pacman keys/cache before any other role runs.
- **Environment exports**: README documents PATH tweaks for Rust and uv; consider adding small `copy` tasks to drop profile snippets (e.g., `/etc/profile.d/lang-paths.sh`) so shell setup is consistent across users and non-interactive sessions.

## Lower-priority improvements
- **Consistent privilege escalation**: Normalize `become: true` across roles and add `become_user` only when required (e.g., `rustup`/`npm`/`go` tasks). This improves readability and predictability.
- **Tagging for selective runs**: Expand tags so users can run `ansible-playbook endor.yml --tags updates` to only run maintenance/upgrades, or `--tags nodejs,neovim` for language tooling only.
- **Testing/CI hooks**: Add a `--check` friendly path by ensuring all shell tasks either support `check_mode: no` or are replaced with modules. A simple lint (`ansible-lint`) run in CI would catch module mix-ups and ordering regressions early.

## Suggested next steps
1. Standardize pacman usage and add upgrade hooks.
2. Swap AUR shell commands for the `aur` module with `use: yay`, then add an `aur_upgrade` task under a `maintenance` tag.
3. Replace script-based installs for pnpm/bun with package-manager installs and add version vars for Node/Bun.
4. Refactor Go/NPM install tasks to be versioned and idempotent, using variables and `creates` paths.
5. Add versioned, checksum-verified installers for Noctalia/claude-code and consolidate portal dependencies.
