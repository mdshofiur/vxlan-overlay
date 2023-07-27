# Creating a Simple VXLAN Overlay Network using Linux Network Namespaces and Bridges


#### In this blog post, we will explore how to set up a simple Virtual Extensible LAN (VXLAN) overlay network using Linux network namespaces and bridges. VXLAN is a popular technology used to create virtualized, layer 2 network segments over an existing IP network. We will create two separate hosts, Host-1 and Host-2, and connect them via a VXLAN tunnel to establish communication between them.

#### Prerequisites:

* Two Linux hosts with a recent version of the Linux kernel that supports VXLAN and network namespaces.
* Superuser (root) access on both hosts

## Step 1: Setting up Host-1

* Create a new network namespace named “red” on Host-1:


```
sudo ip netns add red
```

* Create a virtual Ethernet pair (veth) interface, “red-in” and “red-out,” and move “red-in” into the “red” namespace:

```
sudo ip link add red-in type veth peer name red-out
sudo ip link set red-in netns red
```


* Configure the IP address and bring up the “red-in” interface in the “red” namespace:

```
sudo ip netns exec red ip addr add 192.168.60.13/16 dev red-in
sudo ip netns exec red ip link set red-in up
```

* Create a Linux bridge named “bridge-main” on the host and assign an IP address to it:

```
sudo ip link add bridge-main type bridge
sudo ip addr add 192.168.60.14/16 dev bridge-main
sudo ip link set red-out master bridge-main
sudo ip link set red-out up
sudo ip link set bridge-main up
```


* Add a default route in the “red” namespace through the bridge:


```
sudo ip netns exec red ip route add default via 192.168.60.14

```

* Create a VXLAN tunnel interface, “vxlan-red,” on Host-1 to connect to Host-2:

```
sudo ip link add vxlan-red type vxlan id 100 local 192.168.60.11 remote 192.168.60.12 dev eth1

```

* Attach the “vxlan-red” interface to the “bridge-main” bridge:


```
sudo ip link set vxlan-red master bridge-main
sudo ip link set vxlan-red up
```

## Step 2: Setting up Host-2


* Create a new network namespace named “blue” on Host-2:

```
sudo ip netns add blue

```


* Create a virtual Ethernet pair (veth) interface, “blue-in” and “blue-out,” and move “blue-in” into the “blue” namespace:

```
sudo ip link add blue-in type veth peer name blue-out
sudo ip link set blue-in netns blue
```
* Configure the IP address and bring up the “blue-in” interface in the “blue” namespace:

```
sudo ip netns exec blue ip addr add 192.168.60.15/16 dev blue-in
sudo ip netns exec blue ip link set blue-in up
``` 

* Create a Linux bridge named “bridge-main” on the host and assign an IP address to it (if not already done in Step 1):

```
sudo ip link add bridge-main type bridge
sudo ip addr add 192.168.60.16/16 dev bridge-main
sudo ip link set blue-out master bridge-main
sudo ip link set blue-out up
sudo ip link set bridge-main up
``` 

* Add a default route in the “blue” namespace through the bridge (if not already done in Step 1):

 ```
 sudo ip netns exec blue ip route add default via 192.168.60.16

 ```
 
 * Create a VXLAN tunnel interface, “vxlan-blue,” on Host-2 to connect to Host-1:

```
sudo ip link add vxlan-blue type vxlan id 100 local 192.168.60.12 remote 192.168.60.11 dev eth1
```

* Attach the “vxlan-blue” interface to the “bridge-main” bridge:


```
sudo ip link set vxlan-blue master bridge-main
sudo ip link set vxlan-blue up
```

## Testing the Connectivity:

#### At this point, both Host-1 and Host-2 are set up with their respective network namespaces, bridges, and VXLAN tunnels. To test the connectivity between the two hosts, you can open two terminal windows on each host and execute ping commands between the namespaces:


* On Host-1:

```
sudo ip netns exec red ping 192.168.60.15
```
* On Host-2:

```
sudo ip netns exec blue ping 192.168.60.13
```

#### In this blog post, we have learned how to create a simple VXLAN overlay network using Linux network namespaces and bridges. The setup allows Host-1 and Host-2 to communicate with each other through the VXLAN tunnel. VXLAN is a powerful technology for creating scalable, isolated network segments, and this example can be a foundation for more complex network topologies and use cases.

##### (Note: Please ensure that the interfaces’ IP addresses and other configuration settings match your specific network environment.)

