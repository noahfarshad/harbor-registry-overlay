# Example inventories

Four complete inventories, one per way an air-gapped environment can receive the
kit. Pick the one that matches, copy it to `inventory/`, and replace the values.

| Example | Artifacts from | Certificate | Docker packages |
|---|---|---|---|
| `nexus/` | Nexus raw hosted, basic auth | IdM (`ipa`) | Local yum mirror |
| `artifactory/` | Artifactory generic, access token | IdM (`ipa`) | Local yum mirror |
| `staged-on-registry-host/` | Transfer disk mounted on the registry host | Staged | Staged RPMs |
| `staged-on-control-host/` | Transfer disk mounted on the control host | IdM (`ipa`) | Staged RPMs |

```bash
cp -r examples/nexus inventory
cp inventory/group_vars/all/vault.yml.example inventory/group_vars/all/vault.yml
$EDITOR inventory/group_vars/all/vault.yml && ansible-vault encrypt inventory/group_vars/all/vault.yml
cp artifacts/harbor/harbor_artifacts.yml inventory/group_vars/registry/
make linux-registry HOST=registry01.example.coach
```

`docs/EXAMPLES.md` walks through every step around these files.
