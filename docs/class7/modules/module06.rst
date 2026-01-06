Lab 6: Device Service Clusters (DSC) 
====================================

This lab is designed to help you understand Device and Traffic Groups, as well as the process of building an Active-Standby HA pair. While there is a wizard, we will configuring this manually. 

Base Networking and HA VLAN
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will be creating a high availability cluster using the second BIG-IP
(**bigip2)** in your lab , so let’s prep our current BIG-IP and we will
be creating a high availability VLAN.

1. On **bigip01.f5demo.com** archive your configuration in case you need
   to revert.

a. Go to **System > Archives** and create a new archive.

b. You will be using your third interface (1.3) for Network Failover and
   ConfigSync. This requires certain ports to be open on the Self IP;
   TCP port 4353 for ConfigSync and TCP port 1026 for Network Failover
   and TCP port 6699 for the Master Control Program.

   i.  Build a new untagged VLAN **ha_vlan** on interface **1.3**

   ii. Add a self-IP address to the VLAN, **10.1.30.245** net mask
       **255.255.255.0.**

       1. Under **Port Lockdown**, select **Allow Default**, to open
          ports required for HA communications.

          a. Optionally you could select: **Allow Custom** and add TCP
             ports 4353,1026 and 6699

1. Go to **https://10.1.1.246** which is **bigip02.f5demo.com** and
   login with the credentials **admin/f5UDFrocks!**.

   a. **bigip02** has already been licensed and provisioned. You will
      need to set up the base networking. 


+----------------+----------------+-----------------+-----------------+
| **Interface**  | **Untagged     | **Self IP**     | **Netmask**     |
|                | VLAN**         |                 |                 |
+================+================+=================+=================+
| 1.1            | client_vlan    | 10.1.10.246     | 255.255.255.0   |
+----------------+----------------+-----------------+-----------------+
| 1.2            | server_vlan    | 10.1.20.246     | 255.255.255.0   |
+----------------+----------------+-----------------+-----------------+
| 1.3            | ha_vlan        | 10.1.30.246     | 255.255.255.0   |
+----------------+----------------+-----------------+-----------------+

b. On the **ha_vlan** ip configure set **Port Lockdown** to **Allow
   Default**

c. Build the default gateway destination **0.0.0.0**, mask **0.0.0.0**,
   gateway ip address **10.1.10.1**

d. What is the status your BIG-IPs? Check the upper left-hand corner
   next to the F5 ball.

Configure HA
~~~~~~~~~~~~

1. **On each BIG-IP**, prior to building the Device Trust it is
   recommended renewing the BIG-IP self-signed certificate with valid
   information and re-generating the local Device Trust certificate.

a. Under **System > Device Certificate > Device Certificate** select the
   **Renew…** button

   i.   **Common Name**: <the Hostname of the BIG-IP in the upper left
        corner>

   ii.  **Country**: United States (or your country of preference)

   iii. **Lifetime**: 3650

        1. Lifetime is important, if your cert expires your HA setup
           will fail.

   iv.  Select **Finished**. Your browser will ask to exchange certs
        with the BIG-IP again.

b. Under **Device Management > Device Trust > Local Domain** select
   **Reset Device Trust…**

   i. In the **Certificate Signing Authority** select **Generate New
   Self-Signed Authority** and hit **Update**.

   ii. **On each BIG-IP** configure the device object failover parameters
   the BIG-IP will send to other BIG-IPs that want to be a part of a
   sync-only or sync-failover group.

c. Click **Device Management > Device** and select the local BIG-IP. It will have the self suffix. 

      i.  Under **Device Connectivity** on the top bar select:

          1. **ConfigSync**

          2. Use the Self IP address of the HA VLAN for your **Local Address**.

          3. Hit **Update**

      ii. **Network Failover**

          1. In the **Failover Unicast Configuration** section select
             the **Add** button

          2. Use the Self IP address the HA VLAN for your **Address**

          3. Leave the **Port** at the default setting of 1026

          4. **Note:** Multicast is for Viprion chasses only.

          5. Select **Finished**

      iii. **Mirroring**

         1. **Primary Local Mirror Address**: use the Self IP address of
            the HA VLAN for your

         2. **Secondary Local Mirror Address:** None

         3. Select **Update**

2. On **bigip01.f5demo.com** build the Device Trust.

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

      iv.  Select **Add Device**.

           1. On each BIG-IP check the other BIG-IP in the **Peer
              Authorities** list. **Is all the information there?**

..

   .. image:: ../images/image66.png
      :width: 4.5974in
      :height: 0.53243in

3. If some information is missing delete the trust and try again.

..

   .. image:: ../images/image67.png
      :width: 4.51948in
      :height: 0.50506in

4. What are the statuses of your BIG-IPs now?

    a. They should be **In Sync**. But wait! We haven’t even created a
       device group! But remember the **Device Trust** creates a
       **Sync-Only** group for the certificates under the covers
       (*device-trust-group*) and that should be in sync.

    b. Click on **In Sync** in the upper right corner or **Device
       Management > Overview** to see the **device_trust_group**.

5. On bigip01.f5demo.com create a new **Sync-Failover** device group

   a. **Under Device Management > Device Groups** create a new device
      group.

      i.    **Name:** my-device-group

      ii.   **Group Type**: Sync-Failover

      iii.  Add the members of the group to the **Includes** box and
            select **Finished**.

      iv.   Check **Device Groups** on each BIG-IP.

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

      ix.   **Push the selected device configuration to the group**. It
            could take up to 30 seconds for synchronization to complete.

            1. What are the statuses of your BIG-IPs? Do you have an
               active-standby pair?

            2. Are the configurations the same?

6. Now that you have created your HA environment. HA selections will
   show up for SNAT addressed (not tied to your base network),
   persistence profiles and connection mirroring on virtual servers.

   a. Go to your **Active** BIG-IP.

   b. Go to your persistence profile **my-src-persistence** and check
      the **Mirror Persistence** box.

   c. Go to your **www_vs** virtual server and set the **Default
      Persistence Profile** to **my-src-persistence.**

   d. Synchronize your changes. Did the changes sync?

   e. On each BIG-IP go to **Module Statistics > Local Traffic** and
      bring up the persistence record statistics.

      i.  Go to the home page of you www_vs web service
          (`http://10.1.10.100 <http://10.128.10.100>`__). Refresh a few
          times.

      ii. Check the persistence records on each of your BIG-IPs, you
          should see the records are mirrored on each device.

7. Go to **Device Management > Traffic Groups**. As you can see the
   default traffic group “\ **traffic-group-1**\ ” already exists.

   a. Select **traffic-group-1**, check out the page information and
      then select **Force to Standby**.

   b. What are the statuses of your BIG-IPs? Go to your web page. What
      is the client IP?

   c. Go to your self-IP addresses. What traffic group are they in? What
      does it mean?

   d. Archive your work.