Listeners
############################################

A Listener object is synonyous with a virtual server. In the DNS Delivery interface on a BIG-IP, Listeners are configured to process DNS traffic. 

We will be creating both TCP and UDP based listeners as remember DNS can use both TCP and UDP! 

BIG-IP can be configured for multiple functions from the Listener, starting with simple load balancing, transparent or full caching, along with optional security functions. 

After this module, we will have enabled the BIG-IP to process and cache DNS requests.

.. image:: /class2/media/listeners.png
  :scale: 50%   

.. toctree::
   :maxdepth: 2
   :hidden:
   :glob:

   lab*
