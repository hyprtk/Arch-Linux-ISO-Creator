# Changelog

All notable changes to **Hyprtk-ISO-Creator** are documented in this file.
Dates are in `YYYY-MM-DD` format.

## [Unreleased] - 2026-09-22

### Changed

- **Repository renamed** `Arch-Linux-ISO-Creator` → `Hyprtk-ISO-Creator`
  (<https://github.com/hyprtk/Hyprtk-ISO-Creator>). The README clone URL and the
  `git` remote are updated; GitHub keeps a redirect from the old name.
- **Builder renamed** `arch-iso-builder.sh` → `hyprtk-iso-builder.sh`.
- **Default hyprtk source is now `~/hyprtk`**, falling back to cloning
  `hyprtk/dotfiles` when it is absent. `--hyprtk-dir` / `$HYPRTK_DIR` still take
  precedence.

### Added

- **`hyprtk-usb`** — a host-side USB writer. It `dd`s the iso-hybrid (MBR) ISO to
  a disk and appends a 1 MiB-aligned `hyprtk-persist` ext4 partition in the free
  space, preserving the ISO's two MBR entries (iso9660 + EFI). Flags:
  `--no-persist`, `--size 8G|50%|rest`, `--refresh` (keep an existing persistence
  partition), `--dry-run`. Refuses partitions, mounted disks, the `/` disk,
  too-small targets and bad sizes. The equivalent manual procedure is documented
  in the README.
- **`CHANGELOG.md`** (this file).

### Fixed

- **Papirus icons and their colour scripts were missing from the ISO.** The skel
  builder trimmed `assets/papirus-icons/` out of `~/hyprtk`, which also removed
  `papirus-folders.sh` and the `hyprtk-*.sh` recolour scripts, so the icon-colour
  menu did nothing. They are baked into `/etc/skel` again (the ISO grows by only
  ~4 MiB because squashfs dedups the icon data against the packaged
  `papirus-icon-theme`).

## [2026-09-21]

### Added

- **Live ISO** from the archiso `releng` profile with the hyprtk desktop
  preinstalled: every package in `packages.hyprtk`, the host-built AUR extras,
  matuwall built from source, the dotfiles vendored into `/etc/skel`, and SDDM
  autologin straight into Hyprland for the live user `hyprtk`.
- **`hyprtk-deploy`** — the offline live-to-disk installer (GPT single disk,
  `rsync` clone, GRUB for UEFI + BIOS, identity setup), with non-interactive flags
  (`--target`, `--user`, `--hostname`, … `--yes`) and a refusal to target the live
  medium.
- The `/etc/skel` tree is packed as `skel.tar` (mkarchiso strips overlay modes)
  and `oh-my-zsh` + its three plugins are baked for offline first login.
- The per-user pywal cache is provisioned at install time so the first boot is
  already themed.

### Fixed

- matuwall built from the new C/meson source; AUR GPG verification handled;
  staged AUR packages installed one at a time through a throwaway `pacman.conf`
  (`CheckSpace` off, `SigLevel = Never`).
- `/etc/skel` and `/usr/lib/os-release` staged under `/usr/share/hyprtk-iso/` to
  avoid `pacstrap` file conflicts.

## [2026-02-23]

### Added

- Initial `arch-iso-builder.sh`, building the stock archiso `releng` profile.
