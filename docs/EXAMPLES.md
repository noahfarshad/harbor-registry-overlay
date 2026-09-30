# Harbor registry — worked examples

Everything needed to go from nothing to a running registry in one air-gapped
environment, with example values. Names use `example.coach`; replace them with
your own.

| What | Example value |
|---|---|
| Registry host | `registry01.example.coach`, RHEL 9, IdM client |
| Registry name | `registry.example.coach` → `10.10.10.40` |
| Harbor release | `2.14.0` |
| Nexus | `https://nexus.example.coach/repository/vmware-raw` |
| Artifactory | `https://artifactory.example.coach/artifactory/vmware-generic` |
| Transfer mount | `/var/staging/artifacts` on the registry host, or `/mnt/transfer/artifacts` on the control host |

---

## 1. What you need

| Item | Where it comes from | Section |
|---|---|---|
| Harbor offline installer and its sha256 manifest | Connected side, `make stage-harbor` | 2 |
| A way in: Nexus, Artifactory or attached storage | Inside the environment | 3 |
| Docker Engine and Compose packages | Local yum mirror or staged RPMs | 4 |
| DNS for the registry name, and an IdM service principal | IdM | 5 |
| An inventory and vault | `examples/` | 6 |

---

## 2. Connected side — stage the kit

Stage only:

```bash
make stage-harbor STAGE_ARGS="-e stage_harbor_version=2.14.0"
```

Stage with Docker RPMs, for environments without a yum mirror:

```bash
make stage-harbor STAGE_ARGS="-e stage_harbor_version=2.14.0 -e stage_harbor_docker_rpms=true"
```

Stage and push to a Nexus instance the transfer process replicates:

```bash
make stage-harbor STAGE_ARGS="-e stage_harbor_version=2.14.0 \
  -e stage_harbor_push=true \
  -e stage_harbor_repo_url=https://nexus.example.coach/repository/vmware-raw \
  -e stage_harbor_repo_username=svc-stage -e stage_harbor_repo_password=..."
```

Stage and push to Artifactory with an access token — easiest through a vars file:

```bash
cat > stage-artifactory.yml <<'VARS'
stage_harbor_version: "2.14.0"
stage_harbor_push: true
stage_harbor_repo_url: https://artifactory.example.coach/artifactory/vmware-generic
stage_harbor_repo_headers:
  Authorization: "Bearer <token>"
VARS
make stage-harbor STAGE_ARGS="-e @stage-artifactory.yml"
```

What lands in `artifacts/harbor/`:

```
artifacts/harbor/
  harbor-offline-installer-v2.14.0.tgz
  harbor-offline-installer-v2.14.0.tgz.asc
  harbor_artifacts.yml
  docker/                                   only with stage_harbor_docker_rpms=true
    containerd.io-<ver>.el9.x86_64.rpm
    docker-ce-<ver>.el9.x86_64.rpm
    ...
```

`harbor_artifacts.yml` looks like this, with the real checksum:

```yaml
harbor_version:           "2.14.0"
harbor_installer_file:    "harbor-offline-installer-v2.14.0.tgz"
harbor_installer_sha256:  "<64 hex characters>"
```

---

## 3. Inside the environment — get the kit where the play can reach it

The play looks for the same layout wherever the files are:

```
<artifact_repo_url or artifact_staging>/
  harbor/
    harbor-offline-installer-v2.14.0.tgz
    harbor_artifacts.yml
    docker/*.rpm                            only if docker_offline_source: staged
    tls/registry.example.coach.crt          only if harbor_tls_mode: staged
    tls/registry.example.coach.key
```

### Nexus

Create a raw hosted repository once — in the UI under Repositories → Create
repository → raw (hosted), named `vmware-raw` — or with the REST API:

```bash
curl -u admin -X POST https://nexus.example.coach/service/rest/v1/repositories/raw/hosted \
  -H 'Content-Type: application/json' \
  -d '{"name":"vmware-raw","online":true,
       "storage":{"blobStoreName":"default","strictContentTypeValidation":false,"writePolicy":"ALLOW"}}'
```

Upload from the transfer media:

```bash
cd /media/transfer/harbor
for f in harbor-offline-installer-v2.14.0.tgz harbor_artifacts.yml; do
  curl -u svc-upload --upload-file "$f" \
    "https://nexus.example.coach/repository/vmware-raw/harbor/$f"
done
```

### Artifactory

Use a generic repository, `vmware-generic`, and upload with an access token:

```bash
cd /media/transfer/harbor
for f in harbor-offline-installer-v2.14.0.tgz harbor_artifacts.yml; do
  curl -H "Authorization: Bearer $ARTIFACTORY_TOKEN" -T "$f" \
    "https://artifactory.example.coach/artifactory/vmware-generic/harbor/$f"
done
```

### Attached storage

Mount it where `artifact_staging` points before running the play:

```bash
# on the registry host (artifact_staged_on: target)
mount /dev/sr0 /var/staging/artifacts
# or on the control host (artifact_staged_on: controller)
mount repo.example.coach:/export/transfer /mnt/transfer/artifacts
```

---

## 4. Docker Engine and Compose packages

### Option A — mirror into the environment's yum repository

On the connected side:

```bash
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
dnf reposync --repoid=docker-ce-stable --arch=x86_64 --newest-only \
  --download-metadata -p /srv/mirror/
curl -o /srv/mirror/docker-ce-stable/gpg https://download.docker.com/linux/rhel/gpg
```

Carry `/srv/mirror/docker-ce-stable/` across, publish it on the repository
server, and point the registry host at it — with the existing `add_yum_repo`
role or a repo file:

```ini
[docker-ce-stable-local]
name=Docker CE (local mirror)
baseurl=http://repo.example.coach/yum/docker-ce-stable
enabled=1
gpgcheck=1
gpgkey=http://repo.example.coach/yum/docker-ce-stable/gpg
```

Then `docker_offline_source: yum`.

### Option B — staged RPMs

Stage with `stage_harbor_docker_rpms=true`, keep `harbor/docker/` next to the
installer, and set `docker_offline_source: staged`. If the RPMs are not signed by
a key the host already trusts, also set `docker_offline_disable_gpg_check: true`.

---

## 5. DNS and the IdM service principal

For `harbor_tls_mode: ipa`, from an IdM admin session:

```bash
kinit admin
ipa dnsrecord-add example.coach registry --a-rec=10.10.10.40
ipa dnsrecord-add 10.10.10.in-addr.arpa. 40 --ptr-rec=registry.example.coach.
ipa service-add HTTP/registry.example.coach --force
ipa service-add-host HTTP/registry.example.coach --hosts=registry01.example.coach
```

`--force` is needed because `registry.example.coach` is a name, not an IdM host.
`service-add-host` lets `registry01` request the certificate, and certmonger
renews it from then on.

For `harbor_tls_mode: staged`, put the certificate and key under
`harbor/tls/` as shown in section 3.

---

## 6. Inventory and vault

```bash
cp -r examples/nexus inventory          # or artifactory / staged-on-registry-host / staged-on-control-host
cp artifacts/harbor/harbor_artifacts.yml inventory/group_vars/registry/
cp inventory/group_vars/all/vault.yml.example inventory/group_vars/all/vault.yml
$EDITOR inventory/group_vars/all/vault.yml
ansible-vault encrypt inventory/group_vars/all/vault.yml
```

Check what the host will see before running anything:

```bash
ansible-inventory --host registry01.example.coach
```

Look for `artifact_source`, `harbor_hostname`, `harbor_installer_sha256` and
`firewall_open_tcp_ports`, each coming from the scope you expect.

---

## 7. Deploy and verify

```bash
make linux-registry HOST=registry01.example.coach
make verify-registry HOST=registry01.example.coach
```

Example verify summary:

```
Endpoint:    https://registry.example.coach:443
Health:      healthy
Services:    core, jobservice, log, portal, postgresql, proxy, redis, registry, registryctl
Projects:    library, sup-services, tanzu-packages, tkg
Certificate: OK
```

`library` is created by Harbor itself.

A quick check from any host that trusts the IdM CA:

```bash
curl -sI https://registry.example.coach/v2/     # HTTP 401 = registry up, asking for auth
```

Run one part again with tags:

```bash
ansible-playbook playbooks/linux/configure_registry.yml --limit registry01.example.coach --tags projects
ansible-playbook playbooks/linux/configure_registry.yml --limit registry01.example.coach --tags tls
```

---

## 8. Load content and point the Supervisor at it

Push bundles with the existing scripts, or directly with imgpkg:

```bash
imgpkg copy --tar sup-services/<service>.tar \
  --to-repo registry.example.coach/sup-services/<service> \
  --registry-ca-cert-path /etc/ipa/ca.crt
```

Then in vCenter, Supervisor → Configure → Container Registries → Add Registry:

| Field | Example |
|---|---|
| Registry host URL | `registry.example.coach` |
| Credentials | none for public projects, or a Harbor robot account |
| CA certificate | contents of `/etc/ipa/ca.crt` |

---

## 9. Day 2

Upgrade Harbor — stage the new release, replace the manifest, run the play:

```bash
make stage-harbor STAGE_ARGS="-e stage_harbor_version=<new>"
cp artifacts/harbor/harbor_artifacts.yml inventory/group_vars/registry/
make linux-registry HOST=registry01.example.coach
```

`install_harbor` sees the version change and runs Harbor's upgrade procedure.
Check the release notes for the supported version jump first.

Remove Harbor, keeping its data:

```bash
ansible-playbook playbooks/linux/configure_registry.yml --limit registry01.example.coach \
  -e desired_state=absent
```

Remove everything, including registry content:

```bash
ansible-playbook playbooks/linux/configure_registry.yml --limit registry01.example.coach \
  -e desired_state=absent -e harbor_purge_install=true -e harbor_purge_data=true
```
