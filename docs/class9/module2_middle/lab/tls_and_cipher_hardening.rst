TLS and Cipher Hardening
========================

TLS and cipher hardening reduces the risk of cryptographic downgrade,
weak cipher negotiation, and non-compliant protocol exposure.

In the Middle Layer, hardened TLS profiles enforce modern protocol
versions and strong cipher suites for application traffic.

This mechanism is a critical Middle Layer cryptographic control.

Executive Summary
-----------------

   Applications must enforce modern TLS versions and strong ciphers.
   Legacy protocols and weak ciphers must be disabled unless explicitly
   required and approved.

   Hardening should be implemented with standardized TLS profiles
   and validated using negotiated handshake testing.

Objective
---------

This lab will:

* Identify current TLS protocol and cipher posture on BIG-IP
* Create hardened Client and Server SSL profiles
* Apply hardened profiles to a virtual server
* Validate negotiated protocol and cipher behavior
* Confirm logging, operational posture, and drift resistance

Hardened Enterprise Reference Design
------------------------------------

The goal is to standardize strong TLS posture at the BIG-IP data plane.

.. note::

   TLS hardening should be implemented using dedicated SSL profiles
   rather than per-application ad hoc settings.

   Use separate profiles for client-side and server-side as needed
   and avoid modifying built-in defaults directly.

.. nwdiag::
   :caption: Reference Design – TLS Termination and Hardened Cipher Profiles
   :name: tls-cipher-hardening-reference-design

   nwdiag {
     internet [shape = cloud];

     network clients   { address = "Clients"; }
     network dmz       { address = "DMZ / Client-side VLAN"; }
     network internal  { address = "Server-side VLAN"; }

     clients -- dmz;

     bigip [description = "BIG-IP\nClient SSL: Hardened\nServer SSL: Hardened (as required)"];

     dmz -- bigip;
     internal -- bigip;

     app [description = "Application Servers"];
     internal -- app;
   }

Recommended TLS Posture
-----------------------

+----------------------+-------------------------------+-------------------------------------------+
| Component            | Setting                        | Recommendation                              |
+======================+===============================+===========================================+
| Client-side TLS      | Minimum TLS version           | TLS 1.2 or TLS 1.3                          |
+----------------------+-------------------------------+-------------------------------------------+
| Client-side TLS      | Legacy protocols              | Disable SSLv3, TLS 1.0, TLS 1.1             |
+----------------------+-------------------------------+-------------------------------------------+
| Cipher suites        | Weak ciphers                  | Disable RC4, 3DES, NULL, EXPORT            |
+----------------------+-------------------------------+-------------------------------------------+
| Cipher suites        | Forward secrecy               | Prefer ECDHE suites                          |
+----------------------+-------------------------------+-------------------------------------------+
| Certificates         | Key strength / algorithm       | Prefer ECDSA or RSA 2048+                  |
+----------------------+-------------------------------+-------------------------------------------+
| Server-side TLS      | Re-encryption requirement      | Enable only if required by policy           |
+----------------------+-------------------------------+-------------------------------------------+

Cipher Policy Guidance
----------------------

+-------------------------------+----------------------------------------------+
| Category                      | Examples                                     |
+===============================+==============================================+
| Strong (preferred)            | ECDHE + AES-GCM, TLS 1.3 ciphers             |
+-------------------------------+----------------------------------------------+
| Acceptable (legacy constrained)| ECDHE + AES-CBC (limited use)                |
+-------------------------------+----------------------------------------------+
| Weak (disable)                | RC4, 3DES, NULL, EXPORT, MD5-based suites    |
+-------------------------------+----------------------------------------------+

.. warning::

   Disabling legacy protocols may impact older clients.
   Any exception must be formally approved, documented,
   and time-bounded with a remediation plan.

GUI Configuration Procedure
---------------------------

Step 1 – Identify Current TLS and Cipher Posture
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Log in to the BIG-IP Configuration Utility.
2. Navigate to **Local Traffic → Virtual Servers**.
3. Select the target virtual server.
4. Review the currently assigned profiles:

   - Client SSL profile (if TLS termination is in use)
   - Server SSL profile (if re-encryption is in use)

5. Document current TLS settings and cipher strings.

Step 2 – Create a Hardened Client SSL Profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **Local Traffic → Profiles → SSL → Client**.
2. Click **Create**.
3. Configure:

   - **Name**: ``clientssl_hardened``
   - **Parent Profile**: choose an appropriate baseline (do not modify defaults)

4. Set protocol versions:

   - Disable legacy protocols (SSLv3, TLS 1.0, TLS 1.1) if present
   - Enable TLS 1.2 and TLS 1.3 according to your policy

5. Configure ciphers:

   - Set a strong cipher string aligned to enterprise standards
   - Prefer forward secrecy (ECDHE) and AEAD ciphers (AES-GCM)

6. Click **Finished**.

Step 3 – Create a Hardened Server SSL Profile (If Required)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the application requires TLS between BIG-IP and servers:

1. Navigate to **Local Traffic → Profiles → SSL → Server**.
2. Click **Create**.
3. Configure:

   - **Name**: ``serverssl_hardened``
   - **Parent Profile**: choose an approved baseline

4. Configure protocol versions and ciphers consistent with server capabilities
   and security policy.

5. Click **Finished**.

Step 4 – Apply Hardened Profiles to the Virtual Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **Local Traffic → Virtual Servers**.
2. Select the target virtual server.
3. In the **Configuration** section, update assigned profiles:

   - Replace existing Client SSL profile with ``clientssl_hardened``
   - If applicable, replace Server SSL profile with ``serverssl_hardened``

4. Click **Update**.

Step 5 – Confirm Certificate and Chain Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Navigate to **Local Traffic → SSL Certificates**.
2. Verify the correct certificate and chain are applied
   through the Client SSL profile.

3. Confirm certificate validity:

   - Correct CN/SAN coverage
   - Valid expiration window
   - Strong key algorithm and size

Verification
------------

GUI Verification
~~~~~~~~~~~~~~~~

1. Navigate to **Local Traffic → Virtual Servers**.
2. Confirm the virtual server has:

   - Client SSL: ``clientssl_hardened``
   - Server SSL: ``serverssl_hardened`` (if applicable)

3. Navigate to **System → Logs → Local Traffic**.
4. Confirm there are no SSL negotiation errors for legitimate clients.

CLI Verification (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

From a client host, validate negotiated protocol and cipher.

Test TLS 1.2:

.. code-block:: bash

   openssl s_client -connect <vs-ip>:443 -tls1_2

Test TLS 1.3:

.. code-block:: bash

   openssl s_client -connect <vs-ip>:443 -tls1_3

Attempt a blocked legacy protocol:

.. code-block:: bash

   openssl s_client -connect <vs-ip>:443 -tls1

Expected:

* TLS 1.2/1.3 succeeds (as allowed)
* TLS 1.0 fails (blocked)

Network Validation
~~~~~~~~~~~~~~~~~~

Perform a cipher scan from a test host:

.. code-block:: bash

   nmap --script ssl-enum-ciphers -p 443 <vs-ip>

Expected:

* Weak ciphers not offered
* Only approved cipher suites present
* Protocol versions align with policy

iHealth Validation
------------------

Upload a QKView and confirm:

* No weak protocol exposure findings
* No weak cipher suite findings
* TLS profiles align with enterprise best practices
* No misconfiguration indicators for SSL profiles

Instructor Notes
----------------

.. admonition:: Instructor Guidance
   :class: tip

   Emphasize the difference between:
   * Client-side TLS posture (internet-facing requirements)
   * Server-side TLS posture (internal capability constraints)

   Reinforce that hardening should be performed using dedicated SSL profiles,
   not by modifying default profiles in place.

Lab Challenge – Legacy Compatibility Scenario
---------------------------------------------

Scenario:

A legacy client requires TLS 1.0 and fails after hardening.

Student Tasks:

1. Confirm failure using protocol-specific testing.
2. Validate the current hardened profile configuration.
3. Implement an exception path that does not weaken the primary virtual server.

   Examples:
   * Separate virtual server on a restricted VLAN
   * Separate Client SSL profile limited to the specific legacy requirement
   * Time-bounded exception with audit logging

4. Confirm the primary service remains hardened.

Success Criteria
~~~~~~~~~~~~~~~~

* Primary virtual server supports only approved TLS versions and ciphers
* Weak protocols and ciphers are not offered
* Legitimate clients negotiate successfully
* Any legacy exception is isolated, documented, and time-bounded

Advanced Exercise
~~~~~~~~~~~~~~~~~

Create two Client SSL profiles:

1. ``clientssl_strict`` – TLS 1.3 preferred, strongest ciphers only
2. ``clientssl_compatible`` – TLS 1.2 allowed with limited approved suites

Apply them to two separate virtual servers and validate negotiation
differences using openssl and nmap scanning.