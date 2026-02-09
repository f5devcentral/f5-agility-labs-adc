Validating Resolver
##########################################

.. toctree::
   :hidden:
   :maxdepth: 2
   :glob:

   lab*

In this lab we will use the BIG-IP as a Validating resolver and not send any queries to the back end server. 

The *validating* function of the resolver means that recusive queries are sent requesting DNSSEC, and responses are validated to authenticate validity of  the response!

First lets create a new DNS cache on the BIG-IP:

Navigate to **DNS  ››  Caches : Cache List**

.. image:: /class2/media/cache_list_flyout.png
   :scale: 50

Create a validating resolver cache according to the table below:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "validating-resolver_cache"
   "Resolver Type", "Validating Resolver"
   "Answer default zones", "Checked - Enabled"

.. image:: /class2/media/cache_validating-resolver.png
   :scale: 50

.. admonition:: TMSH

   tmsh create ltm dns cache validating-resolver validating-resolver_cache answer-default-zones yes

