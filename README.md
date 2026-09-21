# Arch-Linux-ISO-Creator

Build a **live Arch Linux ISO with the hyprtk desktop preinstalled and
preconfigured** — a pywal-themed Hyprland session with hyprtk-bar, ready to use
the moment it boots.

The builder starts from the stock archiso `releng` profile and adds:

- every package in [`packages.hyprtk`](packages.hyprtk), verified against your
  sync databases
- the AUR extras in [`aur-packages.txt`](aur-packages.txt), built on the host
  (best-effort, non-fatal)
- **matuwall** (the wallpaper picker), built from source on the host
- the hyprtk dotfiles vendored into `/etc/skel` — every new user gets a
  preconfigured `~/hyprtk` plus the `~/.config` symlinks
- **SDDM autologin** straight into Hyprland for the live user `hyprtk`

## Requirements

- Arch Linux (or an Arch-based host) with `sudo`
- Network access during the build (package download + `dbus-next` for the bar)
- Enough free space: roughly **15 GB** in `/tmp` and **5-8 GB** for the ISO
- For AUR extras: `base-devel` on the host, plus each package's makedepends

## Usage

```bash
git clone https://github.com/hyprtk/Arch-Linux-ISO-Creator.git
cd Arch-Linux-ISO-Creator
./arch-iso-builder.sh
```

Run it as your normal user — the script re-execs itself with `sudo` because
`mkarchiso` needs root. When it finishes the ISO is written to your home
directory.

### Options

| Option | Meaning |
| --- | --- |
| `-y`, `--yes` | Skip the confirmation prompt |
| `--hyprtk-dir DIR` | hyprtk dotfiles source (default: `$HYPRTK_DIR`, `~/hyprtk`, `~/Projects/AI-Projects/hyprtk-merged`, else cloned from `hyprtk/dotfiles`) |
| `--iso-name NAME` | ISO file name (default `hyprtk`) |
| `--iso-label LABEL` | ISO label, max 32 chars (default `HYPRTK_<YYYYMM>`) |
| `--out-dir DIR` | Where the ISO is written (default: your home) |
| `--build-root DIR` | Scratch dir for the profile/work (default `/tmp/hyprtk-iso-build`) |
| `--no-aur` | Skip building the AUR extras |
| `--no-matuwall` | Skip building matuwall from source |
| `--profile-only` | Assemble the profile and stop (no ISO build) |
| `--keep-work` | Keep the assembled profile after the build |

`--profile-only --keep-work` is handy for inspecting what went onto the image
without waiting for a full `mkarchiso` run.

## What the live ISO looks like

- **Live user:** `hyprtk` (password `hyprtk`, passwordless sudo)
- **Login:** SDDM autologins to the **hyprland** session — no greeter
- **Theming:** pywal palette is generated at build time from
  `assets/Wallpapers/default.png`, so the bar, rofi, lock screen and icons are
  already coloured on first boot
- **New users:** creating any account (e.g. from the live session) copies
  `/etc/skel`, so the new user's `~/hyprtk` and `~/.config` symlinks are set up;
  a one-shot systemd user unit (`hyprtk-first-run`) installs the bar and runs
  pywal on first login

## How it fits together

```
arch-iso-builder.sh        # the builder
packages.hyprtk            # official packages baked into the ISO
aur-packages.txt           # AUR extras, built on the host
airootfs/                  # overlay merged onto the releng profile
  etc/sddm.conf.d/         # autologin + Wayland greeter
  etc/sudoers.d/           # live-user passwordless sudo
  etc/skel/                # generated at build time: ~/hyprtk + ~/.config links
  etc/skel/.config/systemd/user/   # hyprtk-first-run unit
  root/customize_airootfs.sh       # creates the live user, installs bar/pywal
  usr/lib/os-release               # "Hyprtk on (Arch Linux)" branding
  usr/local/bin/hyprtk-first-run   # per-user first-run setup
```

The builder copies `releng`, edits `profiledef.sh`, merges the package lists,
generates `/etc/skel` from your hyprtk checkout (trimming `assets/screenshots`,
`assets/papirus-icons`, `distro/` and the root caches), then runs `mkarchiso`.

## AUR extras

`aur-packages.txt` is built with `makepkg --nodeps --skippgpcheck` as your
normal user, then installed into the chroot by `customize_airootfs.sh`. The
stage is **best-effort**: anything that fails to build is reported and skipped,
and the ISO is still produced. Because of `--nodeps`, the host must already
have each package's build dependencies (`--skippgpcheck` skips source PGP
verification; checksums are still enforced). The visually important extras
(`swaylock-effects`, `bibata-cursor-theme`, `sddm-theme-sugar-candy-git`) are
listed out of the box; if they are missing the ISO falls back gracefully
(plain `swaylock` config, default cursor/SDDM theme).

Disable the stage with `--no-aur` if you want a fully reproducible build from
the official repos only.

## Notes

- The live root filesystem is writable through archiso's RAM overlay, so the
  bar's venv and the generated `~/.cache/wal` live in memory and vanish on
  reboot — exactly what you want for a live session.
- The AUR build runs on the **host** and (with `--nodeps`) does not install
  anything to it; build logs are kept under
  `~/.cache/hyprtk-iso/aur/<pkg>.log`.
- `--profile-only` also accepts `ARCHISO_RELENG=/path/to/releng` to use a
  different base profile.
