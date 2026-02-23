Multi-Factor Authentication
===========================

Multi-Factor Authentication (MFA) strengthens administrative access by
requiring a second factor beyond username and password.

In the Middle Layer, MFA reduces the impact of credential compromise
by enforcing stronger authentication for TMUI and SSH access paths.

This mechanism is a critical Middle Layer identity control.

Executive Summary
-----------------

   Administrative access must require MFA whenever supported by
   enterprise identity infrastructure.

   MFA should be enforced through centralized authentication where possible,
   with BIG-IP configured to use external identity services for consistency
   and auditability.

Objective
---------

This lab will:

* Configure BIG-IP to use centralized authentication suitable for MFA
* Apply authentication configuration to administrative access
* Validate successful MFA-based login
* Validate unauthorized access and failed MFA attempts are denied
* Confirm logging and operational posture

Hardened Enterprise Reference Design
------------------------------------

The goal is to integrate BIG-IP administrative authentication with a
centralized identity provider that enforces MFA.

.. note::

   BIG-IP can integrate with enterprise AAA systems such as RADIUS,
   TACACS+, or SAML-based access to TMUI depending on organizational standards.

   The reference design assumes MFA is enforced by the upstream identity system,
   while BIG-IP consumes the resulting authentication decision.

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
| SSH (TCP/22)         | Centralized AAA (RADIUS/   | MFA (or strong step-up) via AAA policy     |
|                      | TACACS+)                   |                                           |
+----------------------+----------------------------+-------------------------------------------+
| Break-glass account  | Local (restricted)         | Stored securely, monitored, least use      |
+----------------------+----------------------------+-------------------------------------------+

.. warning::

   Do not rely on local-only authentication for routine administration.
   Local accounts should be limited to break-glass use, strongly protected,
   and continuously monitored.

GUI Configuration Procedure
---------------------------

Step 1 – Confirm Administrative Access Baseline
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **System → Users**.
3. Identify current administrative accounts and roles.
4. Confirm there is a documented break-glass account with restricted usage
   according to enterprise policy.

Step 2 – Configure Remote Authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This step configures BIG-IP to use centralized authentication suitable for MFA.

1. Navigate to **System → Users → Authentication**.
2. Select the authentication method approved by your enterprise standard:

   * RADIUS
   * TACACS+
   * LDAP (when combined with an upstream MFA enforcement mechanism)
   * SAML (if supported in your TMOS version for administrative access)

3. Configure the remote authentication server settings:

   - Server address(es)
   - Shared secret or key material (where applicable)
   - Timeout / retry parameters
   - Source IP / routing considerations (ensure BIG-IP can reach AAA/IdP)

4. Click **Update** (or **Apply**) to save changes.

Step 3 – Enforce MFA for TMUI Access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Users → User List**.
2. Confirm that administrative users are configured to authenticate via the
   centralized method where applicable.
3. Validate that the AAA/IdP policy requires MFA for administrative roles.

If your enterprise uses an access proxy in front of TMUI (recommended in some environments),
ensure MFA is enforced at the proxy/IdP layer and TMUI is reachable only through that path.

Step 4 – Enforce MFA for SSH Access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **System → Services**.
2. Confirm SSH is enabled only if required.
3. Confirm SSH authentication is aligned with centralized AAA policy.

If SSH is restricted to centralized AAA, ensure local password authentication is not used
for routine administration unless explicitly approved.

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

From an authorized administrative host:

1. Open a browser and navigate to:

   https://<mgmt-ip>

2. Log in using a user configured for centralized authentication.
3. Complete the MFA challenge presented by the enterprise AAA/IdP workflow.
4. Confirm successful login.

Negative test:

1. Attempt login using an account without MFA requirements.
2. Confirm access is denied by policy.

CLI Verification (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

From an authorized host, attempt SSH login:

.. code-block:: bash

   ssh <user>@<mgmt-ip>

Expected:

* Authentication requires AAA interaction
* MFA challenge occurs according to enterprise AAA design
* Access is denied if MFA is not satisfied

Network Validation
~~~~~~~~~~~~~~~~~~

Confirm BIG-IP can reach AAA/IdP services:

.. code-block:: bash

   nc -vz <aaa-server-ip> 1812
   nc -vz <aaa-server-ip> 49

Expected:

* Connectivity to required authentication ports succeeds

iHealth Validation
------------------

Upload a QKView and confirm:

* Administrative authentication is not local-only for routine access
* Remote authentication configuration aligns with enterprise policy
* No high-risk findings indicating weak administrative authentication posture

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   MFA effectiveness depends on consistent enforcement across all access paths.

   Reinforce that:
   * MFA must apply to both TMUI and SSH access where operationally required
   * Local accounts are for break-glass only
   * Logs and audit trails must capture authentication outcomes

Lab Challenge – Local Authentication Drift Scenario
---------------------------------------------------

Scenario:

A local administrator account was created for convenience and is being used
instead of centralized MFA.

Student Tasks:

1. Identify the account under **System → Users**.
2. Confirm it bypasses centralized MFA.
3. Disable or restrict the account according to enterprise break-glass policy.
4. Confirm centralized authentication with MFA is used for routine admin access.

Success Criteria
~~~~~~~~~~~~~~~~

* TMUI administrative access requires centralized authentication with MFA
* SSH administrative access requires centralized authentication with MFA (where applicable)
* Local-only routine admin access is eliminated
* Authentication events are logged and auditable

Advanced Exercise
~~~~~~~~~~~~~~~~~

Simulate an AAA service outage:

1. Validate break-glass access procedure (without disabling centralized AAA permanently).
2. Restore normal AAA operation.
3. Confirm all routine administrative access returns to MFA-enforced centralized authentication.