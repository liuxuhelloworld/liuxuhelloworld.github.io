An IPv4 address is normally written as four unsigned bytes, each ranging from 0 to 255, with the most significant byte first. Bytes are separated by periods for the convenience of human eys.

An IPv6 address is normally written as eight blocks of four hexadecimal digits separated by colons.

Every computer connected to the Internet should have access to a machine called a *domain name server*, generally a Unix box running special DNS software that knows the mappings between hostnames and IP addresses. Most domain name servers only know the addresses of the hosts on their local network, plus the addresses of a few domain name servers at other sites. If a client asks for the address of a machine outside the local domain, the local domain name server asks a domain name server at the remote location and relays the answer to the requester.

# InetAddress
The **java.net.InetAddress** class is Java's high-level representation of an IP address, both IPv4 and IPv6. Usually, it includes both a hostname and an IP address.

Because DNS lookups can be relatively expensive (on the order of several seconds for a request that has to go through several intermediate servers, or one that's trying to resolve an unreachable host) the **InetAddress** class caches the results of lookups.

Besides local caching inside the **InetAddress** class, the local host, the local domain name server, and other DNS servers elsewhere on the Internet also cache the results of various queries. Java provides no way to control this. As a result, it may take several hours for the information about an IP address change to propagate across the Internet. In the meantime, your program may encounter various exceptions, including **UnknownHostException**, **NoRouteToHostException**, and **ConnectionException**, depending on the changes made to the DNS.

Hostnames are much more stable than IP addresses. Use an IP address only when a hostname is not available.

Some IP addresses and some patterns of addresses have special meanings.

The **isLoopbackAddress()** method returns true if the address is the loopback address, false otherwise. The loopback address connects to the same computer directly in the IP layer without using any physical hardware. In IPv4, this address is 127.0.0.1. In IPv6, this address is 0:0:0:0:0:0:0:1 (a.k.a. ::1).

The **isMulticastAddress()** method returns true if the address is a multicast address, false otherwise. Multicasting broadcasts connect to all subscribed computers rather than to one particular computer. In IPv4, multicast addresses all fall in the range 224.0.0.0 to 239.255.255.255. In IPv6, they all begin with byte FF.

# NetworkInterface