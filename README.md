### Домашнее задание №24 (VLAN)
#### 1. Настройка бонда между inetRouter и centralRouter. Настройка производится при помощи модуля nmcli в Ansible.
Проверим работу бондов с отключением линка. Роутер inetRouter:
```console
[root@inetRouter vagrant]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:4f:37:f7
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:18:3d:bd
Slave queue ID: 0
```
```console
[root@inetRouter vagrant]# ip -c a | grep bond0
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
5: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.255.1/30 brd 192.168.255.3 scope global noprefixroute bond0
```
Роутер centralRouter:
```console
[root@centralRouter vagrant]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:b3:6d:79
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:d5:cd:95
Slave queue ID: 0
```
```console
[root@centralRouter vagrant]# ip -c a | grep bond0
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
10: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.255.2/30 brd 192.168.255.3 scope global noprefixroute bond0
```
Отключим интерфейс eth1 на centralRouter, режим аггригирования выбран __round robin__:
```console
[root@centralRouter vagrant]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
^C
--- 192.168.255.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2007ms
```
Поднимем интерфейс eth1:
```console[root@centralRouter vagrant]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=0.702 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.578 ms
64 bytes from 192.168.255.1: icmp_seq=3 ttl=64 time=0.773 ms
64 bytes from 192.168.255.1: icmp_seq=4 ttl=64 time=0.862 ms
^C
--- 192.168.255.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms

```
___
#### 2. Разводим vlan server1/client1 и server2/client2:

