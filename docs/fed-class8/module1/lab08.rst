F5 Medium Details
~~~~~~~~~~~~~~~~~

Note: There may not be any matching packets with a Reset Cause.

#. Now expand the F5 Medium Details.

#. A majority of the information in the Medium Details will be used by F5 support in analyzing packet flow.  The important parts to notice are the Flow ID, Peer ID, and RST Cause.

#. The Flow ID and Peer ID.  These are unique identifiers for the two flows of the connection.  They are only unique within a specific slot and tmm combination.  A flow ID may be reused, so it is possible to see unrelated packets with the same flow ID in a capture before or after the connection you are interested in.

#. In the display filter type 'f5ethtrailer.rstcause'.  Now Select one of the RST packets and expand the F5 Medium Details.  Notice there is a RST Cause value.  In this case the RST Cause is `No Local Listener`.  This indicates that there is no Virtual Server listening on that IP:Port combination.  There are other RST causes.  In the F5 Medium Details you will only see RST Cause for packets that were RST by the F5.
