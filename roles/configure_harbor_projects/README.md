configure_harbor_projects
=========================

Creates the Harbor projects through the API once Harbor reports healthy. Running
it again is safe: an existing project returns 409 and is left alone.

The Supervisor and VKS content needs three projects:

    harbor_projects:
      - { name: sup-services,   public: true }
      - { name: tanzu-packages, public: true }
      - { name: tkg,            public: true }

Role Variables
--------------

| Variable | Default | What it is |
|---|---|---|
| `harbor_projects` | `[]` | Projects to create — set in `group_vars/registry` |
| `harbor_api_url` | built from `harbor_hostname` | API base |
| `harbor_api_validate_certs` | `true` | Keep on once the IdM CA is trusted |

`desired_state: absent` deletes only empty projects.

License
-------

GPL-3.0-only
