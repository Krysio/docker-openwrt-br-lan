# Docker + OpenWrt. Services on the br-lan network

The solution uses a network type `macvlan` and the `veth` kernel module to get around the problem of communication with the host.

### 1. Install
```bash
opkg install luci-app-dockerman
opkg install kmod-macvlan
opkg install kmod-veth
```

### 2. Configuration of network

Edit `/etc/rc.local`, add the following lines:
```
ip link add veth1 type veth
ip link set veth0 up
ip link set veth1 up
```
This will create a virtual ethernet cable.

Edit `/etc/config/network`, add `list ports 'veth0'` to the "br-lan" section:
```
config device
    option name 'br-lan'
    ...
    list ports 'veth0'
```
The first end of the virtual cable is connected to the lan network.

Create a docker network:
```
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=veth1 br-lan
```
The other end of the virtual cable is connected to the docker's network.

### 3. Add services to your network
```bash
docker run --rm -itd --network br-lan --ip 192.168.0.2 --name busybox busybox ash
```

```yml
services:
    pihole:
        container_name: pihole
        image: pihole/pihole:latest
        environment:
            ServerIP: '192.168.1.2'
            ...
        ...
        networks:
            br-lan:
                ipv4_address: '192.168.1.2'

networks:
    br-lan:
        external: true
```