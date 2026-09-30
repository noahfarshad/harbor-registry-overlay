install_harbor
==============

Installs Harbor from its official offline installer. Harbor runs as Docker
Compose services; the installer carries its own images, so nothing is pulled
from outside. A systemd unit brings it up at boot.

`harbor.yml` is created from the `harbor.yml.tmpl` the installer ships, and only
the hostname, ports, TLS paths, passwords and data volume are changed. That keeps
the file correct across Harbor releases. A hand-edited `harbor.yml` is never
overwritten; changed values trigger a re-run of `install.sh`.

The installed release is recorded in `.installed_version`. When
`harbor_version` changes, `tasks/upgrade.yml` runs Harbor's upgrade procedure:
stop, copy the database directory, move the old release aside, unpack the new
one, migrate `harbor.yml` with the new `prepare` image, and reinstall.

Requirements
------------

`install_docker_offline`, `download_harbor_artifacts` and `configure_harbor_tls`
run first — `playbooks/linux/configure_registry.yml` does this in order.

Role Variables
--------------

| Variable | Default | What it is |
|---|---|---|
| `harbor_hostname` | — | **REQUIRED** — the name clients use; set in host_vars |
| `harbor_admin_password` / `harbor_db_password` | — | **REQUIRED** — from vault |
| `harbor_install_parent` | `install_root` | Where the installer unpacks |
| `harbor_data_volume` | `/data` | Registry storage; a separate disk is recommended |
| `harbor_http_port` / `harbor_https_port` | `80` / `443` | Listener ports |
| `harbor_with_trivy` | `false` | Trivy needs an internet-fed database |
| `harbor_purge_install` / `harbor_purge_data` | `false` | Extra clean-up for `desired_state: absent` |

License
-------

GPL-3.0-only
