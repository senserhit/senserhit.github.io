# iptables

echo 1 > /proc/sys/net/ipv4/ip_forward

分析包的走向.

modprobe nf_log_ipv4

echo nf_log_ipv4 >/proc/sys/net/netfilter/nf_log/2

然后查看/var/log/syslog

设置TRACE

iptables -t raw -I PREROUTING  -s 106.12.36.166 -j TRACE

iptables -t raw -I PREROUTING  -p icmp -j TRACE

iptables -t raw -A OUTPUT -s 172.16.8.0/24 -j TRACE

iptables -t raw -A OUTPUT -p icmp -j TRACE

iptables -t raw -I PREROUTING  -s 172.16.8.0/24 -j TRACE

iptables -t raw -I OUTPUT -s 172.16.8.0/24 -j TRACE

Ssh 不通卡在 expecting SSH2_MSG_KEX_ECDH_REPLY  是一个buG sudo ip li set mtu 1200 dev wlan0

接下来又卡在 Next authentication method: gssapi-with-mic  修改/etc/ssh_config/ 把GSSAPIAuthentication 改为no

配置一个NAT就可以

- A POSTROUTING -s 172.16.8.9/32 -o wlp3s0 -j SNAT --to-source 172.16.8.10

然后再配置一个路由, 路由是由内核做出的.

iptables -t nat

192.168.0.4

iptables -t raw -I PREROUTING  -p udp -dport 4500 or -dport 500  -j TRACE

iptables -t nat -A POSTROUTING -s 172.16.8.11/32 -o eth0 -j SNAT --to-source 172.16.8.60

iptables -t nat -A PREROUTING -i eth0 -s 172.16.8.11/32  -d 172.16.8.10 -j REDIRECT --to-destination 172.16.8.10

iptables -t nat -A PREROUTING -i eth0 -s 172.16.8.11/32  -d 172.16.8.10 -j DNAT --to-destination 172.16.8.10

只要做一个转发就可以了.

iptables -A FORWARD -d 172.16.9.0/24 -j ACCEPT