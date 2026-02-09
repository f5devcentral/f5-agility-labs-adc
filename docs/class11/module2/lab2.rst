DNS Profile
#####################################

The DNS profile unlocks all BIG-IP features by making the BIG-IP fully aware of DNS as a protocol. 

Without a DNS profile applied to a listener, the TMOS does not parse DNS requests and load balance UDP/TCP packets. 

Navigate to: **DNS  ››  Delivery : Profiles : DNS**

.. image:: /class2/media/router01_ltm_profile_dns.png

Create a DNS profile as shown in the table below. Check boxes on the right to enable editing and overriding default properties. 

  Note: *AVR sampling in production should be a ratio of queries so as not to overload local database.*

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "example.com_dns_profile"
   "DNS Cache", "Enabled"
   "DNS Cache Name", "transparent_cache"
   "Use BIND Server on Big-IP", "Disabled"
   "Logging", "Enabled"
   "Logging Profile", "example_dns_logging_profile"
   "AVR statistics Sample Rate", "Enabled, 1/1 queries sampled"

.. image:: /class2/media/dns_profile_settings_cache_router01.png
   :scale: 75

.. admonition:: TMSH

   tmsh create ltm profile dns example.com_dns_profile { avr-dnsstat-sample-rate 1 cache transparent_cache defaults-from dns enable-cache yes enable-logging yes log-profile example_dns_logging_profile use-local-bind no }
