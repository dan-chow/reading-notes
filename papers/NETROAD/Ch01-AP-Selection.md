## Characterizing and Improving WiFi Latency in Large-Scale Operational Networks

The three most informative indicators for WiFi latency are channel utilization, the number of online devices, and the signal-to-noise-ratio (SNR), while the commonly considered RSSI used for AP selection is not.

SNMP data is a commonly-used data source for monitoring large-scale EWLANs. Most vendors provide a large range of useful SNMP data on their wireless controllers.

Radio factors include typical universal wireless performance metrics of APs and user devices, for example channel utilization, interference utilization, receiver/transmitter utilization, and #devices (per radio).

Channel utilization is the percentage of time used by all traffic of this channel; interference utilization is the part of channel utilization used by other 802.11 networks on the same channel; receiver/transmitter utilization is the percentage of time the AP receiver/transmitter is busy operating on packets; #devices (per radio) is the number of devices connected to the specific band of the AP.

![Alt text](img/ch1-fig1.PNG)

A natural question is that since 5GHz provides a lower WiFi latency, why are dual-band devices nevertheless connecting to 2.4 GHz? The reason is that the device AP selection heavily depends on RSSI, and since 5 GHz signals attenuate faster, devices are biased towards 2.4 GHz. In contrast, RSSI is not a very important  factor for WiFi latency: good RSSI does not guarantee low WiFi latency. Moreover, when channel utilization exceeds 50%, even high RSSI cannot achieve low latency. Interestingly, we also find that AP vendors such as Cisco provide a mechanism to direct dual-band devices to the 5 GHz band by delaying probe responses of 2.4 GHz to make 5 GHz more attractive.

First, for channel utilization, the EX slow class only appears when channel utilization > 47.5%, end the EX fast class only appears when it ≤ 47.5%. Therefore, channel utilization greater than 47.5% can be deemed as the ***heavy load problem***.

Second, we observe that even when channel utilization is very slow, a large #devices can still impact WiFi latency. FOr example, the leftmost slow node occurs where channel utilization is ≤ 22.5 and #devices > 35.5. Previous studies suggest that this is caused by the ***local contention problem***: a large number of concurrent senders increase the data collision probability and the backoff waiting time, and decrease the achievable channel utilization.

Third, for SNR, we find that the split points of SNR nodes in the decision tree are from 21.5 dB to 25.5 dB. This suggests that SNR less than 20 dB is low, and could impact WiFi latency. Specifically, the fading and noise problem increases the bit error rate and thus increases the MAC layer frame retry times; or decreases the PHY rate and thus increases the transmission time, which both in turn inflate the WiFi latency.