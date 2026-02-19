TMOS - Performance Tuning
=========================

This class covers performance tuning options with TMOS.  Students will learn how to review application traffic flowing through TMOS.  Once the traffic patterns are known, ajustments can be made to optimize performance.  The lab will focus on TCP and HTTP tuning options.

Getting Started
---------------

You will be using our Unified Demo Framework (UDF) environment to complete the tasks in this lab.

Lab Components
--------------

The following components have been included in your lab environment:

* F5 BIG-IP VE (bigip01)
* Ubuntu router - For adding 200ms network latency between the client and BIGIP Virtual Servers
* Ubuntu Client - For generating test traffic
* Ubuntu Webserver
* Windows Client - For Wireshark

Lab Topology Diagram
--------------------

.. image:: ../images/image90.png

Lab Credentials
---------------

The following table lists the management credentials for the components used in the labs:

  
+----------------------+--------------------------------------------------+
| Host                 | Username:Password                                |
+======================+==================================================+
| BIGIP01              | admin:admin.F5demo.com and root:root.F5demo.com  |
+----------------------+--------------------------------------------------+
| Windows Client       | labUser:lab.F5demo.com30 minutes                 |
+----------------------+--------------------------------------------------+
| Linux Client         | SSH Key no password required                     |
+----------------------+--------------------------------------------------+

Timing for Labs
---------------

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
| LAB 2 - Working With TCP Profiles                    | 30 minutes       |
+------------------------------------------------------+------------------+
| LAB 3 - HTTP Tuning                                  | 20 minutes       |
+------------------------------------------------------+------------------+
| Appendix - Optional Packet Decryption Lab            | 15 minutes       |
+------------------------------------------------------+------------------+

Expected time to complete: **1.5 hours**

.. toctree::
   :maxdepth: 1
   :glob:

   Module*/module*
   

