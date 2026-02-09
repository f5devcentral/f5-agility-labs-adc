Results
##################################

#. From the Ubuntu client, open a Web Shell or SSH session. Using the *dig* utility, we can query the listeners. 

   Repeat some of the same queries multiple times

   .. code-block:: console

      dig @10.1.10.53 www.f5.com
      dig @10.1.10.53 +tcp www.wikipedia.org
  
#. Viewing Cache Statistics

   Navigate to: **Statistics  ››  Module Statistics : DNS : Caches  ››  Caches** and then choose **Caches** from the 'Statistics Type' drop-down. 

   .. image:: /class2/media/router01_cache_click_view.png

   Examine the Query, Failure, and Cache details below.

   .. image:: /class2/media/router01_cache_view_details.png

   Login to the BIG-IP using a Web Shell or SSH session.  You can view the contents of the cache with the following TMSH command:

   .. admonition:: TMSH

      tmsh show ltm dns cache records rrset cache transparent_cache

   .. image:: /class2/media/tmsh_show_ltm_dns_cache_records.png

   To view the cache statistics similar to what you saw in the GUI you can use:

   .. admonition:: TMSH

      tmsh show ltm dns cache transparent transparent_cache

#. Clearing Entire Cache

   Navigate to **Statistics > Module Statistics > DNS > Caches**

   Set "Statistics Type" to "Caches".

   Select the cache and click "Clear Cache" to empty the cache. Note, this will clear the actual DNS cache on the BIG-IP. If you want to clear the cache statistics, select the cache and hit the **Reset** button.

#. Back End Visibility

Log onto the Ubuntu Server using a Web SHell. You can then run tcpdump to view DNS queries and see what hits the back end and what does not.

Use the Ubuntu Client to issue DNS queries for various domains.

.. code-block:: console

      tcpdump -nni eth1 port 53

   Hit *Control-C* to exit the *tcpdump* 

