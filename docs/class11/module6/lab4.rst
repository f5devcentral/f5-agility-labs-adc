Local Zone
#####################################

Navigate to: **DNS  ››  Caches : Cache List**

.. image:: /class2/media/select_validating-resolver_cache.png

Select validating-resolver_cache, click "Local Zones", and click "Add"

.. image:: /class2/media/cache_create_local-zone.png

Create a local zone entry according to the following table:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "sorry.example.com"
   "Type", "Static"
   "Records", "sorry.example.com. IN A 10.1.20.252"

.. image:: /class2/media/local_zone.png
   :scale: 50
   
.. admonition:: TMSH

   tmsh modify ltm dns cache validating-resolver validating-resolver_cache local-zones { { name sorry.example.com records add { "sorry.example.com. IN A 10.1.20.252" } type static } }

