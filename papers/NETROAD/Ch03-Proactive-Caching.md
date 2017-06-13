## Proactive Content Caching for Mobile Video Utilization Transportation Systems and Evaluation Through Field Experiments

We aim at turning transportation vehicles into radio base stations and mobile cache servers.

An opportunistic content pushing scheme was proposed in [10] that predicts the moving routes of roaming users and pre-locates content to Wi-Fi spots along their routes. The authors of this paper proposed a similar method called comfort route navigation that recommends a moving route that maximizes throughput or minimizes power consumption instead of the conventional shortest path[22].

Our approach has two phases: proactive caching and video streaming. The proactive caching phase indicates how to distribute content to the nearest cache router/server, which is a train server in this case. Then, the recieved video segments are streamed to the users via high-speed wireless access schemes such as wireless LANs. We called this phase the video streaming phase.

The key function of our system is a delivery scheduler we call smart scheduler. This scheduler determines content quality and the amount of content segments, and also selects delivery locations and timing.

The content bitrate is computed by fulfilling the following three conditions:

- *Proactive Caching:* content segments have to be delivered to station servers before the trains arrive.
- *Continuous Playback:* video playback has to be continuous until arrival at the next station to avoid interruption.
- *Smooth Streaming:* received video segments should be streamed smoothly to the users inside a train.

The smart sheduler needs to be extended for robust scheduling because a current delivery schedule may be collapsed by train delays and users' playback behaviors.

Secure Copy Protocol (SCP) was used in the HTTP based prototype, and Internet message aggregation was used in the NDN based prototype, respectively.

## An Optimal Cache Management Framework for Information Centric Networks with Network Coding

We propose a cache management framework for ICNs based on *software-defined networking* (SDN) where a controller is responsible for determining the optimal caching strategy and content routing via *linear network coding* (LNC).

To manage in-network caches in ICNs, two major issues need to be jointly considered. One is the *caching strategy* that determines which data chunks shall be cached at each *Content router* (CR), and the other is *content routing* that determines where to route content requests and how to deliver content.

In the literature, there are two types of caching strategies: non-cooperative and cooperative. In non-cooperative caching strategies, a CR oppotunistically caches the received data, which may lead to frequent cache updates, sub-optimal cache allocation and caching duplication. In cooperative caching strategies, a CR can sync with its neighboring CRs to determin which set of data chunks to cache.

To apply LNC, the controller may choose deterministic LNC or random LNC. If deterministic LNC is used, the controller shall send a set of *global encoding vectors* (GEVs) to each CR so that the CR can forward a request to LNC-enabled servers to obtain required coded data chunks. In this case, the controller can guarantee that any set of coded data chunks of each content cached in ICNs with size no more than the size of the content is linearly independent. On the other hand, if random LNC is used, the controller can send the number of required coded data chunks to each CR, who will then forward the request to LNC-enabled servers that can generate coded data chunks with random GEVs. In such case, coded data chunks of the same content may be linearly independent to each other with a certain (high) probability. The advantage of the latter scheme is that the computation overhead of the controller can be reduced at the risk of a possible compromise of the linear independence of coded data chunks.

## Cache as a Service: Leveraging SDN to Efficiently and Transparently Support Video-on-Demand on the Last Mile

High quality online video streaming, both live and on-demand, has become an essential part of consumers' every-day lives. The popularity of video streaming has placed a heavy burden on the network infrastructure that now has to transfer an enormous amount of data very quickly to the end-user.

Historically though, the most common use of caching and proxy servers is to serve static web content. Thus, existing solutions (e.g. Squid) are not usually optimised for the high storage, bandwidth and the very demanding application requirements of video delivery (e.g. skipping to a certain part of a video stream).

OpenCache offers a powerful interface that provides *cache as a service*. This is not intrinsically linked to a particular type of content, or to a specifically linked to a particular type of content. The control and decision of what content should be cached is passed on to the network administrator of the ISP, who now has the ability to optimise his netowork's utilisation and external link usage.

The main entity of OpenCache, namely the OpenCache Controller (OCC), orchestrates the VoD caching and distribution functionalities with the aid of a key-value store, that acts as a database. The OCC communicates with the OpenFlow controller of the network via a JSON-RPC interface. A VoD server is the primary source for the video assets and could be located anywhere on the Internet reachable by its IP address. Finally, the OpenCache Nodes (OCNs) are the caches of the service, inherently being deployed in various locations in the network.

The OpenCache Controller (OCC) is the main orchestrator of the in-network caching functionality that OpenCache provides, and implements the following four main operations:

- Receives requests for content of intrest
- Implememts the caching logic
- Manage the available OCNs' resources
- Manage and maintain the OpenFlow flows in the network dynamically

The OCC exposes another JSON-RPC based API that allows the communication of the OCC with a number of OCNs.

Most critically to the operation of OpenCache is the ability to transparently redirect requests for content to a running cache instance, as mentioned previously. More specifically, this is achieved by rewriting the packet header information, and intenttionally forwarding it towards a cache or a client.

In addition, monitoring information, when used in conjunction with the redirecting action described previously, allows us to load balance requests on-the-fly and in real-time.

Three main metrics are used to evaluate the effectiveness of OpenCache; start up delay (key QoE metric), external link utilization and video playback bitrate.

## Performance Evaluation of Comfort Route Navigation Providing High-QoS Communication for Mobile Users

It is well known that communication quality in such wireless networks easily fluctuate because of shared bandwidth among multiple users, degradation of the radio signal, and mobile backhaul congestion.

As a result, it is important to select an optimal wireless infrastructure depending on service requirements and users' situations, such as time of day and locations.

To achieve it, three important phases are necessary; "network discovery phase", "network selection phase" and "network switching phase".

The optimal wireless interfaces should be selected by service requirements, multiple QoS metrics and load-balancing.

In network switching phase, it is known as vertical handover (VHO). VHO is a mechanism which can be solved to switch different wireless interfaces seamlessly without any degradation of communication quality. One of major solution of VHO for real-time traffic is a bi-cast (soft handover) scheme.

NaviComf aids in the navigation of people to their destinations via comfortable paths, which are specified by temperature, humidity, or pedestrian crowds, captured by multi-model sensors.

To create the optimal route, the authors employed the following assumptions: (a) wireless resources can be calculated by the distance between a user and an AP, and (b) each user is allowed to travel towards and/or stay at the broadband spots for a limited time.

The next 200 seconds of TCP throughput fluctuation in cellular and Wi-Fi networks can be also predicted by constructing a stochastic model of TCP throughput, which is a mixture of stationary and non-stationary states.