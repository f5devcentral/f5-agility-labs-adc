Core Layer Labs – Advanced Control-Plane Security
================================================

Objective
---------
Implement advanced controls for high-security and compliance-driven environments.

Lab 1: Appliance Mode and RBAC
------------------------------

Steps
~~~~~
1. Enable appliance mode where operationally appropriate.
2. Implement role-based access control (RBAC) aligned to job functions.
3. Establish and document a break-glass account and procedure.

Validation
~~~~~~~~~~
::

   tmsh list sys global-settings appliance-mode
   tmsh list auth user

iHealth Verification
~~~~~~~~~~~~~~~~~~~~
* D012815 – Appliance mode status

Lab 2: Continuous Monitoring and Incident Readiness
---------------------------------------------------

Steps
~~~~~
1. Send all control-plane logs off-box to SIEM (TMUI/SSH/iControl/audit).
2. Enable alerting for anomalous login patterns and configuration changes.
3. Perform periodic iHealth checks (QKView upload + review Security Best Practices panel).

Validation
~~~~~~~~~~
* SIEM receives control-plane events.
* Alerts trigger on failed login spikes and privileged changes.
* iHealth findings are tracked to closure over time.

Expected Results
~~~~~~~~~~~~~~~~
* Continuous visibility into control-plane activity.
* Documented response actions for suspected compromise.
