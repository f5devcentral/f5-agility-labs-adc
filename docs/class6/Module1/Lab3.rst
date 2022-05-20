Lab 3:  Modify BIG-IP settings, Sync, & Perform Validation
=========================================================================

Lab Tasks:
==========
* Task 1: Modify Self IP Port Lockdown
* Task 2: Verify an Active / Standby "In Sync" State
* Task 3: Create an LTM Node Configuration Object
* Task 4: Perform a Configuration Synchronization between BIG-IPs

**Task 1:  Modify Self IP Port Lockdown**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


On each BIG-IP:



-

+--------------+--------------------------------------------------------------+
| Navigate to: | Network --> Self IPs --> click the VLAN 30 Self IP hyperlink |
+--------------+--------------------------------------------------------------+

..



-  .. image:: ../images/image45.png
      :width: 7.82431in
      :height: 6.11111in

.. _section-3:


-

-  Under the Port Lockdown drop-down, change the value to "Allow
   Default," then click the Update button:

   -  .. image:: ../images/image46.png
         :width: 6.40764in
         :height: 4.37986in

.. _section-4:


-

-

+-----------------------+---------------------------------------------+
| .. rubric:: Question: | Are the devices Active / Standby ? Why or   |
|    :name: question    | why not?                                    |
+=======================+=============================================+
| Answer:               | No, still Active / Active . . . Failover    |
|                       | via unicast Self IPs … port lockdown        |
+-----------------------+---------------------------------------------+

.. _section-5:


-

-  On both BIG-IPs, modify the Internal & External Self IP Port Lockdown
   settings

   -  Change from "Allow None" to "Allow Default"

..



-

+-----------------------+---------------------------------------------+
| .. rubric:: Question: | Are the devices Active / Standby ? Why or   |
|    :name: question-1  | why not?                                    |
+=======================+=============================================+
| Answer:               | Yes, they are now Active / Standby for      |
|                       | Failover.                                   |
+-----------------------+---------------------------------------------+
| Log output:           | Apr 28 12:34:47 bigipB.f5demo.com info      |
|                       | sod[7297]: 010c0085:6: First failover       |
|                       | status message received from device         |
|                       | bigipA.f5demo.com (10.1.1.5) (unicast: ->   |
|                       | 10.1.20.242).                               |
|                       | Apr 28 12:34:47 bigipB.f5demo.com warning   |
|                       | sod[7297]: 010c0084:4: Failover status      |
|                       | message received after 3977.100 second gap, |
|                       | from device bigipA.f5demo.com (10.1.1.5)    |
|                       | (unicast: -> 10.1.20.242).                  |
+-----------------------+---------------------------------------------+

..







-

+----------------------+----------------------+----------------------+
| .. rubric:: Bonus    | How do you know what |                     |
|    Question:         | "ports" are allowed  |                      |
|    :                 | in the "default"     |                      |
| name: bonus-question | port-lockdown list?  |                      |
+======================+======================+======================+
| **Answer:**          | GUI: Help, expand    | | CLI:               |
|                      | Port Lockdown, and   | |                   |
|                      | review:              |                      |
|                      |                      | tmsh list net        |
|                      |                     | self-allow           |
|                      |                      |                      |
|                      | .. image::           |                     |
|                      |  vertopal_ef3ad3b065 |                      |
|                      | ac47a0834351436faa7c | | net self-allow {   |
|                      | 57/media/image47.png | | defaults {         |
|                      |    :width: 2.37986in | | igmp:any           |
|                      |                      | | ospf:any           |
|                      |   :height: 9.10208in | | pim:any            |
|                      |                      | | tcp:domain         |
|                      |                      | | tcp:f5-iquery      |
|                      |                      | | tcp:https          |
|                      |                      | | tcp:snmp           |
|                      |                      | | tcp:ssh            |
|                      |                      | | udp:520            |
|                      |                      | | udp:cap            |
|                      |                      | | udp:domain         |
|                      |                      | | udp:f5-iquery      |
|                      |                      | | udp:snmp           |
|                      |                      | | }                  |
|                      |                      | | }                  |
+----------------------+----------------------+----------------------+





**Task 2: Verify an Active / Standby "In Sync" State **
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-

On each BIG-IP, review the current state.



-  BIG-IP-A:

   -  .. image:: ../images/image48.png
         :width: 4.87014in
         :height: 1.25in

-  BIG-IP-B:

   -  .. image:: ../images/image49.png
         :width: 4.62986in
         :height: 1.24097in

.. _section-7:


-

-  If your BIG-IPs are in a state other than the above, please review
   previous steps to ensure correct configuration.

..



.. _section-8:


-

**Task 3: Create an LTM Node Configuration Object**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



In this task, we will create an LTM node object that will be
synchronized between BIG-IPs. This will validate that ConfigSync is
working correctly. We will perform these changes on the ACTIVE BIG-IP,
and then we will sync these changes to the STANDBY BIG-IP.



On the ACTIVE BIG-IP:



-

+--------------+------------------------------------------------------+
| Navigate to: | Local Traffic --> Nodes --> Node List --> click the  |
|              | "+" sign to create a new node                        |
+--------------+------------------------------------------------------+

..



-  .. image:: ../images/image50.png
      :width: 4.12986in
      :height: 3.56458in

.. _section-9:


-

-  Create the following Windows Server Node Object, and click Finished:

   -  Name = win_server

   -  Address = 10.1.10.199

      -  .. image:: ../images/image51.png
            :width: 6.48125in
            :height: 4.12986in



**Task 4:  Perform a Configuration Synchronization between BIG-IPs**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


~

On the ACTIVE BIG-IP,



-  Notice the "Changes Pending" in the upper-left corner:

   -  .. image:: ../images/image52.png
         :width: 4.5in
         :height: 1.20347in

   -  Click this hyperlink to go to the Overview screen.

   -  Review the recommendations, and perform a ConfigSync to peer:

      -  .. image:: ../images/image53.png
            :width: 7.94444in
            :height: 6.87986in

.. _section-11:


-

-  While the configuration is being pushed, you will see a "Syncing"
   icon display in the middle:

   -  .. image:: ../images/image54.png
         :width: 9.64792in
         :height: 1.37986in

.. _section-12:


-

-  Once the ConfigSync process is complete, your BIG-IPs should indicate
   an "In Sync" state, and be in an Active / Standby cluster:

   -  Verify the sync state:

      -  .. image:: ../images/image55.png
            :width: 7.32431in
            :height: 4.14792in
