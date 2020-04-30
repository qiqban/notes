## CentOS防火墙放行端口

```bash
firewall-cmd --remove-port=8765/tcp --permanent
firewall-cmd --reload
firewall-cmd --state
```