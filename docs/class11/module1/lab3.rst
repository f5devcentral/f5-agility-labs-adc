Results
#################################

#. Navigate to: **DNS  ››  Delivery ›› Load Balancing ›› Pools ›› Pool List**

.. image:: /class2/media/class2_cache_listpool_flyout.png
   :scale: 50

#. Click to select *dns_pool*, and then select *Members*

#. Observe the health status of the pool (green is good)

.. image:: /class2/media/dns_pool_status.png
   :scale: 75

.. admonition:: TMSH

      tmsh show ltm pool dns_pool detail

