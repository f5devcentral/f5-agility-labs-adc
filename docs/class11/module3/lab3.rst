Results
##################################

Now that the BIG-IP has transfered the zone, we can look at status and if needed dump the zone. To check the status:

#. Click on the newly created *example.com* zone and make sure it is showing green for 'Available' indicating that the initial AXFR transfer was successful.

   .. image:: /class2/media/DNS_Express_result1.png


You can use the **dnsxdump** utility to view the DNS Express database information, which includes zone information and statistics.

   * The **DB Dump** section of the **dnsxdump** utility output displays the zone information for all configured DNS Express zones.
   * The **DB Stats** section of the **dnsxdump** utility output displays a cumulative count of records for all configured DNS Express zones.

#. From the Web Shell of SSH session to the BIG-IP:  
 
  Run the following command to see the contents of the DNS Express database from the Advanced Shell (not tmsh): 

   .. code-block:: console

      #dnsxdump | less

   Examine the results

To see or troubleshoot zone transfers, we can refer to the */var/log/ltm* log file. A quick examination of the log should show a successful zone transfer in the lab:

   .. code-block:: console

      #tail -100 /var/log/ltm | grep zxfrd 

   .. image:: /class2/media/mod3lab3.png
