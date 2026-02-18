TMOS - Performance Tuning
=========================

This class covers performance tuning options with TMOS.  Students will learn how to review application traffic flowing through TMOS.  Once the traffic patterns are known, ajustments can be made to optimize performance.  The lab will focus on TCP and HTTP tuning options.

Getting Started
---------------

You will be using our Unified Demo Framework (UDF) environment to complete the tasks in this lab.  All configuration can be completed from a Web browser.

Lab Topology Diagram
--------------------
.. image:: ../images/image90.png

The following components have been included in your lab environment:

* F5 BIG-IP VE (bigip01)
* Ubuntu router - For adding 200ms network latency between the client and BIGIP Virtual Servers
* Ubuntu Client - For generating test traffic
* Ubuntu Webserver
* Windows Client - For Wireshark

Lab Components
---------------

The following table lists the management IP addresses and credentials for the components used in the labs:

.. list-table:: 
   :widths: auto
   :align: center
   :header-rows: 1

   * - Host
     - Management IP
     - username:password
   * - bigip01
     - 10.1.1.1
     - **admin**:admin.F5demo.com *and/or* **root**:root.F5demo.com
   * - Windows Client
     - 10.1.1.6
     - **labUser**:lab.F5demo.com

Timing for Labs
^^^^^^^^^^^^^^^

The time it takes to perform each lab varies and is mostly dependent on
accurately completing steps. This can never be accurately predicted but
we strived to derive an estimate among several people each having a
different level of experience. Below is an estimate of how long it will
take for each lab:

+------------------------------------------------------+------------------+
| LAB Name (Description)                               | Time Allocated   |
+======================================================+==================+
| LAB 1 - Getting to Know Your App Traffic             | 20 minutes       |
+------------------------------------------------------+------------------+
| LAB 2 - Working With TCP Profiles                    | 20 minutes       |
+------------------------------------------------------+------------------+
| LAB 3 - HTTP Tuning                                  | 20 minutes       |
+------------------------------------------------------+------------------+
| Appendix - Optional Packet Decryption Lab            | 15 minutes       |
+------------------------------------------------------+------------------+
Add the lab summary here

Expected time to complete: **1.5 hours**

.. toctree::
   :maxdepth: 1
   :glob:

   Module*/module*

