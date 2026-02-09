UDP Listener
############

Now that all of our profiles are created, we can create our listeners starting with the UDP listener. 

Navigate to: **DNS  ››  Delivery : Listeners : Listener List**

.. image:: /class2/media/router01_create_virtual_flyout.png


Create a UDP listener according to the tables below:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "udp_53_virtual"
   "Destination Address", "10.1.10.53"
   "Service Port (Advanced Settings)", "DNS 53"
   "VLAN and Tunnel Traffic -> Enabled on..", "external"
   "Address Translation", "Enabled"
   "Protocol", "UDP"
   "Protocol Profile (Client)", "example.com_udp-dns_profile"
   "DNS Profile", "example.com_dns_profile"
   "Default Pool", "dns_pool"

.. image:: /class2/media/udp_listener.png
   :scale: 85


.. admonition:: TMSH

   tmsh create gtm listener udp_53_virtual address 10.1.10.53 port 53 translate-address enabled ip-protocol udp pool dns_pool profiles add { example.com_dns_profile  example.com_udp-dns_profile } vlans add { external } vlans-enabled

