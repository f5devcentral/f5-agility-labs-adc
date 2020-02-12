F5 Medium Details
~~~~~~~~~~~~~~~~~

#. Now expand the F5 Medium Details.

   .. image:: /_static/class4/medium-details.png
      :scale: 50 %

#. A majority of the information in the Medium Details will be used by F5 support in analyzing packet flow.  The important parts to notice are the Flow ID, Peer ID, and RST Cause.

#. The Flow ID and Peer ID.  These are unique identifiers for the two flows of the connection.  They are only unique within a specific slot and tmm combination.  A flow ID may be reused, so it is possible to see unrelated packets with the same flow ID in a capture before or after the connection you are interested in.

#. In the display filter type 'f5ethtrailer.rstcause'.  Now Select one of the RST packets and expand the F5 Medium Details.  Notice there is a RST Cause value.  In this case the RST Cause is "RST from BIG-IP internal Linux host".  This indicates that the self-ip on the F5 is not listening on this port.  There are other RST causes.  In the F5 Medium Details you will only see RST Cause for packets that were RST by the F5.

   Also note that if you are performing a packet capture on a VE instance as we are here.  If the interface is not in permiscuous mode you may not get all the data, especially reset causes.  We are not able to get Reset Causes on the virtual servers here but are for the self-ip because of limitations in our Virtualization layer.

   .. image:: /_static/class4/tcp-reset.png
      :scale: 50 % 
