install_docker_offline
======================

Installs Docker Engine and the Compose plugin with no internet access. The
existing `install_docker` role adds `download.docker.com`, which an air-gapped
environment can't reach; this one only uses what is already inside.

| `docker_offline_source` | Behaviour |
|---|---|
| `yum` | Install from the environment's yum repository — the packages must be mirrored there |
| `staged` | Install the RPMs under `artifact_staging/docker_offline_rpm_path` |

Harbor's installer is built for Docker Engine and Compose; it doesn't run on Podman.

Role Variables
--------------

| Variable | Default | What it is |
|---|---|---|
| `docker_offline_source` | `yum` | Package source |
| `docker_offline_packages` | Docker CE set | Packages to install |
| `docker_offline_rpm_path` | `harbor/docker` | Sub-path under `artifact_staging` |
| `docker_offline_allowerasing` | `true` | Resolve conflicts with Podman's docker shim |
| `docker_offline_disable_gpg_check` | `false` | Only for staged RPMs without an imported key |

Example Playbook
----------------

    - hosts: registry
      become: true
      roles:
        - { role: install_docker_offline, tags: [docker, install] }

License
-------

GPL-3.0-only
