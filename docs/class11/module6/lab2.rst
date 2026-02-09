Name Server
##############################

Navigate to **DNS  ››  Delivery : Nameservers : Nameserver List**

.. image:: /class2/media/create_nameserver_flyout.png

Create a nameserver according to the following table:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "localhost"
   "Address","127.0.0.1"

.. image:: /class2/media/create_nameserver_localhost.png

.. admonition:: TMSH

   tmsh create ltm dns nameserver localhost { address 127.0.0.1 tsig-key none }
