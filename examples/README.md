# Example inventory

One inventory. The three arrival questions are settings, not four directories.

| Question | Setting | Example value |
|---|---|---|
| Where do the Harbor files come from? | `nexus_enabled` | `false` — staged on the control host. `true` reads them from Nexus |
| Where does the certificate come from? | `harbor_tls_mode` | `staged`. `ipa` if the server is an IdM client |
| Where does the container engine come from? | `harbor_container_runtime` | `podman`. `docker` for Docker CE |

Artifactory is not a fourth inventory. Set `nexus_enabled: true` and point `artifact_repo_url` at the Artifactory generic repository, or set `artifact_repo_headers` to a bearer token.

```bash
cp -r examples/example inventory
cp inventory/group_vars/all/vault.yml.example inventory/group_vars/all/vault.yml
# set the Harbor passwords, then:
ansible-vault encrypt inventory/group_vars/all/vault.yml
# after the connected side has staged the installer:
cp artifacts/harbor/harbor_artifacts.yml inventory/group_vars/registry/
make linux-registry HOST=registry01.example.coach
```

Switching to Docker CE is a remove and a rebuild:

```bash
make remove-registry HOST=registry01.example.coach
# set harbor_container_runtime: docker in group_vars/registry/main.yml
make linux-registry HOST=registry01.example.coach
```

The transfer tool that builds the folder and checks it is [harbor-airgap-kit](https://github.com/noahfarshad/harbor-airgap-kit). These roles are the build that kit runs, laid out to drop into ansible-automation.
