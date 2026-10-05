# Harbor registry overlay for ansible-automation

The transfer tool is [harbor-airgap-kit](https://github.com/noahfarshad/harbor-airgap-kit). This repository is the same registry build, as a drop-in for ansible-automation: Podman or Docker CE, one example inventory, three settings.

License: GPL-3.0. Built and proved out for [essential.coach](https://essential.coach).
Full write-up: [Air-Gapped VKS on VCF 9: One Inventory, Three Settings](https://essential.coach/air-gapped-vks-on-vcf-9/).

Drop-in additions for an air-gapped Harbor registry, laid out to match the
`ansible-automation` repository: single-purpose roles with the `desired_state`
dispatcher, a paved-path playbook under `playbooks/linux/`, and all
environment-specific values in `group_vars` and `host_vars`.

## What's here

```
roles/
  stage_harbor_artifacts/        connected side
  install_podman_offline/        default runtime
  install_docker_offline/        harbor_container_runtime: docker
  download_harbor_artifacts/
  configure_harbor_tls/
  install_harbor/
  configure_harbor_projects/
  verify_harbor/
playbooks/linux/
  stage_harbor_artifacts.yml
  configure_registry.yml
  verify_registry.yml
  remove_registry.yml               required before a runtime switch
inventory.example/
  hosts.registry                          lines to add to hosts
  group_vars/all/artifacts.yml            environment-wide artifact source
  group_vars/all/vault.yml.example.registry
  group_vars/registry/main.yml            registry tier defaults
  group_vars/registry/harbor_artifacts.yml  placeholder, replaced by staging
  host_vars/registry01.example.coach.yml
docs/HARBOR_REGISTRY.md                   reference
docs/EXAMPLES.md                          every step, with example values
examples/example/                         one inventory, three settings
Makefile.registry                         targets to append to Makefile
```

## Merging

```bash
cp -r roles/* <repo>/roles/
cp playbooks/linux/*.yml <repo>/playbooks/linux/
cp -r inventory.example/group_vars/registry <repo>/inventory.example/group_vars/
cp inventory.example/group_vars/all/artifacts.yml <repo>/inventory.example/group_vars/all/
cp inventory.example/host_vars/registry01.example.coach.yml <repo>/inventory.example/host_vars/
cp docs/HARBOR_REGISTRY.md <repo>/docs/
cat Makefile.registry >> <repo>/Makefile
```

Then add the `[registry]` group from `hosts.registry` to the inventory, add the
vault entries from `vault.yml.example.registry`, and add the three new targets to
`make help`. `docs/HARBOR_REGISTRY.md` covers the rest.

## How the variables layer

| Scope | File | Holds |
|---|---|---|
| Environment | `group_vars/all/artifacts.yml` | `nexus_enabled`, staged or Nexus |
| Tier | `group_vars/registry/main.yml` | `harbor_container_runtime` podman or docker, TLS mode, projects |
| Release | `group_vars/registry/harbor_artifacts.yml` | Version and sha256, generated on the connected side |
| Host | `host_vars/<host>.yml` | `harbor_hostname`, data disk, any per-host source override |
| Secrets | `group_vars/all/vault.yml` | Harbor passwords; repository credentials reuse `vault_repo_*` |

Each air-gapped environment keeps its own inventory, so the same roles and
playbooks run everywhere and only these files differ.

## Checks run

- `ansible-playbook --syntax-check` passes on all three playbooks, merged into the repo.
- `ansible-lint` passes the roles and playbooks at the production profile. Role
  variables use component prefixes (`harbor_`, `docker_offline_`, `artifact_`),
  as the repo's own roles do (`apache_`), so skip `var-naming[no-role-prefix]`.
- Inventory files keep the repo's aligned-column style, which yamllint's default
  `colons` rule flags across the repo already.

## One thing noticed in the repo

With ansible-core 2.21, `collections_paths` in `ansible.cfg` isn't read, so
`ansible.posix.firewalld` can't be found. `collections_path` (singular) works.
