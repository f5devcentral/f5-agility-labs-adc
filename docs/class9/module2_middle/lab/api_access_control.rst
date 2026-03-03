API Access Control – Role-Based Enforcement (AS3 & Declarative Onboarding)
===========================================================================

The Middle Layer enforces authentication and authorization controls
over the BIG-IP control plane. This lab validates secure access
to service extensions (AS3 and Declarative Onboarding) using
role-based authorization and deterministic REST validation.

This lab assumes:

* Network-layer segmentation (Outer Layer) is enforced.
* Administrative authentication (MFA) is in place or delegated to centralized AAA.

This lab focuses on authorization enforcement at the REST framework layer.

Authentication answers:

* Who are you?

Authorization answers:

* What are you allowed to do?

---------------------------------------------------------------------

Executive Summary
-----------------

This lab demonstrates secure API access enforcement for BIG-IP
service extensions through layered controls:

* Network segmentation (Outer Layer)
* REST authentication
* Role-based authorization
* Device-scoped extension protection
* Deterministic authorization failure behavior

Together, these controls enforce least privilege and protect
declarative service extensions from unauthorized invocation.

---------------------------------------------------------------------

Objective
---------

Validate secure, deterministic API access to BIG-IP service extensions
while enforcing:

* Role-based authorization
* Least privilege validation
* Device-scoped extension protection
* Management-plane isolation
* Deterministic authorization failure behavior

---------------------------------------------------------------------

Threat Model
------------

This lab assumes an adversary may attempt to:

* Use valid credentials with insufficient privileges to invoke device-scoped APIs.
* Access the BIG-IP management API from an authorized network without sufficient privileges.
* Abuse overly privileged accounts to deploy unauthorized configurations.
* Invoke device-scoped service extensions (AS3/DO) using partition-scoped roles.
* Submit unauthorized declarative configurations.

Controls validated in this lab mitigate these threats via:

* Management-plane segmentation.
* Strict REST authentication enforcement.
* Role-based authorization at the REST framework layer.
* Explicit denial of device-scoped extension access to non-administrative roles.

---------------------------------------------------------------------

Architecture Overview
---------------------

AS3 and Declarative Onboarding operate under:

::

   /mgmt/shared/*

These endpoints are:

* Device-scoped
* Not partition-scoped
* Protected by REST framework authorization
* Not configurable for partition-level delegation

Authorization Flow:

1. Client authenticates using REST Basic Authentication.
2. REST framework validates credentials.
3. Authorization policy evaluates assigned role.
4. Service extension executes only if role permits.

---------------------------------------------------------------------

Lab Environment Reference
-------------------------

Management IP of BIG-IP:

::

   10.1.1.5

All API testing in this lab will target:

::

   https://10.1.1.5

---------------------------------------------------------------------

Phase 1 – Baseline Validation
-----------------------------

Verify AS3 and DO Installed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Navigate to:

::

   iApps → Package Management LX

Confirm the following packages are installed:

* ``f5-appsvcs``
* ``f5-declarative-onboarding``

.. image:: ../_images/api_access_control-as3_do_check.png
   :alt: Package Management LX showing AS3 and Declarative Onboarding installed
   :align: center
   :width: 900px

---------------------------------------------------------------------

Baseline API Reachability (Administrator)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

From Git Bash on the Windows Jumphost:

.. code-block:: bash

   curl -sk -u 'admin:f5Twister!' \
   https://10.1.1.5/mgmt/shared/appsvcs/info \
   -w "\nHTTP Status: %{http_code}\n"

Expected:

* HTTP Status: 200
* JSON response containing AS3 version information

.. image:: ../_images/api_access_control-baseline_api_reachablity_jumphost.png
   :alt: Git Bash curl to AS3 info endpoint returning HTTP 200
   :align: center
   :width: 900px

---------------------------------------------------------------------

Phase 2 – Least Privilege Enforcement (Operator Denied)
-------------------------------------------------------

Create Restricted User
^^^^^^^^^^^^^^^^^^^^^^

Create a local user with limited privileges:

* Username: ``api_test_user``
* Role: ``Operator``
* Partition: ``Common``
* Terminal Access: ``Disabled``

.. image:: ../_images/api_access_control-create_local_user.png
   :alt: Create local user api_test_user with Operator role
   :align: center
   :width: 900px

---------------------------------------------------------------------

Create AS3 Test Declaration (Git Bash)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a minimal AS3 declaration payload locally on the jumphost. This
payload is intentionally minimal and used only to validate authorization
behavior.

From Git Bash:

.. code-block:: bash

   cat <<'EOF' > test.json
   {
     "class": "AS3",
     "declaration": {
       "class": "ADC",
       "schemaVersion": "3.0.0",
       "id": "lab-test",
       "label": "lab-test"
     }
   }
   EOF

Verify file creation:

.. code-block:: bash

   ls -l test.json

Expected:

* ``test.json`` exists and has a non-zero size

.. image:: ../_images/api_access_control-create_declaration.png
   :alt: Create local user api_test_user with Operator role
   :align: center
   :width: 900px

---------------------------------------------------------------------

Test AS3 Access (GET)
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   curl -sk -u 'api_test_user:f5Twister!' \
   https://10.1.1.5/mgmt/shared/appsvcs/info \
   -w "\nHTTP Status: %{http_code}\n"

Expected:

* HTTP Status: 401

.. image:: ../_images/api_access_control-as3_test_acess.png
   :alt: Operator role denied (HTTP 401) when querying AS3 info endpoint
   :align: center
   :width: 900px

---------------------------------------------------------------------

Test Declarative Onboarding Access (GET)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   curl -sk -u 'api_test_user:f5Twister!' \
   https://10.1.1.5/mgmt/shared/declarative-onboarding/info \
   -w "\nHTTP Status: %{http_code}\n"

Expected:

* HTTP Status: 401

.. image:: ../_images/do_test_access.png
   :alt: Operator role denied (HTTP 401) when querying Declarative Onboarding info endpoint
   :align: center
   :width: 900px

---------------------------------------------------------------------

Test AS3 Declaration Attempt (POST)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   curl -sk -u 'api_test_user:f5Twister!' \
     -H "Content-Type: application/json" \
     -X POST \
     https://10.1.1.5/mgmt/shared/appsvcs/declare \
     --data-binary @test.json \
     -w "\nHTTP Status: %{http_code}\n"

Expected:

* HTTP Status: 401

.. image:: ../_images/as3_test_declaration_post.png
   :alt: Operator role denied (HTTP 401) when attempting AS3 declaration POST
   :align: center
   :width: 900px

---------------------------------------------------------------------

Why This Fails
^^^^^^^^^^^^^^

AS3 and Declarative Onboarding operate under:

::

   /mgmt/shared/*

These are device-scoped service extensions.

Partition-scoped roles (Operator, Application Editor) cannot invoke
shared extension endpoints.

This confirms proper REST authorization enforcement.

---------------------------------------------------------------------

Phase 3 – Administrative Authorization (Allowed)
------------------------------------------------

Test Administrative Access
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   curl -sk -u 'admin:f5Twister!' \
   https://10.1.1.5/mgmt/shared/appsvcs/info \
   -w "\nHTTP Status: %{http_code}\n"

Expected:

* HTTP Status: 200
* JSON response returned

.. image:: ../_images/as3_test_admin_auth_access.png
   :alt: Administrator permitted (HTTP 200) when querying AS3 info endpoint
   :align: center
   :width: 900px

---------------------------------------------------------------------

Deterministic Validation Matrix
-------------------------------

+-------------------------------------------+-----------------------------+
| Test Case                                 | Expected Result             |
+===========================================+=============================+
| Operator GET AS3 info                     | HTTP 401                    |
+-------------------------------------------+-----------------------------+
| Operator GET DO info                      | HTTP 401                    |
+-------------------------------------------+-----------------------------+
| Operator POST AS3 declare                 | HTTP 401                    |
+-------------------------------------------+-----------------------------+
| Administrator GET AS3 info                | HTTP 200                    |
+-------------------------------------------+-----------------------------+

---------------------------------------------------------------------

Security Controls Validated
---------------------------

+-------------------------------------------+-----------+
| Control                                   | Validated |
+===========================================+===========+
| REST authentication enforcement           | ✔         |
+-------------------------------------------+-----------+
| Role-based authorization enforcement      | ✔         |
+-------------------------------------------+-----------+
| Device-scoped extension protection        | ✔         |
+-------------------------------------------+-----------+
| Least privilege validation                | ✔         |
+-------------------------------------------+-----------+
| Deterministic failure behavior            | ✔         |
+-------------------------------------------+-----------+

---------------------------------------------------------------------

Detection & Evidence
--------------------

Relevant log locations:

* ``/var/log/restjavad.*`` – REST authentication and authorization
* ``/var/log/restnoded/restnoded.log`` – AS3 request handling
* **System → Logs → Audit** – User and configuration changes

Authorization failures generate:

* HTTP 401 responses
* REST framework log entries
* No MCP configuration transactions
* No configuration modification events

---------------------------------------------------------------------

Middle Layer Cohesion
---------------------

Within the Middle Layer:

* MFA validates administrative identity.
* TLS enforces secure transport.
* API Access Control enforces privilege boundaries.

Together, these controls prevent credential abuse,
transport downgrade, and unauthorized configuration deployment.

---------------------------------------------------------------------

Success Criteria
----------------

* Operator role denied access to device-scoped endpoints
* Administrator role permitted access
* HTTP 401 returned for unauthorized requests
* No configuration changes occur for denied attempts
* Deterministic privilege boundary confirmed