Lab 1: Prompt and Response Scanning
==========================================================================================

Business problem. You will learn how to protect AI applications from various prompt attacks, such as jailbreaks, prompt injections, and data exfiltration attempts. 
These attacks can lead to unauthorized access, data breaches, and compromised AI model integrity

Following the tasks in the prior **Introduction** Section, you should now be able to access the
UDF lab environment and have received an email from the Calypso SaaS platform.

Task 1: Create project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After logging into the platform you access the Dashboard. From here click on Projects on the left-hand menu.
image::_static/lab1-dashboard.png
:align: center
:alt: Dashboard

The create project dialog will appear. 
Click the radio button to the left of CalypsoAI Chat and click the Create button.
image::_static/lab1-create-project.png
:align: center
:alt: Create Project
Create the new chat project with 
========================== ======================== 
Chat Name                  Model                    
========================== ======================== 
**First inital Last name** genai-azure-openai         
========================== ======================== 
      
image::../_static/lab1-chat.png
:align: center
:alt: Chat Setup

You should now be in your new chat project.
image::_static/lab1-chat-project.png
:align: center
:alt: Chat Project

Task 2: Real-time protection for prompts and responses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here you will be sending prompts to the model you just connected into with-in your project. 
These prompts will be of the safe and unsafe varieties. 
You will observe the results and explore the logs regarding those prompts.

+---------------------------------------------------------------------------------------------------------------+
| 1. Open MinIO WARP (UDF → Components → Traffic‑Gen → Access → Firefox).  The credentials are under lab        |
|    Documentation tab (admin/admin).  If presented with Firefox "Restoring Pages" message, choose "Restore     |
|    Session" button.   As well, permit the pop-up to allow access to clipboard.                                |
|                                                                                                               |
| 2. Select the cluster‑1 profile.                                                                              |
|                                                                                                               |
| 3. Select all 3 buckets, when selected for use they will appear in bright orange.                             |
|                                                                                                               |
| 4. Set Duration to 3 minutes and Concurrency to 20 threads. Conncurrency refers to parallel S3 transactions.  |
|                                                                                                               |
| 5. In WARP Parameters, set Endpoint to 10.1.10.100:9000.                                                      |
|                                                                                                               |
| 6. Click Run Benchmark.                                                                                       |
+---------------------------------------------------------------------------------------------------------------+
| |lab014|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| 7. Open MinIO cluster‑1 Console (UDF → Cluster1‑Node1 → Access → UI). Login: minioadmin / minioadmin          | 
|                                                                                                               |
| 8. Observe that there are 4 AIStor servers spread across 2 clusters, however data charts require normally     |
|    30 minutes or more to popluate so expect no traffic on the right-hand chart.                               |
+---------------------------------------------------------------------------------------------------------------+
| |lab016|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| 9.  Click on the arrow next to time to first byte, in the lower right of the screen.                          | 
|                                                                                                               |
| 10. Observe that once the charts populate, only traffic will be registered with only the first AIStor,        |
|     at address 10.1.10.100 port 9000.  This traffic will task one server, creating a hot spot of load.        |
+---------------------------------------------------------------------------------------------------------------+
| |lab017|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+



Why this matters:  Clients that target a single node are brittleutilization is uneven and scalability suffers. 





Task 3: Baseline: Steer (proxy) the Same Workload through BIG-IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~




+---------------------------------------------------------------------------------------------------------------+
| 1. In WARP, switch target to the BIG‑IP VIP profile BigIP‑cluster‑1.                                          |
|                                                                                                               |
| 2. Set Duration: 5 minutes (300 seconds), Concurrency: 20 threads.                                            |
|                                                                                                               |
| 3. In WARP Parameters, set Endpoint to 10.1.40.160:9000.                                                      |
|                                                                                                               |
| 4. Click Run Benchmark                                                                                        |
|                                                                                                               |
| 5. Log into BIG-IP TMU: Local Traffic → Pools → Cluster‑1.  Confirm 3 pool members are present initially      |
|    Click both Members and Statistics tabs.                                                                    |
|                                                                                                               |
| .. note::                                                                                                     |
|      *due to short run durations, summary analytics may not have appeared in the AIStor dashboard view yet.*  |
+---------------------------------------------------------------------------------------------------------------+
| |lab018|                                                                                                      |
|                                                                                                               |
| |lab019|                                                                                                      |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| 6. Confirm the WARP S3 load generator has run to completions, traffic settings can be seen below.             |
|                                                                                                               |
|                                   |                                                                           |
+---------------------------------------------------------------------------------------------------------------+
| |lab020|                                                                                                      |
|                                                                                                               |
| |lab021|                                                                                                      |
+---------------------------------------------------------------------------------------------------------------+

**Expected outcome**: Traffic is distributed across the **three** nodes behind the VIP.

**Load‑balancing method**: This pool is configured for Least Connections, recommended for S3‑style
workloads to reduce request skew.


Task 4: Scale out easily: add the 4th MinIO AIStor node to the pool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+---------------------------------------------------------------------------------------------------------------+
| 1. In BIG‑IP TMUI: Local Traffic → Pools → Cluster‑1 → Members.  Add a New member and choose Node List menu.  |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+
| |lab041|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| 2. Click **Add.... -> Node List ->** select cl1-nd4                                                           |
|                                                                                                               |
| 3. Set **Service Port** to 9000 and click **Finished**                                                        |                                             
|                                                                                                               | 
|  .. note::                                                                                                    |
|      health checks for the new member will drive the LED from blue to green (ready) shortly                   |
+---------------------------------------------------------------------------------------------------------------+
| |lab042|                                                                                                      |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------------+
| 4. Re-run the WARP workload targetting the BIG-IP Virtual IP (VIP 10.1.40.160:9000)                           |                                                                                                  
|                                                                                                               | 
|  .. note::                                                                                                    |
|      The 4th AIStor node begins processing traffic **immediately**.  All nodes now share load                 |
+---------------------------------------------------------------------------------------------------------------+
| |lab043|                                                                                                      |
|                                                                                                               |
| |lab044|                                                                                                      |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+


**Key Takeaway** There were no client changes and Applications still continue to talk to the same VIP;
topology changes are absorbed by *BIG‑IP* at the dataplane.


Task 5:  Verify the load‑balancing method & pool health
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following steps will guide you through the rich, visual metrics presented for BIG-IP through the AST
dashboards powered by Grafana.   We will verify the load balancing method and pool health.

+---------------------------------------------------------------------------------------------------------------+
| 1. In BIG‑IP TMUI: Local Traffic → Pools → Cluster‑1.                                                         |
|                                                                                                               |
| 2. Confirm Load Balancing Method: Least Connections.                                                          |
|                                                                                                               |
| 3. Check Members tab:  All members green (up) with active connections.                                        |
|                                                                                                               |
| 4. Use the AST tool (to review the Dashboards) UDF -> AST -> Access -> Grafana.                               |
|    Login as admin / admin, when prompted to change password simply retain the value as admin                  |
|                                                                                                               |
| 5. In AST, choose Dashboards - BigIP - Device - Device Pools look at the key metrics, such as Active Pool     |
|            Connections.   For "Pool" in top menu, adjust to "Cluster-1" and examine last 15 minutes.          |
|                                                                                                               |
| 6. Click on the "3 dots" menu → View on any graphical widget to see the full panel.  Click "Refresh" often.   |
|                                                                                                               |
|                                                                                                               |
|                                                                                                               |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+
| |lab045|                                                                                                      |
|                                                                                                               |
+---------------------------------------------------------------------------------------------------------------+

The charts indicate an even distribution for MinIO AIStor nodes, hot spots have been successfully avoided.


Validation
~~~~~~~~~~

MinIO AIStor Console (Main Screen → Data -> arrow next to Time to First Byte): when populated with data points
shows very close to even, per‑node traffic once proxied via VIP (some variability is expected / OK).

BIG‑IP Pool Stats: show all 4 members up with active connections.

End‑user impact: Client endpoint unchanged; backend scaling is transparent.

Troubleshooting
~~~~~~~~~~~~~~~

WARP can't connect
 -> Re‑check endpoint (10.1.10.100:9000 direct vs 10.1.40.160:9000 VIP).
 -> Ensure the http/https scheme matches your setup (WARP must use the correct protocol).

MinIO AIStore UI Metrics don't update
 -> the page isn't real‑time, hit "Refresh" with last 4 hours timerange, towards end of lab to see data

A pool member is down (red)
 -> Verify the MinIO node process.
 -> Review the pool's Monitor and node address/port.

Skew remains after adding cl1‑nd4
 -> Confirm the new member is enabled and passing its monitor.
 -> Ensure Least Connections is applied (not Round Robin).


Clean-Up (Optional)
~~~~~~~~~~~~~~~~~~~

Stop any running WARP tests.

If you temporarily disabled/enabled members, restore their original state.

Leave Cluster‑1 with 4 active members for Labs 2 and 3.


What You Learned - Value of BIG-IP LTM and AIStor
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Decoupling** Clients connect to one VIP; storage topology can evolve freely.

**Efficiency** Least Connections smooths throughput and reduces hotspots.

**Elasticity** Scale by adding/removing pool members without client changes.

**Operational simplicity** Traffic engineering lives at the dataplane, not in every client.



+---------------------------------------------------------------------------------------------------------------+
| **End of Lab 1:**  This concludes Lab 1.  In this lab you ran high rate S3 loads against a single MinIO       |
| AIStor server, leaving other AIStor instances unused.  You then adjusted the load generator to use a virtual  |
| server with a virtual IP address on the F5 BIG-IP.   An origin pool corresponding to three AIStor instances   |
| allow traffic to be spread, per the least connections approach, to all healthy nodes.   A fourth AIStor       |
| instance was added to the pool, exercising all nodes equally and not requiring any client side adjustments,   |
| a major benefit in scenarios with hundreds or possibly thousands of clients.                                  |
+---------------------------------------------------------------------------------------------------------------+
| |labend|                                                                                                      |
+---------------------------------------------------------------------------------------------------------------+

.. |lab000| image:: ../_static/lab1-000.png
   :width: 800px
.. |lab001| image:: ../_static/lab1-001.png
   :width: 800px
.. |lab002| image:: ../_static/lab1-002.png
   :width: 800px
.. |lab003| image:: ../_static/lab1-003.png
   :width: 800px
.. |lab004| image:: ../_static/image_001_WARP_ui.png
   :width: 800px
.. |lab005| image:: ../_static/lab1-005.png
   :width: 800px
.. |lab006| image:: ../_static/lab1-006.png
   :width: 800px
.. |lab007| image:: ../_static/lab1-007.png
   :width: 800px
.. |lab008| image:: ../_static/lab1-008.png
   :width: 800px
.. |lab009| image:: ../_static/lab1-009.png
   :width: 800px
.. |lab010| image:: ../_static/lab1-010.png
   :width: 800px
.. |lab011| image:: ../_static/lab1-011.png
   :width: 800px
.. |lab012| image:: ../_static/lab1-012.png
   :width: 800px
.. |lab013| image:: ../_static/lab1-013.png
   :width: 800px
.. |lab014| image:: ../_static/image_001_WARP_ui.png
   :width: 800px
.. |lab015| image:: ../_static/lab1-015.png
   :width: 800px
.. |lab016| image:: ../_static/a_aistor_traffic_totals.png
   :width: 800px
.. |lab017| image:: ../_static/a_aistor_traffic_per_aistor2.png
   :width: 800px
.. |lab018| image:: ../_static/a_bigip_pool_members_start.png
   :width: 800px
.. |lab019| image:: ../_static/a_bigip-pool-statistics.png
   :width: 800px
.. |lab020| image:: ../_static/a_aistor_traffic_chart.png
   :width: 800px
.. |lab021| image:: ../_static/a_warp_setup.png
   :width: 800px
.. |lab022| image:: ../_static/lab1-022.png
   :width: 800px
.. |lab023| image:: ../_static/lab1-023.png
   :width: 800px
.. |lab024| image:: ../_static/lab1-024.png
   :width: 800px
.. |lab025| image:: ../_static/lab1-025.png
   :width: 800px
.. |lab026| image:: ../_static/lab1-026.png
   :width: 800px
.. |lab027| image:: ../_static/lab1-027.png
   :width: 800px
.. |lab028| image:: ../_static/lab1-028.png
   :width: 800px
.. |lab029| image:: ../_static/lab1-029.png
   :width: 800px
.. |lab030| image:: ../_static/lab1-030.png
   :width: 800px
.. |lab031| image:: ../_static/lab1-031.png
   :width: 800px
.. |lab032| image:: ../_static/lab1-032.png
   :width: 800px
.. |lab033| image:: ../_static/lab1-033.png
   :width: 800px
.. |lab034| image:: ../_static/lab1-034.png
   :width: 800px
.. |lab035| image:: ../_static/lab1-035.png
   :width: 800px
.. |lab036| image:: ../_static/lab1-036.png
   :width: 800px
.. |lab037| image:: ../_static/lab1-037.png
   :width: 800px
.. |lab038| image:: ../_static/lab1-038.png
   :width: 800px
.. |lab039| image:: ../_static/lab1-039.png
   :width: 800px
.. |lab040| image:: ../_static/lab1-040.png
   :width: 800px
.. |lab041| image:: ../_static/a_update_pool_members_list.png
   :width: 800px
.. |lab042| image:: ../_static/a_new_pool_member.png
   :width: 800px
.. |lab043| image:: ../_static/a_data_to_4th_node_big_ip.png
   :width: 800px
.. |lab044| image:: ../_static/a_aistor_chart_showing_data.png
   :width: 800px
.. |lab045| image:: ../_static/a_ast_overview_charts.png
   :width: 800px
.. |labend| image:: ../_static/labend.png
   :width: 800px
