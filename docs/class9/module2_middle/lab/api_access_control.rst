API Access Control (AS3 / DO via CI)
=====================================

API access control restricts programmatic configuration access to BIG-IP
through iControl REST, which is used by AS3 and DO automation pipelines.

In the Middle Layer, API access must be restricted to dedicated CI systems,
enforced through least privilege, network segmentation, and logging.

This mechanism is a critical Middle Layer automation security control.

Executive Summary
-----------------

   AS3 and DO automation must access BIG-IP only through
   explicitly authorized CI systems.

   Management APIs must never be reachable from application VLANs
   or broad internal networks.

Objective
---------

This lab will:

* Identify iControl REST exposure on the management interface
* Create a dedicated least-privilege automation account
* Restrict API access to CI subnet only
* Validate blocked access from unauthorized hosts
* Confirm logging and audit posture

Hardened Enterprise Reference Design
------------------------------------

The goal is to isolate management API access to CI systems over
the OOB management network.

.. note::

   This lab assumes BIG-IP management is OOB-only.
   Management services are not reachable via Self IPs.

.. nwdiag::
   :caption: Reference Design – AS3/DO Automation via OOB Management
   :name: api-access-as3-do-reference-design

   nwdiag {
     network ci        { address = "CI/CD Subnet"; }
     network admin     { address = "Admin Subnet"; }
     network mgmt      { address = "OOB Management Network"; }

     ci -- mgmt;
     admin -- mgmt;

     bigip [description = "BIG-IP\nMgmt Interface\nAS3/DO via iControl REST"];
     mgmt -- bigip;
   }

Recommended Automation Posture
------------------------------

+----------------------+----------------------+----------------------+-------------------------------------------+
| Source               | Destination          | Interface            | Action                                    |
+======================+======================+======================+===========================================+
| CI Subnet            | BIG-IP Mgmt IP       | iControl REST (443)  | Permit (Logged)                           |
+----------------------+----------------------+----------------------+-------------------------------------------+
| Admin Subnet         | BIG-IP Mgmt IP       | TMUI / SSH           | Permit (Logged)                           |
+----------------------+----------------------+----------------------+-------------------------------------------+
| Any                  | BIG-IP Mgmt IP       | Mgmt Ports           | Deny (Logged)                             |
+----------------------+----------------------+----------------------+-------------------------------------------+

.. warning::

   Do not use the built-in admin account for AS3/DO automation.
   Automation must use a dedicated least-privilege account.

GUI Configuration Procedure
---------------------------

Step 1 – Confirm Management Interface Isolation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **System → Platform**.
3. Confirm the Management IP resides on a dedicated OOB subnet.
4. Navigate to **Network → Self IPs**.
5. Confirm no data-plane Self IP exposes management services.

Step 2 – Create Dedicated AS3/DO Automation User
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Users → User List**.
2. Click **Create**.
3. Configure:

   - Username: ``ci_as3_do``
   - Role: Application Editor (or minimum required role)
   - Partition access: Restricted to required partitions
   - Shell: None (if CLI access not required)

4. Click **Finished**.

Step 3 – Restrict Management API by Source IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On the upstream firewall protecting the OOB network:

1. Permit TCP 443 from:
   - CI subnet
   - Authorized admin subnet

2. Deny TCP 443 from:
   - Internal application VLANs
   - Internet-facing networks
   - Any unauthorized internal subnet

3. Enable logging on both permit and deny rules.

Step 4 – Validate AS3/DO API Access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From a CI system:

.. code-block:: bash

   curl -sku ci_as3_do:<password> https://<mgmt-ip>/mgmt/shared/echo

Expected:

* HTTP 200 response
* Authentication successful

Deploy a test AS3 declaration via CI pipeline and confirm success.

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

1. Navigate to **System → Users → User List**.
2. Confirm ``ci_as3_do`` exists and is least privilege.
3. Navigate to **System → Logs → Local Traffic**.
4. Confirm API access from CI subnet is logged.

CLI Verification (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

From unauthorized host:

.. code-block:: bash

   curl -sku ci_as3_do:<password> https://<mgmt-ip>/mgmt/shared/echo

Expected:

* Connection blocked or timeout

Network Validation
~~~~~~~~~~~~~~~~~~

From unauthorized host:

.. code-block:: bash

   nmap -p 443 <mgmt-ip>

Expected:

* Port filtered

iHealth Validation
------------------

Upload a QKView and confirm:

* No public management exposure
* No broad internal API access
* No over-privileged automation account findings

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   Emphasize that automation expands attack surface.

   Reinforce:
   * Dedicated automation account
   * Least privilege
   * OOB-only management access
   * Logging and credential rotation

Lab Challenge – Overexposed CI Scenario
---------------------------------------

Scenario:

The CI system can reach BIG-IP via both management interface
and internal Self IP.

Student Tasks:

1. Identify unintended management exposure.
2. Remove Self IP management reachability.
3. Restrict firewall policy to CI subnet only.
4. Validate unauthorized hosts cannot reach API endpoint.

Success Criteria
~~~~~~~~~~~~~~~~

* AS3/DO automation works from CI subnet only
* No API reachability from application VLANs
* Dedicated least-privilege automation account used
* Unauthorized API access blocked and logged

Advanced Exercise
~~~~~~~~~~~~~~~~~

Implement partition-level separation:

1. Restrict ``ci_as3_do`` to a specific partition.
2. Attempt to deploy AS3 declaration outside allowed partition.
3. Confirm deployment fails due to role restriction.