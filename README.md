# Chromasurf — reComputer R100x CM4

Custom Nerves system for the [Seeed Studio reComputer R1000](https://wiki.seeedstudio.com/recomputer_r/) (CM4, aarch64).

Built on `nerves_system_br` (Buildroot). Headless industrial gateway with RS485, LoRa, and LTE support.

## Hardware

- 3x isolated RS485 (UART2, UART3, UART4)
- GPIO expander (PCA9535) with LoRa/LTE control lines
- LoRaWAN ready (SX1302 + SX1262 via SPI, application-level stack)
- LTE/4G ready (Mini-PCIe slot, USB modem interface)
- RTC (PCF8563W) with hardware watchdog
- Crypto co-processor (ATECC608A)
- TPM 2.0 (Infineon SLB9670)
- EEPROM (24C256) with write-protect
- RGB status LEDs, buzzer
- Dual Ethernet (BCMGENET + R8169)
- M.2 NVMe SSD slot
- WiFi/BT (Broadcom brcmfmac)

## Usage

```bash
# In your Nerves app's mix.exs:
{:chromasurf_recomputer_r100x_cm4, github: "chromasurf/recomputer-r100x-cm4", runtime: false, targets: :recomputer_r100x_cm4}

# Build firmware
MIX_TARGET=recomputer_r100x_cm4 MIX_ENV=prod mix deps.get
MIX_TARGET=recomputer_r100x_cm4 MIX_ENV=prod mix firmware
```

## Building the system

Prebuilt artifacts are published as GitHub releases. To build from source:

```bash
mix deps.get
mix compile
mix nerves.artifact
```

## License

Apache-2.0
