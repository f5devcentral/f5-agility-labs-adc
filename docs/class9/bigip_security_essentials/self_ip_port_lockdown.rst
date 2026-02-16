Self IP Port Lockdown
=====================

Self IP Port Lockdown restricts which control-plane services are reachable
on each VLAN. Even without virtual servers configured, Self IPs may respond
to administrative services unless explicitly limited.

This mechanism is a critical Outer Layer boundary control.

Executive Summary
-----------------

   Production data-plane VLANs should enforce a default-deny posture
   using **Allow Custom**, permitting only explicitly required services.
   Administrative services must not be exposed on DMZ or application VLANs.

Objective
---------

This lab will:

* Configure Self IPs with a least-privilege model
* Prevent unintended exposure of control-plane services
* Validate service reachability
* Confirm correct VLAN segmentation posture

Hardened Enterprise Reference Design
------------------------------------

The goal is to separate control-plane access from data-plane traffic.

.. note::

   This is a reference design. Your topology may differ, but the principle remains:
   use **Allow Custom** on production VLANs and reserve **Allow Default**
   for isolated management or HA networks only.

.. nwdiag::
   :caption: Reference Design – Control Plane Segmentation for Self IPs
   :name: selfip-port-lockdown-reference-design

   nwdiag {
     internet [shape = cloud];
     network dmz     { address = "VLAN: External/DMZ (Data Plane)"; }
     network internal{ address = "VLAN: Internal/App (Data Plane)"; }
     network ha      { address = "VLAN: HA (ConfigSync/Failover)"; }
     network mgmt    { address = "OOB Mgmt Network"; }

     internet -- dmz;

     bigip_a [description = "BIG-IP A\nDMZ: Allow Custom\nInternal: Allow Custom\nHA: Allow Default\nMgmt: Restricted"];
     bigip_b [description = "BIG-IP B\nDMZ: Allow Custom\nInternal: Allow Custom\nHA: Allow Default\nMgmt: Restricted"];

     dmz -- bigip_a;
     dmz -- bigip_b;

     internal -- bigip_a;
     internal -- bigip_b;

     ha -- bigip_a;
     ha -- bigip_b;

     mgmt -- bigip_a;
     mgmt -- bigip_b;
   }

Recommended VLAN Posture
------------------------

+----------------+----------------------------+--------------------+---------------------------------------------+
| VLAN           | Purpose                    | Port Lockdown Mode | Rationale                                    |
+================+============================+====================+=============================================+
| Mgmt (OOB)     | Administrative access      | Allow Default      | Isolated admin network                        |
+----------------+----------------------------+--------------------+---------------------------------------------+
| HA VLAN        | ConfigSync + Failover      | Allow Default      | Dedicated HA communication                    |
+----------------+----------------------------+--------------------+---------------------------------------------+
| External (DMZ) | Client-side data plane     | Allow Custom       | Prevent mgmt/control-plane exposure           |
+----------------+----------------------------+--------------------+---------------------------------------------+
| Internal       | Server-side data plane     | Allow Custom       | Prevent lateral movement to control plane     |
+----------------+----------------------------+--------------------+---------------------------------------------+

Port Lockdown Modes
-------------------

+--------------+---------------------------------------------------+
| Mode         | Behavior                                          |
+==============+===================================================+
| Allow None   | Blocks all services                               |
+--------------+---------------------------------------------------+
| Allow Default| Enables system-defined administrative services    |
+--------------+---------------------------------------------------+
| Allow Custom | Enables only explicitly defined services          |
+--------------+---------------------------------------------------+

.. warning::

   "Allow Default" may expose SSH, HTTPS, SNMP, and other services.
   It should never be used on internet-facing or shared production VLANs.

GUI Configuration Procedure
---------------------------

Step 1 – Access Self IP Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **Network → Self IPs**.
3. Select the target data-plane Self IP (for example: ``external_self``).

Step 2 – Configure Port Lockdown
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Locate the **Port Lockdown** setting.
2. Select **Allow Custom**.

Step 3 – Define Explicit Services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add only services required for your environment.

Typical minimal configuration for data-plane VLANs:

* ``big3d`` (TCP/4353) – if DNS/GTM monitoring is required

Only if operationally necessary:

* ``ConfigSync``
* ``Failover``

Remove all other services.

4. Click **Update**.

Step 4 – Apply to All Production VLAN Self IPs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure consistent configuration across all data-plane VLANs.

When to Use Allow Default
-------------------------

Use **Allow Default** only when the Self IP resides on:

* A dedicated HA VLAN
* An isolated out-of-band management network

Never use Allow Default on DMZ or application VLANs.

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

1. Navigate to **Network → Self IPs**.
2. Confirm data-plane Self IPs show:
   * **Port Lockdown: Allow Custom**
   * Only required services listed

CLI Verification (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   tmsh list net self <self-ip-name>

Confirm:

.. code-block:: text

   allow-service {
       big3d
   }

Network Validation
~~~~~~~~~~~~~~~~~~

From a host on the same VLAN:

.. code-block:: bash

   nmap -p 22,80,161,443,4353 <self-ip-address>

Expected (data-plane VLAN):

* SSH (22) – closed/filtered
* HTTPS (443) – closed/filtered
* SNMP (161) – closed/filtered
* big3d (4353) – open only if required

iHealth Validation
------------------

Upload a QKView and confirm:

* No management interface public exposure heuristics
* No high-risk Self IP findings
* No unintended administrative service exposure

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   Self IPs are frequently misunderstood. Even without virtual servers,
   they may respond to administrative services. Port Lockdown is
   essential to preventing unintended exposure.

   Reinforce that Port Lockdown complements:
   * Firewall controls
   * Management interface isolation
   * AAA enforcement

Lab Challenge – Misconfiguration Scenario
------------------------------------------

Scenario:

A DMZ Self IP is configured with **Allow Default**, and security scans
show SSH and HTTPS reachable.

Student Tasks:

1. Confirm exposure via port scan.
2. Inspect configuration under **Network → Self IPs**.
3. Change to **Allow Custom**.
4. Remove administrative services.
5. Re-scan and confirm exposure is eliminated.

Success Criteria
~~~~~~~~~~~~~~~~

* SSH/HTTPS not reachable on data-plane VLAN
* HA and ConfigSync remain operational
* DNS/GTM monitoring unaffected (if applicable)

Advanced Exercise
~~~~~~~~~~~~~~~~~

Remove a required HA service from an HA VLAN Self IP,
observe synchronization behavior, then restore proper configuration.
