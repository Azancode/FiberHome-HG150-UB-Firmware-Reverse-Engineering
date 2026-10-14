# FiberHome-HG150-UB-Firmware-Reverse-Engineering
This project documents a local firmware reverse-engineering investigation of the FiberHome HG150-UB VDSL2 router. All analysis was performed in a local lab using a firmware image, extracted filesystem, Ghidra, and a configuration backup from the researcher's own test device.

Device Information

Vendor: FiberHome

Model: HG150-UB

Software version: HG150-Ub_V3.0

Hardware revision: HG150-Ub_R3A

Build: 180607_1556

Architecture: MIPS32, big-endian, o32 ABI

Tools Used

Kali Linux

Ghidra

Binwalk

readelf, strings, xxd, and other Linux utilities

Python 3

Python cryptography library

Git

Investigation

1. Firmware analysis

Inspected the firmware image and extracted its filesystem to identify the embedded web server and relevant configuration-management components.

2. Backup handler

Reverse-engineered the cgiBackupSettings() handler in the web server. The handler reads configuration data from flash, passes it to usbencode(), and writes the resulting buffer to the response stream.

3. Cryptographic implementation

Inspected usbencode(), bcmusbdecode(), aes_cbc_encrypt(), and the corresponding decryption routine.

The backup and restore paths use AES-CBC with cryptographic material embedded in the firmware.

4. Backup format

The observed 61,204-byte backup was consistent with a four-byte length header followed by 61,200 bytes of AES-block-aligned ciphertext.

The firmware's decoding logic and the header interpretation were investigated using Ghidra.

5. Validation

Locally decrypted a captured configuration backup and verified that the resulting 61,190-byte configuration began with valid XML, including a DslCpeConfig root element.

This validated the reconstructed encryption parameters and backup-processing path for the tested sample.

Security Observations

The firmware contains embedded key and IV material used by the configuration backup encryption routine.

This design may weaken the confidentiality provided by encryption if an attacker can obtain both the firmware and a compatible configuration backup.

The impact depends on backup availability, the sensitivity of stored configuration values, affected firmware versions, and the attacker's access.

This investigation does not by itself establish remote exploitability or prove that every device or firmware version is affected.

Responsible Disclosure

Any security report should include the affected firmware version, relevant code references, sanitized evidence, and a clear explanation of the demonstrated impact.

Sensitive router configurations, recovered cryptographic material, and proprietary firmware binaries are intentionally excluded from this repository.

Scope and Safety

This project documents authorized local analysis of a device and firmware available to the researcher. It does not include instructions to access other users' routers or retrieve their configuration backups.

Disclaimer

This is an independent research project. No affiliation with or endorsement by FiberHome is implied.
