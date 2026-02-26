Outer Layer Labs – Network Access Controls
=========================================

Objective
---------
Implement quick-win protections for control-plane access.

Prerequisites
-------------
* BIG-IP administrative access
* Network access to management interface
* iHealth access (for validation)

Lab 1: Restrict Management Interface Access
-------------------------------------------

Steps
~~~~~
1. Restrict TMUI and SSH access to trusted source IP ranges.
2. Configure the management interface firewall (if applicable to your version/platform).
3. Apply Self IP port lockdown: default-deny, then explicitly allow required ports/services.

Validation
~~~~~~~~~~
::

   tmsh list sys management-ip
   tmsh list net self

Expected Results
~~~~~~~~~~~~~~~~
* TMUI/SSH reachable only from approved networks.
* No public/routable access to the control-plane.

iHealth Verification
~~~~~~~~~~~~~~~~~~~~
* H444724 – Management interface public access

Lab 2: Session Timeouts and Login Banners
-----------------------------------------

Steps
~~~~~
1. Configure idle logout for TMUI sessions.
2. Configure inactivity timeout for SSH.
3. Configure pre-login and post-login banners.

Validation
~~~~~~~~~~
::

   tmsh list sys httpd auth-pam-idle-timeout
   tmsh list sys global-settings banner

iHealth Verification
~~~~~~~~~~~~~~~~~~~~
* D009908 – Idle logout settings
* D006068 – Login banner configuration
