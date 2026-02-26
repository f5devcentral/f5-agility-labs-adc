Multi-Factor Authentication
===========================

Multi-Factor Authentication (MFA) strengthens administrative access by
requiring a second factor beyond username and password.

In the Middle Layer, MFA prevents credential-only administrative compromise
by integrating BIG-IP administrative authentication with a centralized
AAA or identity provider that enforces MFA for TMUI and SSH access paths.

This mechanism is a critical Middle Layer identity control.

.. admonition:: Executive Summary
   :class: important

   Administrative access must require MFA whenever supported by
   enterprise identity infrastructure.

   BIG-IP should delegate authentication to a centralized AAA or IdP
   that enforces MFA, while maintaining deterministic local fallback
   for break-glass recovery.

Objective
---------

This lab will:

* Configure BIG-IP to use centralized authentication suitable for MFA
* Validate remote AAA reachability before enabling authentication
* Enable deterministic fallback to local authentication
* Validate successful MFA-based login
* Validate failure conditions and fallback behavior
* Confirm operational visibility and audit posture

.. warning::

   Misconfiguration of remote authentication can result in administrative lockout.

   Before changing authentication source:

   * Ensure console or SSH access is available.
   * Confirm a documented break-glass local administrator account exists.
   * Validate network reachability to the remote AAA server.
   * Test login before ending the session.

Hardened Enterprise Reference Design
------------------------------------

The goal is to integrate BIG-IP administrative authentication with a
centralized identity provider that enforces MFA.

.. note::

   BIG-IP can integrate with enterprise AAA systems such as:

   * RADIUS
   * TACACS+
   * LDAP (when upstream MFA is enforced)
   * SAML (where supported for administrative access)

   The reference design assumes MFA enforcement occurs at the upstream
   identity provider. BIG-IP consumes the authentication decision.

.. nwdiag::
   :caption: Reference Design – BIG-IP Administrative Access with Centralized MFA
   :name: mfa-reference-design

   nwdiag {
     internet [shape = cloud];

     network admin     { address = "Authorized Admin Subnet"; }
     network bigipmgmt { address = "BIG-IP Management IP"; }
     network idp       { address = "Enterprise AAA / IdP (MFA Enforced)"; }

     admin -- bigipmgmt;
     bigipmgmt -- idp;
   }

Recommended MFA Posture
-----------------------

+----------------------+----------------------------+-------------------------------------------+
| Access Path          | Authentication Source      | Requirement                                |
+======================+============================+===========================================+
| TMUI (HTTPS/443)     | Centralized AAA / IdP      | MFA enforced by AAA / IdP                  |
+----------------------+----------------------------+-------------------------------------------+
| SSH (TCP/22)         | Centralized AAA            | MFA or strong step-up policy               |
+----------------------+----------------------------+-------------------------------------------+
| Break-glass account  | Local (restricted)         | Stored securely, monitored, least use      |
+----------------------+----------------------------+-------------------------------------------+

GUI Configuration Procedure
---------------------------

Step 1 – Confirm Administrative Baseline
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to TMUI from the lab jump host.
2. Navigate to **System → Users → User List**.
3. Confirm the presence of a documented local break-glass administrator.
4. Verify partition access and role assignments.

Step 2 – Validate AAA Reachability (Required)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before enabling remote authentication, validate connectivity.

Enter TMSH:

.. code-block:: bash

   run util bash
   ping -c 3 <radius-server-ip>

Expected:

* Successful ICMP replies.
* No "Destination Host Unreachable" errors.

If unreachable, correct routing before proceeding.

Step 3 – Configure Remote Authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Users → Authentication**.
2. Set:

   * Authentication: **Advanced**
   * User Directory: **Remote – RADIUS**
   * Service Type: **Authenticate Only**
   * Fallback to Local: **Enabled**

3. Configure:

   * Host (RADIUS server IP)
   * Port (default 1812)
   * Shared secret

4. Click **Update**.

Fallback Behavior
~~~~~~~~~~~~~~~~~

If the RADIUS server is reachable:

* Authentication is delegated to the AAA server.

If the RADIUS server is unreachable:

* BIG-IP attempts remote authentication.
* After timeout, authentication falls back to local accounts (if enabled).
* If fallback is disabled, administrative access may fail entirely.

Step 4 – Configure Remote Role Groups
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Users → Remote Role Groups**.
2. Click **Create**.
3. Define:

   * Attribute String (based on AAA attribute policy)
   * Assigned Role (e.g., Administrator)
   * Partition Access (All or as required)

4. Click **Finished**.

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

1. Open a browser from the lab jump host.
2. Navigate to:

   https://<mgmt-ip>

3. Log in using a centralized AAA user.
4. Complete MFA challenge (if enforced by IdP).
5. Confirm successful login.

Negative Test – Remote Unreachable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Temporarily configure an unused RADIUS IP.

Expected:

* Login fails or times out.
* If fallback is enabled and valid local credentials are used,
  login succeeds.

Packet-Level Verification (Advanced)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From TMSH:

.. code-block:: bash

   run util bash
   tcpdump -ni mgmt udp port 1812

Attempt login.

Expected:

* RADIUS Access-Request packets observed.

iHealth Validation
------------------

Upload a QKView and confirm:

* Authentication source is remote.
* Fallback is configured.
* No findings related to weak administrative authentication posture.

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   Reinforce the following principles:

   * Always validate AAA reachability before enabling remote authentication.
   * Always configure deterministic fallback.
   * Always test login before ending configuration.
   * Remote authentication without reachability validation is operationally unsafe.

Lab Challenge – Authentication Drift Scenario
---------------------------------------------

Scenario:

A local administrator account is being used instead of centralized MFA.

Student Tasks:

1. Identify the account under **System → Users**.
2. Confirm it bypasses centralized authentication.
3. Restrict or document it as break-glass only.
4. Validate centralized MFA is used for routine administrative access.

Success Criteria
~~~~~~~~~~~~~~~~

* Remote authentication source configured.
* RADIUS reachability validated.
* Remote role group mapped correctly.
* Fallback to local authentication enabled.
* TMUI login validated.
* RADIUS traffic observable via packet capture (optional).
* Local-only routine administrative access eliminated.

Advanced Exercise
~~~~~~~~~~~~~~~~~

Simulate AAA outage:

1. Change RADIUS IP to unused address.
2. Attempt login.
3. Validate fallback behavior.
4. Restore correct server IP.
5. Confirm normal operation resumes.