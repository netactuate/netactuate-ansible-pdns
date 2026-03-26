# netactuate-ansible-pdns — AI Provisioning Context

Deploys PowerDNS authoritative on anycast nodes. Runs after BGP deployment.

## Security — Read This First

`pdns_db_password` and `pdns_api_key` must NEVER be plaintext in production.
Use ansible-vault:
```bash
ansible-vault encrypt_string 'real-password' --name 'pdns_db_password'
```

## Sequencing

1. `netactuate-ansible-bgp-bird2`: `createnode.yaml` → `bgp.yaml`
2. **This repo**: `pdns-auth.yaml`

## Required Inputs

- Existing inventory from BGP deployment
- `pdns_anycast_ip` — customer's anycast IP
- `pdns_db_password` — secure password for MariaDB
- `pdns_api_key` — secure key for PowerDNS API

## What to Do

1. Set variables in `group_vars/all`
2. Install community.mysql collection: `ansible-galaxy collection install community.mysql`
3. Run: `ansible-playbook pdns-auth.yaml`

## Validation

```bash
dig @ANYCAST_IP version.bind chaos txt
```

Should return PowerDNS version.

## Files

- `pdns-auth.yaml` — main playbook (pre_tasks: MariaDB + schema; role: pdns config)
- `roles/pdns/templates/pdns.conf.j2` — PowerDNS config
- `group_vars/all` — credentials (CHANGEME placeholders)

## Config Location on Nodes

- `/etc/powerdns/pdns.conf` — main config (mode 0600)
- MariaDB database `pdns` — zone and record storage
- PowerDNS API: `http://127.0.0.1:8081` (localhost only)
