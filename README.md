# netactuate-ansible-pdns

Deploys PowerDNS authoritative server on existing anycast worker nodes for globally
distributed DNS with BGP anycast routing. Every node serves the same zones, and
DNS queries are automatically routed to the nearest PoP.

## What This Deploys

On each existing anycast node:

1. MariaDB for zone storage
2. PowerDNS authoritative server with MySQL backend
3. PowerDNS API enabled (localhost only)
4. DNS listening on the anycast IP and localhost

## Security

**Never commit real credentials.** The `group_vars/all` file contains placeholder
values for `pdns_db_password` and `pdns_api_key`. For production deployments, use
ansible-vault:

```bash
ansible-vault encrypt_string 'your-real-password' --name 'pdns_db_password'
ansible-vault encrypt_string 'your-real-api-key' --name 'pdns_api_key'
```

Then run playbooks with `--ask-vault-pass` or `--vault-password-file`.

## Prerequisites

- Existing anycast nodes from [netactuate-ansible-bgp-bird2](https://github.com/netactuate/netactuate-ansible-bgp-bird2)
  with BGP sessions established
- Ansible venv set up (from the BGP deployment)
- `community.mysql` Ansible collection:
  ```bash
  ansible-galaxy collection install community.mysql
  ```

## Configuration

### group_vars/all

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `pdns_db_name` | string | `pdns` | MariaDB database name |
| `pdns_db_user` | string | `pdns` | MariaDB user |
| `pdns_db_password` | string | `CHANGEME` | MariaDB password — use vault |
| `pdns_api_key` | string | `CHANGEME` | PowerDNS API key — use vault |
| `pdns_admin_email` | string | `admin@example.com` | SOA contact email |
| `pdns_anycast_ip` | string | required | Anycast IP PowerDNS listens on |

### Inventory

Reuse the `hosts` file from your BGP deployment.

## Deployment

```bash
source .venv/bin/activate
ansible-playbook pdns-auth.yaml
```

## Validation

```bash
dig @YOUR_ANYCAST_IP version.bind chaos txt
```

Expected: response from PowerDNS with version string.

To test a zone (after creating one via the API or pdnsutil):

```bash
dig @YOUR_ANYCAST_IP example.com SOA
```

## Managing Zones

Use `pdnsutil` on any node or the PowerDNS API:

```bash
# SSH to a node
ssh ubuntu@<node-ip>
sudo pdnsutil create-zone example.com ns1.example.com
sudo pdnsutil add-record example.com @ A 3600 192.0.2.1
```

## Related Resources

- [netactuate-ansible-bgp-bird2](https://github.com/netactuate/netactuate-ansible-bgp-bird2)
- [PowerDNS Documentation](https://doc.powerdns.com/)

## Need Help?

- NetActuate support: support@netactuate.com
