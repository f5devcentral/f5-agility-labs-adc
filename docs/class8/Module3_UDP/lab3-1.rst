Task 1: Enabling Layer 7 LB for HTTP
====================================

By default, TMOS load balances HTTP traffic at layer 4.  If a client connects to an HTTP Virtual Server and requests a log of files and resources through typical browsing through a web site, the load balancing decision .is done during the initial TCP connection.  All of the requests will goo to a single pool member.

In most cases this could be OK many applications being front-end by a CDN <<or other examples??>> requests forom multiple end clients could get aggregated before hitting the Virtual Server.  With TMOS, it is possible to load balance at the application layer (Layer 7 LB) by enabling the OneCOnnect profile on the HTTP Virtual Server.

Here is an example (need a graphic):

A live-streaming TV provider is front-ending their application with a CDN and has a back-end cache layer of 50 servers ahead of the video ingest servers.  The CDN may merge multiple requests together on the way in to the Virtual Server - Requests for channel A, B, C, D all within a single TCP stream.  With L7 LB, TMOS can break out each request and send it to a specific pool member.  This allows for more warm cache hits and a reduction is cache layer storage redundancy as all channel data doesn't have to be on each cache layer server. <<boil this down?>>


1. From the BIGIP01 UI, go to **Local Traffic > Pools > Pool List**
2. Click on **web02_pool**
3. Click on **Statistics** from the top right menu
   
   .. image:: ../images/http_pool_stats_button.png
       :width: 400px

4. Clear any pool stats if they are not already all zeros. Click the check mark next to the **Status** dropdown on the left to select all pool members. Then clcik reset at the bottom to clear all of the stats.

   .. image:: ../images/http_pool_stats_reset.png
       :width: 700px

5. Click the **Auto Refresh** dropdown and set the time to 10 seconds
   
   .. image:: ../images/http_pool_stats_refresh10.png
       :width: 450px

6. From the Ubuntu-Client SSH window, run the following command::

      ~/web02_testing.sh 


   The script will run a curl times requesting a list of 5 files 2 times from Virtual Server **web01_vs1**

7. From BIGIP01 UI, check the pool stats.  You should see all 10 requests have gone to a single server
   
   .. image:: ../images/http_pool_stats_base.png
       :width: 900px

8. Clear the pool stats
9. 