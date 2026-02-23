IP Allowlisting
===============

IP allowlisting restricts administrative access to the BIG-IP device
based on source IP address.

Even when upstream firewall controls are in place, device-level
source restrictions provide an additional enforcement layer
for control-plane services.

This mechanism is a critical Outer Layer boundary control.

Executive Summary
-----------------

   Administrative access must be restricted by explicit source IP.
   Only approved administrative and monitoring systems should be
   permitted to initiate SSH, TMUI, or SNMP connections.

   Broad internal access is not acceptable without documented justification.

Objective
---------

This lab will:

* Restrict TMUI access by source IP
* Restrict SSH access by source IP
* Restrict SNMP access to monitoring systems only
* Validate unauthorized access attempts are blocked
* Confirm least-privilege administrative posture

Hardened Enterprise Reference Design
------------------------------------

The goal is to enforce explicit source-IP restrictions for all
management-plane services.

.. note::

   Source-IP allowlisting complements upstream firewall controls.
   Both layers should enforce least privilege.

.. nwdiag::
   :caption: Reference Design – BIG-IP Administrative IP Allowlisting
   :name: ip-allowlisting-reference-design

   nwdiag {
     internet [shape = cloud];

     network admin     { address = "Authorized Admin Subnet"; }
     network monitor   { address = "Monitoring Systems"; }
     network internal  { address = "Application Network"; }
     network firewall  { address = "Upstream Firewall"; }
     network mgmt      { address = "BIG-IP Management IP"; }

     internet -- firewall;
     internal -- firewall;
     admin -- firewall;
     monitor -- firewall;
     firewall -- mgmt;
   }

Recommended Administrative Access Posture
-----------------------------------------

+----------------------+----------------------+--------------------+-------------------------------------------+
| Source               | Destination          | Service            | Action                                    |
+======================+======================+====================+===========================================+
| Authorized Admin     | BIG-IP Mgmt IP       | SSH (22)           | Permit                                    |
+----------------------+----------------------+--------------------+-------------------------------------------+
| Authorized Admin     | BIG-IP Mgmt IP       | HTTPS (443)        | Permit                                    |
+----------------------+----------------------+--------------------+-------------------------------------------+
| Monitoring Systems   | BIG-IP Mgmt IP       | SNMP (161)         | Permit                                    |
+----------------------+----------------------+--------------------+-------------------------------------------+
| Any                  | BIG-IP Mgmt IP       | Mgmt Ports         | Deny                                      |
+----------------------+----------------------+--------------------+-------------------------------------------+

Administrative Services of Concern
-----------------------------------

+----------------+-----------------------------------------+
| Port           | Service                                 |
+================+=========================================+
| TCP 22         | SSH                                     |
+----------------+-----------------------------------------+
| TCP 443        | TMUI / HTTPS                            |
+----------------+-----------------------------------------+
| UDP 161        | SNMP                                    |
+----------------+-----------------------------------------+
| TCP 4353       | iControl REST / big3d (if required)     |
+----------------+-----------------------------------------+

.. warning::

   Allowing entire RFC1918 ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
   defeats the purpose of allowlisting. Only explicitly approved
   administrative or monitoring subnets should be permitted.

GUI Configuration Procedure
---------------------------

Step 1 – Confirm Management IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **System → Platform**.
3. Document the Management IP address.

Step 2 – Validate TMUI Access Scope
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. From an authorized administrative host,
   browse to:

   https://<mgmt-ip>

2. Confirm successful login.

3. From an unauthorized host, attempt the same.
4. Confirm connection fails.

If your platform supports management access restriction fields,
ensure only approved source IPs are listed.

Otherwise, enforce allowlisting via upstream firewall policy.

Step 3 – Validate SSH Access Scope
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Services**.
2. Confirm SSH is enabled only if required.

From an authorized host:

.. code-block:: bash

   ssh admin@<mgmt-ip>

Confirm login succeeds.

From an unauthorized host:

.. code-block:: bash

   ssh admin@<mgmt-ip>

Confirm connection is blocked.

Step 4 – Restrict SNMP to Monitoring Systems
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → SNMP**.
2. Confirm SNMP is enabled only if required.
3. Review SNMP community configuration.

Ensure only approved monitoring systems are permitted
to poll the device.

If direct source-IP restriction is not configurable
in the GUI, enforce via upstream firewall:

* Permit UDP 161 from monitoring systems only
* Deny all other sources
* Enable logging

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

1. Navigate to **System → Logs → Local Traffic**.
2. Review management access entries.
3. Confirm only authorized source IPs appear.

CLI Verification (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

From unauthorized host:

.. code-block:: bash

   nmap -p 22,443,161 <mgmt-ip>

Expected:

* Ports filtered or closed

Network Validation
~~~~~~~~~~~~~~~~~~

Attempt access from unauthorized host:

.. code-block:: bash

   curl -k https://<mgmt-ip>

Expected:

Connection blocked or refused.

iHealth Validation
------------------

Upload a QKView and confirm:

* No public management exposure heuristics
* No high-risk administrative service findings
* No unrestricted SNMP exposure

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   IP allowlisting is often assumed to be handled by perimeter firewalls.
   However, device-level validation ensures policy enforcement even if
   upstream controls are misconfigured.

   Reinforce that allowlisting must:
   * Be explicit
   * Be minimal
   * Be logged
   * Be periodically reviewed

Lab Challenge – Broad Internal Access Scenario
----------------------------------------------

Scenario:

The firewall permits the entire internal network
(10.0.0.0/8) to access SSH and HTTPS.

Student Tasks:

1. Identify overly broad source definition.
2. Replace with explicit administrative subnet.
3. Validate unauthorized internal hosts are blocked.
4. Confirm logging of denied attempts.

Success Criteria
~~~~~~~~~~~~~~~~

* Only authorized administrative IPs can access SSH and TMUI
* Only monitoring systems can access SNMP
* Unauthorized hosts are blocked
* Logging confirms enforcement
* No broad internal ranges remain without justification

Advanced Exercise
~~~~~~~~~~~~~~~~~

Temporarily remove SNMP restriction for monitoring systems.
Observe expanded exposure via port scan,
then restore least-privilege configuration.