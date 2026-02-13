Exercise 2: Tenant Creation and Migration
=========================================

In this exercise, each student must create a new tenant and migrate a configuration into it; individual settings may differ for Student A and Student B, as noted below.

Navigate to Tenant Deployments and click Add
Create a new tenant with the following new settings (all others leave as default) 
Name: r5900-X-tenant-a or r5900-X-tenant-b
Type: BIG-IP
Image: BIGIP-21.0.0.1-0.0.13  
IP Address: 10.193.5.30+X for tenant-a 10.193.5.50+X for tenant-b
Prefix Length: 24
Gateway: 10.193.5.254
VLANS: Check both VLANs 
vCPUs: 8
Virtual Disk Size: 100
Click Save & Close


 
.. image:: images/image18.png
   :alt: image18.png
   :width: 80%

CLI:	ssh as user admin 

.. code-block:: none

   r5900-1# config
   r5900-1(config)# tenants tenant r5900-<X>-tenant-<Y> config image BIGIP-21.0.0.1-0.0.13.ALL-F5OS.tar.bundle vcpu-cores-per-node 8 mgmt-ip 10.193.5.<30+X> prefix-length 24 gateway 10.193.5.254 vlans [ <10+X> <30+X> ] nodes 1 running-state configured memory 29184 storage size 100
   r5900-1(config-tenant-r5900-2-tenant-a)# commit
   r5900-1# exit
   
   

At this stage, the tenant configuration is complete; however, the tenant has not been started, and no resources have been provisioned. You can view the tenant state in Dashboard → Tenant Overview or Tenant Management → Tenant Deployments.

To start the tenant, either select the tenant and click the Deploy button, or edit the tenant and change the State setting to Deployed.

 
.. image:: images/image19.png
   :alt: image19.png
   :width: 80%


CLI from config mode
r5900-1# show tenants
	See that newly created tenant is in ‘configured' state

.. code-block:: none

   r5900-1# config
   r5900-1(config)# tenants tenant r5900-2-tenant-a config running-state deployed
   r5900-1# commit
   r5900-1# exit
   r5900-1# exit
   
   
   

After you commit the change, F5OS will start deploying the tenant. The initial deployment requires creating the virtual disk, which takes longer than updating an existing tenant's running state. While the tenant is being created, the status will show as Starting in the GUI and CLI.  The tenant should be available in 3-5 minutes.

The tenant Status will show Running once it has booted; you can monitor the tenant startup by continuously pinging the management IP from your workstation -- successful replies indicate tenant services are coming up and you can log in.

r5900-1# show tenants tenant state status
NAME              STATUS
---------------------------
r5900-1-tenant-a  Running

Note: New tenants are created with a default TMOS install. For initial login use root/default (SSH); change the root password when prompted -- this password is temporary and will revert to the lab password Appworld26 after the UCS load completes.

Once the tenant is running, and management IP is reachable:
<<JHart:  Since the password update process is a bit funky where admin/root expiration dates don't both get reset let's just have them ssh as root for this step>>
SSH into the tenant as user "root" with password "default"
ssh root@10.193.5.(30+X For Student A or 50+X for Student B)
After resetting the password in the CLI remain in the CLI for the next steps.

Next, import the master key from the tenant you're migrating. This is necessary when the UCS contains encrypted passwords; without the master key, the UCS may fail to load. If the master key is unavailable, manually edit the configuration to remove encrypted elements so the UCS can load without errors.

Load the master key from the bash shell (see K9420). For this lab, both the UCS and master key files are on a web server 10.193.5.2 that can be downloaded to a laptop then uploaded to the BIG-IP tenant, or simple directly downloaded to the tenant via HTTPS.  

Direct download to the tenant: 

<Testers, the UCS files and keys have been moved to 10.193.5.2 - this is the IP we will use for Appworld. > 
Filenames for Student A:
UCS file:	r5900-<X>a.ucs
Key file:	r5900-<X>a.key

Filenames for Student B:
UCS file:	r5900-<X>b.ucs
Key file:	r5900-<X>b.key


Using curl from the BIG-IP tenant (remember to change the files names to your student and station assignments):
<<JHart - Does it flow better and lead to less mistakes if we have block for Student A and Student B here?  The students will still need their respective station ID/Number but maybe separate block will result in fewer loads of the same UCS file between student A/B

Also...  If the students login to the tenant UI and change the password, the root password's expiration date is not reset so when logging into the CLI, students will be prompted for password change again.  Does setting the root password in the UI set the flag - yes.  Should we have student's login to the tenant CLI initially?>>

<BVL: the end of lab reset commands set both PWs . My first version had the last-change and root in the lab, but this is really more of a bug that we may fix, so I thought removing it was less confusing> 

.. code-block:: none

   [root@localhost:Active:Standalone] config # curl -k https://10.193.5.2/r5900-2a.key -o /var/tmp/r5900-2a.key
   
   [root@localhost:Active:Standalone] config # curl -k https://10.193.5.2/r5900-2a.ucs -o /var/tmp/r5900-2a.ucs
   
   [root@localhost:Active:Standalone] config # ls -la /var/tmp/r5900-*

-rw-r--r--. 1 root root       25 Jan 14 11:13 /var/tmp/r5900-1a.key
-rw-r--r--. 1 root root 10918287 Jan 14 11:13 /var/tmp/r5900-1a.ucs

Next, re-key the master key from the txt file downloaded, as an example: 
[root@localhost:Active:Standalone] config # cat /var/tmp/r5900-1a.key
U5hNcJqbR0W4pjILPNa5/Q==
[root@localhost:Active:Standalone] config # f5mku -r U5hNcJqbR0W4pjILPNa5/Q==
Rekeying Master Key...
[root@localhost:Active:Standalone] config # f5mku -K
U5hNcJqbR0W4pjILPNa5/Q==

The final tenant migration step is to load the UCS file, using the platform-migrate option which ignores network interfaces and other items in the load process

tmsh
root@(localhost)(cfg-sync Standalone)(Active)(/Common)(tmos)# load sys ucs /var/tmp/r5900-1a.ucs platform-migrate
Replace all configuration on the system? (y/n) y
…
…
Platform migrate loaded successfully. Saving configuration.
/var/tmp/r5900-1a.ucs is loaded.
root@(i5000-a)(cfg-sync Standalone)(INOPERATIVE)(/Common)(tmos)#

Within a minute or two the BIG‑IP status will change to Active and the configuration will be loaded. View the configuration in the UI or with tmsh to examine the virtual server, pool, and other settings. The Pool should be passing health checks and available along with the Virtual Servers

Within F5OS, the virtctl command allows for a virtual console to any tenant. In addition, ssh can be used to enable virtual console to tenants with the following configuration. Begin by viewing the system aaa settings:

r5900-1# show system aaa authentication users
                  AUTHORIZED  LAST        TALLY                  EXPIRY
USERNAME          KEYS        CHANGE      COUNT  ROLE            STATUS
--------------------------------------------------------------------------
admin             -           2026-01-25  0      admin           enabled

.. code-block:: none

   r5900-8-tenant-a  -           0           0      tenant-console  locked
   r5900-8-tenant-b  -           0           0      tenant-console  locked

root              -           2026-01-08  0      root            enabled

Notice that for each tenant, a username has been created with specific role of tenant-console. To enable use of this, a few configuration items must be done.

First set the password for the account (this is only to scp to console, it is not a user login role) 

.. code-block:: none

   r5900-1# config
   r5900-1(config)# system aaa authentication users user <tenant USERNAME> config set-password

Set Account to enabled 

.. code-block:: none

   r5900-1(config)# system aaa authentication users user <tenant USERNAME> config expiry-status enabled
   r5900-1(config-user-r5900-1-tenant-b)# exit

Set the last-change date

.. code-block:: none

   r5900-1(config)# system aaa authentication users user <tenant USERNAME> config last-change <date in format YYYY-MM-DD>
   r5900-1(config-user-r5900-1-tenant-b)# commit
   r5900-1(config-user-r5900-1-tenant-b)# exit
   
   
   
   

Test console access to your tenant from your workstation/desktop:
ssh <tenant-name>@<F5OS IP> -p 7001 using the password set above
Log into the tenant console as root or admin


//End of Exercise 2
