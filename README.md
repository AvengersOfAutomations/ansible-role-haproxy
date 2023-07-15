# Ansible Playbook For Setup (Haproxy, Keepalived)

## Roles Variables

### Install and Configure Option
Choose which tools install on your host

```yaml
docker: true
haproxy: true
keepalived: true
sysctl_tuning: true
```

### Haproxy Config
Haproxy cloud install with docker container or linux service

```yaml
haproxy_install_type: service # service / docker-compose
haproxy_health_check_port: 8888
# Golabal configs
haproxy_global:
  user: haproxy
  group: haproxy
  chroot: /var/lib/haproxy
  home_dir: /opt/haproxy
  version: 2.8
  maxconn: 300000
# dashboard config
haproxy_dashboard:
  user: admin
  password: admin
# Haproxy timeout configs
haproxy_timeout:
  connect: 5000
  client: 50000
  server: 50000
# Haproxy monitoring configs
haproxy_monitoring_config:
  port: 9090
  uri: /haproxy?stats
  refresh: 10s
  maxconn: 100
  timeout:
    client: 300s
    server: 300s
    connect: 300s
    queue: 300s
# Haproxy logging configs
haproxy_logging:
  driver: json-file
  maxfile: '3'
  maxsize: 100m
```

### Keepalived Config
Dynamically Add/Remove VRRP Instances

```yaml
vrrp_auth_type: PASS

# vrrp health check haproxy config
vrrp_script_chk_haproxy: 
  script: "killall -0 haproxy"
  interval: 2
  weight: 2

vrrp_instances:
  - vrrp_instance_name: VI_1
    virtual_ipaddress: "84.106.57.166/27"
    virtual_router_id: "56"
    auth_password: "141a653b895287bf9"
  - vrrp_instance_name: VI_2
    virtual_ipaddress: "84.106.57.167/32"
    virtual_router_id: "57"
    auth_password: "141a653b895287ff9"
  - vrrp_instance_name: VI_3
    virtual_ipaddress: "84.106.57.168/32"
    virtual_router_id: "58"
    auth_password: "141a653b895287vf9"
```

### Keepalived Prometheus Exporter Config
```yaml
keepalived_exporter_config:
  version: 1.3.0
  port: 9165
  path: /metrics
```

### Sysctl Config
Dynamically Add/Remove sysctl parameters
```yaml
sysctl_config:
  - name: net.ipv4.ip_forward  # Enable IP forwarding
    value: 1
  - name: net.ipv6.conf.all.disable_ipv6 # Disable IPv6 for all network interfaces
    value: 1 
  - name: net.ipv6.conf.default.disable_ipv6 # Disable IPv6 for the default network interface
    value: 1  
  - name: net.ipv4.ip_local_port_range # Set the range of local ports for outgoing connections
    value: 32768 60999  
  - name: net.core.default_qdisc # Set the default queuing discipline to Fair Queueing
    value: fq  
  - name: net.ipv4.tcp_congestion_control # Set TCP congestion control algorithm to BBR
    value: bbr
  - name: fs.file-max # Set the maximum number of file handles
    value: 1000000
  - name: net.core.rmem_max # Set the maximum receive socket buffer size
    value: 104857600
...
```

## Example Playbook

## Playbook

**install.yaml** file

```yaml
- name: install-haproxy
  hosts: all
  become: true
  become_user: root
  roles:
    - ansible-role-haproxy
```

### Inventory

**inventory.ini** file
```ini
[haproxy]
10.100.177.5 vrrp_state=MASTER vrrp_priority=101 vrrp_ip_interface=ens192
10.100.177.6 vrrp_state=BACKUP vrrp_priority=100 vrrp_ip_interface=ens192

[all:vars]
ansible_user=root
ansible_connection=ssh
ansible_ssh_port=22
```

### Installation

```
ansible-playbook -i inventory.ini install.yaml
```
