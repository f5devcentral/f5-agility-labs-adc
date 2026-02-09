iRule assignment
############################

Assigned the iRule to the DNS Listeners:

Navigate to: **DNS  ››  Delivery : Listeners : Listener List**

.. image:: /class2/media/listener-assignment-1.png
   :scale: 50

Navigate to the *udp_53_virtual* listener:  

.. image:: /class2/media/mod7lab2-listener.png

Navigate to the iRules section

.. image:: /class2/media/mod7lab2-irule.png

Click Manage button and assign the iRule

.. image:: /class2/media/irule-assignment-5-c.png

.. admonition:: TMSH

   tmsh modify gtm listener all rules { DNS-query-filtering }
