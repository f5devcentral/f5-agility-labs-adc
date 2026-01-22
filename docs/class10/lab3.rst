
Lab 3: Cluster Health Monitoring with Read/Write Quorum
====================================

AI data pipelines depend on **consistent, correct S3 access**. Storage failures can lead to partial writes,
corrupted data, or stalled training jobs. Simply passing traffic to unhealthy nodes risks **pipeline disruption**
and **data loss**.

**Technical Problem**

- Without quorum checks, clients may send writes when a cluster cannot maintain consistency.

- Failures cascade into downstream AI jobs.

- Operators need automated controls to switch between **read/write** states, ensuring safe traffic handling.

**Solution with BIG-IP LTM**

BIG-IP integrates with MinIO health endpoints to monitor quorum readiness. With custom monitors, it can:

- **Block writes** when quorum is lost.
- **Shift traffic** to a read-only pool to keep dashboards and queries alive.
- **Automatically restore writes** once quorum returns.
- **Outcome**: AI workloads remain consistent and responsive even under node failures.

Task 1. Validate healthy write quorum
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In **BIG-IP TMUI**:

- Navigate to Local **Traffic → Pools → cluster1-write-quorum**.
- Confirm all 4 members are **green**.

|lab400|


**Expectation:** Pool entirely healthy; write quorum is satisfied.

Review the Monitors, under Local Traffic, where you will see one for read and one for write quorum.
Open the minio-health-check to see the configuration of the monitor.

|lab401|

A custom monitor allows one to specify URL to send requests to and custom strings on which to check the
validity of the response. This monitor just checks for the HTTP return code, 200 Okay suggest server health.

- Using HTTP "HEAD" as opposed to "GET" lowers network impact as only the HTTP response code is sent, no content is delivered
- F5 provides more advanced monitors, Extended Application Verfication (EAV), allowing more advanced actions such as using an S3
  access token/secret to upload a small object, such as the current UNIX timestamp, and immediately downloading the object.
- In this lab, the monitors are already applied to their respective pools.

Task 2.  Run baseline workload (repesenting typical read/write load)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open the MinIO Warp bench tool (**UDF -> Components -> Traffic-Gen -> Access -> Firefox**)

- Select the target: **BigIP-cluster-1 (healthcheck + quorum)**
- Select **all** three buckets.
- Put sliders on **Duration** of 10 mins and **Concurrency** at 20 threads
- Make sure that the IP address in WARP Parameters is a new BIG-IP virtual server at **10.1.40.162:9000**

|lab402|

Click the **Run Benchmark** button to start a long, ten minutes of high rate S3 load.

|lab403|

We observe all members of the pool cluster-1-write-quorum are getting close to the same number of total HTTP (S3) requests.

Task 3.  Disable one node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In UDF, open **UDF -> Components -> Jump Host → Access → Web Shell** (be careful not to inadvertently use WIN-JUMP-HOST).

- Check the active user: #whoami
- If it returns **root**, switch to user ubuntu: #su - ubuntu

|lab404|

- Change to /home/ubuntu/minio directory
- run the ansible playbook $ansible-playbook cluster1-stop-one-node.yml

|lab405|





In the BIG-IP Pool being used, called cluster1-write-quorum, click on the **Members / Statistics** tab and observe 1 marked **red**, this is the expected behavor.

|lab406|

Note that the red, down node in the above screenshot has no current TCP connections terminating upon it.

**Expectation:**  Write quorum still exists, as three nodes remain up, only one node is down.

Task 4.  Disable a second node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the JumpHost shell, issue the following Ansible command to take down a second node in the pool.

|lab407|

Ansible takes out another node. However, because of the **write quorum** health check in the monitor, which uses a specific endpoint,
the entire pool will be taken out because the healthcheck no longer returns 200 OK and therefore write write quorum isn't achieved.

BIG-IP marks the entire pool as **red**.

|lab408|

Notice that all nodes are down, however a few TCP connections remain active.  No new S3 traffic will be proxied to these nodes by the corresponding
virtual server, however existing transactions may run to completion.  Within seconds all nodes will display zero active connections.


.. image:: _static/lab3-appworld2025-topology-diagram.png

F5 Distributed Cloud AWS VPC Site
---------------------------------

In addition to protecting resources using F5 Distributed Cloud WAF/WAAP enforcement at an F5 Regional Edge (RE), you can also deploy a
Customer Edge (CE) that may or may not be exposed to the public Internet. CE nodes may be deployed in physical data centers and/or public 
cloud environments.

Once a CE has been deployed, it unlocks two additional topologies.

**1. Client -> RE -> CE -> Protected resource**

Leveraging F5 Distributed Cloud REs to provide WAF and other services upstream, then proxying the clean traffic to the protected resource via the CE. It is recommended that a firewall rule be placed at the site with the CE to only allow traffic from an RE. This ensures that all traffic is scrubbed upstream before entering the site.

**2. Client -> CE -> Protected resource**

In this scenario, the CE advertises the services directly.  While this topology sacrifices some functionality such as 
volumetric DDoS protection and anycast availability from the Distributed Cloud global network, there are some use cases where it can be beneficial.  
One such example is when clients and protected resources are both local to each other without having to traverse the Internet.

With either toplogy, two encrypted tunnels are automatically created between the CE and the two closest REs.  At a minimum these redundant tunnels provide
control-plane communication to the CE. Through the Distributed Cloud Console or via API your configuraton commands traverse these tunnels.

In the event of an Internet outage at a CE site, local survivability will continue to provide data plane services locally for a period of time.  
During this time, control plane services are suspended and will resume upon Internet connection reestablishment.

While a single CE node may be adequate for non-production environments, a high-availability cluster of at least 3 CE's is highly recommended for production.

This lab has auto deployed an AWS site with a CE node for you. You may walk through this process using the F5 Distributed Cloud Simulator if you wish.

https://simulator.f5.com/s/cloud2cloud_via_sites_brownfield/nav/aws/005/0

Continue with the steps below to allow secure connectivity to the AWS hosted application. 


Task 1. Create Private Origin Pool
----------------------------------

In Lab #1 we created an origin pool that was accessible via the Public Internet without using a CE deployment.  In Lab #2 we are still leveraging Internet access via the
RE but now connecting to the origin server throught a CE deployment.  In this topology the CE IPSec tunnels will carry both control-plane and data-plane traffic down to the orgin. 
Now with Lab# 3 we will create an origin pool that will provide internal resources discovered with local DNS by the CE node that is deployed in our lab AWS environment.


We will first create an Origin Pool that refers to the "Private Endpoint" site in our lab environment.

+------------------------------------------------------------------------------------------------------------+
|| 1. Start in F5 Distributed Cloud Console and switch back to the **Multi-Cloud App Connect** workspace.    |
||                                                                                                           |
|| 2. Navigate the menu to go to **Manage->Load Balancers->Origin Pools**. Click on **Add Origin Pool**.     |
||                                                                                                           |
|| 3. Enter the following variables:                                                                         |
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+
|                                                                                                            |
|   ================================= ======                                                                 |
|   *Variable*                        *Value*                                                                |
|   ================================= ======                                                                 |
|   Name                              **[NAMESPACE]-private-pool**                                           |
|   ================================= ======                                                                 |
|                                                                                                            |
+------------------------------------------------------------------------------------------------------------+
||                                                                                                           |
|| 4. Click on **Add Item** under the section **Origin Servers**                                             |
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+
|                                                                                                            |
|   ================================= ============================================                           |
|   *Variable*                        *Value*                                                                |
|   ================================= ============================================                           |
|   Select Type of Origin Server      **DNS Name of Origin Server on given Sites**                           |
|   DNS Name                          **private.lab.f5demos.internal**                                       |
|   Site                              **system/appworld-awsnet**                                             |
|   ================================= ============================================                           |
|                                                                                                            |
+------------------------------------------------------------------------------------------------------------+   
|                                                                                                            |
| |lab301|                                                                                                   | 
|                                                                                                            |
||                                                                                                           |
|| 5. Click on **Apply** to return to the previous screen.                                                   |
||                                                                                                           |
|| 6. Below the **Origin Servers** section fill in the Origin Server Port information                        |     
||                                                                                                           |
+------------------------------------------------------------------------------------------------------------+
|                                                                                                            |
|                                                                                                            |
|   ================================= =======                                                                |
|   *Variable*                        *Value*                                                                |
|   ================================= =======                                                                |
|   Port                              **8080**                                                               |
|   ================================= =======                                                                |
|                                                                                                            |
+------------------------------------------------------------------------------------------------------------+   
||                                                                                                           |
|| 7. Click **Save and Exit**.                                                                               |
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
|   Name                              **[NAMESPACE]-private-lb**                                                                    |
|   Domains                           **[NAMESPACE].aws.lab.f5demos.com**                                                           |
|   Select type of Load Balancer      **HTTP**                                                                                      |
|   Automatically Manage DNS Records  **No/Unchecked**                                                                              |
|   ================================= =======                                                                                       |
|                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------+                                                                                       
|                                                                                                                                   |
|  |lab311|                                                                                                                         |
||                                                                                                                                  |
|| 4. Under **Origin Pools** Click **Add Item**                                                                                     |
||                                                                                                                                  |
|  |lab302|                                                                                                                         |
||                                                                                                                                  |
|| 5. Select the recently created **[NAMESPACE]-private-pool** under **Origin Pool** and then click **Apply**                       |
||                                                                                                                                  |
|  |lab303|                                                                                                                         |
||                                                                                                                                  |
|| 6. Now you can see your Origin Pool has been added to the HTTP Load balancer configuration                                       |
||                                                                                                                                  |
|  |lab304|                                                                                                                         |
||                                                                                                                                  |
|| 7. Now we want to control how this Load Balancer is advertised, we will select the **Other Settings** on the left hand side.     |
||    This will auto-scroll the configuations towards the bottom of the Load Balancer configuration section labeled                 |
||    **Other Settings**                                                                                                            |
||                                                                                                                                  |
|| |lab305|                                                                                                                         |
||                                                                                                                                  |
|| 8. Under **VIP Advertisement**, change it to **Custom**  then select **Configure**                                               |
||                                                                                                                                  |
|| |lab306|                                                                                                                         |
||                                                                                                                                  |
|| 9. In the **List of Sites to Advertise**, Click on **Add Item**                                                                  |
||                                                                                                                                  |
|| |lab307|                                                                                                                         |
||                                                                                                                                  |
|| 10. For **Site Network** select **Outside Network**                                                                              |
||                                                                                                                                  |
|| 11. For **Site Reference** select **system/appworld-awsnet**                                                                     |
||                                                                                                                                  |
|| |lab308|                                                                                                                         |
||                                                                                                                                  |
|| 12. Click on **Apply** and once again **Apply** on the next screen.                                                              |
||                                                                                                                                  |
|                                                                                                                                   |       
+-----------------------------------------------------------------------------------------------------------------------------------+

Task 3: Configure WAF Policy
-----------------------------
Now that we have our load balancer and orign server configured we want to make sure we are protecting the origin server.  Here we   
are easily applying a pre-existing shared WAF policy to our loadbalancer.  The shared WAF policy is available for all namespaces
under this tenant.

+-----------------------------------------------------------------------------------------------------------------------------------+
|| 1. Under the **Web Application Firewall** section                                                                                |
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
+-----------------------------------------------------------------------------------------------------------------------------------+

Task 4: Verify Configuration
-----------------------------

You should now be able to go to the DNS name that you created in this Load Balancer configuration.  
The FQDN we used in our example is http://[NAMESPACE].aws.lab.f5demos.com/.  

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


Task 5: Verify DNS
-------------------

You can verify that you are connecting directly to AWS by comparing the DNS of the two hosts.

+-----------------------------------------------------------+
| .. code-block::                                           |
|                                                           |
|   $ dig +short [NAMESPACE].aws.lab.f5demos.com            |
|   52.4.72.136                                             |
|                                                           |
|   $ dig -x 52.4.72.136 +short                             |
|   ec2-52-4-72-136.compute-1.amazonaws.com                 |
+-----------------------------------------------------------+

+-----------------------------------------------------------+
| .. code-block::                                           |
|                                                           |
|   $ nslookup [NAMESPACE].aws.lab.f5demos.com              |
|                                                           |
|   Server:  2a01:cb04:765:e00:a6ce:daff:fe11:96e           |
|   Address: 2a01:cb04:765:e00:a6ce:daff:fe11:96ea#53       |
|                                                           |
|   Non-authoritative answer:                               |
|   Name: [NAMESPACE].aws.lab.f5demos.com                   |
|   Address: 52.4.72.136                                    |
+-----------------------------------------------------------+

Task 6: Verify WAF Protection
-----------------------------

In this topology we are sending traffic to the AWS EIP that's attached to the CE node in the AWS VPC.

We then connect to the AWS resource via it's Private IP address.  

+-----------------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                                   |
| Using some of the sample attacks below, add the URI path & variables to your application to generate                              |
| security event data.                                                                                                              |
|                                                                                                                                   |
|    * /?cmd=cat%20/etc/passwd                                                                                                      |
|    * /product?id=4%20OR%201=1                                                                                                     |
|    * /cart?search=aaa'><script>prompt('Please+enter+your+password');</script>                                                     |
|                                                                                                                                   |
| Just like in Lab 1, you should see a block page when adding the attacks to the URL.  The difference in this case is that the      |
| enforcement of the WAF policy is occurring on the load balancer on the CE node that is deployed in the AWS lab environment        |
| instead of the Regional Edge in Distributed Cloud.                                                                                |
|                                                                                                                                   |
|   |lab313|                                                                                                                        |                                      
|                                                                                                                                   |
+-----------------------------------------------------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------------------------------------------------+
| **End of Lab 3**.  Congratulations, you have successfully configured and secured application access within AWS!  This marks       |
| the end of the lab.                                                                                                               |
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
.. |lab308| image:: _static/lab3-appworld2025-task2-lb-site-changee.png
   :width: 800px
.. |lab309| image:: _static/screenshot-global-vip-private.png
   :width: 800px
.. |lab310| image:: _static/lab3-appworld2025-waf-block-message.png
   :width: 800px
.. |lab311| image:: _static/lab3-appworld2025-task2-lb-updated.png
   :width: 800px 
.. |lab312| image:: _static/screenshot-global-vip-private.png
   :width: 800px 
.. |lab313| image:: _static/lab3-appworld2025-waf-block-message.png
   :width: 800px
.. |lab400| image:: _static/c_write_quorom_pool.png
   :width: 800px
.. |lab401| image:: _static/c_health_monitor.png
   :width: 800px
.. |lab402| image:: _static/c_warp_params.png
   :width: 800px
.. |lab403| image:: _static/c_traffic_to_all_nodes.png
   :width: 800px
.. |lab404| image:: _static/c_connect_to_linux_host.png
   :width: 800px
.. |lab405| image:: _static/c_ansible_take_one_node_down_2.png
   :width: 800px
.. |lab406| image:: _static/c_one_node_down.png
   :width: 800px
.. |lab407| image:: _static/c_take_down_second_node.png
   :width: 800px
.. |lab408| image:: _static/c_all_nodes_down.png
   :width: 800px
.. |labend| image:: _static/labend.png
   :width: 800px
