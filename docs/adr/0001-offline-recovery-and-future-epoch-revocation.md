# ADR-0001: Require offline recovery and future-epoch revocation

**Status:** accepted

Version 1 requires a user-held offline recovery secret in addition to normal
surviving-device recovery, because a surviving-device-only model can
permanently strand a self-hosted workspace after device loss. The daemon never
receives that secret or the workspace content-encryption key. Revocation takes
effect immediately for new daemon requests and rotates future payload-key
epochs for remaining authorized devices; v1 does not re-encrypt retained
operations or snapshots. This cannot erase data a revoked device already
decrypted, and losing every authorized device and the recovery secret remains
permanent loss.

## Consequences

The exact recovery-secret format, display, storage advice, recovery package,
authority replacement, and new-epoch distribution remain Phase 0 protocol
work. Test vectors must prove that recovery material never reaches the daemon
and that revoked devices cannot use a future epoch.
