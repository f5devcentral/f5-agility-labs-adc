TCP Profile
###################################

Like the UDP profile, a TCP profile controls properties of TCP connections on the BIG-IP. 

Navigate to: **DNS  ››  Delivery : Profiles : Protocol : TCP**

.. image:: /class2/media/router01_create_tcp_profile.png


Create a TCP profile as shown in the following table.
 
.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "example.com_tcp-dns_profile"
   "Parent Profile", "f5-tcp-lan"

.. image:: /class2/media/router01_create_tcp_profile_properties.png
   :scale: 50

.. admonition:: TMSH

   tmsh create ltm profile tcp example.com_tcp-dns_profile defaults-from f5-tcp-lan
