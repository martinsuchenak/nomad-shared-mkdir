# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The version reported by the plugin's `fingerprint` operation is kept in sync with
the git tag for each release.

## [Unreleased]

## [0.1.0] - 2026-06-20

### Added
- **Self-healing reconciliation** on `fingerprint`: registers any cluster
  volumes that exist on other nodes but are missing a registration on the local
  node, pointing them at the shared NFS path. This recovers nodes that were down
  or absent when a volume was originally created.
- The reconciliation runs at most once per boot or Nomad service restart,
  guarded by a marker file keyed to the boot ID and the Nomad agent's PID/start
  time, so repeated fingerprint calls are cheap no-ops.
- Log rotation for the fingerprint sync log (`/tmp/nomad-shared-mkdir-sync.log`),
  rotated at 1 MB keeping a single previous copy.
- "How It Works" section in the README documenting the `create`, `delete`, and
  `fingerprint` reconciliation flows and the shared-filesystem assumption.

### Changed
- Bumped the plugin `fingerprint` version from `0.0.1` to `0.1.0`.

## [0.0.1]

### Added
- Initial unified Nomad dynamic host volume plugin.
- `create`: makes the local directory and fans out registration to every other
  `ready` node sharing the same path.
- `delete`: removes the local directory and deregisters sibling registrations
  sharing the volume name, guarded against recursion.

[Unreleased]: https://github.com/martinsuchenak/nomad-shared-mkdir/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/martinsuchenak/nomad-shared-mkdir/releases/tag/v0.1.0
[0.0.1]: https://github.com/martinsuchenak/nomad-shared-mkdir/releases/tag/v0.0.1
