TCP Listeners
####################################

Next, create the TCP listener 

Navigate to: **DNS  ››  Delivery : Listeners : Listener List**

.. image:: /class2/media/router01_create_virtual_flyout.png

Create two TCP listeners according to the table below:

**Pro-tip: You can use the 'Repeat' button to easily create the second virtual server**

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "tcp_53_virtual"
   "Destination Address", "10.1.10.53"
   "Service Port (Advanced Settings)", "DNS 53"
   "VLAN and Tunnel Traffic -> Enabled on..", "external"
   "Address Translation", "Enabled"
   "Protocol", "TCP"
   "Protocol Profile (Client)", "example.com_tcp-dns_profile"
   "DNS Profile", "example.com_dns_profile"
   "Pool", "dns_pool"
   
.. image:: /class2/media/tcp_listener_profile.png
   :scale: 75

.. admonition:: TMSH

   tmsh create gtm listener tcp_53_virtual address 10.1.10.53 port 53 translate-address enabled ip-protocol tcp pool dns_pool profiles add { example.com_dns_profile  example.com_tcp-dns_profile } vlans add { external } vlans-enabled
