## zcash_script

[![Latest Version][version-badge]][version-url]
[![Rust Documentation][docs-badge]][docs-url]

[version-badge]: https://img.shields.io/crates/v/zcash_script.svg
[version-url]: https://crates.io/crates/zcash_script
[docs-badge]: https://img.shields.io/badge/docs-latest-blue.svg
[docs-url]: https://docs.rs/zcash_script

Rust implementation of the Zcash Script language..

### Publishing New Releases

This crate maintains **two concurrent release lines**: a `main`-based release (`0.5.x`) and a
backport release on a separate branch (`0.4.x`). Both must be released together when
publishing fixes that apply to both lines.

#### Main-line release (`0.5.x`)

1. Decide the release level (`patch`, `minor`, or `major`).
2. Bump the version and apply changelog/doc replacements:
   ```bash
   cargo release version --verbose --execute --allow-branch '*' -p zcash_script patch # [ major | minor ]
   cargo release replace --verbose --execute --allow-branch '*' -p zcash_script
   ```
3. Update `CHANGELOG.md`.
4. Open a PR with the changes, get it reviewed, and wait for CI to pass.
5. Create a [new GitHub release](https://github.com/ZcashFoundation/zcash_script/releases/new) for the tag.
6. Publish the crate:
   ```bash
   cargo release publish --verbose --execute -p zcash_script
   ```

#### Backport release (`0.4.x`)

1. Create a base branch from the previous release tag (e.g. `zcash_script-v0.4.2`):
   ```bash
   git checkout -b backport/zcash_script-0.4.3-base zcash_script-v0.4.2
   git push origin backport/zcash_script-0.4.3-base
   ```
2. Create a working branch from the base branch:
   ```bash
   git checkout -b backport/zcash_script-0.4.3
   ```
3. Cherry-pick the relevant commits from `main` (excluding release/merge commits).
4. Bump the version and apply replacements on the backport branch:
   ```bash
   cargo release version --verbose --execute --allow-branch '*' -p zcash_script patch
   cargo release replace --verbose --execute --allow-branch '*' -p zcash_script
   ```
5. Update `CHANGELOG.md`.
6. Open a PR from `backport/zcash_script-0.4.3` targeting `backport/zcash_script-0.4.3-base`, get it reviewed, and wait for CI to pass.
7. Create a [new GitHub release](https://github.com/ZcashFoundation/zcash_script/releases/new) for the tag.
8. Publish the crate from the backport tag:
   ```bash
   cargo release publish --verbose --execute -p zcash_script
   ```

## Experimental WOTS callback

This fork adds an `experimental-wots` feature for the disposable-key Zakura
regtest demo. It is off by default. Execution additionally requires the caller
to set `Flags::CHECKWOTS` and implement `SignatureChecker::check_wots`; the
default callback rejects every authorization.

Opcode `0xba` consumes a 32-byte key commitment and five chunks each for the
2,176-byte public key and 2,144-byte signature. Chunks are 512 bytes except for
the final 128-byte public-key chunk and 96-byte signature chunk. The caller
checks the commitment and signature against its transaction digest. The
interpreter does not implement the cryptographic primitive or wallet key state.

The demo uses a WOTS-only P2SH redeem script, `<32-byte key commitment> 0xba`.
An ordinary EC output remains a separate output type; this template has no EC
fallback or branch selector. The signature chunks are pushed first, followed by
the public-key chunks and the redeem script. P2SH executes that redeem script,
which pushes the commitment before calling the opcode.
