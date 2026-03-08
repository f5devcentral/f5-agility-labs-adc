Lab 5: Device Service Clusters (DSC) 
====================================

By now you have a working HTTP virtual server and pool. Let's take a moment and review what we have learned about the traffic flow through the BIG-IP. 

We have a client that is sending HTTP requests to the BIG-IP, which then forwards those requests to the pool members. The pool members respond with HTTP responses, which are sent back to the client through the BIG-IP.

Standard Virtual Server Packet Flow
-----------------------------------

On a standard virtual server application would follow something similar to the flow below:

	1. An IPv4 or IPv6 virtual server receive the connection request​
	2. The TCP connection between the BIG-IP and the client is set up​

      I. at this point we can adjust how TCP works between the BIG-IP and the client using the client TCP profile and its parameters​
      II. or log or manipulate the connection using iRules​
	3. The virtual server establish and encrypted session between the BIG-IP and the client.​
	
      I. Again we can manipulate the SSL setup via iRules​
	4. The decrypted data hits the HTTP profile and the HTTP information is parsed and processed according to the profile parameters, and iRules could be applied​
	5. And so on, until we make the load balancing decision in TMOS​
	6. Manipulate the HTTP stream as required​
	7. Re-encrypt the stream if desired ​
	8. At that point we create a new TCP connection based on the server-side TCP profile to the selected server​
	9. And send the connection to the IPv4 or IPv6 pool member​

Now that we have a working virtual server and pool, let's take a look at how we can make our BIG-IP highly available. 

In this lab we will be creating a **high availability cluster** using two BIG-IP devices. We will configure device trust, device groups, and traffic groups to ensure that if one BIG-IP fails, the other can take over and continue to serve traffic.

This lab is designed to help you understand Device and Traffic Groups, as well as the process of building an Active-Standby HA pair. While there is a wizard, we will configuring this manually. 

Base Networking and HA VLAN
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will be creating a high availability cluster using the second BIG-IP
(**bigip02)** in your lab , so let’s prep our current BIG-IP and we will
be creating a high availability VLAN.

1. On **bigip01.f5demo.com** archive your configuration in case you need
   to revert.

a. Go to **System > Archives** and create a new archive.

b. You will be using your third interface (1.3) for Network Failover and
   ConfigSync. This requires certain ports to be open on the Self IP;
   TCP port 4353 for ConfigSync and TCP port 1026 for Network Failover
   and TCP port 6699 for the Master Control Program.

   i.  Build a new untagged VLAN **ha_vlan** on interface **1.3**

   ii. Add a self-IP name **ha_ip** and address **10.1.30.245** net mask
       **255.255.255.0.**

       1. Under **Port Lockdown**, select **Allow Default**, to open
          ports required for HA communications.

          a. Optionally you could select: **Allow Custom** and add TCP
             ports 4353,1026 and 6699

1. Go to **bigip02.f5demo.com** and log into GUI with the credentials provided in Documentation section.

   a. **bigip02** has already been licensed and provisioned. You will
      need to set up the base networking. 


   +----------------+----------------+----------------+-----------------+-----------------+
   | **Interface**  | **Untagged     | **IP Name**    | **Self IP**     | **Netmask**     |
   |                | VLAN**         |                |                 |                 |
   +================+================+================+=================+=================+
   | 1.1            | client_vlan    | client_ip      | 10.1.10.246     | 255.255.255.0   |
   +----------------+----------------+----------------+-----------------+-----------------+
   | 1.2            | server_vlan    | server_ip      | 10.1.20.246     | 255.255.255.0   |
   +----------------+----------------+----------------+-----------------+-----------------+
   | 1.3            | ha_vlan        | ha_ip          | 10.1.30.246     | 255.255.255.0   |
   +----------------+----------------+----------------+-----------------+-----------------+

b. On the **ha_vlan** ip configure set **Port Lockdown** to **Allow
   Default**

c. Build the default gateway def_gw in Routes with destination **0.0.0.0**, mask **0.0.0.0**,
   gateway ip address **10.1.10.1**

d. What is the status your BIG-IPs? Check the upper left-hand corner
   next to the F5 ball.

Configure HA
~~~~~~~~~~~~

1. **On each BIG-IP**, prior to building the Device Trust it is
   recommended renewing the BIG-IP self-signed certificate with valid
   information and re-generating the local Device Trust certificate.

   a. Under **System > Certificate Management > Device Certificate Management** select the
      **Renew…** button

      i.   **Common Name**: <the Hostname of the BIG-IP in the upper left
           corner>

      ii.  **Country**: United States (or your country of preference)

      iii. **Lifetime**: 3650

         Lifetime is important, if your cert expires your HA setup
         will fail.

      iv.  Select **Finished**. Your browser will ask to exchange certs
        with the BIG-IP again, and refresh the page.

   b. Under **Device Management > Device Trust > Local Domain** select
      **Reset Device Trust…**

      In the **Certificate Signing Authority** select **Generate New
      Self-Signed Authority** and hit **Update**.

2. **On each BIG-IP** configure the device object failover parameters
      the BIG-IP will send to other BIG-IPs that want to be a part of a
      sync-only or sync-failover group.

      Click **Device Management > Device** and select the local BIG-IP. It will have the self suffix. 

         i.  On the top bar select **ConfigSync**.

            Use the Self IP address of the HA VLAN for your **Local Address**.

            Hit **Update**.

         ii. On the top bar select **Failover Network**.

            In the **Failover Unicast Configuration** section select the **Add** button.

            Use the Self IP address the HA VLAN for your **Address**.

            Leave the **Port** at the default setting of 1026.

            **Note:** Multicast is for Viprion chasses only.

            Select **Finished**.

         iii. On the top bar select **Mirroring**.

            **Primary Local Mirror Address**: use the Self IP address of the HA VLAN.

            **Secondary Local Mirror Address:** None.

            Select **Update**.

3. On **bigip01.f5demo.com** build the Device Trust.

   a. Under **Device Management > Device Trust > Device Trust Members** and
      select **Add** to add other BIG-IP(s) you will trust.

      i.   **Device IP Address**: <management IP address of the BIG-IP
           to add>

           1. You could use any Self IP if the out-of-band management
              interface is not configured.

      ii.  Enter the Administrator Username and Password of the BIG-IP
           you are trusting.

      iii. Select **Retrieve Device Information**

           1. The certificate information and name from the other BIG-IP
              should appear

           2. Select **Device Certificate Matches**

      iv. Select **Add Device**. Now you should see the other BIG-IP in the list of trusted devices.

      Check on the other BIG-IP bigip02 in the **Peer and Subordinate Devices** list to verify that bigip01 is trusted. 


      .. image:: ../images/image66.png

         Is all the information there?

       v. Go to **bigip02.f5demo.com** and check the **Device Management > Device Trust > Peer and Subordinate Devices** list. 
      
         Do you see the first BIG-IP in the list?

 
      .. image:: ../images/image67.png

   b. If some information is missing delete the trust and try again.

4. What are the statuses of your BIG-IPs now?

    a. They should be **In Sync**. But wait! We haven’t even created a
       device group! But remember the **Device Trust** creates a
       **Sync-Only** group for the certificates under the covers
       (*device-trust-group*) and that should be in sync.

    b. Click on **In Sync** in the upper right corner or **Device
       Management > Overview** to see the **device_trust_group**.

5. On **bigip01.f5demo.com** create a new **Sync-Failover** device group

   a. **Under Device Management > Device Groups** create a new device
      group.

      i.    **Name:** my-device-group

      ii.   **Group Type**: Sync-Failover

      iii.  Add the members of the group to the **Includes** box and
            select **Finished**.

      iv.   Check **Device Groups** on bigip02 BIG-IP to verify that the new device group is there.

      v.    Did you have to create the Device Group on the other BIG-IP?

            1. Is the full configuration synchronized yet? (No! Only the
               Device Group is sync’d)

      vi.   What is your sync status?

            1. It should be **Awaiting Initial Sync**

      vii.  Click on the sync status or go to **Device
            Management > Overview** (or click on **Awaiting Initial**
            Sync) of the BIG-IP with the **good/current** configuration.

      viii. Click the device with the configuration you want to
            synchronize. **Sync Options** should appear.

      ix.   **Push the selected device configuration to the group** should be selected by default so click on **Sync**. It
            could take up to 30 seconds for synchronization to complete.

            1. What are the statuses of your BIG-IPs? Do you have an
               active-standby pair?

            2. Are the configurations the same?

6. Now that you have created your HA environment. HA selections will
   show up for SNAT addressed (not tied to your base network),
   persistence profiles and connection mirroring on virtual servers.

   a. Go to your **Active** BIG-IP.

   b. Go to your persistence profile **my-src-persistence** and check
      the **Mirror Persistence** box and Update

   c. Go to your **www_vs** virtual server and set the **Default
      Persistence Profile** to **my-src-persistence.**

   d. Your Active BIG-IP is now in **Changes Pending** state. Click on the status and **Sync** your changes. Did the changes sync?

   e. On each BIG-IP go to **Module Statistics > Local Traffic** and
      bring up the persistence record statistics.

      i.  Log back into the RDP session of your Ubuntu Jumpbox, go to the home page of you www_vs web service
          (`http://10.1.10.100 <http://10.128.10.100>`__). Refresh a few
          times.

      ii. Check the persistence records on each of your BIG-IPs, you
          should see the records are mirrored on each device.

7. On your Active BIG-IP, go to **Device Management > Traffic Groups**. As you can see the
   default traffic group “\ **traffic-group-1**\ ” already exists.

   a. Select **traffic-group-1**, check out the page information and
      then select **Force to Standby**.

   b. What are the statuses of your BIG-IPs? Go to your web page. What
      is the client IP?

   c. Go to your self-IP addresses. What traffic group are they in? What
      does it mean?

   d. Archive your work.