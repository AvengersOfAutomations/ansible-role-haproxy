<p align="center">
  <a href="https://github.com/SepehrImanian/terraform-provider-haproxy">
    <img src="./assets/haproxy.png" alt="minio-provider-terraform" width="200">
  </a>
  <h1 align="center" style="font-weight: bold">Ansible Role For Setup (Haproxy, Keepalived, Data Plane API)</h1>
</p>

## Roles Variables

### Install and Configure Option
Choose which tools install on your host

```yaml
docker: true
haproxy: true
keepalived: true
sysctl_tuning: true
haproxy_exporter: true
keepalived_exporter: true
data_plane_api: true
ufw: true
```

### Haproxy Config
Haproxy could install with docker container or linux service

```yaml
haproxy_install_type: docker-compose # service / docker-compose

# Golabal configs
haproxy_global:
  home_dir: /opt/haproxy
  version: 2.6
  maxconn: 300000

# Dashboard config
haproxy_dashboard:
  user: admin
  password: admin

# Haproxy timeout configs
haproxy_timeout:
  connect: 30s
  client: 60s
  server: 60s

# Haproxy monitoring configs
haproxy_monitoring_config:
  port: 9090
  uri: /stats
  refresh: 10s

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

### Data Plane API Config
```yaml
dataplaneapi:
  user: admin
  password: adminpwd
  api:
    command: /usr/bin/dataplaneapi
    host: 0.0.0.0
    port: 5555
    haproxy_bin: /usr/sbin/haproxy
    config_file: /usr/local/etc/haproxy/haproxy.cfg
    reload_cmd: "kill -SIGUSR2 1"
    restart_cmd: "kill -SIGUSR2 1"
    reload_delay: 5
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

### UFW Config
Dynamically Add/Remove Rules and Ports
```yaml
ufw_state: enabled # enabled/disabled/reloaded

ufw_rules:
  - rule: allow
    src: 172.16.0.0/24
  - rule: allow
    src: 172.20.0.0/24
  
ufw_ports:
  - rule: allow
    port: 80
    proto: tcp
  - rule: allow
    port: 443
    proto: tcp
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
