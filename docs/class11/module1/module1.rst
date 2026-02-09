Transparent Cache
#####################################

.. toctree::
   :hidden:
   :maxdepth: 2
   :glob:

   lab*

In this module we will implement all the configuration objects required for a transparent DNS cache on the BIG-IP.

Enabling a transparent cache offloads the back end DNS servers from responding to every query which frees resources on the servers. 

.. image:: /class2/media/transparent_cache.png

Log into *BIG-IP DNS* using either the TMUI or webshell interface with credentials u:**admin** p:**agility2021**

Navigate to **DNS  ››  Caches ›› Cache List**

then click the *Create* button to create a transparent cache with the following settings: 

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "transparent_cache"
   "Resolver Type", "Transparent"

.. image:: /class2/media/cache_transparent.png
  :scale: 50

.. admonition:: TMSH

   tmsh create ltm dns cache transparent transparent_cache

