# Changelog

All notable changes to this project are documented here.

## [1.0.0] — 2026-09-30

### Added

- Initial public release. Ansible overlay that stands up an air-gapped Harbor registry: stage artifacts on a connected host, install Docker offline, install Harbor, configure TLS and projects, and verify. Four complete example inventories: Nexus, Artifactory, a transfer disk on the registry host, and a transfer disk on the control host.

### Notes

- All customer-specific identifiers have been replaced with generic example values.
- Configuration files use placeholder credentials (REPLACE-WITH-*) that must be replaced with your own before use.
- Hostnames follow the *.example.coach pattern.
