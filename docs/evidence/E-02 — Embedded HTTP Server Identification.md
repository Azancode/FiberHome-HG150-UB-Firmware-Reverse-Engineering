# E-02 — Embedded HTTP Server Identification

## Objective

Identify the executable format and target architecture of the web server embedded in the FiberHome HG150-UB firmware.

## Artifact

- **Path in extracted filesystem:** `bin/httpd`
- **Format:** ELF executable
- **Architecture:** MIPS32, 32-bit
- **Endianness:** Big-endian (MSB)
- **ELF version:** Version 1 (SYSV)
- **Linking:** Dynamically linked
- **Dynamic loader:** `/lib/ld-uClibc.so.0`
- **Section headers:** Absent

## Procedure

The following command was executed against the extracted HTTP server:

```bash
file rootfs-extracted/bin/httpd
```

## Observed Output

```text
ELF 32-bit MSB executable, MIPS, MIPS32 version 1 (SYSV),
dynamically linked, interpreter /lib/ld-uClibc.so.0,
no section header
```

## Interpretation

The binary targets a 32-bit, big-endian MIPS environment. Its dynamic loader indicates that it was built for a system using uClibc.

The lack of section headers limits some forms of static analysis but does not prevent disassembly or decompilation. Ghidra can analyze executable code using the ELF program information and processor-specific instruction decoding.

The executable was used to investigate the router's configuration-backup processing path.

## Limitations

This command establishes the executable's reported format and architecture. It does not independently verify every ABI detail or prove a vulnerability.

The inspected executable is excluded from this repository.
