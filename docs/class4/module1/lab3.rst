F5 Specific tcpdump Switches
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section is for informative value and nothing will be done in the lab environment.

F5 has added some F5 specific switches to the tcpdump utility on the F5.  These switches give additional information on your packet captures.  These switches are places after the interface option in the command line as follows:

#. :n gives low details

    a. for example: **tcpdump -nni 0.0:n -s0 -w/var/tmp/capture.pcap**

    b. This will give basic information such as whether the captured traffic is ingress or egress to the F5.  It will also give the TMM instance the traffic is on as well as the Chassis slot processing the traffic.  This option also lists the virtual server name that processes the traffic.

    .. image:: /_static/class4/tcpdump-n.png

#. :nn gives medium details

    a. for example: **tcpdump -nni 0.0:nn -s0 -w/var/tmp/capture.pcap**

    b. This option will give all the low detail information plus the following: 

       * Flow ID number
       * Peer Flow ID number
       * TCP RST cause
       * Flow type
       * HA unit
       * Ingress Slot
       * Ingress port
       * Priority

    .. image:: /_static/class4/tcpdump-nn.png

#. :nnn gives high details

    a. for example: **tcpdump -nni 0.0:nnn -s0 -w/var/tmp/capture.pcap**

    b. This option will give all the low and medium details plus the following

       * Peer IP Protocol
       * Peer VLAN
       * Peer Remote address
       * Peer local address
       * Peer remote port
       * Peer local port

    .. image:: /_static/class4/tcpdump-nnn.png
