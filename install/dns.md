单master k8s, dns配置

# cat /etc/dnsmasq.d/k8s.conf 
port=53
domain-needed
bogus-priv
no-negcache
cache-size=10000
bind-interfaces
expand-hosts
listen-address=::1,127.0.0.1,192.168.146.90
dns-forward-max=15000

address=/registry.crc.test/192.168.146.90
address=/.apps.test.crc.test/192.168.146.90
address=/whoami.crc.test/192.168.146.90
address=/log-collection.crc.test/192.168.146.90
address=/0509-test.crc.test/192.168.146.90