## RFRA: Random Forests Rate Adaptation for Vehicular Networks

Vehicular communications, however, are subject to fast topology and channel changes, as well as short connectivity spans.

Firstly, the movement of the nodes is limited in space by the length and width of the roads, and is regulated by traffic lights and speed limitations. Furthermore, moving cars do in general first approach a roadside unit and later leaves the communication area, which results in a certain pattern of the signal strength over time. Finally, GPS devices provide valuable information, such as position, speed, and acceleration of a node.

Our approach exploits SNR samples obained over a time window to characterize the propagation environment. We also use GPS information such as position and speed to increase the prediction accuracy.

![Alt text](img/ch4-fig1.PNG)

As input feature vector we select the variables SNR, current speed, current position, as well as data rate employed. The output of Random Forests is the prediction if the packet transmitted under the conditions described by the input variables will be received successfully by the RSU.

The car combines SNR information (obtained from packets originated at the RSU) with speed and propagation distance to predict the success probability for each data rate and decide on the most appropriate rate for transmission.

The SNR samples collected over the last 25 ms are more important than the propagation distance. Interestingly, the speed is the input variable with the lowest relevance.

**Goodput** is defined as rate (in bits/seconds) of correctly received data frames excluding headers. Unless differently specified, we show the goodput measured by the RSU aggregated over all transmitting nodes. **Packet Error Rate (PER)** is the ratio of erroneous data packets to the total data packets transmitted. **Data Rate** refers to the average data rate (in bits/second) used by the transmitting nodes during a simulation run. **Accuracy** represents if the data rate has been precisely selected. To evaluate this, we obtain (off-line) the performance of every data rate as function of the SNR.

## CARS: Context-Aware Rate Selection for Vehicular Networks

*Rate Adaptation* is the problem of selecting the best transmission rate based on the real-time link quality, so as to obtain maximum throughput at all times.

![Alt text](img/ch4-fig2.PNG)

In fact, the highest bitrate of 54Mbps causes the transmission range to be 60m, whereas at the lowest bitrate of 6Mbps, the transmission is 240m.

Even though the RSSI spikes are larger, the large-scale path-loss is much more significant than small-scale fading.

In vehicular networks, a MAC-layer packet scheduling algorithm can cause clients to periodically transmit packets in short bursts and then stay quiet.

The context information used in CARS broadly consists of information about the environment that is available to the node and which has an effect on the packet delivery probability. Such information could include the position, speed and acceleration of the vehicle, the distance from the neighboring vehicle, and environment factors such as location, time of day, weather, type of road and traffic density.

Even at very high mobility (vehicular speed of 65mph), in 100 ms, the vehicle moves less than 3 meters. Therefore, we set the recalculation frequency to be 100 ms.

To model the effect of distance, we could use the free space path loss model, the two ray propagation model or a more complex fading model. Models such as the delay tap model or ray models with delay profiles can be used to model the effect of speed.