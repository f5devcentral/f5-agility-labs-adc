TMOS - Performance Tuning
=========================

This class covers performance tuning options with TMOS.  Students will learn how to review application traffic flowing through TMOS.  Once the traffic patterns are known, adjustments can be made to optimize performance.  The lab will focus on TCP, UDP and HTTP tuning options.

Getting Started
---------------

You will be using our Unified Demo Framework (UDF) environment to complete the tasks in this lab.

Lab Components
--------------

The following components have been included in your lab environment:

* BIGIP01 (F5 BIG-IP VE)
* Router - For adding 200ms network latency between the client and BIG-IP Virtual Servers
* Ubuntu-Client - For generating test traffic
* Ubuntu-Webserver - Hosting HTTP content and DNS services
* Windows-Client - For Wireshark

Lab Topology Diagram
--------------------

.. image:: /images/udf_lab_diagram.png
    :width: 900px


Lab Credentials
---------------

The following table lists the management credentials for the components used in the labs:


+----------------+-------------------------------------+
| Host           | Username:Password                   |
+================+=====================================+
| BIGIP01 TMUI   | admin:admin.F5demo.com              |
+----------------+-------------------------------------+
| Windows Client | labUser:lab.F5demo.com              |
+----------------+-------------------------------------+
| Linux Client   | No password required for Web Shell  |
+----------------+-------------------------------------+

Timing for Labs
---------------

The time it takes to perform each lab varies from person to person.  Below is an estimate of how long each lab should take:


+------------------------------------------------------+------------------+
| LAB Name                                             | Time Allocated   |
+======================================================+==================+
| Into - Working With The Lab                          | 8 minutes        |
+------------------------------------------------------+------------------+
| LAB 1 - Viewing Application Data                     | 20 minutes       |
+------------------------------------------------------+------------------+
| LAB 2 - Working With TCP Profiles                    | 30 minutes       |
+------------------------------------------------------+------------------+
| LAB 3 - Layer 7 LB For HTTP And UDP                  | 20 minutes       |
+------------------------------------------------------+------------------+

Expected time to complete: **1.5 hours**

.. toctree::
   :maxdepth: 1
   :glob:

   Module*/module*
   

