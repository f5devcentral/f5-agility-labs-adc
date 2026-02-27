Task 2: Layer 7 LB for UDP
==========================

Even though EDP is connectionless, TMOS will load balance UDP to the same pool member based on the client's source IP:Port by default.  This behavior can be changed through use of UDP Profiles.