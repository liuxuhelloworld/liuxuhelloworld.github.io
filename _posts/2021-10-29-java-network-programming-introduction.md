The Java core API includes well-designed interfaces to most network features. Indeed, there is very little application layer network software you can write in C or C++ that you can't write more easily in Java. In brief, it is easy for Java applications to send and receive data across the Internet.

# Network Basics
A *network* is a collection of computers and other devices that can send data to and receive data from one another, more or less in real time.

Each machine on a network is called a *node*. Nodes that are fully functional computers are also called *hosts*.

Every node has an *address*, a sequence of bytes that uniquely identifies it. Addresses are assigned differently on different kinds of networks. Ethernet addresses are attached to the physical Ethernet hardware. Internet addresses are normally assigned to a computer by the organization that is responsible for it.

On some kinds of networks, nodes also have text names that help human beings identify them.

All modern computer networks are *packet-switched* networks: data traveling on the network is broken into chunks call *packets* and each packet is handled separately. Each packet contains information about who sent it and where it is going. The most important advantage of breaking data into individually addressed packets is that packets from many ongoing exchanges can travel on one wire, which makes it much cheaper to build a network: many computers can share the same wire without interfering. Another advantage of packets is that checksums can be used to detect whether a packet was damaged in transit.

A *protocol* is a precise set of rules defining how computers communicate: the format of addresses, how data is split into packets, and so on. There are many different protocols defining different aspects of network communication. Open, published protocol standards allow software and equipment from different vendors to communicate with one another. A web server doesn't care whether the client is a Unix workstation, an Android phone, or an iPad, because all clients speak the same HTTP protocol regardless of platform.

# The Layers of a Network
To hide most of the complexity from the application developer and end user, the different aspects of network communication are separated into multiple layers. Each layer represents a different level of abstraction between the physical hardware and the information being transmitted. In theory, each layer only talks to the layers immediately above and immediately below it. Separating the network into layers lets you modify or even replace the software in one layer without affecting the others, as long as the interfaces between the layers stay the same. The layer model decouples the application protocols from the physics of the network hardware and the topology of the network connections.

## TCP/IP four layer model
- application layer
- transport layer
- internet layer
- host-to-network layer

90% of the time of your Java code will work in the application layer and only talk to the transport layer. The other 10% of the time, you will be in the transport layer and talking to the application layer or the internet layer.

## host-to-network layer
The host-to-network layer defines how a particular network interface, such as an Ethernet card or a WiFi antenna, sends IP datagrams over its physical connection to the local network and the world.

Whichever physical links you encounter, the APIs you use to communicate across those networks are the same. What makes that possible is the internet layer.

## internet layer
The internet layer defines how bits and bytes of data are organized into the larger groups called packets, and the addressing scheme by which different machines find each other.

The Internet Protocol (IP) is the most widely used internet layer protocol in the world and the only internet layer protocol Java understands.

IPv4 and IPv6 are two very different protocols that do not interoperate on the same network without special gateways or tunneling protocols, Java hides almost all the of differences from you.

In both IPv4 and IPv6, data is sent across the internet layer in packets called *datagrams*. Each IPv4 datagram contains a header between 20 and 60 bytes long and a payload that contains up to 65535 bytes of data. In practice, most IPv4 datagrams are much smaller, ranging from a few dozen bytes to a little more than eight kilobytes. An IPv6 datagram contains a larger header and up to four gigabytes of data.

Several address blocks and patterns are special. All IPv4 addresses that begin with 10., 172.16. through 172.31. and 192.168. are unassigned. They can be used on internal networks, but no host using addresses in these blocks is allowed onto the global Internet. 

Besides routing and addressing, the second purpose of the internet layer is to enable different types of host-to-network layers to talk to each other. The internet layer is responsible for connecting heterogenous networks to each other using homogeneous protocols.

## transport layer
The transport layer is responsible for ensuring that packets are received in the order they were sent and that no data is lost or corrupted.

TCP was layered on top of IP to give each end of a connection the ability to acknowledge receipt of IP packets and request retransmission of lost or corrupted packets. Furthermore, TCP allows the packets to be put back together on the receiving end in the same order they were sent.

UDP is an unreliable protocol that does not guarantee that packets will arrive at their destination or that they will arrive in the same order they were sent. UDP is suitable to use if the order of the data isn't particularly important and if the loss of individual packets won't completely corrupt the data stream.

ICMP, the Internet Control Message Protocol, which uses raw IP datagrams to relay error messages between hosts. The best-known use of this protocol is in the **ping** program. Java does not support ICMP, nor does it allow the sending of raw IP datagrams. 

The only protocols Java supports are TCP and UDP, and application layer protocols built on top of these. All other transport layer, internet layer, and lower layer protocols such as ICMP, IGMP, ARP, RARP, RSVP, and others can only be implemented in Java programs by linking to native code.

## application layer
The application layer is where most of the network parts of your programs spend their time. Your programs can define their own application layer protocols as necessary.

Domain Name System (DNS) was developed to translate hostnames that humans can remember into numeric Internet addresses.

# The Internet
The Internet is the world's largest IP-based network.

When a company or organization wants to set up an IP-based network connected to the Internet, their ISP assigns them a block of addresses. Each block has a fixed prefix. However, the lowest address in all block used to identify the network itself, and the largest address is a broadcast address for the network, so you have two fewer available addresses than you might expect.

## NAT (Netwrok Address Translation)
In NAT-based networks most nodes only have local, non-routable addresses selected from either 10.x.x.x, 172.16.x.x to 172.31.x.x, or 192.168.x.x. The routers that connect the local networks to the ISP translate these local addresses to a much smaller set of routable addresses.

The router watches outgoing and incoming connections and adjusts the addresses in the IP packets. Exactly how it keeps track of which connections come from and are aimed at which internal computers is not particularly important to a Java programmer. As long as your machines are configured properly, this process is mostly transparent.

Eventually, IPv6 should make most of this obsolete. NAT will be pointless.

## firewalls
The hardware and software that sit between the Internet and the local network, checking all the data that comes in or out to make sure it's legal, is called a firewall. The firewall is responsible for inspecting each packet that passes into or out of its network interfaces and accepting it or rejecting it according to a set of rules.

Filtering is usually based on network addresses and ports. More intelligent firewalls look at the contents of the packets to determine whether to accept or reject them.

## proxy servers
If a firewall prevents hosts on a network from making direct connections to the outside world, a proxy server can act as a go-between. 

For example, a machine that is prevented from connecting to the external network by a firewall would make a request for a web page from the local proxy server instead of requesting the web page directly from the remote web server. The proxy server would then request the page from the web server and forward the response back to the original requester. 

One of the security advantages of using a proxy server is that external hosts only find out about the proxy server. They do not learn the names and IP addresses of the internal machines, making it more difficult to hack into internal systems.

Whereas firewalls generally operate at the level of the transport or internet layer, proxy servers normally operate at the application layer.

Proxy servers can also be used to implement local caching.

The biggest problem with proxy servers is their inability to cope with all but a few protocols. Generally established protocols like HTTP, FTP, and SMTP are allowed to pass through, which newer protocols like BitTorrent are not. It is a particular disadvantage for Java programmers because it limits the effectiveness of custom protocols. In Java, it is easy and often useful to create a new protocol that is optimized for your application.

# Client/Server Model
Most modern network programming is based on client/server model. A client initiates a conversation while a server waits for clients to start conversations with it.

Not all applications fit easily into a client/server model. Another sort of connection is *peer-to-peer*. The telephone system is the classic example of a peer-to-peer network.

Java does not have explicit peer-to-peer communication in its core networking API. However, applications can easily offer peer-to-peer communications in several ways, most commonly by acting as both a server and a client.