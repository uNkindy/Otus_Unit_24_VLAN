### Домашнее задание №24 (VLAN)
#### 1. Написан [Vagrantfile](https://github.com/uNkindy/Otus_Unit_24_VLAN/blob/main/Vagrantfile) в который добавлены 4 виртуальные машины: testServer1, testClient1, testServer2, testClient2. 
#### 2. Настройка бонда между inetRouter и centralRouter. Настройка производится при помощи модуля nmcli в Ansible.
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
При поднятии интерфейса eth1 пинги продолжили идти по bond-интерфейсу.
___
#### 3. Разводим vlan testServer1/testClient1 и testServer2/testClient2:

настроим VLAN между testServer1(2) и testClient1(2) при помощи модуля для Ansible __nmcli__:
- настроим VLAN 100 между testServer1 eth1 и testClient1 eth1 и eth3 на centralRouter и проверим:
```console
[root@testServer1 vagrant]# ip -c a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86050sec preferred_lft 86050sec
    inet6 fe80::5054:ff:fe4d:77d3/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:a3:5b:c2 brd ff:ff:ff:ff:ff:ff
4: eth1.100@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:a3:5b:c2 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.254/24 brd 10.10.10.255 scope global noprefixroute eth1.100
       valid_lft forever preferred_lft forever
    inet6 fe80::2c8f:6500:e19a:96a2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
```console
[root@testServer1 vagrant]# tcpdump -vvv -i eth1 -e icmp
tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
11:17:13.300780 08:00:27:74:d5:98 (oui Unknown) > 08:00:27:a3:5b:c2 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 54984, offset 0, flags [DF], proto ICMP (1), length 84)
    10.10.10.1 > testServer1: ICMP echo request, id 3075, seq 99, length 64
11:17:13.300858 08:00:27:a3:5b:c2 (oui Unknown) > 08:00:27:74:d5:98 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 19157, offset 0, flags [none], proto ICMP (1), length 84)
```
- тоже самое сделаем для testServer2 и testClient2 и проверим:
```console
[root@testClient2 vagrant]# ip -c a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85359sec preferred_lft 85359sec
    inet6 fe80::5054:ff:fe4d:77d3/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:16:24:df brd ff:ff:ff:ff:ff:ff
    inet6 fe80::a00:27ff:fe16:24df/64 scope link 
       valid_lft forever preferred_lft forever
4: eth1.101@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:16:24:df brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global noprefixroute eth1.101
       valid_lft forever preferred_lft forever
    inet6 fe80::3f94:c569:4512:3f7d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
```console
[root@testServer2 vagrant]# tcpdump -vvv -i eth1 icmp -e
tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
11:27:17.183509 08:00:27:16:24:df (oui Unknown) > 08:00:27:87:50:54 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 101, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 50196, offset 0, flags [DF], proto ICMP (1), length 84)
    10.10.10.1 > testServer2: ICMP echo request, id 3176, seq 54, length 64
11:27:17.183557 08:00:27:87:50:54 (oui Unknown) > 08:00:27:16:24:df (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 101, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 33171, offset 0, flags [none], proto ICMP (1), length 84)
    testServer2 > 10.10.10.1: ICMP echo reply, id 3176, seq 54, length 64
```
По логам видно, что между testServer1 и testClient1 ходят пинги по 100 VLAN, между testServer2 и testClient2 ходят пинги по 101 VLAN.