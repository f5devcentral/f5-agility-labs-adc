Control-Plane Segmentation Validation
=====================================

Control-plane segmentation must be validated to ensure that
administrative services are reachable only through intended paths.

This lab ties together Outer Layer controls by verifying that:

* Management services are accessible only via the management interface
* Data-plane VLANs do not expose control-plane services
* No unintended lateral movement paths exist
* Administrative access paths are deterministic and controlled

Executive Summary
-----------------

   Segmentation is not complete until it is validated.
   This lab confirms that IP Allowlisting and Self IP Port Lockdown
   work together to eliminate alternate management paths.

Objective
---------

This lab will:

* Identify BIG-IP management and data-plane interfaces
* Validate management service reachability on the management interface
* Validate management service non-reachability on data-plane Self IPs
* Document an exposure matrix as evidence of segmentation posture

Hardened Enterprise Reference Design
------------------------------------

.. note::

   Validation should be performed from representative network locations:
   * Authorized administrative host (management network)
   * Data-plane host (DMZ or internal VLAN)

.. nwdiag::
   :caption: Reference Design – Segmentation Validation Paths
   :name: control-plane-segmentation-validation-reference-design

   nwdiag {
     network mgmt    { address = "Mgmt Network (Admin Host)"; }
     network dmz     { address = "DMZ Network (Data Plane Host)"; }
     network internal{ address = "Internal Network (Data Plane Host)"; }

     bigip [description = "BIG-IP\nMgmt IP: Allowed (Admin only)\nSelf IPs: No mgmt services"];

     mgmt -- bigip;
     dmz -- bigip;
     internal -- bigip;
   }

Recommended Validation Targets
------------------------------

Validate the following interfaces:

+----------------------+---------------------------+-----------------------------+
| Interface Type       | Example Address           | Expected Mgmt Service Reach |
+======================+===========================+=============================+
| Management IP        | 10.1.1.x                  | Reachable (authorized only) |
+----------------------+---------------------------+-----------------------------+
| External Self IP     | 10.1.10.x                 | Not reachable               |
+----------------------+---------------------------+-----------------------------+
| Internal Self IP     | 10.1.20.x                 | Not reachable               |
+----------------------+---------------------------+-----------------------------+

Services in scope:

* SSH (TCP 22)
* HTTPS/TMUI (TCP 443)

---------------------------------------------------------------------

Lab Procedure
-------------

Step 1 – Identify Interface Inventory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Document:
   * Management IP (System → Platform)
   * External Self IP (Network → Self IPs)
   * Internal Self IP (Network → Self IPs)

.. image:: ../_images/segval_01_platform_mgmt_ip.png
   :alt: Platform screen showing management IP
   :align: center

.. image:: ../_images/segval_02_selfip_list.png
   :alt: Self IP list showing external and internal Self IPs
   :align: center

Record the following values:

* Mgmt IP: ``<mgmt-ip>``
* External Self IP: ``<external-self-ip>``
* Internal Self IP: ``<internal-self-ip>``

---------------------------------------------------------------------

Step 2 – Validate Management IP Reachability (Authorized Path)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the authorized administrative host (management network):

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 443
   Test-NetConnection <mgmt-ip> -Port 22

Expected:

* TcpTestSucceeded: True (for required services)

.. image:: ../_images/segval_03_mgmt_reachability.png
   :alt: PowerShell output showing management IP reachability
   :align: center

---------------------------------------------------------------------

Step 3 – Validate Data-Plane Self IP Non-Reachability (DMZ/Internal)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From a data-plane host (for example, DMZ subnet 10.1.10.0/24):

.. code-block:: powershell

   Test-NetConnection <external-self-ip> -Port 443
   Test-NetConnection <external-self-ip> -Port 22

Expected:

* TcpTestSucceeded: False

.. image:: ../_images/segval_04_external_selfip_blocked.png
   :alt: PowerShell output showing external self IP ports blocked
   :align: center

From an internal host (if available):

.. code-block:: powershell

   Test-NetConnection <internal-self-ip> -Port 443
   Test-NetConnection <internal-self-ip> -Port 22

Expected:

* TcpTestSucceeded: False

.. image:: ../_images/segval_05_internal_selfip_blocked.png
   :alt: PowerShell output showing internal self IP ports blocked
   :align: center

.. note::

   If an internal test host is not available in the lab environment,
   validate the internal Self IP from the management host and document
   that the test is not from the native internal VLAN.

---------------------------------------------------------------------

Step 4 – Document the Exposure Matrix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complete the following matrix:

+----------------------+--------------------+--------------------+
| Target Interface     | TCP 22 (SSH)       | TCP 443 (HTTPS)    |
+======================+====================+====================+
| Management IP        | Expected: Open     | Expected: Open     |
+----------------------+--------------------+--------------------+
| External Self IP     | Expected: Blocked  | Expected: Blocked  |
+----------------------+--------------------+--------------------+
| Internal Self IP     | Expected: Blocked  | Expected: Blocked  |
+----------------------+--------------------+--------------------+

---------------------------------------------------------------------

Validation Summary
------------------

If segmentation is correctly enforced:

* Management services are reachable only through the management interface
* Data-plane VLAN Self IPs do not expose SSH or HTTPS
* No alternate administrative access paths exist

Outer Layer Alignment
---------------------

This lab validates the combined effect of:

* IP Allowlisting (who may access management services)
* Self IP Port Lockdown (where management services are exposed)

Success Criteria
----------------

* Mgmt IP reachable for authorized host on required ports
* External Self IP not reachable on SSH/HTTPS
* Internal Self IP not reachable on SSH/HTTPS
* Evidence captured via screenshots and exposure matrix