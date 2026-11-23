# CLAUDE.md

Nerves system `:recomputer_r100x_cm4` for the Seeed reComputer R100x (CM4, aarch64), a headless gateway,
repo `chromasurf/recomputer_r100x_cm4`. Build, release and the shared CM4 stack are described in the
monorepo's `CLAUDE.md` (`../../CLAUDE.md`).

## Stack

- `nerves_system_br` 1.35.3: Buildroot 2026.08.1, Erlang/OTP 29, fwup 1.17
- Linux 6.18.50 (`raspberrypi/linux` `stable_20260911`), rpi-firmware 1.20260915
- Nerves toolchain 15.3 (GCC 15.3, glibc 2.43)
- No WebKit, no boot splash. `vc4` is a module (`CONFIG_DRM_VC4=m`) for an optional HDMI display.
- Builds under Nerves 1.15 and Nerves 2 (`rootfs.tar`, `CONFIG_EROFS_FS`, `CONFIG_SQUASHFS_ZSTD`)

## Build

```bash
mix deps.get
mix compile               # full Buildroot build in the Apple container VM
mix generate_fwup_conf    # renders fwup.conf from fwup.conf.eex; mix compile runs it too
```

- The host toolchain comes from `.tool-versions`; outside a mise shell use `mise exec -- mix …`.
- The artifact checksum covers the files in `source_finger_print_files/0` in `mix.exs`, README and CHANGELOG
  included. Any change there produces a new artifact.
- `mix nerves_system_linter` starts a full build.

## Files

| File | Content |
|---|---|
| `nerves_defconfig` | Buildroot packages, toolchain, kernel source |
| `linux-6.18.defconfig` | Kernel config; pstore, EROFS, NVMe, TPM SPI |
| `linux/ili9881c_fix.patch` | Panel patch shared with the reTerminals, rebased for every kernel bump |
| `config.txt` | Boot config at stock clocks, `gpu_mem=16`; V1.1 hardware active, V1.0 commented; ends with `include app.txt` |
| `app.txt` | Empty except for a comment; the app replaces it via `CHROMASURF_CONFIG_APP` |
| `cmdline-a.txt`, `cmdline-b.txt` | `root=/dev/mmcblk0p5` / `p6`, `loglevel=5`, `oops=panic` |
| `fwup.conf.eex` | A/B with tryboot; boot files as deltas against the other slot |
| `fwup_include/fwup-common.conf` | Partition layout; `APP_PART_OFFSET` stays fixed for OTA from v1.x |
| `fwup-ops.conf` | factory-reset, revert, validate, status |
| `post-createfs.sh` | Writes `images/kernel8.img.gz`, the gzip kernel |
| `ramoops-pi4-overlay.dts` | pstore overlay, 128 KiB |
| `rootfs_overlay/etc/erlinit.config` | Console `ttyS0` |
| `examples/` | App code for reference, not part of the system |

Overclocked, a board had 5 watchdog resets in 45 h, at stock clocks 0 in 47 h; the clocks stay at stock.
`gpu_mem` takes effect only in `config.txt`; the firmware reads it before `include app.txt`.

## Packages (`package/`)

| Package | Purpose |
|---|---|
| `rethings` | Seeed device tree overlays (`reComputer-R100x`); `TPM_RST` on PCA9535 pin 4 is hogged high for the SLB9670 on spi1.0 |
| `rtc-pcf8563w` | RTC with 120 s watchdog |
