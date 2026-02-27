Administrative TLS Hardening – TMUI & iControl REST
===================================================

Administrative HTTPS services on the BIG-IP management interface
must enforce modern TLS versions and strong cipher suites.

This lab hardens TLS posture for:

* TMUI (HTTPS management interface)
* iControl REST (management-plane API)

This mechanism is a critical Middle Layer cryptographic control.

This lab focuses on **management-plane TLS hardening**.
Data-plane TLS hardening for application virtual servers
is covered in the TLS and Cipher Hardening lab.

Executive Summary
-----------------

Administrative interfaces must not support legacy TLS versions
or weak cipher suites.

Weak protocol exposure on the management interface increases
the risk of downgrade attacks, cryptographic exploitation,
and compliance violations.

Hardening must be validated using deterministic handshake testing.

Threat Scenario
---------------

In the absence of administrative TLS hardening:

* Attackers may negotiate TLS 1.0 or TLS 1.1.
* Weak ciphers (e.g., 3DES) may be offered.
* Downgrade attacks may force weaker protocol selection.
* Credential interception risk increases under legacy crypto.
* Compliance audits (PCI DSS, NIST) may flag non-compliant exposure.

Administrative TLS hardening reduces these risks by enforcing
modern cryptographic standards on management-plane services.

Objective
---------

This lab will:

* Observe baseline TLS posture of the management interface
* Harden management TLS protocol versions
* Enforce strong cipher posture
* Validate deterministic protocol enforcement
* Confirm administrative access remains functional

.. warning::

   Modifying management-plane TLS settings can result in
   administrative lockout.

   Before applying changes:

   * Ensure console access is available.
   * Confirm an authorized administrative host is accessible.
   * Do NOT close your current session until validation succeeds.

Hardened Enterprise Reference Design
------------------------------------

The management interface should:

* Allow TLS 1.2 and TLS 1.3 only
* Disable TLS 1.0 and TLS 1.1
* Enforce strong ECDHE-based cipher suites
* Align with enterprise cryptographic policy

Middle Layer Cohesion
---------------------

Within the Middle Layer:

* MFA validates **administrative identity**.
* Administrative TLS hardening protects **management transport security**.
* API Access Control enforces **privilege boundaries**.

Together, these controls prevent credential abuse,
downgrade attacks, and unauthorized configuration changes.

---------------------------------------------------------------------

Phase 1 – Baseline Management TLS Observation
---------------------------------------------

From the authorized administrative host:

Test TLS 1.0 (Expected: May Succeed in Default State)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1

Test TLS 1.1 (Expected: May Succeed in Default State)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1_1

Test TLS 1.2 (Expected: Success)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1_2

Capture:

* Negotiated protocol
* Negotiated cipher suite

.. figure:: ../_images/administrative-tls-hardening-01-baseline-tls-observation.png
   :align: center
   :alt: Baseline TLS handshake against management interface

   Baseline TLS handshake results for the management interface prior to hardening.

If TLS 1.0 or TLS 1.1 succeeds, legacy protocol exposure is confirmed.

---------------------------------------------------------------------

Phase 2 – Harden Management TLS Configuration
---------------------------------------------

1. Navigate to the appropriate management SSL configuration
   interface (version-dependent).

2. Disable:

   * TLS 1.0
   * TLS 1.1

3. Enable:

   * TLS 1.2
   * TLS 1.3

4. Enforce a strong cipher posture aligned to enterprise standards.

.. note::

   TLS 1.3 cipher suites are managed separately from legacy
   cipher strings in newer TMOS versions. Ensure alignment
   with enterprise cryptographic policy.

Apply configuration.

.. figure:: ../_images/administrative-tls-hardening-02-hardened-config.png
   :align: center
   :alt: Hardened management TLS configuration screen

   Management TLS configuration updated to disable legacy protocols
   and enforce modern cryptographic standards.

---------------------------------------------------------------------

Phase 3 – Deterministic Validation
-----------------------------------

Test TLS 1.0 (Expected: Failure)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1

Expected Result:

* Handshake failure
* No cipher negotiated

.. figure:: ../_images/administrative-tls-hardening-03-tls10-blocked.png
   :align: center
   :alt: TLS 1.0 handshake failure after hardening

Test TLS 1.1 (Expected: Failure)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1_1

Expected Result:

* Handshake failure
* No cipher negotiated

.. figure:: ../_images/administrative-tls-hardening-04-tls11-blocked.png
   :align: center
   :alt: TLS 1.1 handshake failure after hardening

Test TLS 1.2 (Expected: Success)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1_2

.. figure:: ../_images/administrative-tls-hardening-05-tls12-success.png
   :align: center
   :alt: TLS 1.2 handshake success after hardening

Test TLS 1.3 (Expected: Success)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -tls1_3

.. figure:: ../_images/administrative-tls-hardening-06-tls13-success.png
   :align: center
   :alt: TLS 1.3 handshake success after hardening

Optional – Weak Cipher Validation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Attempt a deprecated cipher (Expected: Failure)

.. code-block:: bash

   openssl s_client -connect <mgmt-ip>:443 -cipher DES-CBC3-SHA

Expected Result:

* Handshake failure

---------------------------------------------------------------------

Deterministic Validation Matrix
-------------------------------

+-------------------------------------+---------------------+---------------------+
| Test Case                           | Expected Result     | Observed Result     |
+=====================================+=====================+=====================+
| TLS 1.0 handshake                   | Fail                |                     |
+-------------------------------------+---------------------+---------------------+
| TLS 1.1 handshake                   | Fail                |                     |
+-------------------------------------+---------------------+---------------------+
| TLS 1.2 handshake                   | Success             |                     |
+-------------------------------------+---------------------+---------------------+
| TLS 1.3 handshake                   | Success             |                     |
+-------------------------------------+---------------------+---------------------+
| Weak cipher negotiation attempt     | Fail                |                     |
+-------------------------------------+---------------------+---------------------+

---------------------------------------------------------------------

Security Controls Validated
---------------------------

+-------------------------------------+-----------+
| Control                             | Validated |
+=====================================+===========+
| Legacy protocol elimination         | ✔         |
+-------------------------------------+-----------+
| Strong cipher enforcement           | ✔         |
+-------------------------------------+-----------+
| Deterministic handshake validation  | ✔         |
+-------------------------------------+-----------+
| Management-plane cryptographic hardening | ✔   |
+-------------------------------------+-----------+

---------------------------------------------------------------------

Success Criteria
----------------

* TLS 1.0 and TLS 1.1 are disabled on the management interface
* TLS 1.2 and TLS 1.3 negotiate successfully
* Weak ciphers are not offered
* Administrative access remains functional
* Deterministic enforcement confirmed via handshake testing