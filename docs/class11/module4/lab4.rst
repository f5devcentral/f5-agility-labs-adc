Results
#####################################

Lets look at the results from both the client along with the logged messages on the BIG-IP.

From the CLI on the BIG-IP run *tail -f /var/log/ltm | grep -i tmm*

From the Ubuntu Client , terminal prompt run these two queries: *dig @10.1.10.53 example.com; dig @10.1.10.53 example.com +dnssec*

The BIG-IP log will show the queries, and the Ubuntu Client the unsigned and signed responses. Do you see the different in logged messages on the BIG-IP?

.. image:: /class2/media/mod4lab4.png
   :scale: 50
