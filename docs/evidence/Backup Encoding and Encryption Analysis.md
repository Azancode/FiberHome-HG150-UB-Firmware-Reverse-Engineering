# E-04 — Backup Encoding and Encryption Analysis

## Objective

Document the firmware's configuration-backup encoding and decoding routines and identify the cryptographic operations used to process exported configuration data.

## Target

- **Device:** FiberHome HG150-UB
- **Firmware:** HG150-Ub_V3.0
- **Binary analyzed:** `httpd`
- **Analysis method:** Static analysis using Ghidra

## E-04A — Backup Encoding (`usbencode()`)

### Observations

Static analysis of `usbencode()` identified the following processing steps:

1. Calculates the output buffer size based on the input length and 16-byte block alignment.
2. Allocates a buffer for the encoded result.
3. Copies a four-byte length value into the beginning of the output buffer.
4. Copies the configuration data after the length field.
5. Initializes the AES key and initialization vector from values embedded in the firmware.
6. Calls `aes_set_key()` with a key-length argument of `0x100`.
7. Calls `aes_cbc_encrypt()` to encrypt the configuration data.

The resulting buffer is returned to the caller for subsequent output.

### Evidence

- `E-04A-usbencode-buffer.png` — Buffer allocation, length field, and configuration-data copy.
- `E-04A-usbencode-encryption.png` — Cryptographic setup and encryption call.

## E-04B — Backup Decoding (`bcmusbdecode()`)

### Observations

Static analysis of `bcmusbdecode()` identified the following steps:

1. Reads the length value from the supplied input buffer.
2. Calculates the expected data length after block alignment.
3. Checks whether the supplied buffer length matches the expected length plus the four-byte header.
4. Allocates memory for the decoded configuration data.
5. Copies the encrypted data into the allocated buffer.
6. Initializes AES key material and an initialization vector from values embedded in the firmware.
7. Calls `aes_set_key()` and `aes_cbc_decrypt()` to decrypt the data.

The function returns a failure value if the input length check fails or the memory allocation cannot be completed.

### Evidence

- `E-04B-bcmusbdecode.png` — Input-length validation, allocation, and decryption path.

## E-04C — AES-CBC Encryption Routine (`aes_cbc_encrypt()`)

### Observations

The decompiled routine processes the input in 16-byte blocks. For each block, it:

1. XORs the current input block with the current chaining value.
2. Passes the resulting block to `aes_encrypt()`.
3. Copies the encrypted block into the chaining buffer.
4. Advances the input, chaining, and output pointers by 16 bytes.

This control flow is consistent with AES-CBC encryption.

### Evidence

- `E-04C-aes-cbc-encrypt.png` — The block-processing loop, XOR operation, encryption call, and chaining-value update.

## Security-Relevant Observations

The analyzed firmware contains configuration-backup routines that use AES encryption and decryption with key and initialization-vector values embedded in the binary.

Hardcoded cryptographic material can weaken the confidentiality of exported configuration backups if an attacker can obtain the firmware and access a backup encrypted with the same material. The practical impact depends on the accessibility of the backup, the sensitivity of the configuration data, and the cryptographic implementation and deployment details.

## Limitations

- These findings are based on static analysis of the extracted firmware.
- The AES mode identification is based on the observed block-processing and chaining logic.
- This evidence alone does not establish remote exploitability or an authentication bypass.
- The presence of embedded key and IV values does not independently demonstrate that a remote attacker can retrieve a backup.
- Sensitive key and IV values, configuration contents, credentials, and unredacted backup files are intentionally excluded from this documentation.

## Conclusion

Static analysis identified a configuration-backup encoding and decoding path that uses AES-based encryption, with a block-processing routine consistent with AES-CBC. The firmware embeds the associated key and IV material in its binary. Further impact assessment should consider backup endpoint access controls, the sensitivity of exported settings, and whether the cryptographic material is shared across devices or firmware builds.
