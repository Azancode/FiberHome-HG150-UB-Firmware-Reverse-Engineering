# E-03 — Configuration Backup Handler Analysis

## Objective

Trace the configuration-backup handler in the embedded HTTP server and identify how configuration data is processed before being returned to the client.

## Binary Under Analysis

- **Device:** FiberHome HG150-UB
- **Binary:** `rootfs-extracted/bin/httpd`
- **Analysis tool:** Ghidra
- **Function identified:** `cgiBackupSettings()`

## Methodology

The HTTP server was imported into Ghidra and analyzed as a big-endian, 32-bit MIPS executable.

The decompiled code was inspected to identify the backup handler and follow its calls into the configuration-management and encoding routines.

## Observed Processing Flow

The decompiled handler showed the following sequence:

1. **Configuration retrieval:** Calls `cmsMgm_readConfigFlashToBuf()` to obtain configuration data in a buffer.
2. **Encoding:** Passes the configuration buffer to `usbencode()`.
3. **Buffer handling:** Checks the encoding result and releases the original configuration buffer when appropriate.
4. **HTTP response:** Writes the resulting output buffer using `fwrite()`.

The handler also contained a log message indicating that configuration settings were being backed up.

## Security Relevance

Tracing this function established a connection between the HTTP backup operation and the encoding implementation. It provided a concrete starting point for investigating the backup file structure and the cryptographic routines used by the firmware.

The function's behavior alone does not establish an authentication bypass or remote vulnerability. Those conclusions require separate evidence concerning access controls and attacker capabilities.

## Evidence to Attach

A sanitized Ghidra screenshot should show:

- The `cgiBackupSettings()` function name or analysis label.
- The configuration retrieval call.
- The call to `usbencode()`.
- The response-writing call to `fwrite()`.

Before publication, inspect the entire screenshot for cryptographic constants, private configuration values, credentials, and unrelated sensitive strings.

## Limitations

This analysis describes the observed code path in the inspected firmware sample. It does not establish that other firmware revisions use the same implementation or that the endpoint is remotely exploitable.

The original binary and unredacted research artifacts are excluded from this repository.
