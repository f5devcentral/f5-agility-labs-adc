Firewall Enforcement
====================

Upstream firewall enforcement restricts which control-plane services
are reachable before traffic ever reaches the BIG-IP device.

Even when Self IP Port Lockdown is properly configured, boundary-layer
controls must prevent unauthorized networks from initiating management-plane
connections.

This mechanism is a critical Outer Layer boundary control.

Executive Summary
-----------------

   BIG-IP control-plane services must be reachable only from explicitly
   authorized administrative subnets.

   Internet-facing and application networks must never have direct
   access to management services.

Objective
---------

This lab will:

* Restrict management-plane access at the upstream firewall
* Permit access only from authorized administrative subnets
* Block management services from untrusted networks
* Validate blocked access attempts
* Confirm logging and drift prevention posture

Hardened Enterprise Reference Design
------------------------------------

The goal is to enforce deterministic boundary-layer protection
in front of the BIG-IP control plane.

.. note::

   Upstream enforcement ensures unauthorized traffic is blocked
   before reaching the device. BIG-IP hardening complements,
   but does not replace, firewall segmentation.

.. nwdiag::
   :caption: Reference Design – Upstream Firewall Protecting BIG-IP Control Plane
   :name: firewall-enforcement-reference-design

   nwdiag {
     internet [shape = cloud];

     network admin     { address = "Authorized Admin Subnet"; }
     network dmz       { address = "DMZ / Internet-Facing"; }
     network internal  { address = "Application Network"; }
     network firewall  { address = "Upstream Firewall"; }
     network mgmt      { address = "BIG-IP Management IP"; }

     internet -- firewall;
     dmz -- firewall;
     internal -- firewall;
     admin -- firewall;
     firewall -- mgmt;
   }

Recommended Firewall Posture
----------------------------

+----------------------+----------------------+--------------------+-------------------------------------------+
| Source               | Destination          | Service            | Action                                    |
+======================+======================+====================+===========================================+
| Authorized Admin     | BIG-IP Mgmt IP       | SSH, HTTPS         | Permit (Logged)                           |
+----------------------+----------------------+--------------------+-------------------------------------------+
| Monitoring Systems   | BIG-IP Mgmt IP       | SNMP (if required) | Permit (Logged)                           |
+----------------------+----------------------+--------------------+-------------------------------------------+
| Any                  | BIG-IP Mgmt IP       | Management Ports   | Deny (Logged)                             |
+----------------------+----------------------+--------------------+-------------------------------------------+

Management Ports of Concern
---------------------------

+----------------+-----------------------------------------+
| Port           | Service                                 |
+================+=========================================+
| TCP 22         | SSH                                     |
+----------------+-----------------------------------------+
| TCP 443        | TMUI / HTTPS                            |
+----------------+-----------------------------------------+
| UDP 161        | SNMP                                    |
+----------------+-----------------------------------------+
| TCP 4353       | iControl / big3d (if applicable)        |
+----------------+-----------------------------------------+

.. warning::

   Broad "Any Internal" source rules are equivalent to no segmentation.
   Only explicitly authorized subnets should be permitted.

GUI Configuration Procedure (BIG-IP Validation)
-----------------------------------------------

Step 1 – Confirm Management IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **System → Platform**.
3. Document the Management IP address.

Step 2 – Confirm No Self IP Exposure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **Network → Self IPs**.
2. Confirm data-plane VLANs are not exposing management services.
3. Verify Port Lockdown posture aligns with enterprise standards.

Step 3 – Review Logs for Access Attempts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Logs → Local Traffic**.
2. Filter for management-plane connections.
3. Confirm only authorized source IPs are present.

Firewall Configuration Procedure
---------------------------------

Step 1 – Define Authorized Subnet Objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On the upstream firewall:

1. Create address objects for:
   * Authorized administrative subnets
   * Monitoring systems (if applicable)

Step 2 – Create Permit Rule
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Source: Authorized Admin Subnet
2. Destination: BIG-IP Management IP
3. Service: Required management ports only
4. Action: Permit
5. Logging: Enabled

Step 3 – Create Explicit Deny Rule
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Source: Any
2. Destination: BIG-IP Management IP
3. Service: Management ports
4. Action: Deny
5. Logging: Enabled

Ensure rule ordering enforces least privilege.

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

1. From an authorized host, confirm TMUI access succeeds.
2. From an unauthorized host, confirm TMUI access fails.
3. Review **System → Logs → Local Traffic** for expected behavior.

CLI Verification (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

From unauthorized host:

.. code-block:: bash

   curl -k https://<mgmt-ip>

Expected result:

Connection timeout or refusal.

Network Validation
~~~~~~~~~~~~~~~~~~

From unauthorized host:

.. code-block:: bash

   nmap -p 22,443 <mgmt-ip>

Expected:

* Ports filtered or closed

iHealth Validation
------------------

Upload a QKView and confirm:

* No public management exposure findings
* No high-risk management-plane heuristics
* No control-plane exposure alerts

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   Firewall enforcement is the first defensive boundary.
   It prevents unauthorized traffic from reaching BIG-IP.

   Reinforce that device-level controls do not replace
   upstream segmentation.

Lab Challenge – Drift Scenario
------------------------------

Scenario:

A temporary firewall rule was added to allow full internal access
for troubleshooting and was never removed.

Student Tasks:

1. Identify the overly broad rule.
2. Replace with least-privilege source restriction.
3. Validate unauthorized hosts are blocked.
4. Confirm logging is enabled.

Success Criteria
~~~~~~~~~~~~~~~~

* Only authorized administrative subnets can access management services
* Unauthorized sources are blocked
* All management-plane attempts are logged

Advanced Exercise
~~~~~~~~~~~~~~~~~

Temporarily disable logging on the deny rule.
Observe visibility impact, then restore proper logging configuration.