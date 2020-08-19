F5 Low Details
~~~~~~~~~~~~~~

#. Apply a new display filter of 'tcp.port == 80' to the capture.

|image3|

#. Select the packet with the GET request in the info field.

#. In the middle section Expand F5 Ethernet Trailer.

#. Then expand F5 Low Details.

#. Notice the Ingress value is True (IN).  This is from the perspective of the F5.  The traffic is inbound to the F5.

#. The low details also gives the Slot value (always be 1 for an appliance).  The TMM number in the image is 3.

#. The most important value here is the VIP.  In this case it is `/Sample_04/A1/serviceMain-Redirect-`.  Notice this is the port 80 VIP for this particular destination IP.  The VIP is configured with a redirect to SSL.

#. The next packet in the capture is a HTTP 302 redirect to the SSL vip.

.. |image3| image:: images/image3.PNG
   :width: 6.32107in
   :height: 4.33645in
