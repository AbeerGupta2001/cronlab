
## Network Types

#### Common Terminology

|**Network Type**|**Definition**|
|---|---|
|Wide Area Network (WAN)|Internet|
|Local Area Network (LAN)|Internal Networks (Ex: Home or Office)|
|Wireless Local Area Network (WLAN)|Internal Networks accessible over Wi-Fi|
|Virtual Private Network (VPN)|Connects multiple network sites to one `LAN`|

## WAN

The WAN (Wide Area Network) is commonly referred to as `The Internet`. When dealing with networking equipment, we'll often have a WAN Address and LAN Address. The WAN one is the address that is generally accessed by the Internet. That being said, it is not exclusive to the Internet; a WAN is just a large number of LANs joined together. Many large companies or government agencies will have an "Internal WAN" (also called Intranet, Airgap Network, etc.). Generally speaking, the primary way we identify if the network is a WAN is to use a WAN Specific routing protocol such as BGP and if the IP Schema in use is not within RFC 1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16).

## LAN / WLAN

LANs (Local Area Network) and WLANs (Wireless Local Area Network) will typically assign IP Addresses designated for local use (RFC 1918, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16). In some cases, like some colleges or hotels, you may be assigned a routable (internet) IP Address from joining their LAN, but that is much less common. There's nothing different between a LAN or WLAN, other than WLAN's introduce the ability to transmit data without cables. It is mainly a security designation.

## VPN

There are three main types `Virtual Private Networks` (`VPN`), but all three have the same goal of making the user feel as if they were plugged into a different network.

#### Site-To-Site VPN

Both the client and server are Network Devices, typically either `Routers` or `Firewalls`, and share entire network ranges. This is most commonly used to join company networks together over the Internet, allowing multiple locations to communicate over the Internet as if they were local.

#### Remote Access VPN

This involves the client's computer creating a virtual interface that behaves as if it is on a client's network. Hack The Box utilizes `OpenVPN`, which makes a TUN Adapter letting us access the labs. When analyzing these VPNs, an important piece to consider is the routing table that is created when joining the VPN. If the VPN only creates routes for specific networks (ex: 10.10.10.0/24), this is called a `Split-Tunnel VPN`, meaning the Internet connection is not going out of the VPN. This is great for Hack The Box because it provides access to the Lab without the privacy concern of monitoring your internet connection. However, for a company, `split-tunnel` VPN's are typically not ideal because if the machine is infected with malware, network-based detection methods will most likely not work as that traffic goes out the Internet.

### Split-Tunnel VPN (what HTB uses)

**Example:** You connect to Hack The Box's VPN. Your routing table only adds a rule for `10.10.10.0/24` (the HTB lab network).

That means:

- Traffic to `10.10.10.5` (a lab machine) → goes through the VPN tunnel
- Traffic to `google.com`, Netflix, YouTube, etc. → goes through your normal home internet, **not** the VPN

**Why HTB likes this:** They don't want to see or slow down your regular internet traffic — they only care about lab traffic. It's efficient and respects your privacy.

### Full-Tunnel VPN (what companies usually use)

**Example:** You work at "Acme Corp" and connect to their VPN. The routing table adds a rule like `0.0.0.0/0` (meaning "literally everything").

That means:

- Traffic to internal file servers → goes through the VPN
- Traffic to `google.com`, YouTube, random websites → **also** goes through the VPN, out to the internet via Acme's network, then back to you. 
#### SSL VPN

This is essentially a VPN that is done within our web browser and is becoming increasingly common as web browsers are becoming capable of doing anything. Typically these will stream applications or entire desktop sessions to your web browser. A great example of this would be the HackTheBox Pwnbox.


We can divide the entire network topology area into three areas:

#### 1. Connections

| `Wired connections`  | `Wireless connections` |
| -------------------- | ---------------------- |
| Coaxial cabling      | Wi-Fi                  |
| Glass fiber cabling  | Cellular               |
| Twisted-pair cabling | Satellite              |
| and others           | and others             |
#### 2. Nodes - Network Interface Controller (NICs)

|   |   |   |   |
|---|---|---|---|
|Repeaters|Hubs|Bridges|Switches|
|Router/Modem|Gateways|Firewalls||

Network nodes are the `transmission medium's connection points` to transmitters and receivers of electrical, optical, or radio signals in the medium. A node may be connected to a computer, but certain types may have only one microcontroller on a node or may have no programmable device at all.

Network topologies are divided into the following eight basic types:

|   |   |
|---|---|
|Point-to-Point|Bus|
|Star|Ring|
|Mesh|Tree|
|Hybrid|Daisy Chain|

## Point-to-Point

The simplest network topology with a dedicated connection between two hosts is the `point-to-point` topology. In this topology, a direct and straightforward physical link exists only between `two hosts`. These two devices can use these connections for mutual communication.

`Point-to-point` topologies are the basic model of traditional telephony and must not be confused with `P2P` (`Peer-to-Peer` architecture).
#### Point-To-Point Topology

![image](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_p2p.png)

## Bus

All hosts are connected via a transmission medium in the bus topology. Every host has access to the transmission medium and the signals that are transmitted over it. There is no central network component that controls the processes on it. The transmission medium for this can be, for example, a `coaxial cable`.

Since the medium is shared with all the others, only `one host can send`, and all the others can only receive and evaluate the data and see whether it is intended for itself.

#### Bus Topology

![Diagram showing a connection between Host A and Host B.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_bus.png)

## Star

The star topology is a network component that maintains a connection to all hosts. Each host is connected to the `central network component` via a separate link. This is usually a router, a hub, or a switch. These handle the `forwarding function` for the data packets. To do this, the data packets are received and forwarded to the destination. The data traffic on the central network component can be very high since all data and connections go through it.

#### Star Topology

![Diagram showing connections between Hosts A, B, C, D, E, and F.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_star.png)

## Ring

The `physical` ring topology is such that each host or node is connected to the ring with two cables:

- One for the `incoming` signals and
- another for the `outgoing` ones.
 
A `logical` ring topology is based on a physical star topology, where a distributor at the node simulates the ring by forwarding from one port to the next.

The information is transmitted in a predetermined transmission direction. Typically, the transmission medium is accessed sequentially from station to station using a retrieval system from the central station or a `token`. A token is a bit pattern that continually passes through a ring network in one direction, which works according to the `claim token process`.

#### Ring Topology

![Diagram showing a router connecting Hosts A, B, C, and D.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_ring.png)

## Mesh

Many nodes decide about the connections on a `physical` level and the routing on a `logical` level in meshed networks. Therefore, meshed structures have no fixed topology. There are two basic structures from the basic concept: the `fully meshed` and the `partially meshed` structure.

Each host is connected to every other host in the network in a `fully meshed structure`. This means that the hosts are meshed with each other. This technique is primarily used in `WAN` or `MAN` to ensure high reliability and bandwidth

In the `partially meshed structure`, the endpoints are connected by only one connection. In this type of network topology, specific nodes are connected to exactly one other node, and some other nodes are connected to two or more other nodes with a point-to-point connection.
#### Mesh Topology

![Network diagram showing Hosts A, B, C, and D connected through Routers A, B, C, and D.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_mesh.png)

## Tree

The tree topology is an extended star topology that more extensive local networks have in this structure. This is especially useful when several topologies are combined. This topology is often used, for example, in larger company buildings.

There are both logical tree structures according to the `spanning tree` and physical ones. Modular modern networks, based on structured cabling with a hub hierarchy, also have a tree structure. Tree topologies are also used for `broadband networks` and `city networks` (`MAN`).

## Hybrid

Hybrid networks combine two or more topologies so that the resulting network does not present any standard topologies. For example, a tree network can represent a hybrid topology in which star networks are connected via interconnected bus networks. However, a tree network that is linked to another tree network is still topologically a tree network. A hybrid topology is always created when `two different` basic network topologies are interconnected.

#### Hybrid Topology

![Network diagram showing Hosts A, B, C, D, E, and F connected through Routers A, B, and C.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_hybrid.png)

## Daisy Chain

In the daisy chain topology, multiple hosts are connected by placing a cable from one node to another.

Since this creates a chain of connections, it is also known as a daisy-chain configuration in which multiple hardware components are connected in a series. This type of networking is often found in automation technology (`CAN`).
#### Daisy Chain Topology

![Network diagram showing Hosts A, B, D, and E connected to Host C.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/topo_daisy-chain.png)

## Proxies

A proxy is when a device or service sits in the middle of a connection and acts as a mediator. The `mediator` is the critical piece of information because it means the device in the middle must be able to inspect the contents of the traffic. Without the ability to be a `mediator`, the device is technically a `gateway`, not a proxy.

If you have trouble remembering this, proxies will almost always operate at Layer 7 of the OSI Model. There are many types of proxy services, but the key ones are:

- `Dedicated Proxy` / `Forward Proxy`
- `Reverse Proxy`
- `Transparent Proxy`

For example, in a corporate network, sensitive computers may not have direct access to the Internet. To access a website, they must go through a proxy (or web filter). This can be an incredibly powerful line of defense against malware, as not only does it need to bypass the web filter (easy), but it would also need to be `proxy aware` or use a non-traditional C2 (a way for malware to receive tasking information). If the organization only utilizes `FireFox`, the likelihood of getting proxy-aware malware is improbable.

#### Forward Proxy

![Diagram showing Host A sending an HTTP request through a Forward Proxy to the Internet, reaching a Webserver. Host B is also connected.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/34/redesigned/forward_proxy.png)


## (Non-) Transparent Proxy

All these proxy services act either `transparently` or `non-transparently`.

With a `transparent proxy`, the client doesn't know about its existence. The transparent proxy intercepts the client's communication requests to the Internet and acts as a substitute instance. To the outside, the transparent proxy, like the non-transparent proxy, acts as a communication partner.

If it is a `non-transparent proxy`, we must be informed about its existence. For this purpose, we and the software we want to use are given a special proxy configuration that ensures that traffic to the Internet is first addressed to the proxy. If this configuration does not exist, we cannot communicate via the proxy. However, since the proxy usually provides the only communication path to other networks, communication to the Internet is generally cut off without a corresponding proxy configuration.