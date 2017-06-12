### An implementation of multipath TCP in ns3

A user may want to leverage these different interfaces into using concurrently several paths to achieve the following goals:

- Seamless mobility.
- Bandwidth aggregation.
- Higher confidentiality.
	- If a flow of data is split over several paths, it may become harder for an attacker to reconstitute the whole connection flow.
- Lower response time.
	- Sending duplicated packets on several paths can increase the probability for the data to follow uncongested paths.
