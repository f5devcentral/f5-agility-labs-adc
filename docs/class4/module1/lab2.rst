tcpdump Filters
~~~~~~~~~~~~~~~

This section is for informative value and nothing will be done in the lab environment.

When running tcpdump capture from the F5 you should always use a filter to limit the volume of traffic you will gather.

#. Host Filters

   **tcpdump host 192.168.2.5**
   This will filter the packet capture to only gather packets going to or coming from the host 192.168.2.5.

   **tcpdump src host 192.168.2.5**
   This will filter the packet capture to only gather packets coming from 192.168.2.5.

   **tcpdump dst host 192.168.2.5**
   This will filter the packet capture to only gather packets going to 192.168.2.5.


#. Port Filters

   **tcpdump port 443**
   This will filter the packet capture to only gather packets with a source or destination of port 443.

   **tcpdump src port 1055**
   This will capture traffic being sourced from port 1055.

   **tcpdump dst port 443**
   This will capture traffic destined for port 443.
