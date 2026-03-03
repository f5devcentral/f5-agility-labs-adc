Control-Plane Segmentation Validation
=====================================

Control-plane segmentation must be empirically validated to ensure that
administrative services are reachable only through intended network paths.
Configuration alone does not guarantee isolation.

This lab validates the combined effect of Outer Layer controls by confirming:

* Management services are accessible only via the management interface
* Data-plane VLANs do not expose control-plane services
* No unintended lateral movement paths exist
* Administrative access paths are deterministic and controlled

This lab validates network-layer segmentation only.
Authentication and authorization enforcement are validated separately
in the Middle Layer.

Executive Summary
-----------------

   Segmentation design must be empirically validated.
   This lab confirms that IP Allowlisting and Self IP Port Lockdown
   work together to eliminate alternate management paths and
   enforce control-plane isolation.

Threat Scenario
---------------

In the absence of proper segmentation validation:

* A data-plane Self IP may inadvertently expose SSH or HTTPS.
* Broad source access to the management IP may allow lateral movement.
* Routing misconfigurations may create alternate administrative paths.
* Firewall rules may not reflect actual device-level exposure.

Segmentation validation ensures that:

* Only authorized administrative hosts can reach the management IP.
* Data-plane VLAN interfaces do not expose management services.
* No unintended control-plane access paths exist.

Objective
---------

This lab will:

* Inventory BIG-IP management and data-plane interfaces
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

* SSH (TCP 22) – if enabled
* HTTPS/TMUI (TCP 443)

---------------------------------------------------------------------

Lab Procedure
-------------

Step 1 – Identify Interface Inventory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Document:
   * Management IP (**Top Right Corner**)
   * External & Internal Self IP (**Network → Self IPs**)

.. image:: /class9/module1_outer/_images/control-plane-segmentation-01-platform-mgmt-ip.png
   :alt: Platform screen showing management IP configuration
   :align: center
   :width: 900px

Platform configuration showing the BIG-IP management IP address.

.. image:: /class9/module1_outer/_images/control-plane-segmentation-02-selfip-inventory.png
   :alt: Self IP inventory showing external and internal Self IPs
   :align: center
   :width: 900px

Self IP inventory identifying external and internal data-plane interfaces.

Record the following values:

* Mgmt IP: ``<mgmt-ip>``
* External Self IP: ``<external-self-ip>``
* Internal Self IP: ``<internal-self-ip>``

---------------------------------------------------------------------

Step 2 – Validate Management IP Reachability (Authorized Path)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the authorized administrative host (management network, e.g., 10.1.1.0/24):

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 443
   Test-NetConnection <mgmt-ip> -Port 22

Expected:

* TcpTestSucceeded: True (for enabled services)

.. image:: /class9/module1_outer/_images/control-plane-segmentation-03-mgmt-ip-reachable.png
   :alt: PowerShell output showing management IP reachable on TCP 22 and 443 from an authorized host
   :align: center
   :width: 900px

Authorized-path validation showing SSH (22) and HTTPS (443)
reachable on the management IP.

---------------------------------------------------------------------

Step 3 – Validate Data-Plane Self IP Non-Reachability
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From a data-plane host (for example, DMZ subnet 10.1.10.0/24):

.. code-block:: powershell

   Test-NetConnection <external-self-ip> -Port 443
   Test-NetConnection <external-self-ip> -Port 22

Expected:

* TcpTestSucceeded: False

.. image:: /class9/module1_outer/_images/control-plane-segmentation-04-external-selfip-blocked.png
   :alt: Validation showing external Self IP blocked on TCP 22 and 443
   :align: center
   :width: 900px

External Self IP validation showing SSH (22) and HTTPS (443) not reachable.

From an internal host (if available, e.g., 10.1.20.0/24):

.. code-block:: powershell

   Test-NetConnection <internal-self-ip> -Port 443
   Test-NetConnection <internal-self-ip> -Port 22

Expected:

* TcpTestSucceeded: False

.. image:: /class9/module1_outer/_images/control-plane-segmentation-05-internal-selfip-blocked.png
   :alt: Validation showing internal Self IP blocked on TCP 22 and 443
   :align: center
   :width: 900px

Internal Self IP validation showing SSH (22) and HTTPS (443) not reachable.

.. note::

   If a native internal test host is not available, validation may be
   performed from the management host. This confirms service non-exposure,
   but does not represent the native VLAN path.

.. note::

   ICMP echo responses may still succeed.
   This lab validates TCP service reachability only.

---------------------------------------------------------------------

Step 4 – Validate Unauthorized Access to Management IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From a non-authorized host (outside the approved admin subnet):

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 443
   Test-NetConnection <mgmt-ip> -Port 22

Expected:

* TcpTestSucceeded: False

.. image:: /class9/module1_outer/_images/control-plane-segmentation-06-mgmt-ip-blocked-unauthorized.png
   :alt: Validation showing management IP blocked from unauthorized host
   :align: center
   :width: 900px

Validation confirming that unauthorized hosts cannot reach management services.

---------------------------------------------------------------------

Step 5 – Document the Exposure Matrix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complete and document the following matrix:

+----------------------+--------------------+--------------------+--------------------+--------------------+
| Target Interface     | TCP 22 Expected    | TCP 22 Observed    | TCP 443 Expected   | TCP 443 Observed   |
+======================+====================+====================+====================+====================+
| Management IP        | Open (authorized)  |                    | Open (authorized)  |                    |
+----------------------+--------------------+--------------------+--------------------+--------------------+
| Management IP        | Blocked (unauth)   |                    | Blocked (unauth)   |                    |
+----------------------+--------------------+--------------------+--------------------+--------------------+
| External Self IP     | Blocked            |                    | Blocked            |                    |
+----------------------+--------------------+--------------------+--------------------+--------------------+
| Internal Self IP     | Blocked            |                    | Blocked            |                    |
+----------------------+--------------------+--------------------+--------------------+--------------------+

This matrix serves as documented evidence that control-plane segmentation
is functioning as designed.

---------------------------------------------------------------------

Validation Summary
------------------

If segmentation is correctly enforced:

* Management services are reachable only through the management interface
* Only authorized administrative hosts can reach the management IP
* Unauthorized hosts cannot reach the management IP
* Data-plane VLAN Self IPs do not expose SSH or HTTPS
* No alternate administrative access paths exist

Outer Layer Alignment
---------------------

This lab validates the combined effect of:

* IP Allowlisting – controls **who** may access management services
* Self IP Port Lockdown – controls **where** management services are exposed

Together they enforce:

* Deterministic administrative access paths
* Network-layer least privilege
* Control-plane isolation
* Zero Trust segmentation principles

Success Criteria
----------------

* Mgmt IP reachable for authorized host on required ports
* Mgmt IP not reachable for unauthorized hosts
* External Self IP not reachable on SSH/HTTPS
* Internal Self IP not reachable on SSH/HTTPS
* Exposure matrix completed and documented as evidence