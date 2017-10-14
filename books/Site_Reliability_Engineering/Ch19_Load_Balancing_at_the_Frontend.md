## Chapter 19: Load Balancing at the Frontend

### Power Isn’t the Answer

- Trafc load balancing is how we decide which of the many, many machines in our datacenters will serve a particular request. Ideally, traffic is distributed across multiple network links, datacenters, and machines in an “optimal” fashion. But what does “optimal” mean in this context? There’s actually no single answer, because the optimal solution depends heavily on a variety of factors:
	- The hierarchical level at which we evaluate the problem (global versus local)
	- The technical level at which we evaluate the problem (hardware versus software)
	- The nature of the traffic we’re dealing with

### Load Balancing Using DNS

- Before a client can even send an HTTP request, it often has to look up an IP address using DNS. This provides the perfect opportunity to introduce our first layer of load balancing: DNS load balancing. The simplest solution is to return multiple A or AAAA records in the DNS reply and let the client pick an IP address arbitrarily.

- The first problem is that it provides very little control over the client behavior: records are selected randomly, and each will attract a roughly equal amount of traffic. Another potential problem stems from the fact that usually the client cannot determine the closest address.

- Of course, none of these solutions are trivial, due to a fundamental characteristic of DNS: end users rarely talk to authoritative nameservers directly. Instead, a recursive DNS server usually lies somewhere between end users and nameservers. This server proxies queries between a user and a server and often provides a caching layer. The DNS middleman has three very important implications on traffic management:
	- Recursive resolution of IP addresses
	- Nondeterministic reply paths
	- Additional caching complications

- Fortunately, we can integrate the authoritative DNS server with our global control systems that track traffic, capacity, and the state of our infrastructure.

### Load Balancing at the Virtual IP Address

- In practice, the most important part of VIP implementation is a device called the network load balancer. The balancer receives packets and forwards them to one of the machines behind the VIP. These backends can then further process the request.

- There are several possible approaches the balancer can take in deciding which backend should receive the request. The first (and perhaps most intuitive) approach is to always prefer the least loaded backend. The alternative is to use some parts of a packet to create a connection ID (possibly using a hash function and some information from the packet), and to use the connection ID to select a backend.

- Fortunately, there is an alternate solution that doesn’t require keeping the state of every connection in memory, but won’t force all connections to reset when a single machine goes down: consistent hashing.

- Returning to the larger question: how exactly should a network load balancer forward packets to a selected VIP backend?
	- One solution is to perform a Network Address Translation. However, this requires keeping an entry of every single connection in the tracking table, which precludes having a completely stateless fallback mechanism.
	- Another solution is to modify information on the data link layer (layer 2 of the OSI networking model). By changing the destination MAC address of a forwarded packet, the balancer can leave all the information in upper layers intact, so the backend receives the original source and destination IP addresses. The backend can then send a reply directly to the original sender—a technique known as Direct Server Response (DSR).

- Our current VIP load balancing solution uses packet encapsulation. A network load balancer puts the forwarded packet into another IP packet with Generic Routing Encapsulation (GRE), and uses a backend’s address as the destination. A backend receiving the packet strips off the outer IP+GRE layer and processes the inner IP packet as if it were delivered directly to its network interface. The network load balancer and the backend no longer need to exist in the same broadcast domain; they can even be on separate continents as long as a route between the two exists.