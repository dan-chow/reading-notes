## SERO: A Model-driven Seamless Roaming Framework for Wireless Mesh Network with Multipath TCP

Based on the throughput model, we propose a hybrid handoff strategy that uses multiple WiFi interfaces for data transmission and employs 3G augmentation to bridge the network interruption caused by handoff and to guarantee the total throughput above a predefined threshold for roaming devices.

Wireless mesh network (WMN) is such a technology that organizes wireless routers in a mesh topology using ad hoc mode to form a wireless communication backbone, which employs distributed multi-hop routing protocol to connect the wired infrastructure and provides Internet access to wireless devices via densely deployed mesh APs.

MPTCP creates several TCP subflows for a single application, each of which may take a different path through the network to achieve maximum bandwidth usage and robustness.

Layer-2 handoff are widely used by mobile devices to switch between APs while maintaining transparent to the upper layer protocols, which may cause network delay as long as several seconds. Layer-3 handoff uses IP-in-IP tunnel to enable routing to a roaming domain without changing IP address.

We focus on seamless roaming of multihomed devices exploiting both layer-2 handoff (switching from AP to AP) and vertical handoff (switching between WiFi and Cellular connections) to maintain active network connectivity and achieve maximum throughput with Multipath TCP protocol.

MPTCP enables seamlessly switching between network connections without modification of hardware and applications, which can be applied to overcome the above drawbacks.

The measurement-based solution can cope with the dynamics of wireless network characteristics due to mobility, and the model-driven strategy allows using empirical model to estimate future performance, which helps to optimize handoff decision to satisfy various objectives such as maximizing network download throughput, minimizing 3G usage, and reducing handoff latency and frequency, etc.

Based on the measured data, three models are constructed: the *propagation model* which predicts the change of RSSI due to movement; the *throughput model without handoff* which estiates the download throughput based on measuring RSSI and the number of nearby users; and the *throughput model during handoff* which describes the interruption and recovery of network service in layer-2 handoff process.

Specifically, we use the propagation model to predict the change of RSSI; we apply the Logistic function to depict the throughput during layer-2 handoff; and we derive a Shannon-like equation for throughput estimation based on the measurement of RSSI and number of users.

MPTCP enables an application-level handoff that applications can use multiple wireless interfaces simulataneously or alternatively.

The most common background noise in wireless communication is Gaussian White Noise, which fluctuates slowly around an average value.

TCP throughput is not only affected by the received signal strength, but also affected by the number of hosts contending for communication channel.

During layer-2 handoff, a WiFi interface takes the following actions including disconnecting from the current AP passively or actively, detecting nearby APs by listening to the beacon frames, and reconnecting to a new AP without changing the IP address.

The number of users *m* can also be obtained in realtime by either sending a query to the AP or by listening the RTS/CTS frames sent by the nearby wireless devices.

On one hand, we target at maximizing the WiFi throughput exploiting multipath transmission while reducing the delay and cost caused by handoff; on the other hand, we would like to provide the guarantee of a minimum throughput threshold for TCP download.

## Energy and Performance of Smartphone Radio Bundling in Outdoor Environments

Most of today's mobile devices come equipped with both cellular LTE and WiFi wireless radios, making radio bundling (simulataneous data transfers over multiple interfaces) both appealing and practical.

Our results show that MPTCP achieves only a fraction of the total performance gain possible, and that its energy-agnostic design leads to considerable power consumption by the CPU.

Today's mobile data access is dominated by public or outdoor locations, where WiFi cannot match the performamce of residential/private environments due to range limitations and uncontrolled contention between public WiFi users.

Bundling always outperforms single radio access methods in network throughput. More importantly, performance gain varies significantly across different RF environment conditions, and is sensitive to the relative throughput between WiFi and LTE links.

Bundling dual radios increases instantaneous power draw, but total energy is lowered by reduction in transfer time. More importantly, we show that the CPU can be a key contributor to energy consumption, and must be carefully considered in any energy-aware bundling solution.

Finally, our measurements and analysis show that performance gains from bundling are heavily dependent on traffic partitioning algorithms, and naive approaches can actually perform worse than single radio operations.

Using Android phones, we overcome this restriction by leveraging a small set of undocumented Android APIs, *i.e.* core API calls implemented in the *ConnectivityManager* class. The key is an API call that forces the cellular netowrk to remain on even after a WiFi connection is made. Using these APIs, we reconfigured the Android operating system to split the routing table and forward data traffic to both WiFi and cellular radio interfaces.

We use the Apach HTTP Java library to open HTTP connections, since it allows us to specify a local network interface when opening an HTTP connection.

Existing works have studied MPTCP as a method to using multiple radios simultaneously.

In post-processing, we synchronized the power profile from the power meter, tcpdump and signal strength traces. We derived the power draw by the radio interface(s) by subtracting the CPU power from the total power. We inferred the different power state machine for each interface.

*First*, the carrier frequencies of LTE (700/1700MHz) and WiFi (2.4GHz) are widely separated, so the two co-located radios do not physically interfere with each other. *Second*, modern smartphones have two CPU cores, and multi-threaded applications such as our bundling application minimize CPU contention by spreading load across cores.

A more detailed analysis on the traces reveals that the performance gap is mostly caused by the fact that MPTCP often "sacrifices" its throughput to maintain fairness between MPTCP and non-MPTCP clients.

Bundling, with optional traffic partitioning, can consistently boost throughput for most smartphone media applications that demand high throughput. The bundling gain comes from two factors, *radio independence, i.e.* the LTE and WiFi operate independently from each other, and *optimal traffic partitioning* that accurately projects radio throughput over time to fully utilize both radios.

Across our experiments with both phones we found that the LTE radio consumes at least 50% of the total power draw. On average, it consumes 5+ time more than power than the WiFi radio.

Alghough Bundling uses an extra radio and thus consumes more power draw (radio+CPU), its throughput improvement reduces data transfer time and effectively compensates the increase in power draw.

## Towards Wifi Mobility without Fast Handover

By using a transport layer mobility solution, we sidestep the need to upgrade all AP and client hardware for mobility and, more importantly, allow a client to utilize multiple APs at the same time.

**AP selection** is the problem of choosing the right AP based on signal strength, available wireless bandwidth, end-to-end bandwidth, RTT, current load.

However, by running two simultaneous MPTCP subflows, the combined throughput is surprisingly good. Repeated runs showed this result is robust, and we also confirmed this via ns2 simulation. MPTCP connection statistics show that traffic is flowing entirely over one of the two subflows, while the other one is starved, experiencing repeated timeouts.

In effect, we are witnessing a capture effect at the MPTCP level triggered by the interation with the WiFi MAC.

The wide variability and burstiness of losses and successes in frame delivery is well documented in literature.

For our client downloading from two APs, when one has a slightly worse channel, it will lose a frame and double its contention window before retrying, leaving the other AP to better utilize the channel.

When one AP experiences repeated timeouts, the other AP will send a packet, thus reducing the tail.

In the experiment *minstrel* favors higher rates even at low signal strengths (with lower frame delivery rate), leading to more retries per packet for the far away AP. Each retry imposes a longer backoff time on the transmitter, allowing the AP with better signal strength to win the contention more often and thus to send more packets.

We also verified in the simulator that when two senders use the same rate, the MAC preference for the better sender holds regardless of the maximum number of retransmissions allowed (0-10). What happens when the AP farthest from the client sends using lower rates, thus reducing its frame loss rate? Simulations showed that the effect on total throughput can be drastic: the client connecting to both APs can receive less then half of the throughput it would get via the best AP. This is because lower rates give the farthest AP and the closest one similar loss rates and thus changes of winning the contention for the medium. However, packets sent at lower bitrate occupy more airtime, thus decreasing the throughput experienced by the client.

When the client is close to one of the APs, the results differed from 802.11a/g: the throughput obtained with MPTCP was only approximately half the throughput obtainable via the closest AP.

This issue is not limited to 802.11n: any rate control algorithm that offers packet-level fairness between multiple senders in CS range greately harms the combined throughput achievable by MPTCP with multiple APs.

In summary, a client that associates to multiple APs and spreads traffic over them with MPTCP will receive close-to-optimal performance is strongly dependent on the rate adaptation algorithms employed by the APs, and these are outside the client's control.

In our context, **on-off** can be implemented either by using the MP_BACKUP mechanism in MPTCP which allows prioritizing subflows, or by relying on WiFi power save mode.

In particular, our MPTCP client uses local WiFi information to find out which APs are preferable, and relays this information to the sender as additional loss notification.

When deciding which AP it should prefer, the client needs to estimate the time *T_i* it takes on average for AP *i* to successfully send a packet, assuming the AP is alone in accessing the wireless medium.

We avoid this problem by leveraging the "retry" bit present in the MAC header of every frame, signaling whether the frame is a retransmission.

Our client monitors the beacons received over a 2s period, and switches to a new AP when it receives Δ more beacons than the current AP.

## An implementation of multipath TCP in ns3

A user may want to leverage these different interfaces into using concurrently several paths to achieve the following goals:

- Seamless mobility.
- Bandwidth aggregation.
- Higher confidentiality.
	- If a flow of data is split over several paths, it may become harder for an attacker to reconstitute the whole connection flow.
- Lower response time.
	- Sending duplicated packets on several paths can increase the probability for the data to follow uncongested paths.
