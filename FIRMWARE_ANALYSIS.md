# Firmware offset analysis

Firmware was downloaded with the local XperiFirm tool and extracted under
`/tmp/xperia-fw-offset-analysis`.

| Device / firmware | XBL | ClockDxe SHA-256 | UfsDxe SHA-256 |
| --- | --- | --- | --- |
| Xperia 1 SO-03L, 55.2.D.3.22-R15C, CID 1319-1360 | 210 | `287793300d51a1d472f405c5c03ffef5813b7abc54551a2c840048d8ba39daf7` | `08bbe3e0a56454826dd5ef221a1faf933b98e7746461cb299f0b4f08cdfba1c2` |
| Xperia 5 SO-01M, 55.2.D.3.22-R15C, CID 1319-8175 | 210 | `287793300d51a1d472f405c5c03ffef5813b7abc54551a2c840048d8ba39daf7` | `08bbe3e0a56454826dd5ef221a1faf933b98e7746461cb299f0b4f08cdfba1c2` |
| Xperia 5 901SO, 55.1.B.0.622-R10B, CID 1319-8987 | 117 | `aef056f582f1f5e1fdcdfe4bd66abb367efc6ffcdb8f397119bd81aad31f5d0c` | `8390e8ffe93b0abf4327992a2aff31d4aecce3c9f2726e7cd37873fb2d01e046` |

In the SO-03L and SO-01M ClockDxe PE images, the bit-clear routine starts at
RVA `0x11B1C`. Its instructions clear a bit in the target word and return. The
old profile's `0x11B7C` is inside a separate function's zero-return epilogue,
so it is not the bit-clear routine. The two firmware builds above contain
byte-identical ClockDxe and UfsDxe images; this establishes module identity,
not that every runtime address or exploit prerequisite matches.

The UfsDxe PE uses `.text` RVA `0x1000` through `0x18FFF` and `.data` RVA
`0x19000` through `0x1BFFF`. The PoC's `ufs_listener_offset = 0x1B7A8` falls in
`.data`, despite being used as a callable address. Its meaning cannot be
confirmed from this static image alone. The transfer-buffer layout, runtime
module bases, LinuxLoader offsets, command-table range, and Xperia 1 exploit
compatibility likewise require a matching runtime memory map or device-side
observation. They are retained as original PoC data only in the SO-01M profile
and left unset in the Xperia 1 template; do not treat them as validated for the
downloaded firmware based only on this report.
