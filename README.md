# infra-ansible-legacy

Legacy Ansible playbook used to provision and deploy a **telco environment** stack (API + database + cache layers) across multiple environments. Archived here as a portfolio reference — credentials have been stripped and replaced with `ansible-vault` placeholders (see `group_vars/all.yml`).

## Purpose

Automates provisioning of the servers backing a telco API platform: base OS setup, PHP7 runtime, and the supporting services (Nginx, MySQL, Redis) needed to run the API across `dev`, `preprod`, and `prod` environments.

## Architecture & layout

The repo follows standard Ansible inventory conventions, with variables scoped by **group** and **host** rather than hardcoded in tasks — this is what lets the same `common` role/playbook run unmodified across every environment.

```
.
├── hosts.ini              # inventory: defines the [system] group (local dev host)
├── playbook.yml           # entry point: applies the "common" role to all hosts
├── group_vars/
│   ├── all.yml            # vars shared by every host (env flag, SSH/become auth)
│   ├── api.yml             # per-environment debug flag for the "api" group (dev/preprod/prod)
│   └── mysql.yml           # env flag for the "mysql" group
├── host_vars/
│   ├── server.dev.yml      # dev: local dev server, http_port 985
│   ├── api.preprod.yml     # preprod: nginx/mysql/redis sub-hosts, ports (80, 303)
│   └── api.prod.yml        # prod host vars (currently empty stub)
└── roles/
    └── common/             # Galaxy-scaffolded role: base OS + PHP7 provisioning
        ├── tasks/main.yml  # adds Sury PHP repo/APT key, installs base toolchain
        ├── defaults/, vars/, handlers/, meta/
        └── tests/          # Travis CI syntax-check harness (test.yml + inventory)
```

**Organization pattern:** environment identity (`dev` / `preprod` / `prod`) and per-host config (ports, service topology) are pushed down into `group_vars` / `host_vars`, keeping `roles/common` environment-agnostic. Each environment's stack (`nginx_*`, `mysql_*`, `redis_*`) is declared as a sub-host block under its environment key in `host_vars`, mapping the deployment topology (which services run, which ports they bind) without touching playbook logic.

## Stack deployed

- **OS bootstrap**: apt housekeeping, common CLI tooling (curl, wget, git, vim, tree, keychain, sshpass, pkg-config, libyaml-dev...)
- **PHP 7** via the Sury/DEB.SURY.ORG APT repository (Debian Buster)
- **Node.js / Yarn** for front-end asset tooling alongside the PHP stack
- **Nginx** — reverse proxy / web server (preprod: port 80)
- **MySQL** — primary datastore (preprod: port 303)
- **Redis** — cache layer (preprod: port 303)

## Environments

| Env | Inventory host(s) | Notes |
|---|---|---|
| dev | `server.dev` | local, single-host, port 985 |
| preprod | `api.preprod` (+ nginx/mysql/redis sub-hosts) | ports 80/303 |
| prod | `api.prod` | host vars stub, mirrors preprod topology |

## Running

```bash
ansible-playbook -i hosts.ini playbook.yml
```

Requires vault-encrypted values for `vault_ansible_ssh_pass` / `vault_ansible_become_pass` referenced in `group_vars/all.yml`:

```bash
ansible-vault encrypt_string 'yourpassword' --name 'vault_ansible_ssh_pass'
```

## Status

Legacy project (2019), kept as-is for reference. Not maintained for current Ansible versions.
