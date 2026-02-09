Results
#######

With the iRule applied, DNS queries will be processed and log messages sent out. Open a shell to the BIG-IP and run :

.. code-block:: bash

   tail -f /var/log/ltm

Now run DNS queries from the Ubuntu Client:

.. code-block:: bash
   
   dig @10.1.10.53 www.f5.com

And analyze the results:

.. code-block:: bash

   Feb 13 14:47:14 ip-10-1-1-4 info tmm[10647]: 2020-02-13 14:47:13 ip-10-1-1-4.us-west-2.compute.internal qid 29530 from 10.1.10.4#43881: view none: query: www.f5.com IN A +E (10.1.10.53%0)
   Feb 13 14:47:14 ip-10-1-1-4 info tmm3[10647]: Rule /Common/DNS-query-filtering <DNS_REQUEST>: Category /Common/Uncategorized matching www.f5.com is not filtered
   Feb 13 14:47:14 ip-10-1-1-4 info tmm[10647]: 2020-02-13 14:47:14 ip-10-1-1-4.us-west-2.compute.internal qid 29530 to 10.1.10.4#43881: [NOERROR qr,rd,ra] response: www.f5.com. 30 IN CNAME dwbfwz8xncgmg.cloudfront.net; dwbfwz8xncgmg.cloudfront.net. 60 IN A 99.86.33.52; dwbfwz8xncgmg.cloudfront.net. 60 IN A 99.86.33.5; dwbfwz8xncgmg.cloudfront.net. 60 IN A 99.86.33.9; dwbfwz8xncgmg.cloudfront.net. 60 IN A 99.86.33.53;

The query *www.f5.com* did not match any categories, and was resolved. Now lets try a matching query:

.. code-block:: bash

   dig @10.1.10.53 www.tmz.com

Notice the DNS response is quite different as well as the log entry on the BIG-IP.

.. code-block:: bash

   Feb 13 15:27:37 ip-10-1-1-4 info tmm[10647]: Rule /Common/DNS-query-filtering <DNS_REQUEST>: BLOCKED: Category /Common/Entertainment matching www.tmz.com is filtered.
   Feb 13 15:27:37 ip-10-1-1-4 info tmm[10647]: 2020-02-13 15:27:36 ip-10-1-1-4.us-west-2.compute.internal qid 32427 to 10.1.10.4#55151: [NOERROR qr,rd,ad] response: www.tmz.com. 111 IN A 10.1.20.252;

You can experiment with various queries to see the catagory of the domain name via the log messages. If you want to add a new category, edit the iRule accordingly.

To list current categories, from the BIG-IP enter the TMSH shell with *tmsh*, then run the following command:

.. code-block:: bash
  
   root@(ip-10-1-1-4)(cfg-sync Standalone)(TimeLimitedModules::Active)(/Common)(tmos)# list sys url-db url-category
