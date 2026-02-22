# Networking Cheatsheet

## IP & Interfaces
| Command | Description |
|---------|-------------|
| `ip addr show` | Show all interfaces |
| `ip addr add 10.0.0.1/24 dev eth0` | Add IP |
| `ip addr del 10.0.0.1/24 dev eth0` | Remove IP |
| `ip link set eth0 up/down` | Enable/disable |
| `ip link show` | Link status |

## Routing
| Command | Description |
|---------|-------------|
| `ip route show` | Show routes |
| `ip route add 10.0.0.0/24 via 192.168.1.1` | Add route |
| `ip route del 10.0.0.0/24` | Delete route |
| `ip route add default via 192.168.1.1` | Default gateway |

## DNS
| Command | Description |
|---------|-------------|
| `dig example.com` | DNS lookup |
| `dig +short example.com` | Quick lookup |
| `nslookup example.com` | DNS query |
| `host example.com` | Simple lookup |
| `cat /etc/resolv.conf` | DNS config |
| `resolvectl status` | systemd-resolved |

## Diagnostics
| Command | Description |
|---------|-------------|
| `ping -c 5 host` | Connectivity test |
| `traceroute host` | Trace route |
| `mtr host` | Combined ping+traceroute |
| `ss -tuln` | Listening ports |
| `ss -tp` | Active connections |
| `netstat -tlnp` | Legacy port listing |
| `curl -I https://example.com` | HTTP headers |
| `wget -q -O- url` | Download to stdout |

## Firewall (ufw)
| Command | Description |
|---------|-------------|
| `sudo ufw status` | Show status |
| `sudo ufw enable/disable` | Toggle firewall |
| `sudo ufw allow 22/tcp` | Allow SSH |
| `sudo ufw deny 23/tcp` | Deny telnet |
| `sudo ufw allow from 10.0.0.0/8` | Allow subnet |
| `sudo ufw delete allow 80/tcp` | Remove rule |

## Firewall (iptables)
| Command | Description |
|---------|-------------|
| `iptables -L -v -n` | List rules |
| `iptables -A INPUT -p tcp --dport 22 -j ACCEPT` | Allow SSH |
| `iptables -A INPUT -j DROP` | Drop all other |
| `iptables -F` | Flush all rules |
| `netfilter-persistent save` | Save rules |

## SSH
| Command | Description |
|---------|-------------|
| `ssh user@host` | Connect |
| `ssh -p 2222 user@host` | Custom port |
| `ssh-keygen -t ed25519` | Generate key |
| `ssh-copy-id user@host` | Copy key |
| `scp file user@host:~/` | Copy file |
| `rsync -avz src/ user@host:dst/` | Sync files |

## Network Analysis
| Command | Description |
|---------|-------------|
| `sudo tcpdump -i eth0` | Capture packets |
| `sudo tcpdump -i eth0 port 80` | Filter port |
| `nmap -sV host` | Service scan |
| `nmap -sn 192.168.1.0/24` | Ping sweep |
| `iftop` | Bandwidth per connection |
| `nload` | Bandwidth per interface |
