Partitions and RBAC
===================


Objective
---------

This lab demonstrates basic access control using partitions.


Lab Steps
---------

Create a Partition
++++++++++++++++++

#. Navigate to **System -> Users -> Partition List**.
#. Click **Create** in the upper right corner.
#. Name the partition **App01**

 .. image:: ./_images/partition-1.png

#. Click **Finished**.


Add Users 
+++++++++

#. Navigate to **System -> Users -> User List**.
#. Click **Create**.
#. Name the user **App01Admin** and choose any password you can remember for the next few steps.
#. Select the role **Manager** and assign it to the partition you just created (**App01**).

 .. image:: ./_images/partition-2.png

#. Click **Add**, ensure that the role and partition are displayed in the box below, then click **Finished**.

 .. image:: ./_images/partition-3.png

#. Create another user by once navigating to **System -> Users -> User List**.
#. Name the user **App01Operator** and assign a passowrd.
#. Select the role **Operator** and assign it to the partition **App01** then click **Finished**.

 .. image:: ./_images/partition-4.png


Explore & Create Objects
++++++++++++++++++++++++

#. Logout of the TMUI and lock back in as **App01Admin**.
#. Since it's you're first time logging in, you'll regrettably be asked to change your password.  While this can be an annoyance, this is the default behavior and an important security control.  Again, choose a password you can remember.
#. If you've done everything correctly so far, you'll notice that your user is locked to the **App01** Partition.

 .. image:: ./_images/partition-5.png

#. Navigate to **Local Traffic -> Virtual Servers**.
#. **Click** any of the Virtual Servers in the list.
#. You'll notice that all of the options are greyed out and you cannot make changes.  This is because these objects are in the common partition which is owned by box level administrators and you do not have sufficient rights with this account.
#. Navigate to **Local Traffic -> Pools**.
#. Click **Create**.
#. Name the pool **App01Pool** and select the **http health monitor**.
#. Add a node with the IP Address **10.1.10.50** on **port 80** (don't forget to click add).  It should look like this:

 .. image:: ./_images/partition-6.png

#. If it looks correct, click **Finished.**
#. **Return** to the **virtual servers list**.
#. Click **Create**.
#. Configure the virtual server as follows:

    +-----------------+---------------------+
    | Setting         | Value               |
    +=================+=====================+
    | Name            | App01               |
    +-----------------+---------------------+
    | IP Address      | 10.1.10.75          |
    +-----------------+---------------------+
    | Port            | 80                  |
    +-----------------+---------------------+
    | HTTP Profile    | http                |
    +-----------------+---------------------+
    | SNAT            | Auto Map            |
    +-----------------+---------------------+
    | Default Pool    | App01Pool           |
    +-----------------+---------------------+

#. Click **Finished**.
#. You'll notice that the Virtual Server was created in the **App01** Partition instead of Common like the others.

Test Limited Access
+++++++++++++++++++

#. Log out of the TMUI and login as **App01Operator**.
#. Change your password one more time.
#. Navigate to **Local Traffic -> Virtual Servers**.
#. Attempt to edit **App01**.  You'll notice that now you can't. 
#. Navigate to **Local Traffic -> Pools**.
#. Click on **App01Pool**.  You'll notice that this is also greyed out:

 .. image:: ./_images/partition-7.png

#. Click on the **Members** tab.  Even though all other options are greyed out, you'll notice that you have the ability to enable and disable pool members:

 .. image:: ./_images/partition-8.png


**That Concludes this lab.**

.. note::


  If you'd like to learn more about user roles, see `this article <https://techdocs.f5.com/en-us/bigip-17-0-0/big-ip-systems-user-account-administration/user-roles.html>`_




