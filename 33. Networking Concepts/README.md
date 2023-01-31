#### Networking

Switching Concepts 

Check interface on a host
ip link

Check interface configuration on an interface
 ip addr

Configure IP Address
ip addr add 192.168.1.10/24 dev eth0

To view kernel's routing table
route

Configure Default Gateway
ip route add 192.168.2.0/24 via 192.168.1.1

ip route add 0.0.0.0/0 via 192.168.1.1 

#### Routing

Setting up linux host as a router

Host forwarding packets between interfaces

cat /proc/sys/net/ipv4/ip_forward
1

To persist changes across reboots

/etc/sysctl.conff

net.ipv4.ip_forward = 1

#### DNS

/etc/hosts
192.168.1.11 db

DNS Resolution configuration

/etc/resolv.conf
nameserver 192.168.1.100
nameserver 8.8.8.8
serach xxxx.com

To configure the public DNS on the nameserver:

add an entry:

forward all to 8.8.8.8

Order between hosts file and DNS server:
1. hosts file
2. DNS server

Changing this order: 
/etc/nsswitch.com

hosts:   files dns

To test DNS use nslookup or dig. It only queries nameservers and not hosts file

nslookup www.google.com
dig www.google.com

Examples of DNS server solutions
- CoreDNS
- PowerDNS
- BIND9
- Amazon Route53
- Istio


#### Network namespaces

Create new network namespace

ip netns add <network_ns_name>

List namespaces

ip netns

View network interfaces in a specified network namespace

ip netns exec <network_ns_name> ip link

Establishing connectivity between namespaces using virtual connectivity

ip link add <veth-name-01> type veth peer name <veth-name-02>

Assign each virtual interface to their respective network namespace

ip link set <network_ns_name-01> netns <veth-name-01>
ip link set <network_ns_name-02> netns <veth-name-02>

Assign the respective interfaces an ip address

ip -n <network_ns_name-01> addr add 192.168.15.1/24 dev <veth-name-01>
ip -n <network_ns_name-01> link set <veth-name-01> up

ip -n <network_ns_name-02> addr add 192.168.15.2/24 dev <veth-name-02>
ip -n <network_ns_name-02> link set <veth-name-02> up

Ping between the network namespaces

ip netns exec <network_ns_name-01> ping 192.168.15.2

Check the ARP table of <network_ns_name-01>

ip netns exec <network_ns_name-01> arp

Linux Bridge 

Creating another interface like eth0, named v-net-0

ip link add v-net-0 type bridge

to list the links

ip link

turn the link up

ip link set dev v-net-0 up

To the virtual namespaces, the bridge is like a virtual switch which they can connect to. 

Connecting the namespaces to the new virtual switch

Delete previous links configured on the virtual interfaces

ip -n <network_ns_name-01> link del <veth-name-01>

Create new connection of the namespaces to the bridge

ip link add <veth-name-01> type veth peer name <veth-name-01-bridge>
ip link add <veth-name-02> type veth peer name <veth-name-02-bridge>

Connect the interfaces to the namespaces

ip link set <veth-name-01> netns <network_ns_name-01>
ip link set <veth-name-01-bridge> master v-net-0

ip link set <veth-name-02> netns <network_ns_name-02>
ip link set <veth-name-02-bridge> master v-net-0

Setting IP Addresses for the links

ip -n <network_ns_name-01> addr add 192.168.15.1/24 dev <veth-name-01>
ip -n <network_ns_name-02> addr add 192.168.15.2/24 dev <veth-name-02>
ip -n <network_ns_name-01> link set <veth-name-01> up
ip -n <network_ns_name-02> link set <veth-name-02> up

Establish connectivity between the host and the namespaces, configure the host ip:

ip addr 192.168.15.5/24 dev v-net-0

Configure bridge to reach the LAN network. Add an entry in the routing table

ip netns exec <network_ns_name-02> ip route add 192.168.1.0/24 via 192.168.15.5

Enable NAT on the host

iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE

To reach the internet:

ip netns exec <network_ns_name-02> ip route add default via 192.168.15.5

Add port forwading rule to reach the namespaces

iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT

#### Docker Networking

Not attaching the container to any network

docker run --netwrok none nginx

Attaching container to the host network

docker run --netwrok host nginx

Bridge Network where both docker host and container are attached to

docker run nginx

The bridge network is created by default on the docker host. To list:

docker network ls

On the host the network is named *docker 0* 

ip link

docker achieve this by running:

ip link add docker0 type bridge

To see the ip address cidr assignet to the brifge

ip addr

To list the namespace

ip netns

docker inspect <docker-id>

How docker does port forwrarding from host port 8080 to container port: 80

docker run -p 8080:80 nginx

iptables \
-t nat \
-A DOCKER \
-j DNAT \
--dport 8080\
--to-destination <container_ip>:80


List the rules in iptables

iptables -nvL -t nat

Summary of the steps - Networking in containers:

- Create network namespace
- Create bridge network /interface
- Create vETH pairs (pipe, virtual cable)
- Attach vEth to namespace
- Attach other vEth end to bridge
- Assign IP Addresses
- Bring the interface up
- Enable NAT - IP Masquerade


Container Network Interface (CNI)
- A set of standards that define how programs should be devewloped to solve network challenges in a container runtime environment
- CNI define how plugins should be developed and how container runtimes should invoke them
- CNI Supported plugins include: BRIDGE, VLAN, IPVLAN, MACVLAN, WINDOWS, DHCP, host-local
- Third party plugins available include: weaveworks, flannel, cilium, NSX etc.
- Docker CRI does not implement CNI, it has its own set of standards called CNM (Container Network Model)