# Unofficial ESP32-C3 port (scan-only)

This fork adds an **unofficial** board profile `MARAUDER_C3_SUPERMINI` for bare ESP32-C3
boards (e.g. "C3 SuperMini", 4 MB flash, native USB-Serial/JTAG). It is not supported by
upstream; please don't report issues from this build to the upstream project.

## What it is
- Serial CLI only (no display, no touch, no SD).
- WiFi 2.4 GHz: AP/station scanning, probe/beacon sniffing, channel hopping.
- BLE scanning via NimBLE (the C3 Arduino core ships NimBLE only, no Bluedroid).
- **Scan-only build:** `SCAN_ONLY_BUILD` makes the CLI refuse all attack, spam, spoof,
  evil-portal and karma commands. Use it for analysis of your own networks/devices only.

## Changes vs. upstream v1.20.0
- `configs.h`: new `MARAUDER_C3_SUPERMINI` profile (`SCAN_ONLY_BUILD`, `HAS_BT`,
  `HAS_NIMBLE_2`, `HAS_IDF_3`), hardware name "ESP32-C3 (unofficial)".
- `CommandLine.cpp`: refuses attack/spam/spoof/portal/karma commands when `SCAN_ONLY_BUILD`.
- `FirmwareMetadata.cpp`: reports the esp32c3 chip ID.
- `AXP192.cpp`: use `Wire` on C3 (it has no `Wire1`).

## Build (arduino-cli)
Toolchain used: arduino-cli 1.5.1, esp32 Arduino core **3.3.4**, NimBLE-Arduino 2.3.8 and the
library versions from the upstream build workflow.

    arduino-cli compile \
      --fqbn "esp32:esp32:esp32c3:CDCOnBoot=cdc,PartitionScheme=min_spiffs" \
      --build-property "compiler.cpp.extra_flags=-DMARAUDER_C3_SUPERMINI" \
      --build-path out esp32_marauder

Notes:
- `CDCOnBoot=cdc` is required, otherwise the CLI is not reachable over the native USB port.
- Like upstream, the core needs `-Wl,-zmuldefs` in `compiler.c.elf.extra_flags` and
  `-fno-exceptions` in the esp32-arduino-libs `cpp_flags`.

## Flash

    esptool --chip esp32c3 --port COMx write-flash 0x0 out/esp32_marauder.ino.merged.bin

Back up your original firmware first (`esptool read-flash 0 ALL backup.bin`).

## Use
Open a serial terminal at 115200 baud. Useful commands: `scanall`, `list -a`, `list -c`,
`sniffbeacon`, `sniffprobe`, `sniffbt`, `stopscan`, `settings -s ChanHop enable`.

License: GPL-3.0, same as upstream.
