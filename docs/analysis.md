# FiberHome HG150-UB Firmware Reverse Engineering: Methodology and Findings

## 1. Objective

This investigation examines the firmware and configuration-backup mechanism of the FiberHome HG150-UB VDSL2 router. The primary objective was to understand how the firmware processes configuration backups, identify the relevant code paths, reconstruct the backup file format, and validate the implementation through local analysis.

The work was performed in a local research environment using a firmware image, an extracted filesystem, a configuration backup from the researcher's test device, and reverse-engineering tools.

This document describes the methodology, observations, and limitations of the investigation. Sensitive configuration data and recovered cryptographic material are intentionally excluded.

## 2. Device and Firmware Identification

The following device information was identified during the investigation:

| Property | Observed value |
|---|---|
| Vendor | FiberHome |
| Model | HG150-UB |
| Software version | HG150-Ub_V3.0 |
| Hardware revision | HG150-Ub_R3A |
| Build identifier | 180607_1556 |
| CPU architecture | MIPS32, big-endian |
| ABI | o32 |
| Firmware image size | 16,777,216 bytes |

The firmware image was obtained from a publicly accessible third-party firmware archive. Its SHA-256 hash was recorded to support reproducibility and identify the exact sample examined.

The firmware was treated as a research artifact. No firmware modifications or deployments to third-party devices were required for this analysis.

## 3. Tools and Environment

The investigation used the following tools and technologies:

- **Kali Linux:** Research environment.
- **Binwalk:** Firmware signature identification and filesystem extraction.
- **Ghidra:** Static analysis and decompilation of embedded binaries.
- **Linux command-line utilities:** `file`, `strings`, `xxd`, `grep`, and related tools.
- **Python 3:** Binary inspection, format validation, and local decryption testing.
- **Python cryptography library:** Cryptographic validation.
- **Git:** Project version control.

Tool versions and exact commands should be recorded when preparing a fully reproducible research environment.

## 4. Firmware Inspection and Extraction

### 4.1 Initial inspection

The firmware image was examined with Binwalk to identify embedded structures and determine how its contents were organized.

The inspection identified several relevant signatures, including:

- LZMA-compressed data.
- A Broadcom 96345 firmware header.
- A SquashFS version 4 filesystem.
- Additional compressed regions.

The firmware header contained the board identifier `63381REF1`. The filesystem metadata reported a SquashFS image size of approximately 10 MB and a creation timestamp of 2017-06-12.

These values describe the examined image and should not be interpreted as proof that every HG150-UB hardware or firmware revision uses the same layout.

### 4.2 Filesystem extraction

The filesystem was extracted for offline inspection. This made it possible to examine the embedded web server, libraries, and configuration-management components without executing the firmware on a live device.

The extracted web server was located at:

`rootfs-extracted/bin/httpd`

Binary inspection identified it as a 32-bit, big-endian MIPS ELF executable using the o32 ABI and dynamically linked against uClibc. The binary was stripped, limiting the availability of original function and symbol names.

Relevant supporting components included cryptographic libraries and Broadcom/CMS-related libraries associated with device configuration and management.

## 5. Identifying the Configuration-Backup Handler

Ghidra was used to inspect the embedded web server and locate the configuration-backup processing path.

The function identified during analysis as `cgiBackupSettings()` was examined in its decompiled form.

The observed processing sequence was:

1. Read configuration data into a buffer through `cmsMgm_readConfigFlashToBuf()`.
2. Pass the configuration buffer to `usbencode()`.
3. Check the encoding result.
4. Release the original configuration buffer when appropriate.
5. Write the resulting encoded buffer to the HTTP response stream using `fwrite()`.

The function also contained a log message indicating that configuration settings were being backed up.

This call chain established a concrete link between the HTTP backup handler and the firmware's encoding implementation. It provided a basis for investigating the backup format rather than relying on assumptions about the file's appearance.

## 6. Reverse-Engineering the Encoding and Decoding Routines

### 6.1 Encoding routine

The function identified as `usbencode()` was examined to understand how the backup buffer was constructed.

The decompiled code showed that the routine:

- Calculates a block-aligned output length.
- Allocates an output buffer with additional space for a length field.
- Copies a four-byte length value into the beginning of the output.
- Copies the configuration data after the length field.
- Initializes cryptographic state using material embedded in the firmware.
- Invokes an AES-CBC encryption routine over the aligned data region.

The cryptographic constants are omitted from this document to avoid unnecessarily distributing recovered secrets.

### 6.2 Decoding routine

The corresponding `bcmusbdecode()` routine was also examined.

Its logic used the same embedded cryptographic material and invoked an AES-CBC decryption routine. The code included a length-consistency check before proceeding with the decoded data.

Examining both the encoding and decoding paths helped establish that the observed cryptographic behavior was part of the firmware's backup/restore implementation, rather than an inference based solely on the output file.

### 6.3 CBC implementation

The relevant CBC encryption routine processes the input in 16-byte blocks. For each block, it combines the plaintext block with the preceding chaining value using XOR, encrypts the result, and updates the chaining value.

This behavior is consistent with AES in Cipher Block Chaining (CBC) mode.

The key-setup call used a parameter of `0x100`. This is consistent with a 256-bit key if, as inferred from the implementation, the argument represents key length in bits. The exact interpretation depends on the semantics of the underlying key-setup function.

The implementation also uses a fixed initialization vector. The recovered key and IV are not reproduced here.

## 7. Reconstructing the Backup File Format

A configuration backup captured from the researcher's test device was inspected as a binary file.

The observed file had the following properties:

| Property | Observed value |
|---|---:|
| Total backup size | 61,204 bytes |
| Header size | 4 bytes |
| Ciphertext region size | 61,200 bytes |
| Length value in header | 61,190 bytes |
| AES block size | 16 bytes |

The first four bytes were interpreted as a big-endian length value of 61,190. The remaining 61,200 bytes form a region whose length is divisible by the AES block size.

The resulting format is consistent with a four-byte length header followed by block-aligned AES-CBC ciphertext.

The length field corresponds to the recovered configuration's observed length, while the ciphertext region is aligned to the cipher's block size. The implementation does not appear to rely on a conventional PKCS#7 padding marker in the inspected CBC routine; padding and buffer initialization must therefore be understood in the context of the firmware's actual allocation and encoding logic.

The format description above applies to the captured sample and the analyzed code path. Other firmware versions may implement different formats or validation rules.

## 8. Local Decryption and Validation

The encryption parameters and file structure inferred from the decompiled routines were used to validate the backup locally.

The validation process involved:

1. Reading the four-byte length field.
2. Separating the ciphertext from the header.
3. Checking that the ciphertext length was compatible with AES's 16-byte block size.
4. Applying AES-CBC decryption with the parameters reconstructed from the firmware.
5. Examining the resulting bytes and checking the decoded length.
6. Validating the output as XML.

The resulting configuration was 61,190 bytes long and began with an XML declaration and a `DslCpeConfig` root element. The output was predominantly printable text and was consistent with a router configuration document.

This successful validation provides evidence that the reconstructed format and cryptographic processing matched the examined backup sample.

The original configuration may contain sensitive information, including network settings, account identifiers, credentials, or other device-specific values. The decoded document is therefore excluded from this repository.

## 9. Security Observations

### 9.1 Embedded cryptographic material

The examined firmware contains fixed cryptographic material used by the configuration-backup encryption routine.

If the same material is reused across devices or firmware installations, and an attacker can obtain both a compatible firmware image and an encrypted configuration backup, the encryption may provide substantially less confidentiality than expected.

The practical impact depends on factors that this investigation has not fully established, including:

- Whether the same cryptographic material is reused across affected devices and firmware versions.
- Whether an attacker can obtain a target device's backup.
- Whether additional access controls protect backup retrieval.
- Whether the restore process accepts modified or malformed backup files.

A fixed initialization vector also weakens the design relative to using a fresh, unpredictable IV for each encryption operation. AES-CBC alone does not provide integrity or authenticity protection.

### 9.2 Backup endpoint observations

During local testing, backup responses obtained with and without the tested session cookie were identical in size and content.

This observation is not, by itself, proof of an authentication bypass. Establishing an access-control vulnerability would require a clear understanding of the endpoint's intended authorization requirements, the device's authentication state, and whether the tested requests should have been permitted.

No claim of remote exploitation is made on the basis of this observation alone.

### 9.3 Limitations of the current findings

The investigation established the relationship between the backup handler, the encoding and decoding routines, and a successfully decoded local sample.

It did not establish:

- That every HG150-UB device uses the same key, IV, or backup format.
- That an unauthenticated remote attacker can retrieve configuration backups.
- That the tested device is vulnerable to remote code execution.
- That modified configuration backups can bypass integrity checks or authorization.
- That a vulnerability has been confirmed by the vendor.

These questions require additional evidence and, where appropriate, controlled testing on an authorized device.


## 10. Conclusion

This investigation traced the FiberHome HG150-UB configuration-backup workflow from the HTTP handler into the firmware's encoding and decoding routines. Static analysis identified an AES-CBC-based processing path, and local validation reproduced a configuration document from an examined backup sample.

The primary security concern is the use of embedded cryptographic material in the backup mechanism. Its real-world severity depends on key reuse, backup accessibility, the sensitivity of stored configuration, and the presence or absence of additional protections.

The findings provide a documented reverse-engineering case study in embedded Linux firmware analysis, binary format reconstruction, cryptographic implementation review, and responsible security research.
