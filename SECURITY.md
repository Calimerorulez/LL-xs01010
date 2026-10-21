# Public documentation and device-secret policy

This is a **public** research repository. Publish only sanitized platform-level technical findings.

## Never commit
- PON serial numbers, vendor-assigned ONU identifiers, LOID and PLOAM passwords
- ONT-equipment identity tuples copied from an operator-provided device
- Device MAC addresses, EEPROM contents, unique calibration/factory data
- Unredacted U-Boot environment or boot logs (may expose identity/secrets)
- `/etc/shadow`, SSH host/private keys, passwords, tokens and credentials
- Raw NAND/MTD/UBI dumps, full firmware dumps, config backups or support bundles
- Internal/home/public IP mappings, account-specific/provider authentication details
- Unreviewed binary firmware, proprietary SDK files or source under unclear licenses

## Publication procedure
1. Capture originals **privately**, off GitHub.
2. Extract only relevant generic facts into Markdown.
3. Review all outputs for secrets, addresses and identifiers.
4. Review staged changes with `git diff --cached` before pushing.
5. Treat accidental disclosure as compromise: remove sensitive material from history, rotate credentials/identities when applicable, and assess exposure.

A `.gitignore` is only a guardrail: it does not sanitize files already tracked or text pasted into Markdown.

## Operational safety
No optical-registration claim is implied. No flashing, writing flash/environment, or altering production PON credentials is part of this documentation.
