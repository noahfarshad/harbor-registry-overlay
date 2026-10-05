# Changelog

## [1.1.0] - 2026-10-05

The four example inventories were four answers to three questions. The roles now match the air-gap kit.

- `configure_registry.yml` installs Podman or Docker CE from `harbor_container_runtime`. It does not pull in an Ansible collection for firewalld.
- `remove_registry.yml` is required before that setting changes. The data volume stays.
- `examples/example` is the one inventory. Nexus, a staged certificate, and Podman are the defaults. Artifactory is a repository URL, not a second tree.
- The transfer tool remains harbor-airgap-kit. This repo is the build that drops into ansible-automation.


All notable changes to this project are documented here.

## [1.0.0] — 2026-09-30

### Added

- Initial public release. Ansible overlay that stands up an air-gapped Harbor registry: stage artifacts on a connected host, install Docker offline, install Harbor, configure TLS and projects, and verify. Four complete example inventories: Nexus, Artifactory, a transfer disk on the registry host, and a transfer disk on the control host.

### Notes

- All customer-specific identifiers have been replaced with generic example values.
- Configuration files use placeholder credentials (REPLACE-WITH-*) that must be replaced with your own before use.
- Hostnames follow the *.example.coach pattern.
