Walled Garden
#####################################

Navigate to: **DNS  ››  Caches : Cache List**

Click "validating-resolver_cache"

.. image:: /class2/media/select_validating-resolver_cache.png

Select validating-resolver_cache, click "Response Policy Zones", and then click "Add"

.. image:: /class2/media/cache_enable_rpz.png
   :scale: 50

Create a response policy zone entry according to the following table:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Zone", "rpz.example.com"
   "Action", "Walled Garden"
   "Walled Garden", "sorry.example.com"

.. image:: /class2/media/cache_enable_rpz_details.png

.. admonition:: TMSH

   tmsh modify ltm dns cache resolver validating-resolver_cache response-policy-zones add { rpz.example.com { action walled-garden walled-garden sorry.example.com } }

