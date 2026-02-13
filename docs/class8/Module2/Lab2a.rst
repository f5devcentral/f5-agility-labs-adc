Lab 1: Getting to Know Your Application Traffic
===============================================

In Lab 1, you will see several options for reviewing the application flows through the BIGIP at the HTTP and TCP layers.  With more details about the traffic, adjustments can be made to improve the speed of the data through the BIGIP.

Lab Tasks:
**********

* Task 1: Review AVR Data
* Task 2: Use iRules to log traffic data
* Task 3: Anything else?

Task 1: Review AVR Data
-----------------------

AVR is already provisioned with custom HTTP and TCP AVR profiles created and assigned to Virtual Servers.  When AVR profiles are initially applied to Virtual Servers, it takes around 5 minutes for the data to appear in the logs.  In the lab, the AVR profiles are already assigned and trafficc is running in the backgound to pre-populate the logs.

#. Connect to TMUI of BIGIP01 using the following credentials:

    User:     admin
    Password: admin.F5demo.com
    
#. From the left menu, select Statistics > Analytics > HTTP > Overview

  .. image:: ../images/avr_http_selection.png

    You will see the AVR overview graphs showing HTTP data flowing through the system.  On the right side of the screen, there are filtering options that we will use later.  By default, the view shows data from the last hour for all Virtual Servers with an attached HTTP AVR profile and refreshes every 5 minutes.  Feel free to change the visible time frame using the dropdown near the top left.  Since the lab systems have been running off for a few weeks you will have data for the Last week, Last month but may not see much for the Last day. 

  .. image:: ../images/avr_http_overview.png

#. Use AVR to see what is happening on the system at the HTTP layer.

    Reset the view to Last hour from the top left dropdown.  Use the Response Code filter on the right side to see what cades are available.  With the lab traffic you will have at least 200, 301, and 404 and response codes available.  Let's use the filters to see what is happening beyond 200 responses.  Click on 301 and the graphs will update showing just the data related to 301 response codes.  Click on URLs above on the right to see what URLs are causing the 301 (redirects).

  .. image:: ../images/avr_http_responses.png

    For lab, 301 responses are a result of redirect.html.  Redirects may be necessary or could be left over from a previous application migration.  You can take these results back to the application team(s) to see if redirects are expected.  It could be something as simple as clients requesting /login instead of /login/ that is causing the redirects.  Unneccesary redirects add time to the client connections and uses resources on the BIGIP and/or application servers.

    Go back to the Resonse Codes filter and click 301 to deselect it, then click on 404.  Look at the URLs filter, it will show the cause of the 404's - file_64kd.txt.  In the lab this cause is simple, the URL should be /file_64k**b**.txt not k**d**.txt.  In a production environment, you may see many other URLs resulting in 404s.  They could be simple typos from the client or from bad links in the applications.  The 404 filter may also show you evidence of someone scanning your application(s).  The point of this is learn how to see what is happening with application traffic.

  .. image:: ../images/avr_http_typo.png


<<Keep for code examples>>
#. **Navigate to**: Network > Trunks > Trunk List, then click the "+" button to create a new Trunk:

   .. image:: ../images/image1.png

#. Provide a Trunk Name, and move the respective Available interface to the "Members" section.

#. Click Repeat to define your next trunk.

   When you define the last trunk, you may select the "Finished" button

   -  Internal Trunk:
   
    .. image:: ../images/image2.png


    .. image:: ../images/image3.png

   -  External Trunk:

     .. image:: ../images/image4.png

   -  HA Trunk:

     .. image:: ../images/image5.png

   -  View of Trunk List after creating all three trunks:

     .. image:: ../images/image6.png


Task 2: Create BIG-IP VLANs
===========================

In Task 2, we will define our VLANs on our BIG-IPs.  Our VLANs will be associated with their respective trunk from Task 1.

#. On both BIG-IP devices, configure VLANs under the Network configuration section.

   Use the following table to create & define your three VLANs:

   +------------+----+-----------+----------+
   |Name        |Tag |Interface  | Tagging  |
   +============+====+===========+==========+
   |int_vlan_10 | 10 |int_trunk  | Untagged |
   +------------+----+-----------+----------+
   |ext_vlan_20 | 20 |ext_trunk  | Untagged |
   +------------+----+-----------+----------+
   |HA_vlan_30  | 30 |HA_trunk   | Untagged |
   +------------+----+-----------+----------+

#. **Navigate to**: Network > VLANs > VLAN List, then click the "+" button to create a new VLAN:

     .. image:: ../images/image7.png

#. Create the respective VLANs per the table above.

   -  Internal VLAN:

     .. image:: ../images/image8.png

     .. image:: ../images/image9.png

   -  External VLAN:

     .. image:: ../images/image10.png

   -  HA VLAN:

     .. image:: ../images/image11.png

   -  View of the VLAN List after all VLANs have been defined, and associated to their respective Trunk:

     .. image:: ../images/image12.png

Task 3: Create BIG-IP Self IPs
==============================

In Task 3, we will configure our Local Self IPs of each BIG-IP.  These IPs will be our L3 connectivity to our BIG-IP networks.

#. On both BIG-IP devices, configure their respective Self IPs under the Network configuration section.

   Use the following table to create & define your three Self IPs:

   .. list-table:: 
      :widths: auto
      :align: center
      :header-rows: 1
   
      * - BIG-IP
        - Name
        - IP address
        - Netmask
        - VLAN
        - Port Lockdown
      * - bigipA
        - self_vlan10
        - 10.1.10.241
        - 255.255.255.0
        - int_vlan_10
        - Allow None (default)
      * - bigipA
        - self_vlan20
        - 10.1.20.241
        - 255.255.255.0
        - ext_vlan_20
        - Allow None (default)
      * - bigipA
        - self_vlan30
        - 10.1.30.241
        - 255.255.255.0
        - HA_vlan_30
        - Allow None (default)
      * - bigipB
        - self_vlan10
        - 10.1.10.242
        - 255.255.255.0
        - int_vlan_10
        - Allow None (default)
      * - bigipB
        - self_vlan20
        - 10.1.20.242
        - 255.255.255.0
        - ext_vlan_20
        - Allow None (default)
      * - bigipB
        - self_vlan30
        - 10.1.30.242
        - 255.255.255.0
        - HA_vlan_30
        - Allow None (default)


#. **Navigate to**: Network > Self IPs, then click the "+" button to create a new Self IP:

     .. image:: ../images/image13.png

#. Create the respective Self IPs per the table above.

   -  Self IP, VLAN 10:

     .. image:: ../images/image14.png

   -  Self IP, VLAN 20:

     .. image:: ../images/image15.png

   -  Self IP, HA VLAN 30:

     .. image:: ../images/image16.png

   -  Example view of the Self IP List from BIG-IP-A after all Self IPs have been defined:

     .. image:: ../images/image17.png


Lab Summary
***********
In this lab, you setup basic BIG-IP network-level configuration settings.  After completion of these lab tasks, you should have network connectivity between the devices on all VLANs.  These configuration objects will assist with the subsequent labs.

This completes Lab 1.