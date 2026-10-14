# Evidence Index

This directory contains sanitized evidence supporting the FiberHome HG150-UB firmware reverse-engineering investigation.

All evidence should be reviewed before publication to ensure it does not disclose device credentials, private configuration data, recovered cryptographic material, or other sensitive information.

## Evidence Register

| ID | Evidence | Purpose | Status |
|---|---|---|---|
| E-01 | Firmware identification | Documents the firmware image properties and identified signatures | To add |
| E-02 | Filesystem and binary identification | Records the architecture and properties of the embedded HTTP server | To add |
| E-03 | Backup handler in Ghidra | Shows the configuration-backup processing path | To add |
| E-04 | Encoding and decoding routines | Documents the relationship between the backup routines | To add |
| E-05 | Backup format analysis | Illustrates the length header and block-aligned ciphertext region using sanitized metadata | To add |
| E-06 | Local validation | Documents successful XML-format validation without exposing configuration contents | To add |

