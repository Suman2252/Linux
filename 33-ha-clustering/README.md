# 🏢 Chapter 33: High Availability & Clustering

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-33%20of%2034-blue?style=for-the-badge" alt="Chapter 33">
</p>

---

## 📑 Table of Contents

- [HA Concepts](#ha-concepts)
- [Pacemaker & Corosync](#pacemaker--corosync)
- [HAProxy — Load Balancing](#haproxy--load-balancing)
- [Keepalived — Floating IP](#keepalived--floating-ip)
- [Database HA](#database-ha)
- [GlusterFS — Distributed Storage](#glusterfs--distributed-storage)
- [Practice Exercises](#-practice-exercises)

---

## HA Concepts

```
Active-Passive (Failover):
  ┌──────────┐         ┌──────────┐
  │  Node A  │────────▶│  Node B  │
  │ (active) │  sync   │(standby) │
  └──────────┘         └──────────┘
        ↑ VIP: 10.0.0.100
     Clients

Active-Active (Load Balanced):
                ┌──────────┐
  Clients ────▶ │ HAProxy  │
                └─────┬────┘
              ┌───────┼───────┐
        ┌─────▼──┐ ┌──▼────┐ ┌▼───────┐
        │ Node A │ │Node B │ │ Node C │
        └────────┘ └───────┘ └────────┘
```

| Term | Description |
|------|-------------|
| **Uptime** | Percentage of time system is available |
| **RTO** | Recovery Time Objective — max downtime allowed |
| **RPO** | Recovery Point Objective — max data loss allowed |
| **SLA** | Service Level Agreement — guaranteed uptime |
| **Five 9s** | 99.999% uptime = ~5 minutes downtime/year |
| **Split-brain** | Both nodes think they're primary |
| **STONITH** | Shoot The Other Node In The Head (fencing) |
| **Quorum** | Majority agreement (prevents split-brain) |

---

## Pacemaker & Corosync

The standard Linux HA stack.

```bash
# Install (on all nodes)
sudo apt install pacemaker corosync pcs

# Set password for cluster user
sudo passwd hacluster

# Start PCS daemon
sudo systemctl enable --now pcsd

# Authenticate nodes (from one node)
sudo pcs host auth node1 node2 node3

# Create cluster
sudo pcs cluster setup mycluster node1 node2 node3

# Start cluster
sudo pcs cluster start --all
sudo pcs cluster enable --all

# Status
sudo pcs status
sudo pcs cluster status
sudo crm_mon -1                       # One-shot status
```

### Configure Resources

```bash
# Add a floating IP
sudo pcs resource create cluster_vip ocf:heartbeat:IPaddr2 \
  ip=10.0.0.100 cidr_netmask=24 op monitor interval=30s

# Add Nginx resource
sudo pcs resource create webserver ocf:heartbeat:nginx \
  configfile=/etc/nginx/nginx.conf op monitor interval=10s

# Colocation (keep resources together)
sudo pcs constraint colocation add webserver with cluster_vip INFINITY

# Order (start VIP before web server)
sudo pcs constraint order cluster_vip then webserver

# Prefer specific node
sudo pcs constraint location webserver prefers node1=100

# STONITH/Fencing (required for production!)
sudo pcs stonith create fence_node1 fence_ipmilan \
  ipaddr=192.168.1.200 login=admin passwd=secret \
  pcmk_host_list=node1

# Disable STONITH (testing only!)
sudo pcs property set stonith-enabled=false
```

---

## HAProxy — Load Balancing

```bash
sudo apt install haproxy
sudo vim /etc/haproxy/haproxy.cfg
```

```
global
    log /dev/log local0
    maxconn 4096
    user haproxy
    group haproxy
    daemon
    stats socket /var/run/haproxy.sock mode 660 level admin

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms
    retries 3

frontend http_front
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/site.pem
    redirect scheme https code 301 if !{ ssl_fc }
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /health
    server web1 10.0.0.11:8080 check
    server web2 10.0.0.12:8080 check
    server web3 10.0.0.13:8080 check backup

listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats auth admin:password
    stats refresh 10s
```

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg   # Validate
sudo systemctl enable --now haproxy
# Stats dashboard: http://server:8404/stats
```

### Balance Algorithms

| Algorithm | Description |
|-----------|-------------|
| `roundrobin` | Rotate evenly |
| `leastconn` | Fewest active connections |
| `source` | Same client → same server (sticky) |
| `uri` | Same URI → same server (cache friendly) |

---

## Keepalived — Floating IP

```bash
sudo apt install keepalived

# /etc/keepalived/keepalived.conf (Primary)
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass mysecret
    }
    virtual_ipaddress {
        10.0.0.100/24
    }
    track_script {
        check_haproxy
    }
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight 2
}

# On Backup node: change state to BACKUP, priority to 99
```

---

## Database HA

### PostgreSQL Streaming Replication

```bash
# Primary: /etc/postgresql/16/main/postgresql.conf
wal_level = replica
max_wal_senders = 3
synchronous_standby_names = 'standby1'

# Replica setup
pg_basebackup -h primary -D /var/lib/postgresql/16/main -U replicator -R
# -R creates standby.signal and connection info
```

### Galera Cluster (MySQL/MariaDB)

```bash
# Multi-master synchronous replication
sudo apt install mariadb-server galera-4

# /etc/mysql/mariadb.conf.d/60-galera.cnf
[mysqld]
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_on=ON
wsrep_cluster_name="my_cluster"
wsrep_cluster_address="gcomm://node1,node2,node3"
wsrep_node_address="10.0.0.11"
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Bootstrap first node
sudo galera_new_cluster
# Start remaining nodes normally
```

---

## GlusterFS — Distributed Storage

```bash
sudo apt install glusterfs-server
sudo systemctl enable --now glusterd

# From one node, probe peers
sudo gluster peer probe node2
sudo gluster peer probe node3
sudo gluster peer status

# Create replicated volume
sudo gluster volume create gvol0 replica 3 \
  node1:/data/brick node2:/data/brick node3:/data/brick

# Start volume
sudo gluster volume start gvol0
sudo gluster volume info gvol0

# Mount on clients
sudo mount -t glusterfs node1:/gvol0 /mnt/shared
```

---

## 🏋️ Practice Exercises

1. **(VMs)** Set up a 2-node Pacemaker cluster with a floating IP
2. **HAProxy**: Configure load balancing across 2 web servers
3. **Keepalived**: Implement floating IP failover
4. **Stats**: Access the HAProxy stats dashboard
5. **Failover**: Test failover by stopping a service on the active node
6. **PostgreSQL**: Set up primary-replica streaming replication
7. **GlusterFS**: Create a replicated volume across 2 nodes

---

<p align="center">
  <a href="../32-embedded-linux/README.md">← Previous: Embedded Linux</a> · <a href="../README.md">🏠 Home</a> · <a href="../34-disaster-recovery/README.md">Next: Disaster Recovery →</a>
</p>
