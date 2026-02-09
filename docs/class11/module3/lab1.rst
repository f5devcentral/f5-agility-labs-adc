Name Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First, we define the Ubuntu server as a *nameserver* and initiate a zone transfer.

Navigate to **DNS  ››  Delivery : Nameservers : Nameserver List**

.. image:: /class2/media/create_nameserver_flyout.png

Create a nameserver according to the following table:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   "Name", "master.example.com"
   "Address", "10.1.20.4"

.. image:: /class2/media/nameserver.png
   :scale: 90
   

.. admonition:: TMSH

   tmsh create ltm dns nameserver master.example.com { address 10.1.20.4 }
