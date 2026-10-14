# E-05 — Backup File Format Analysis

## Objective

Document the observed structure of a configuration-backup file exported from the FiberHome HG150-UB router and compare it with the encoding routine identified during static analysis.

## Target

- **Device:** FiberHome HG150-UB
- **Firmware:** HG150-Ub_V3.0
- **Analysis type:** Local backup-file inspection and static analysis
- **Tools:** Linux command-line utilities and Ghidra

## E-05A — Original Backup File

### Observations

Two captured backup files were compared:

- `backup-no-cookie.bin`
- `router-config-export.bin`

Both files had the same observed size and SHA-256 hash.

| Property | Observed value |
|---|---|
| File size | 61,204 bytes |
| SHA-256 | `b9278c44d22848710a2b29920a977972e9b548980ad3e74cb6dc394920fbaf90` |
| First four bytes | Interpreted as a big-endian length field |
| Remaining bytes | 61,200 bytes |
| Remaining length divisible by 16 | Yes |

The first four bytes, interpreted as a big-endian integer, represent 61,190. This matches the length of the locally decrypted configuration data.

The remaining 61,200 bytes are divisible by the 16-byte AES block size, which is consistent with block-aligned encrypted data.

### Evidence

Record sanitized terminal output showing the file sizes, hashes, and length-field interpretation. Do not include the contents of the configuration file.

## E-05B — Comparison With the Firmware Implementation

The `usbencode()` routine identified in E-04 writes a four-byte length field before the configuration data and encrypts the data using an AES-CBC-style routine.

The observed backup structure is consistent with this implementation:

1. A four-byte length field.
2. An encrypted data region whose size is a multiple of 16 bytes.

The observed file contains 61,204 bytes in total: four bytes for the length field and 61,200 bytes for the encrypted region.

The length field indicates 61,190 bytes of configuration data, leaving 10 bytes of additional space in the encrypted region. This is consistent with padding or alignment to a 16-byte boundary, although the precise padding semantics should be established from the implementation and a controlled test.

## E-05C — Local Decryption Result

A local decryption test produced a 61,190-byte output whose beginning was recognized as XML configuration data.

The output began with an XML declaration and a `DslCpeConfig` root element. This is consistent with the length value recorded in the original backup file.

The decrypted file was inspected locally. Its configuration contents are intentionally excluded from this report because they may contain sensitive network settings, credentials, or other private information.

### Evidence

If screenshots are included, use sanitized terminal output showing only:

- The decrypted file's size.
- Its file type or XML recognition.
- The presence of the XML declaration and root element, with configuration values omitted.

Do not include screenshots showing passwords, Wi-Fi keys, account credentials, public IP addresses, serial numbers, or other sensitive settings.

## Security-Relevant Observations

The backup format does not appear to be a plain XML export. Its structure and the matching firmware code support the conclusion that the configuration is encrypted before being exported.

However, the local decryption result shows that the configuration can be recovered when the required decryption material and processing method are available.

If the same cryptographic material is embedded in firmware distributed to users, an individual who obtains the firmware may be able to analyze the backup-encryption scheme. The practical confidentiality impact depends on the accessibility of backups, the reuse of cryptographic material, and the sensitivity of the configuration data.

## Limitations

- The analysis concerns a backup obtained during local testing.
- Matching file hashes establish that the two captured files are identical; they do not establish an authorization vulnerability.
- The observed structure is consistent with the identified encoding routine but does not, by itself, prove that every firmware version uses the same format.
- Successful local decryption does not establish remote exploitability or an authentication bypass.
- Sensitive decrypted configuration data and cryptographic material are not included in this repository.

## Conclusion

The examined backup file contains a four-byte length field followed by a 16-byte-aligned encrypted region. The length value matches the size of the locally recovered XML configuration, and the observed structure is consistent with the firmware's backup-encoding routine. The evidence documents the backup format and local validation without claiming an unverified remote security impact.
