# Changelog

This project does NOT follow semantic versioning. The version increases as
follows:

1. Major version updates are breaking updates to the build infrastructure.
   These should be very rare.
2. Minor version updates are made for every major Buildroot release. This
   may also include Erlang/OTP and Linux kernel updates. These are made four
   times a year shortly after the Buildroot releases.
3. Patch version updates are made for Buildroot minor releases, Erlang/OTP
   releases, and Linux kernel updates. They're also made to fix bugs and add
   features to the build infrastructure.

## v1.4.1

* Changes
  * TPM: `reComputer-R100x` declared `TPM_RST` (PCA9535 pin 4) without
    `gpio-hog`, so the line stayed an input at 0, the SLB9670 stayed in reset and
    `tpm_tis_spi spi1.0` failed with -110. `TPM_RST` is now hogged high.
  * `config.txt`: drop `tpm-spi-slx9670`. `reComputer-R100x` already defines
    the TPM (spi1.0, spi0.0 with `uart2`); the second node on spi0 never
    probed, because gpio9 is assigned to the UART. The overlay is no longer built
    or written. Remove the line from an own `config.txt` too.
  * `config.txt`: pstore with `ramoops-pi4`, 128 KiB (kernel console, panic
    dumps, pmsg); before, only with the overlay in an own `config.txt`.
  * Kernel command line: `oops=panic`, `sysctl.kernel.panic_on_rcu_stall=1` and
    `loglevel=5`. An oops or an RCU stall reboots the gateway through the
    watchdog; KERN_NOTICE messages reach the pstore console. README,
    "Reboot on Kernel Errors", shows how to switch it off.
  * fwup: `config.txt` ends with `include app.txt`. `CHROMASURF_CONFIG_APP`
    points at an own `app.txt` that adds or overrides lines, without a copy
    of the whole `config.txt`; the system's `app.txt` holds only a
    comment. The firmware reads `gpu_mem` before it follows `include`.
  * The artifact checksum covers `package/` (kernel modules, overlays) and
    `app.txt`; before, a change there produced no new artifact.

## v1.4.0

Moves to Buildroot 2026.08.1, Erlang/OTP 29.1.2 and Linux 6.18.50, takes over
the boot-time changes from the reTerminal systems, and runs under Nerves 1 and
Nerves 2.

* Package updates
  * [nerves_system_br 1.35.3](https://github.com/nerves-project/nerves_system_br/releases/tag/v1.35.3):
    Buildroot 2026.08.1, Erlang/OTP 29.1.2, fwup 1.17.0. Fixes the missing
    available-memory patch on OTP 29 (`system_memory_high_watermark`).
  * Linux 6.18.50 (`raspberrypi/linux` `stable_20260911`), rpi-firmware 1.20260915
    `linux/ili9881c_fix.patch` follows the new `rzw,t101p136cq` entry in the
    panel match table.
  * Target toolchain: the Nerves toolchain 15.3.0 (GCC 15.3, glibc 2.43)
    instead of the Arm GNU Toolchain 15.2 (glibc 2.42). Apps compile their NIFs
    with the same toolchain the system is built with.

* Changes
  * `cmdline-a.txt` / `cmdline-b.txt`: no `rootfstype=squashfs`, so the same
    system boots SquashFS and EROFS root filesystems.
  * `kernel8.img` is the gzip-compressed kernel; `post-createfs.sh` writes
    `images/kernel8.img.gz`, `images/Image` stays in the artifact.
  * `config.txt`: `kernel=` and `device_tree=` named explicitly,
    `initial_turbo=60`, `boot_delay=0`, `boot_delay_ms=0`.
  * `config.txt`: stock clocks. `over_voltage=6`, `arm_freq=2147`,
    `gpu_freq=750` and `arm_boost=1` are gone: overclocked, a board in continuous
    operation froze with a watchdog reset (`rsts=0x20`) 5 times in 45 h; at stock
    clocks none in 47 h.
  * Kernel: `CONFIG_DRM_VC4=m` (with `SND` / `SND_SOC`, which vc4 depends on).
    `config.txt` already loaded `vc4-kms-v3d`, but the kernel had no vc4 driver,
    so HDMI had no Linux output.
  * Only `bcm2711-rpi-cm4.dtb` is built and written; the Pi 4B and Pi 400
    device trees are gone.
  * fwup: boot partition files are written as deltas against the other slot on
    upgrade (`require-fwup-version` 1.12.0).
  * fwup: `CHROMASURF_CONFIG_TXT`, `CHROMASURF_CMDLINE_A` and
    `CHROMASURF_CMDLINE_B` in the environment of `mix firmware` replace
    `config.txt`, `cmdline-a.txt` and `cmdline-b.txt`; a copy of `fwup.conf`
    is no longer needed.
  * `fwup.conf` is rendered from `fwup.conf.eex` before every `mix compile`;
    the checksum covers only the template and now also the `linux/` patches.
  * Kernel: `CONFIG_SQUASHFS_ZSTD=y` and `CONFIG_EROFS_FS=y`. Selected in the app's config
    with `-comp zstd` (Nerves 1, `mksquashfs_flags`) or `rootfs_type: :erofs`
    (Nerves 2).
  * `config/config.exs` starts `nerves_bootstrap` and `cli/0` sets
    `default_target: :target`, replacing the `loadconfig` alias (as in
    `nerves_system_rpi4`). `mix docs` works with parallel dependency
    compilation (`MIX_OS_DEPS_COMPILE_PARTITION_COUNT`).
  * Nerves 2: `rootfs.tar` is built again (Nerves 2 assembles the root
    filesystem from it), `nerves` allows `~> 2.0`, toolchain package 15.3.1.

## v1.3.3

Build tooling — no functional changes to the image.

* Changes
  * `mix.exs`: require `{:nerves, "~> 1.15"}` and drop both the
    `nerves_container` dependency and the explicit `build_runner:` selection
    added in v1.3.2. Nerves 1.15.0 ships the Apple `container` build runner
    upstream (the same code) and selects it on Apple Silicon by itself, falling
    back to Docker elsewhere. Build volumes carry over unchanged — same names,
    same `.container_id`, same `/home/nerves/project` working directory — so
    this costs no rebuild.

## v1.3.2

Build tooling — no functional changes to the image.

* Changes
  * macOS builds can now run in an Apple `container` Linux VM (no Docker or
    remote build server needed) via
    [nerves_container](https://github.com/chromasurf/nerves_container) — it is
    auto-selected on Apple Silicon and falls back to Docker otherwise. Upstream
    support is proposed in nerves-project/nerves#1190.

## v1.3.1

Documentation-only update — no functional changes to the image.

* Changes
  * README: add a device photo, correct the kernel documentation (Linux 6.18,
    `linux-6.18.defconfig`), and document the OTP 29 / Elixir 1.20+ requirement.

## v1.3.0

Minor update for a major Buildroot release: tracks `nerves_system_br` v1.34.0,
bringing a new Erlang/OTP, Linux kernel, and toolchain. Headless gateway system,
so display/browser refinements don't apply.

**Breaking for downstream:** the image now ships Erlang/OTP 29, so consuming
projects must build with OTP 29 (Elixir 1.20+). This board uses an ATECC608A —
`nerves_key_pkcs11` must be `~> 1.3` to avoid a TLS segfault.

* Changes
  * Remove the `pigpio` package — dropped upstream in Buildroot 2026.05 (no
    longer maintained). Use `libgpiod` / `circuits_gpio` for GPIO.

* Updated dependencies
  * [nerves_system_br v1.34.0](https://github.com/nerves-project/nerves_system_br/releases/tag/v1.34.0)
    (Erlang/OTP 29.0.2, Buildroot 2026.05, GCC 15.3.0)
  * Linux kernel 6.18 (Raspberry Pi `stable_20260527` tag), RPi firmware 1.20260521

## v1.2.1

Security/bug fix update that tracks `nerves_system_br` v1.33.9 (up from
v1.33.4). This is a headless gateway system, so the display/browser-related
kiosk refinements don't apply here.

* Changes
  * Switch the runtime cgroup hierarchy to cgroup v2 — kernel config (PSI,
    CFS bandwidth) plus an `erlinit` `cgroup2` mount — and re-enable the memory
    controller with `cgroup_enable=memory cgroup_memory=1` on the kernel command
    line.

* Updated dependencies
  * [nerves_system_br v1.33.9](https://github.com/nerves-project/nerves_system_br/releases/tag/v1.33.9)
    (Erlang/OTP 28.5.0.1, Buildroot 2025.11.3, fwup 1.16.0, glibc 2.43
    compatibility fix, Raspberry Pi WiFi roaming patch)

## Earlier releases

Detailed notes for v1.2.0 and earlier live in the
[GitHub releases](https://github.com/chromasurf/recomputer-r100x-cm4/releases):
v1.2.0, v1.1.2, v1.1.1, v1.1.0, v1.0.0.
