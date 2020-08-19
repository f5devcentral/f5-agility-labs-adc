F5 High Details
~~~~~~~~~~~~~~~

#. Now Expand the F5 High Details on one of the SSL sessions data packets.  (You can type in a display filter of '(f5ethtrailer.peeraddr) && (tls.record.content_type == 23)' to retrieve the session we are looking for). Make sure you select a packet that has Application Data in the Info field.

#. The F5 High Details contains a lot of valuable information.  In the example above we see that the Peer IP Protocol is 6 or TCP.

#. The Peer VLAN is going to be the VLAN that the traffic enters the F5 from.  In the example this is VLAN 4093 which is the internal VLAN.

#. The Peer remote address for an Outbound packet will be the Application server IP address, if it were Inbound the address would be the Client Address.  The Peer Remote address will always be an address that is not on the BIG-IP directly, in this case 10.1.10.31. The second Peer remote address is the IPv6 version of the address.

#. The Peer local address is the SNAT address for this particular Virtual Server, in this case 10.1.10.10.  Once again the IPv6 version is below that.

#. Next we see the Peer remote port.  The remote port is the port that the traffic is coming from on the remote side.  In this case the client is requesting to port 443.

#. Next we have the Peer local port.  This is the port the F5 is talking with the remote side on.  In this case it is port 50228.  If this were a request coming from the client, the local port would be 443 as the Virtual Server is configured with port 443 and the remote port would be a port > 1024.

#. The F5 High Details provide great information on what pool member is selected on a particular conversation that you are tracking down.  This makes it possible to determine which server may be having issues in an application problem.

High Details and Other Field Dissectors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. In order to see how the other field dissectors becomes useful, enter a display filter of 'ip.addr==10.1.10.31'.

#. Notice that some of the Source and Destination fields in the results do not contain the ip address of 10.1.10.31.  Select one of the packets that does not have 10.1.10.31 as the source or destination like below.

#. Is the display filter broken?

#. No it is not.  Notice in the F5 High Details that the Peer remote address is 10.1.10.31.  This is the pool member selected for the pool in the virtual server.

#. With the other field dissectors value turned on in Wireshark, you can set a display filter of the pool member you may be troubleshooting and see which virtual is being selected on the traffic.  Or you can set a display filter of the client and see all the traffic to the pool members even with a SNAT pool selected.
