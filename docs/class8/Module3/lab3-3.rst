Task 3: Layer 7 LB for UDP
==========================

Even though UDP is connectionless, TMOS will load balance UDP to the same pool member based on the client's source IP:Port by default.  This behavior can be changed through use of UDP Profiles by enabling the **Datagram LB** option.

Looking at a DNS design where you want to load balance across a large pool of DNS servers, the local DNS servers may send queries in batches using the same ephemeral source port resulting in the requests going to the same pool member.  Enabling Datagram LB in the UDP profile allows TMOS to send each query to the most capable pool member.

   
1. From BIGIP01 UI, go to **Local Traffic > Pools > Pools List** and click on **dns_pool**
2. Click on **Statistics** from the menu bar

   .. image:: ../images/dns_stats_menu.png
       :width: 500px


3. Reset the statistics for all pool members if they are not already all zeros

   .. image:: ../images/udp_pool_reset.png
       :width: 500px


4. Set **Auto Refresh** to 20 seconds if it isn't already set from earlier stats reviews
5. From the Ubuntu-Client SSH window, run the following command::

      ~/dns_loop.sh


   The script runs 1000 DNS queries through UDP Virtual Server **dns_vs1**.  The Virtual Server is currently configured with a custom UDP Profile but all settings are default.


6. From BIGIP01 UI, check the pool stats.  You should see all 1000 queries have gone to a single server

   .. image:: ../images/dns_stats_base.png
       :width: 750px


7. From BIGIP01 UI, go to **Local Traffic > Profiles > Protocol > UDP** and click on **udp_lab** from the list of profiles
8. On the right side, check the boxes for **Idle Timeout** and **Datagram LB**
9. Change Idle Timeout to 1 second since the DNS response is quick and check the Datagram LB option.  Click the **Update** button at the bottom of the page to save the settings.

   .. image:: ../images/udp_profile_mod.png
       :width: 750px

   
   **Note:** Setting the Idle Timeout to Immediate will result in the response traffic from the server being returned with the actual server IP since there is no table to map the return traffic to the Virtual Server IP.  For one-way traffic like UDP log data a setting of Immediate will be fine as there is no server response.  

10. This UDP Profile is already assigned to Virtual Server **dns_vs1**
11. From BIGIP01 UI, go to **Local Traffic > Pools > Pools List** and click on **dns_pool**
12. Reset the pool stats again.
13. From the Ubuntu-Client SSH window, run the following command::

       ~/dns_loop.sh


14. From BIGIP01 UI, check the pool stats.  You should see all 1000 queries evenly split across the pool members

    .. image:: ../images/dns_stats_datagram_lb.png
        :width: 750px


This concludes Lab 3 and is the end of the class.


