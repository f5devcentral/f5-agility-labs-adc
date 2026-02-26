API Access Control – AS3 & Declarative Onboarding (CI Validation)
==================================================================

The Middle Layer enforces authentication, authorization, and API control
over the BIG-IP control plane. This lab validates secure access to
service extensions (AS3 and Declarative Onboarding) using token-based
authentication and role-based authorization.

Objective
---------

Validate secure, deterministic API access to BIG-IP service extensions
while enforcing:

* Token-based authentication
* Role-based authorization
* Least privilege iteration
* Token lifecycle management
* Management-plane network isolation
* Deterministic CI behavior

Threat Model
------------

This lab assumes an adversary may attempt to:

* Reach the BIG-IP management plane from unauthorized networks (control-plane exposure).
* Reuse leaked credentials or embed static passwords in automation (credential abuse).
* Steal or replay authentication tokens (token theft / replay).
* Abuse overly privileged service accounts (excessive authorization).
* Exploit differences between partition-scoped roles and device-scoped service extensions (RBAC gaps).
* Submit unauthorized or destructive declarations via AS3/DO (configuration integrity risk).

Controls validated in this lab mitigate these threats via:

* Management-plane network isolation (jumphost-only access).
* Token-based authentication with enforced expiration.
* Role-based authorization enforcement (demonstrated least-privilege attempt and escalation).
* Deterministic, idempotent declarative deployment suitable for CI/CD.

Architecture Overview
---------------------

.. figure:: ../images/module2/api_access_control_flow.png
   :align: center
   :alt: API Access Control Flow

   Token-based CI access to BIG-IP service extensions via management plane.

Control Flow:

1. CI/Jumphost authenticates to ``/mgmt/shared/authn/login``
2. BIG-IP issues short-lived auth token
3. CI uses ``X-F5-Auth-Token`` header
4. REST framework validates:
   * Authentication
   * Role authorization
   * Partition scope
5. Service extension (AS3) processes declaration

Phase 1 – Baseline Validation
-----------------------------

Verify AS3 and DO Installed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Navigate to:

``iApps → Package Management LX``

Confirm:

* ``f5-appsvcs``
* ``f5-declarative-onboarding``

Management Plane Segmentation Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

From the Windows Jumphost:

.. code-block:: powershell

   curl.exe -k https://10.1.1.6/mgmt/shared/echo

Expected:
JSON response returned.

From an external workstation:

.. code-block:: bash

   curl -k https://10.1.1.6/mgmt/shared/echo

Expected:
Connection failure.

.. admonition:: Security Principle
   :class: important

   The BIG-IP management interface must only be reachable from authorized
   management networks (CI/jumphost subnet). API security depends on
   network segmentation as the first control boundary.

Phase 2 – Least Privilege Attempt (Intentional Failure)
-------------------------------------------------------

Restrict Service Account
^^^^^^^^^^^^^^^^^^^^^^^^

Modify ``svc_as3do``:

* Role: Application Editor
* Partition: Common
* Terminal Access: Disabled

Re-authenticate and test:

.. code-block:: powershell

   curl.exe -k -s "$BIGIP/mgmt/shared/appsvcs/info" -H "X-F5-Auth-Token: $TOKEN"

Expected:

.. code-block:: text

   401 Authorization failed

Why This Fails
^^^^^^^^^^^^^^

AS3 and Declarative Onboarding operate under:

``/mgmt/shared/*``

These are device-scoped service extensions, not partition-scoped LTM objects.

Partition-level roles (Application Editor) cannot invoke shared
extension endpoints.

.. admonition:: Architectural Insight
   :class: note

   Service extensions execute at the device scope. Partition scoping
   applies only to configuration objects (virtual servers, pools, etc.),
   not to shared REST extensions.

This failure is expected and confirms authorization enforcement.

Phase 3 – Secure Escalation (Enterprise Balance)
------------------------------------------------

Adjust ``svc_as3do``:

* Role: Resource Administrator
* Partition: All
* Terminal Access: Disabled

Re-authenticate and test:

.. code-block:: powershell

   curl.exe -k -s "$BIGIP/mgmt/shared/appsvcs/info" -H "X-F5-Auth-Token: $TOKEN"

Expected:

.. code-block:: json

   {"version":"3.53.0","release":"7","schemaCurrent":"3.53.0","schemaMinimum":"3.0.0"}

This confirms:

* Authentication success
* Proper authorization
* Correct privilege boundary

Phase 4 – Token Lifecycle Validation
-------------------------------------

Allow token to expire or force expiration.

Expected response:

.. code-block:: text

   401 X-F5-Auth-Token has expired

Re-authenticate:

.. code-block:: powershell

   $login = curl.exe -k -s -X POST "$BIGIP/mgmt/shared/authn/login" `
     -H "Content-Type: application/json" `
     -d "{ `"username`":`"$USER`", `"password`":`"$PASS`", `"loginProviderName`":`"tmos`" }"

   $TOKEN = ($login | ConvertFrom-Json).token.token

This validates:

* Token TTL enforcement
* Pipeline resiliency requirement
* Secure re-authentication flow

Phase 5 – Deterministic CI Deployment
--------------------------------------

Deploy minimal AS3 declaration:

.. code-block:: powershell

   curl.exe -k -s -X POST "$BIGIP/mgmt/shared/appsvcs/declare" `
     -H "Content-Type: application/json" `
     -H "X-F5-Auth-Token: $TOKEN" `
     --data-binary "@ci_validation.json"

Expected:

.. code-block:: json

   {
     "results": [
       {
         "code": 200,
         "message": "no change"
       }
     ]
   }

This confirms:

* Idempotent declaration behavior
* Deterministic CI execution
* Proper service account permissions

Final Hardened Configuration
----------------------------

Service Account: ``svc_as3do``

* Role: Resource Administrator
* Partition: All
* Terminal Access: Disabled
* Management Plane: Restricted to jumphost subnet
* Authentication: Token-based in CI pipelines

Security Controls Validated
---------------------------

+-------------------------------------+-----------+
| Control                             | Validated |
+=====================================+===========+
| Token-based authentication          | ✔         |
+-------------------------------------+-----------+
| Token expiration enforcement        | ✔         |
+-------------------------------------+-----------+
| Role-based authorization            | ✔         |
+-------------------------------------+-----------+
| Partition scoping behavior          | ✔         |
+-------------------------------------+-----------+
| Least privilege iteration           | ✔         |
+-------------------------------------+-----------+
| Management plane segmentation       | ✔         |
+-------------------------------------+-----------+
| CI determinism (idempotent deploy)  | ✔         |
+-------------------------------------+-----------+

Detection & Evidence
--------------------

The following system locations provide operational evidence of API access,
authentication failures, and service extension activity.

Authentication & Authorization Events
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* ``/var/log/restjavad.*`` – iControl REST authentication and authorization events
* ``/var/log/ltm`` – Management login events
* ``System → Logs → Audit`` (GUI) – User and configuration changes

Expired Token / 401 Events
^^^^^^^^^^^^^^^^^^^^^^^^^^

Failed token usage generates REST framework responses:

* HTTP 401 Unauthorized
* Logged under REST framework logs (restjavad)

AS3 / Service Extension Activity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* ``/var/log/restnoded/restnoded.log`` – AS3 request handling
* ``/var/log/ltm`` – Configuration commit operations
* ``iApps → Application Services`` – Declarative deployment state

Network Access Validation
^^^^^^^^^^^^^^^^^^^^^^^^^

Management plane access can be verified via:

* Self IP Port Lockdown configuration (Outer Layer)
* AFM management ACLs (if provisioned)
* Network firewall policy enforcement

These log locations provide traceability for:

* Unauthorized access attempts
* Token misuse
* Service account activity
* Declarative configuration changes

Executive Summary
-----------------

This lab demonstrates secure API access enforcement for BIG-IP service
extensions through layered controls:

* Network isolation (Outer Layer)
* Token-based authentication (Middle Layer)
* Role-based authorization (Middle Layer)
* Deterministic declarative deployment (Operational Integrity)

Together, these controls align with Zero Trust principles and enterprise
CI/CD security practices.