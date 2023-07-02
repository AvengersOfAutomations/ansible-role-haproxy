# Ansible Playbook For Haproxy With Keepalived

## Roles Variables

```yaml
# Install and configure these tools
docker: true
haproxy: true
keepalived: true

# Haproxy Config
haproxy_home_dir: /opt/haproxy
haproxy_image_version: 2.8

# Keepalived Config
float_ip_auth_pass: 141a653b895287bf9
float_ip_virtual_router_id: 56
float_ip_interface: ens192
float_ips:
    - "45.94.255.4/27"
    - "45.94.255.6/32"
    - "45.94.255.18/32"
```

## Example Playbook

### Inventory

```ini
[haproxy]
172.16.10.151 vrrp_state=MASTER vrrp_priority=101
172.16.10.152 vrrp_state=BACKUP vrrp_priority=100
```

### Installation

```
ansible-playbook -i inventory.ini
```