
Lab 2: Fine-grained Policy Enforcement & Bucket Migration
====================================

AI training and fine-tuning workloads generate highly variable request rates. Spikes in requests per second
(RPS) can saturate storage clusters, disrupting other workloads and risking missed SLAs.
At the same time, data migrations are common — moving buckets between clusters or rebalancing capacity.
Migrations need to be surgical and transparent, without requiring client reconfiguration.

**Technical Problem**

- No central control: Clients flood nodes with requests, overwhelming clusters.
- Data migrations require manual endpoint changes or application rewrites.
- Lack of policy enforcement leads to instability and risk during transitions.

**Solution wiht BIG-IP Local Traffic Manager (LTM)**

- **iRules** can be applied to cap connections, control RPS, and enforce thresholds at the dataplane.
- **Local Traffic Policies** redirect traffic based on bucket or host headers.
- **Outcome**: Clusters are stabilized under load, migrations are executed seamlessly, and clients keep using the same VIP.


Task 1. Review the Lab Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These values align with the UDF topology. Keep them unchanged unless your
environment differs.

======================== ========================================= ==================================
Component                Purpose                                   Where to access
======================== ========================================= ==================================
BIG‑IP VIP for Cluster‑1 Single front door for MinIO cluster       WARP parameters: 10.1.40.160:9000
------------------------ ----------------------------------------- ----------------------------------
Cluster‑1 MinIO AIStor   Primary storage cluster                   10.1.10.100-103:9000
------------------------ ----------------------------------------- ----------------------------------
Cluster‑2 MinIO AIStor   Migration target for bucket A             10.1.20.100:9000
------------------------ ----------------------------------------- ----------------------------------
WARP GUI                 Generate high-RPS S3 workloads            UDF → Traffic-Gen → Firefox
------------------------ ----------------------------------------- ----------------------------------
BIG‑IP TMUI              Attach iRules, configure policies         UDF → BIG‑IP → Access → TMUI
======================== ========================================= ==================================
      

Task 2: Rate Limiting S3 Traffic with iRules
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following steps will create a massive spike in sudden S3 activity, and an approach to throttle down a specific source
of the excessive load being received.

+---------------------------------------------------------------------------------------------------------------+
| 1. Open MinIO WARP (UDF → Components → Traffic‑Gen → Access → Firefox).                                       |
|                                                                                                               |
| 2. Set the load target to Endpoint: 10.1.40.160:9000 (BIG-IP VIP for Cluster-1).                              |
|                                                                                                               |
| 3. Duration: 5 minutes, Concurrency 50 threads.                                                               |
|                                                                                                               |
| 4. Click Run Benchmark.                                                                                       |
+---------------------------------------------------------------------------------------------------------------+
| |lab314|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


+---------------------------------------------------------------------------------------------------------------+
| 1. Open BIG-IP TMUI (UDF → Components → BIG-IP 21 → Access → TMUI).  The credentials are under lab            |
|    Documentation tab (admin/bigip123).                                                                        |
|                                                                                                               |
| 2. Select Statistics -> Dashbaord                                                                             |
|                                                                                                               |
| 3. Set Dashboard type pulldown (upper left) to "LTM" and click under View (upper right) to "Minio-Cluster-1"  |
|                                                                                                               |
| 4. Since traffic is already underway, the moment the spike started may not be visible as displayed below.     |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+
| |lab315|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+



Task 3: Apply Rate Limiting iRule
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following steps will demonstrate how one S3 source address may be throttled, allowing only a specific number of 
transactions over time, once a threshold has been first exceeded.


+---------------------------------------------------------------------------------------------------------------+
| 1. In BIG-IP TMUI go to Local Traffic -> iRules -> iRule List                                                 | 
|                                                                                                               |
| 2.  Choose "Show All" in list display control at bottom right of iRule list.                                  |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+
| |lab316|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+


In Lab #1 we created an origin pool that was accessible via the Public Internet.
This lab exercise will create an origin pool that is not accessible via the Public Internet; in this case the application is local to the deployed
CE node in the public cloud. The only way the application can be accessed is via the RE nodes of Distributed Cloud.

First, you will create an Origin Pool that refers to the "Private Endpoint" site in our lab environment.

+------------------------------------------------------------------------------------------------------------+
|| 1. Start in F5 Distributed Cloud Console and switch back to the **Multi-Cloud App Connect** workspace.    |
||                                                                                                           |
|| 2. Navigate the menu to go to **Manage->Load Balancers->Origin Pools**. Click on **Add Origin Pool**.     |
||                                                                                                           |
|| 3. Enter the following variables:                                                                         |
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+
|                                                                                                            |
|   ================================= =======                                                                |
|   *Variable*                        *Value*                                                                |
|   ================================= =======                                                                |
|   Name                              **[NAMESPACE]-private-ce-pool**                                        |
|   ================================= =======                                                                |
|                                                                                                            |
+------------------------------------------------------------------------------------------------------------+
||                                                                                                           |
|| 4. Click on **Add Item** under the **Origin Servers**.                                                    |
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+
|                                                                                                            |
|   ================================= =======                                                                |
|   *Variable*                        *Value*                                                                |
|   ================================= =======                                                                |
|   Select Type of Origin Server      **DNS Name of Origin Server on given Sites**                           |
|   DNS Name                          **private.lab.f5demos.internal**                                       |
|   Site                              **system/appworld-awsnet**                                             |
|   Select Network on the Site        **Inside Network**                                                     |
|   ================================= =======                                                                |
+------------------------------------------------------------------------------------------------------------+   
|                                                                                                            |
| |lab301|                                                                                                   | 
|                                                                                                            |
||                                                                                                           |
|| 5. Click on **Apply** to return to the previous screen.                                                   |
||                                                                                                           |
|| 6. Below the **Origin Servers** section fill in the Origin Server Port information.                       |     
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+
|                                                                                                            |
|                                                                                                            |
|   ================================= ========                                                               |
|   *Variable*                        *Value*                                                                |
|   ================================= ========                                                               |
|   Port                              **8080**                                                               |
|   ================================= ========                                                               |
|                                                                                                            |
+------------------------------------------------------------------------------------------------------------+   
||                                                                                                           |
|| 7. Click **Save and Exit**                                                                                |
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+



Task 2. Create and Deploy a HTTP Load Balancer on F5 Distributed Cloud CE 
---------------------------------------------------------------------------

In the first lab we were connecting to a F5 Distributed Cloud Load Balancer that was deployed in a RE.
Now we will deploy a Load Balancer on the CE node that was deployed in the AWS VPC (CE location).

+-----------------------------------------------------------------------------------------------------------------------------------+
|| 1. Start in F5 Distributed Cloud Console and switch back to the **Multi-Cloud App Connect** context.                             |
||                                                                                                                                  |
|| 2. Navigate the menu to go to **Manage->Load Balancers-> HTTP Load Balancers**.  Click on **Add HTTP Load Balancer**.            |
||                                                                                                                                  |
|| 3. Enter the following variables:                                                                                                |
||                                                                                                                                  |
+-----------------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                                   |
|                                                                                                                                   |
|   ================================= =======                                                                                       |
|   *Variable*                        *Value*                                                                                       |
|   ================================= =======                                                                                       |
|   Name                              **[NAMESPACE]-private-ce-lb**                                                                 |
|   Domains                           **[NAMESPACE]-2.lab-sec.f5demos.com**                                                         |
|   Select type of Load Balancer      **HTTP**                                                                                      |
|   Automatically Manage DNS Records  **No/Unchecked**                                                                              |
|   ================================= =======                                                                                       |
|                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------+                                                                                       
|                                                                                                                                   |
|  |lab311|                                                                                                                         |
||                                                                                                                                  |
|| 4. Under **Origin Pools** Click **Add Item**.                                                                                    |
||                                                                                                                                  |
|  |lab302|                                                                                                                         |
||                                                                                                                                  |
|| 5. Select the recently created **[NAMESPACE]-private-ce-pool** under Origin pool and then click **Apply**.                       |
||                                                                                                                                  |
|  |lab314|                                                                                                                         |
||                                                                                                                                  |
|| 6. Now you can see your Origin Pool has been added to the HTTP load balancer configuration.                                      |
||                                                                                                                                  |
|  |lab304|                                                                                                                         |
||                                                                                                                                  |
|| 7. Now we want to confirm how this Load Balancer is advertised, we will select the **Other Settings** on the left hand side.     |
||    This will auto-scroll the configuation towards the bottom of the Load Balancer configuration section labeled                  |
||    **Other Settings**.                                                                                                           |
||                                                                                                                                  |
|| 8. Under **VIP Advertisement** Confirm it is set to **Internet**. This will allow *XC* to advertise the Virtual IP address.      |
||                                                                                                                                  |
|| |lab305|                                                                                                                         |
||                                                                                                                                  |
||                                                                                                                                  |
|                                                                                                                                   |       
+-----------------------------------------------------------------------------------------------------------------------------------+

Task 3: Configure WAF Policy
----------------------------
Now that we have our load balancer and orign server configured we want to make sure we are protecting the origin server.  Here you
can easily applying a pre-existing shared WAF policy to our load balancer.  The shared WAF policy is available for all namespaces
under this tenant.

+-----------------------------------------------------------------------------------------------------------------------------------+
|| 1. Under the **Web Application Firewall** section.                                                                               |
||                                                                                                                                  |
|| 2. Choose the following options:                                                                                                 |
||                                                                                                                                  | 
+-----------------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                                   |
|   =============================== =================================                                                               |
|   *Variable*                      *Value*                                                                                         |
|   =============================== =================================                                                               |
|   Web Application Firewall (WAF)  **Enable**                                                                                      |
|   Select App Firewall             **shared/base-appfw**                                                                           |
|   =============================== =================================                                                               |
|                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------+
||                                                                                                                                  |
|| 3.  Scroll to the botton of the screen and click **Save and Exit** to create the HTTP Load Balancer.                             |
||                                                                                                                                  |
||                                                                                                                                  |
+-----------------------------------------------------------------------------------------------------------------------------------+

Task 4: Verify Configuration
-----------------------------

You should now be able to go to the DNS name that you created in this Load Balancer configuration.  
The FQDN we used in our example is http://[NAMESPACE]-2.lab-sec.f5demos.com/.  

+-----------------------------------------------------------------------------------------------------------------------------------+
| .. note::                                                                                                                         |
|    *It can take a minute or so before the configuration is pushed to the CE node, appworld-awsnet.  If the verify does not*       |
|    *work on first attempt, please try again after waiting for a minute.*                                                          |
|                                                                                                                                   |
||  The private demo app should look like the following:                                                                            |
||                                                                                                                                  |
|  |lab312|                                                                                                                         |
|                                                                                                                                   |
|                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------+


Task 5: Verify WAF Protection
------------------------------

In this topology we are sending traffic to the application via the public IP advertised on *XC*, which then sends to the application
in AWS via the connection to the CE node in AWS.

+-----------------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                                   |
| Using some of the sample attacks below, add the URI path & variables to your application to generate                              |
| security event data.                                                                                                              |
|                                                                                                                                   |
|    * /?cmd=cat%20/etc/passwd                                                                                                      |
|    * /product?id=4%20OR%201=1                                                                                                     |
|    * /cart?search=aaa'><script>prompt('Please+enter+your+password');</script>                                                     |
|                                                                                                                                   |
| Just like in Lab 1, you should see a block page when adding the attacks to the URL.                                               |
|                                                                                                                                   |
| .. note::                                                                                                                         |
|    *Try using the AI Assistant to get a detailed explanation on the attacks that are blocked.  You can view that by selecting*    |
|    *the event in the Security Dashboard, then selecting "Explain with AI" for the event, which is located under "Actions".*       |
|                                                                                                                                   |
|   |lab313|                                                                                                                        |                                      
|                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------------------------------------------------+
| **End of Lab 2**.  In this lab you configured a global load balancer with a WAF policy on a CE node running in AWS for a          |
| private end point. That private end point was only accessible via the global load balancer.                                       |
+-----------------------------------------------------------------------------------------------------------------------------------+
|  |labend|                                                                                                                         |
+-----------------------------------------------------------------------------------------------------------------------------------+

.. |lab300| image:: _static/lab3-appworld2025-topology-diagram.png
   :width: 800px
.. |lab301| image:: _static/lab3-appworld2025-task1-originserverr.png
   :width: 800px
.. |lab302| image:: _static/lab3-appworld2025-task2-lb-add-origin-pool.png
   :width: 800px
.. |lab303| image:: _static/lab3-appworld2025-task2-lb-add-origin-pool2.png
   :width: 800px
.. |lab304| image:: _static/lab3-appworld2025-task2-lb-origin-pool-added.png
   :width: 800px
.. |lab305| image:: _static/lab3-appworld2025-task2-lb-other-settings.png
   :width: 800px
.. |lab306| image:: _static/lab3-appworld2025-task2-lb-change-vip-advertisement.png
   :width: 800px
.. |lab307| image:: _static/lab3-appworld2025-list-sites-advertise.png
   :width: 800px
.. |lab308| image:: _static/lab3-appworld2025-task2-lb-site-change.png
   :width: 800px
.. |lab309| image:: _static/screenshot-global-vip-private.png
   :width: 800px
.. |lab310| image:: _static/lab3-appworld2025-waf-block-message.png
   :width: 800px
.. |lab311| image:: _static/lab2-appworld2025-task2-lb.png
   :width: 800px 
.. |lab312| image:: _static/screenshot-global-vip-private.png
   :width: 800px 
.. |lab313| image:: _static/lab3-appworld2025-waf-block-message.png
   :width: 800px 
.. |lab314| image:: _static/b_warp_parameters_lab2.png
   :width: 800px
.. |lab315| image:: _static/b_dashboard_no_irule.png
   :width: 800px
.. |labend| image:: _static/labend.png
   :width: 800px
