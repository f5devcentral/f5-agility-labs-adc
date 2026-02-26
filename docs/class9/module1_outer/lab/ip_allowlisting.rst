IP Allowlisting
===============

IP Allowlisting restricts administrative access to the BIG-IP device
based on source IP address. Only explicitly approved management and
monitoring systems should be able to initiate connections to the
management interface.

This mechanism is a critical Outer Layer boundary control.

Executive Summary
-----------------

   Administrative access must be restricted by explicit source IP.
   Broad internal access (e.g., 10.0.0.0/8) is not acceptable without
   documented justification.

   IP Allowlisting complements Self IP Port Lockdown by controlling
   who may access management services.

Objective
---------

This lab will:

* Identify management-plane exposure
* Restrict SSH access by source IP
* Restrict TMUI (HTTPS) access by source IP
* Restrict SNMP access to monitoring systems
* Validate unauthorized access attempts are blocked

Hardened Enterprise Reference Design
------------------------------------

.. note::

   IP Allowlisting should exist at multiple layers:
   upstream firewall and device-level enforcement.

.. nwdiag::
   :caption: Reference Design – Administrative Access Control
   :name: ip-allowlisting-reference-design

   nwdiag {
     internet [shape = cloud];
     network admin     { address = "Authorized Admin Subnet"; }
     network monitor   { address = "Monitoring Systems"; }
     network firewall  { address = "Upstream Firewall"; }
     network mgmt      { address = "BIG-IP Management IP"; }

     internet -- firewall;
     admin -- firewall;
     monitor -- firewall;
     firewall -- mgmt;
   }

Recommended Administrative Posture
----------------------------------

+----------------------+----------------------+--------------------+--------------------------------------+
| Source               | Destination          | Service            | Action                               |
+======================+======================+====================+======================================+
| Authorized Admin     | BIG-IP Mgmt IP       | SSH (22)           | Permit                               |
+----------------------+----------------------+--------------------+--------------------------------------+
| Authorized Admin     | BIG-IP Mgmt IP       | HTTPS (443)        | Permit                               |
+----------------------+----------------------+--------------------+--------------------------------------+
| Monitoring Systems   | BIG-IP Mgmt IP       | SNMP (161)         | Permit                               |
+----------------------+----------------------+--------------------+--------------------------------------+
| Any                  | BIG-IP Mgmt IP       | Mgmt Ports         | Deny                                 |
+----------------------+----------------------+--------------------+--------------------------------------+

Administrative Services of Concern
----------------------------------

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

   Allowing entire RFC1918 ranges defeats the purpose of allowlisting.
   Only explicitly approved administrative subnets should be permitted.

---------------------------------------------------------------------

Lab Procedure
-------------

Step 1 – Identify Management IP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **System → Platform**.
3. Document the Management IP address.

.. figure:: ../_images/ip-allowlisting-01-ssh-access-all-addresses.png
   :alt: SSH access enabled with IP Allow set to All Addresses
   :align: center
   :width: 900px

   Baseline configuration showing SSH access enabled and IP Allow set to All Addresses (unrestricted management-plane exposure).

---------------------------------------------------------------------

Step 2 – Inspect SSH Access Scope
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Platform → Configuration**.
2. Scroll to **SSH Access**.
3. Review the **SSH IP Allow** configuration.

If configured broadly (e.g., 0.0.0.0/0 or large internal range),
administrative access may be overly permissive.

.. image:: ../_images/ipallow_02_ssh_allow_scope.png
   :alt: SSH IP Allow configuration
   :align: center

---------------------------------------------------------------------

Step 3 – Restrict SSH to Approved Subnet
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Under **SSH IP Allow**, choose:
   **Specify Range**
2. Enter only the authorized admin subnet
   (for example: 10.1.1.0/24).
3. Click **Update**.

.. figure:: ../_images/ip-allowlisting-02-ssh-restricted-to-10.1.1.0-24.png
   :alt: SSH access restricted to management subnet 10.1.1.0/24
   :align: center
   :width: 900px

   Remediation: SSH access restricted to the management subnet (10.1.1.0/24) using IP Allow.

---------------------------------------------------------------------

Step 4 – Validate SSH Restriction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From an authorized administrative host:

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 22

Expected:

* TcpTestSucceeded: True

From an unauthorized host:

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 22

Expected:

* TcpTestSucceeded: False

.. figure:: ../_images/ip-allowlisting-03-ssh-allowed-from-mgmt.png
   :alt: SSH access allowed from management subnet
   :align: center
   :width: 900px

   Validation from the management subnet (10.1.1.0/24) showing SSH access permitted.

.. figure:: ../_images/ip-allowlisting-04-ssh-blocked-from-non-mgmt.png
   :alt: SSH access blocked from non-management network
   :align: center
   :width: 900px

   Validation from a non-management host showing SSH connection attempt timed out due to IP allowlisting.

---------------------------------------------------------------------

Step 5 – Review SNMP Client Allow List
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → SNMP → Agent**.
2. Review the **Client Allow List**.
3. Ensure only approved monitoring systems are listed.

Remove broad entries such as:

* 0.0.0.0/0
* Entire internal ranges

.. figure:: ../_images/ip-allowlisting-05-snmp-client-list.png
   :alt: SNMP Client Allow List configuration
   :align: center
   :width: 900px

   SNMP Client Allow List restricted to approved monitoring systems (no broad network ranges).

---------------------------------------------------------------------

Step 6 – Validate HTTPS (TMUI) Access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From authorized host:

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 443

Expected:

* TcpTestSucceeded: True

From unauthorized host:

.. code-block:: powershell

   Test-NetConnection <mgmt-ip> -Port 443

Expected:

* TcpTestSucceeded: False

.. figure:: ../_images/ip-allowlisting-06-https-validation.png
   :alt: HTTPS management access validation
   :align: center
   :width: 900px

   Validation showing HTTPS (TMUI) access restricted to the management subnet.

---------------------------------------------------------------------

Validation Summary
------------------

After remediation:

* SSH restricted to authorized admin subnet
* HTTPS restricted to authorized admin subnet
* SNMP restricted to monitoring systems
* Unauthorized hosts blocked

Outer Layer Alignment
---------------------

IP Allowlisting protects:

* **Who** may access management services.

Self IP Port Lockdown protects:

* **Where** management services are exposed.

Together they enforce:

* Least privilege
* Deterministic administrative access
* Control-plane isolation
* Zero Trust segmentation principles

Success Criteria
----------------

* Only approved administrative subnet can access SSH and HTTPS
* Only approved monitoring systems can access SNMP
* No broad internal ranges permitted
* Unauthorized access attempts fail
* Management access remains operational for approved sources
