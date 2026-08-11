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
