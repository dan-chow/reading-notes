## CS2P: Improving Video Bitrate Selection and Adaptation with Data-Driven Throughput Prediction

Bitrate adaptation is critical to ensure good quality-ofexperience (QoE) for Internet video. Several efforts have
argued that accurate throughput prediction can dramatically
improve the efficiency of (1) initial bitrate selection to lower
startup delay and offer high initial resolution and (2) midstream bitrate adaptation for high QoE.


Our analysis also reveals two key insights that form the
basis for our proposed design. First, we observe that similar sessions (i.e., sessions sharing the same critical features
such as ISP, location) tend to have similar initial and average
throughput values and even exhibit similar structural properties in throughput variation.
Second,
even though the observed throughputs for each video chunk
within a session are inherently noisy, they do exhibit natural stateful evolving behaviors.
Building on these data-driven insights, we develop the
CS2P (Cross Session Stateful Predictor) approach for improving bitrate selection and adaptation (Figure 1).


fig_1_Overall_workflow_of_CS2P.PNG




The player uses bitrate selection and adaptation algorithms that choose the bitrate levels for future chunks to deliver the highest possible QoE. Here, the adaptation algorithm needs to balance multiple QoE considerations as discussed in prior work [15,16,22,47]. These include the initial
startup latency for the video to start playback, the amount
of rebuffering the user experiences during the session, the
average bitrate of the rendered video, and the smoothness
of the rendered video as measured by the number of bitrate switches. 


Observation 1: There is a significant amount of throughput variability within a video session, and simple predictive
models (e.g., looking at recent epochs) do not work.


We tried a range of simple prediction models used in prior
work [24, 30, 47] for predicting the throughput of the next
epoch based on past observations in the session. These include: (1) Last-Sample (LS, using the observation of the
last epoch), (2) Harmonic-Mean (HM, harmonic mean of
past measurements), and (3) Auto-Regressive (AR, a classical timeseries modeling technique). 


Observation 2: The evolution of the throughput within a
session exhibits stateful/persistent characteristics, which if
captured can lead to improved prediction.


fig_2_Stateful_behaviors_in_session_throughput..PNG


Figure 4a gives a visual example from our dataset. We can
clearly observe some states within the throughput variation.
We investigate the throughput variation across two consecutive epochs for a broader set of sessions and find similar stateful behaviors in these sessions. 
We can
observe a clustered trend in the distribution of these points,
i.e., there are some discrete states and the session throughput
changes across these states (red circles in Figure 4b).
