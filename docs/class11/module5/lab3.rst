Results
#####################

Now lets look at results. Tail the *ltm* log on the BIG-IP

tail -f /var/log/ltm | grep tmm

From a Web shell on the Ubuntu Client, start wtih some DNS queries. 

First, issue a DNS query that returns no response: 

   .. code-block:: console

      dig @10.1.10.53 nope.f5.com

.. image:: /class2/media/nxdomain.png

Next, set the DNSSEC OK bit in the query (DO): 

   .. code-block:: console

      dig @10.1.10.53 dnssec-deployment.org +dnssec

.. image:: /class2/media/dnssec-resolver-results.png

Finally, set the DNSSEC but but observe how the response is different: 

   .. code-block:: console

      dig @10.1.10.53 www.google.com +dnssec


