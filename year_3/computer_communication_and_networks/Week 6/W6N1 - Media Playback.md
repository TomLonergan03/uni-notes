# Overview
With streaming of videos becoming so prevalent in today's Internet, scale becomes a challenge: How do we reach close to a billion devices all streaming our content? All of which have different capabilities? (wired vs mobile; bandwidth rich versus bandwidth poor)
The solution is a *distributed, application-level infrastructure* designed for this purpose.
# Encoding Videos
Videos are a sequence of images displayed at a constant rate.
To save on the number of bits required to stream a video, videos can be encoded to use redundancy **within and between** images.
> [!summary] Definition
> **Spatial Redundancy**: Redundancy within an image. (eg, an N pixel long strip of purple can be encoded as N bits all of value 'purple', or as the value 'purple' followed by N to indicate that the value is duplicated N times.)
> 
> **Temporal Redundancy**: Redundancy between images. (eg, a pixel that is black for N consecutive frames can have black encoded into each of the N frames, or it can be encoded as black followed by N to say this pixel will only change after N frames.)
## Encoding Varients
**Constant Bit Rate (CBR)** encoding is a video encoded at a fixed rate.

**Variable Bit Rate (VBR)** encoding allows the encoding bit rate to change as the amount of spatial or temporal coding changes.
# Streaming Stored Video
The main challenge of streaming video is that the server-to-client bandwidth will vary over time due to varying usage of the network and thus varying congestion levels. This will cause packet loss, which in turn will create playback delays or poor video quality.
There are further challenges, including:
- *Continuous Playout*: Once started, there should be no variation in the playback speed (stuttering). To prevent this, client-side buffering will be required to match the playout requirements.
- Packets may be lost and require retransmission
- The client may alter playback through rewind, fast forward, or pause.
# DASH
> [!abstract] Definition - DASH
> **Dynamic, Adaptive Streaming over HTTP (DASH)** is an encoding standard where videos are encoded by the server at different rates. The video is split into small chunks, encoded, and referenced on a *manifest file*.
> From there, the client can measure the server-to-client bandwidth and choose the maximum coding rate (highest quality) permitted at the time given the current bandwidth.

The client can "intelligently" determine *when* to request a chunk (to prevent buffer starvation or overflow), the encoding rate to request (bandwidth dependent) and where to request the chunk from (using the manifest file)