Load Balancing
####################################

Create a new pool for back end load balancing of DNS queries. The Ubuntu server will be the single pool member.

Navigate to: **DNS >> Delivery : Load Balancing : Pools : Pool List**

.. image:: /class2/media/class2_dns__pool_create_flyout.png
  :scale: 40

Create a pool according to the following table:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "dns_pool"
   "Health Monitors", "example.com_dns_monitor"
   "Node Name", "dns01_node"
   "Address", "10.1.20.4"
   "Service Port", "53"

.. image:: /class2/media/create_pool.png


.. admonition:: TMSH

   tmsh create ltm pool dns_pool members add { dns01_node:53 { address 10.1.20.4 }  } monitor example.com_dns_monitor
