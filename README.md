# Xperia bootloader unlock PoC

This proof of concept depends on firmware-specific memory addresses and offsets.
Those values are loaded from a device profile instead of being compiled into
`main.c`.

The Xperia 5 Docomo SO-01M profile includes original PoC values and a ClockDxe
bit-clear RVA checked against extracted firmware. The remaining runtime values
are not verified against that firmware. The Xperia 1 profile is only a
template; its runtime offsets have not been established. See
[`FIRMWARE_ANALYSIS.md`](FIRMWARE_ANALYSIS.md) for extracted firmware hashes
and the limits of the static analysis. Create a separate profile for each
model and firmware build after measuring and confirming its runtime values.

## Build and select a profile

Install the compiler and libusb development package, then run:

```sh
./build.sh
./blu profiles/xperia5-so01m-p42.conf
```

The program defaults to the included Xperia 5 profile. Pass a different profile
path as the first argument to select another device configuration.

Copy `profiles/xperia1-template.conf` to a filename that identifies the exact
model and firmware, then replace the placeholder values. The program rejects
missing, duplicate, unknown, malformed, or out-of-range profile entries before
it opens USB. Do not reuse offsets across models or firmware versions without
verifying them on the target.

## Profile fields

All values accept decimal or `0x`-prefixed hexadecimal notation. Address fields
are absolute addresses; fields ending in `_offset` are relative to the relevant
image or buffer. `command_table_start` and `command_table_end` are the absolute
start and exclusive end addresses of the command table region.

| Field | Meaning |
| --- | --- |
| `clock_dxe_addr`, `clock_bitclear_offset` | ClockDxe base and bit-clear routine offset |
| `ufs_dxe_addr`, `ufs_listener_offset` | UfsDxe base and RPMB listener parameter offset |
| `usb_buffer_addr` | Address of the USB download buffer |
| `original_rpmb_protocol_addr` | Original RPMB protocol pointer to restore |
| `original_listener_offset`, `listener_alloc_size` | Original listener position in the transfer and allocation size |
| `rpmb_protocol_offset` | Protocol structure offset within listener parameters |
| `listener_buffer_offset`, `protocol_buffer_offset` | Listener and protocol positions in the crafted transfer buffer |
| `linux_loader_base`, `cmdline_pointer_offset` | Linux loader base and command-line pointer offset |
| `oem_unlock_string_offset`, `unlock_handler_offset` | String and handler offsets in the Linux loader image |
| `command_table_start`, `command_table_end` | Command table memory range |

This is experimental code that writes to bootloader memory. Use only on a device
you own and with a profile verified for its exact firmware. Incorrect values can
hang or damage the device.
