E-06 — Backup Configuration XML Structure

Objective

Document the structure of the XML configuration recovered during local analysis of the FiberHome HG150-UB router backup file.

The purpose of this evidence is to demonstrate the recovered file format and identify configuration areas that may contain sensitive information, without publishing private router settings.

Device and Firmware

Device: FiberHome HG150-UB

Firmware version: HG150-Ub_V3.0

Hardware revision: HG150-Ub_R3A

Firmware build: 180607_1556

Analysis Summary

The locally decrypted backup output begins with an XML declaration and a configuration root element:

<?xml version="1.0"?>
<DslCpeConfig version="3.0">

This indicates that the recovered configuration is represented as structured XML rather than an opaque binary document.

The analysis was performed on a local copy of the backup. Actual configuration values, credentials, and other sensitive data are intentionally excluded from this report.

XML Structure Review

The recovered XML can be inspected locally to identify:

The root configuration element and version attribute.

Nested configuration sections and their hierarchy.

Configuration keys related to network interfaces and connectivity.

Authentication-related fields that require careful handling.

Whether configuration values are stored as readable text or encoded representations.

These are analysis categories, not a claim that every category was confirmed in the recovered file.

Security Considerations

Configuration backups can contain sensitive information, depending on the device and enabled features. Examples may include:

Wireless network names and passwords.

ISP connection settings.

Administrative credentials or authentication material.

Network addressing and service configuration.

Device-specific identifiers.

The presence of any particular item must be verified from the local artifact before making a finding.

If sensitive values are present, they should remain private and must not be included in public screenshots, Git commits, or repository files.

Evidence Handling

Original firmware and backup artifacts remain local.

Decrypted configuration contents are not included in this repository.

Screenshots should show only structural information, with sensitive values redacted.

No live network testing or modification of third-party devices is required for this analysis.
