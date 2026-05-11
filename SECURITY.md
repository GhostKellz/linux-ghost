# Security Policy

## Scope

linux-ghost is a custom kernel build configuration (PKGBUILD, patches, config) for Arch Linux. We do not maintain the Linux kernel source itself — upstream kernel security is handled by kernel.org and CachyOS.

This policy covers:
- Security issues in our patches (`patches/`)
- Dangerous misconfigurations in our kernel config (`config/`)
- Vulnerabilities introduced by our PKGBUILD build process
- Supply chain issues (compromised source URLs, checksum integrity)

## Supported Versions

Only the latest release is supported with security updates.

| Version | Supported |
|---------|-----------|
| 7.0.x   | Yes       |
| < 7.0   | No        |

## Reporting a Vulnerability

**Do not open a public issue for security vulnerabilities.**

Email: **ghost@ghostkellz.sh**

Include:
- Description of the vulnerability
- Which file(s) are affected (patch, config, PKGBUILD)
- Steps to reproduce or proof of concept
- Impact assessment

You should receive a response within 72 hours. If confirmed, a fix will be released as a new tagged version.

## Kernel Security

For vulnerabilities in the Linux kernel itself, report to the upstream kernel security team:
- https://www.kernel.org/doc/html/latest/process/security-bugs.html

For CachyOS-specific patches:
- https://github.com/CachyOS/linux/issues

## Build Verification

All source tarballs and patches use `SKIP` checksums in the PKGBUILD. Users building from source should verify:
- The CachyOS source tarball matches the expected release tag
- Patches from `patches/` match the committed versions in this repository
- Remote patches (CachyOS, linux-tkg) are fetched over HTTPS
