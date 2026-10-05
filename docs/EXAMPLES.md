# Harbor registry — one example

Names use `example.coach`. Replace them. There is one inventory, `examples/example`. The old four directories (Nexus, Artifactory, transfer on the registry host, transfer on the control host) were four ways of answering the same three questions. Those answers are settings now.

| Question | Setting | In the example |
|---|---|---|
| Where do the Harbor files come from? | `nexus_enabled` | `false`. The build reads the staged transfer. `artifact_staged_on: controller` |
| Where does the certificate come from? | `harbor_tls_mode` | `staged` |
| Where does the container engine come from? | `harbor_container_runtime` | `podman` |

Artifactory is the same `nexus_enabled: true` path with `artifact_repo_url` pointed at the generic repository. It is not a separate inventory.

```bash
cp -r examples/example inventory
cp inventory/group_vars/all/vault.yml.example inventory/group_vars/all/vault.yml
ansible-vault encrypt inventory/group_vars/all/vault.yml
make linux-registry HOST=registry01.example.coach
```

To move that host to Docker CE, remove Harbor first, change `harbor_container_runtime` to `docker`, then build again:

```bash
make remove-registry HOST=registry01.example.coach
make linux-registry HOST=registry01.example.coach
```

The folder that crosses the air gap, and the checks on it, are built by [harbor-airgap-kit](https://github.com/noahfarshad/harbor-airgap-kit). This overlay is the Ansible build those commands run, in the ansible-automation layout.
