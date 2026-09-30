configure_harbor_tls
====================

Puts the registry certificate in place.

| `harbor_tls_mode` | Behaviour |
|---|---|
| `ipa` | `ipa-getcert request` for `harbor_tls_principal`; certmonger tracks and renews it |
| `staged` | Copies a certificate and key from attached storage |

For `ipa`, create the service principal once, from any IdM admin session:

    ipa service-add HTTP/registry.example.coach
    ipa service-add-host HTTP/registry.example.coach --hosts=registry01.example.coach

If `harbor_hostname` is an alias rather than the host's own FQDN, add the alias
as a DNS record and let the host manage the principal as above.

Role Variables
--------------

| Variable | Default | What it is |
|---|---|---|
| `harbor_tls_mode` | `ipa` | Certificate source |
| `harbor_tls_cert` / `harbor_tls_key` | `/etc/pki/tls/...{{ harbor_hostname }}` | Paths Harbor reads |
| `harbor_tls_principal` | `HTTP/{{ harbor_hostname }}` | IdM service principal |
| `harbor_tls_staged_cert` / `_key` | under `artifact_staging/harbor/tls/` | Sources for `staged` |

License
-------

GPL-3.0-only
