
Lab 2: Load Balancing, Monitoring and Persistence
=================================================

Objectives:

-  Configure and review the Ratio load balancing method

-  Build and test priority groups

-  Build a content monitor that looks for a receive string

-  Build and review simple (source IP) persistence and cookie
   persistence

Ratio Load Balancing
--------------------

1. Go to **Local** **Traffic>Pools** and select **www_pool** and then
   **Members** from the top bar or you could click on the **Members**
   link in the Pool List screen.

.. image:: ../images/image69.png
   :width: 5.35046in
   :height: 1.60014in

a. Note: When we created the pool, we performed all our configuration on
   one page, but when we modify a pool the **Resource** information is
   under the **Members** tab

2. Under **Load** **Balancing** section

   a. Change the **Load** **Balancing** **Method** to **Ratio (Member)**

   b. As you look at the drop down menu, notice most load balancing
      methods have two options, **(Node)** or **(Member)**. You should know the
      difference between the two.

.. image:: ../images/image25.png
   :width: 5.01042in
   :height: 2.59576in

c. Don’t forget the **Update** button

d. Then under **Current Members**

   i.  Select the first member in the pool **10.1.20.11:80**.

   ii. Under the **Configuration** section

       1. Change the **Ratio** of the member to 3

.. image:: ../images/image26.png
   :width: 4.26042in
   :height: 4.04175in

e. Select the **Update** button

3. Verification

   a. Check the pool statics by selecting **Statistics** in the top bar,
      if you are still in **Local Traffic> Pools** or by going to
      **Statistics>Module Statistics>Local Traffic** and selecting
      **Pool** from **Statistics Type**.

   b. Reset the statistics for your **www_pool** pool by checking the
      boxes next to the pool members and hitting the **Reset** button

      i.   Log back into Ubuntu Jumphost and browse to your **www_vs** **(10.1.10.100)** virtual server

      ii.  Refresh the browser screen several times (use “<ctrl>” F5)

      iii. Select the **Refresh** button on the **Statistics** screen

      iv.  How many Total connections has each member taken?

      v.   Is the ratio of connections correct between the members?

   c. Now go back and put the pool back to Round Robin Load Balancing
      Method

      i.   Reset the statistics

      ii.  Refresh the virtual server page several times

      iii. Refresh the statistics

      iv.  Does the ratio setting have any impact now?

Priority Groups Lab
-------------------

Let’s look at priority groups. In this scenario we will treat the
**.13** server as if it was is in a disaster recovery site that can be
reached over a backhaul. To maintain at least two members in the pool
for redundancy and load sharing, our customer would like to use it
during maintenance periods or if one of the two other pool members
fails.

1. Go to **Local Traffic>Pools>www_pool**

   a. Select the **Members** tab.

      i.  Set the **Load Balancing Method** back to **Round Robin**

      ii. Set the **Priority Group Activation** to **Less than …** **2**
          Available Members.

.. image:: ../images/image27.png
   :width: 5.4375in
   :height: 2.03332in

b. Don’t forget the **Update** button

c. Select the pool members **10.1.20.11** and **10.1.20.12** and set
   their **Priority Group** to **2**.

   i. This will allow you to change the priority on that particular
      member.

.. image:: ../images/image28.png
   :width: 4.49091in
   :height: 4.26042in

2. Review your settings and let’s see how load balancing reacts now.

   a. Select the **Statistics** tab.

   b. Reset the pool statistics.

   c. In your RDP session to Jumpbox, browse to your virtual server and refresh several times.

   d. Refresh you statistics on bigip01.

   e. Are all members taking connections?

   f. Which member isn’t taking connections?

3. Let’s simulate a maintenance window, or an outage, by disabling a
   pool member in the highest priority group. As this will drop the
   number of active members below 2, this should cause the low priority
   group to be activated.

4. Select the member in the Priority Group 2 and Disable that pool
   member.

   a. Select the **Disable** button

..

   .. image:: ../images/image29.png
      :width: 6.9739in
      :height: 1.24444in

a. The status indicator now goes to black, indicating the member has
   been disabled

5. Once again, select **Statistics**, reset the pool statistics, browse
   to the virtual server and see which pool members are taking hits now.

.. tip:: Once you are done testing re-enable your disabled pool member.

Monitor Labs
------------

Objective:

-  Build a default monitor for nodes

-  Build a content monitor for your pool

Default Monitors

1. Go to **Local Traffic>Nodes**, note the Status.

   a. Notice there are Nodes in this table even though we never
      specifically configured them under the Node portion of the GUI.
      Each time a unique IP address is placed in a pool, by default, a
      corresponding node entry is added and assigned the default monitor
      (if any).

   b. Select the **Default Monitor** tab.

.. image:: ../images/image30.png
   :width: 4.2837in
   :height: 2.06685in

c. Notice we have several options, for nodes you want a generic monitor,
   so we will choose **icmp**.

d. Select **icmp** from the **Available** box and hit the arrows button pointing to the left to place
   it in the **Active** box.

e. Click on the **Update** button to finalize your changes.

2. Select **Node List** or **Statistics** from the top tab.

    a. What is the Status of the Nodes?

3. Select **Statistics>Module Statistics>Local Traffic**

    a. What is the Status of your Nodes, Pool and Virtual Server?

Content Monitors
----------------

The default monitor simply tells us the IP address is accessible, but we really don’t know the status of the particular application the node supports. We are now going to create a monitor to specifically test the application we are interested in. We are going to basic contnet check of our web site to determine it the servers are responding properly.

1. Browse to **http://10.1.10.100**. A number of content items you could use to check the site status.  You could check for text on this page. You could view the source code and check for a text string not normally visible to the user. You can also look in the HTTP header information being returned. 

2. We will be looking for the HTTP status “\ **200 OK**\ ” in the HTTP header information as our receive string to determine availability.

3. Select **Local Traffic>Monitor** on the side-bar and select the plus
   (**+**) sign or the **Create**

.. image:: ../images/image32.png
   :width: 3.78002in
   :height: 2.46226in

a. Now we can create a monitor to check the content of our web page to
   ensure things are running properly.

   i.  **Name**: www_test

   ii. **Type**: HTTP

.. image:: ../images/image33.png
   :width: 1.93333in
   :height: 2.56016in

b. Once you have selected your parent (Type) monitor, you can access the **Configuration** section

   i.   **Send String**: Enter the command to retrieve the page you want **GET /index.php HTTP/1.0\\r\\n\\r\\n**

   ii.  In the Receive String box put “\ **200 OK**\ ” (no quotes)

   .. image:: ../images/image34.png
      :alt: Monitor configuration
      :scale: 75

.. note:: The receive string is not case sensitive.

c. Click **Finish** and you will be taken back to **Local
   Traffic>Monitors**

..

   .. image:: ../images/image35.png
      :width: 0.94444in
      :height: 0.55556in

4.  Where is your new Monitor?

    a. **Hint:** Check the lower right hand corner of the Monitors list,
       here you can go to the next page or view all Monitors

    b. You can change the number of records displayed per page in
       **System>Preferences**

5.  Go to **Local Traffic>Pools>www_pool** and choose **Properties**
    from the top bar.

    a. Remove the **http** monitor from the Active box.

    b. Select the **www_test** monitor from the Available monitor’s
       window in the **Configuration** section and move it to the Active
       window.

.. image:: ../images/image36.png
   :width: 3.76042in
   :height: 3.10417in

6. Hit **Update** to apply the change.

    a. Select **Statistics** from the tabs.

    b. What is the status of the pool and its members?

7. Go to **Local Traffic>Virtual Servers**, what is the status of your virtual server?

    a. Browse to your **www_vs** virtual server. Which members are
       taking traffic?

    b. Just for fun, let's reverse the monitor. Go to Properties of the pool and change the **Reverse** setting to **Yes**. Update the pool and browse to your virtual server again.
    
       Now when **200 OK** is returned it indicates the server is not responding successfully. You can
       see where this would be useful if you were looking for a 404 (bad
       page) response. 
       
       Once completed, please go back and **undo** your changes to the monitor. 



Persistence Labs
----------------
       
In this lab we will configure a couple types of persistence and view their behavior. For persistence, profiles will have to be created and attached to our virtual server.
       
       Lab Requirements:
       
       -  Prior to beginning the lab verify your **www_pool** has been set to
          the following parameters:
       
          -  **Load Balancing Method**: Round Robin
       
          -  **Priority Group Activation**: Disable
       
             -  The members **Ratio** and **Priority** **Group** mean nothing
                since we aren’t using Ratio load balancing and Priority Groups
                are disabled.
       
          -  Hit **Update**
       
          -  Hit your virtual server several times, you should see all 3
             servers respond.
       
Simple (Source Address) Persistence
-----------------------------------
       
       1. Go to **Local** **Traffic>Profiles** and select the **Persistence**
          tab.
       
          a. From the **Persistence Profiles** screen select the **Create**
             button.
       
         .. image:: ../images/image37.png
            :scale: 75
      
       b. At the **New Persistence Profile** screen enter:
       
          i.  **Name**: my-src-persist
       
          ii. **Persistence** **Type**: Source Address Affinity
       
       .. image:: ../images/image38.png
          :width: 2.61621in
          :height: 2.04167in
       
       c. This will add the **Configuration** section to the **General**
          **Properties** section.
       
          i. Note the parent profile.
       
       d. In the **Configuration** section, set the
       
          i.   **Timeout**: 60 seconds
       
          ii.  **Prefix Length**: None
       
               1. This is the default, and is a /32 prefix (255.255.255.255 mask).
       
               2. Each new IP address will create a new persistence record.

.. hint::
   You can’t change the settings until you have checked the Custom box.  This is very useful in knowing which configuration items were modified from the default.

|
          iii.  Click the Finished button.
       
       e. You have just created your first custom Profile.
       
          i. Note the check box for your new custom profile isn’t grayed out
             and can be selected to allow you to delete the profile if desired.
       
       f. Now let’s attach our new profile to the virtual server.
       
          i. Go to **Local Traffic>Virtual Server** and ….
       
             1.   Select **www_vs** and the **Resources** tab or ….
       
             2. Take the shortcut directly to the **Resources** of the virtual server. (Can you find it?)

      .. note:: 
         When we created the Virtual Server everything was on a single page, we find when we return to modify the Virtual Server the Properties and Resources are on different pages.
      
      |
       g. Set the **Default Persistence Profile** to **my-src-persist**.
       
       .. image:: ../images/image39.png
          :width: 3.41667in
          :height: 1.90957in
       
       h. Don’t forget to **Update** before leaving the page. *(Be careful,
          someday I will quit telling you that.)*
       
       I. Testing Source Address Affinity
       
          i.   At this point you may want to open a second browser window to
               the management GUI of bigip01.
       
          ii.  For one management window go to **Statistics>Module
               Statistic>Local Traffic**
       
          iii. Select **Persistence Records** for the **Statistics Type** menu
       
       .. image:: ../images/image40.png
          :width: 4.47075in
          :height: 2.22917in
       
       j. At this point you will see that Persistence Records statistics
          display has been disabled by default. A TMSH database command is
          required to activate it.
       
          i. SSH to bigip01 or open a WebShelll session from the Access dropdown menu.
       
          ii. At the prompt enter: **tmsh**
       
          iii. At the TMSH prompt enter the command in the **Persistence Value** GUI.

               modify sys db ui.statistics.modulestatistics.localtraffic.persistencerecords value true
               
Tab completion will make this a little easier
            
       2. Now, in this window you can watch you persistence records. You may
          want to set **Auto Refresh** to 20 seconds.
       
       .. image:: ../images/image41.png
          :width: 4.8125in
          :height: 1.80366in
       
       3. In your other management GUI window go to **www_pool** and clear the
          member statistics.
       
          a. Open a browser session to your virtual server and refresh several
             times.
       
          b. How many members are taking traffic?
       
          c. Check your Persists Records window, are the any persistence
             records?
       
             i. If you are not Auto Refreshing, don’t forget to hit Refresh
       
          d. Refresh your web page prior to the Age column reaching 60. What
             happens?
       
Cookie Persistence (Cookie Insert)
----------------------------------
       
       1. Go to **Local Traffic>Profiles>Persistence** tab and hit **Create**
       
       a. Let’s name our profile **my_cookie_insert** (original isn’t it)
       
       b. Our **Persistence Type** will be **Cookie**
       
       c. This brings us to the **Configuration** section.
       
       .. image:: ../images/image42.png
          :width: 3.59403in
          :height: 3.15625in
       
       2. As you can see the default **Cookie Method** is **HTTP** **Cookie**
          **Insert**, so we won’t have to modify the Cookie Method
       
       a. The BIG-IP will also create a cookie name for you using a combination
          of “\ **BIGipServer**\ ” and the pool name the virtual server
          service. We will take this default also.
       
       b. We will use a session cookie. Which means the cookie is deleted when
          the browser is closed.
       
       c. Select **Finished**
       
       d. Now attach your cookie persistence profile to your virtual server’s
          **Default Persistence Profile**
       
       Go to **Local Traffic>Virtual Server>www_vs>Resources** tab
       
       e. Set the **Default Persistence Profile** to **my_cookie_insert**
       
       f. Hit **Update**
       
       g. Whoa! Did you just get this error message?
       
       .. image:: ../images/image43.png
          :width: 4.64151in
          :height: 1.83072in
       
       h. Remember what we said earlier about some Profiles requiring
          prerequisite Profiles? Since we are looking in the HTTP header for
          the cookie the prerequisite for the Cookie Profile is the HTTP
          profile.
       
       3. We will have to go to the virtual server to add the HTTP profile,
          prior to adding the Cookie Persistence profile.
       
          a. Select the **Properties** tab on your virtual server
       
          b. Go to **HTTP Profile (client)** in the **Configuration** section and select
             the default HTTP (**http**) profile.
       
       .. image:: ../images/image44.png
          :width: 3.13229in
          :height: 3.69328in
       
       c. Hit the **Update** button
       
       d. Now we can go back to the **Resource** tab and add our cookie
          persistence profile.
       
       4. Testing cookie persistence.
       
          a. If you wish you can watch the member statistics to validate your
             persistence.
       
          b. Open a new browser session to your virtual server and refresh
             several times.
       
          c. Does the page ever change?
       
          d. Did you hit a different server?
       
          e. Refresh several times. Are you hitting the same server?
          f. Let's take a look at the cookie.
       
             i. On the web page of the demo application right click and select **Inspect**. This should work on most browsers.
             ii. In the **Element** bar of the Inspect window select **Network** and refresh your web page.
             iii. In the **Name** section of the Inspect window on the content select **f5demo.css** and **Cookies** on the **Name** bar.  Here you will see the BIG-IP cookie that was inserted.

             .. tip::
               **Inspect** is your friend.  Learn to use it.  It will aid in debugging, iRules, monitors and more.
       
            .. image:: ../images/image45.png
               :alt: Inspect page window
               :scale: 50
               :align: center

         g. Before proceeding to the next lab **please** remove the persistence profile from the virtual server. 
