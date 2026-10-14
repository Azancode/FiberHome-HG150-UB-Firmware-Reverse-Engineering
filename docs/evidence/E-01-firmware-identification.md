# E-01 — Firmware Identification

## Objective

Record the identity and integrity fingerprint of the firmware image examined during the FiberHome HG150-UB reverse-engineering investigation.

## Environment

- **Device:** FiberHome HG150-UB
- **Firmware filename:** `GD25Q128_20180910_222553.BIN`
- **Image size:** 16,777,216 bytes (16 MiB)
- **SHA-256:** `efa8dbe272112e5690d8ad21b6a24fbdc56a8ff891493da5aab2cdaf85e782cd`

## Procedure

The following commands were executed against the original local firmware image:

```bash
sha256sum GD25Q128_20180910_222553.BIN
file GD25Q128_20180910_222553.BIN
```

## Observed Results

- SHA-256 calculation completed successfully.
- The calculated digest matched the fingerprint recorded for this research sample.
- The `file` utility identified the top-level image as generic data rather than assigning it a recognized standard file type.

The generic file identification is not evidence of corruption. Vendor firmware may use proprietary headers or contain several embedded data formats.

## Interpretation

The SHA-256 digest provides a reproducible fingerprint for identifying the exact firmware sample analyzed. Binwalk and subsequent filesystem extraction were used to inspect its internal structures.
