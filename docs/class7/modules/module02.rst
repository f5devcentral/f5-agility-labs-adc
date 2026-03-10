
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

1. Go to **Local Traffic >> Pools** and select **www_pool** and then
   **Members** from the top bar or you could click on the **Members**
   link in the Pool List screen.

   .. image:: ../images/image69.png
      :width: 460px


   .. note::
      When we created the pool, we performed all our configuration on one page, but when we modify a pool the **Resource** information is    under the **Members** tab


2. Under **Load Balancing** section

   a. Change the **Load Balancing Method** to **Ratio (Member)**

   b. As you look at the drop down menu, notice most load balancing
      methods have two options, **(Node)** or **(Member)**. You should know the
      difference between the two.

      .. image:: ../images/image25.png
         :width: 600px


   c. Don’t forget the **Update** button

   d. Then under **Current Members**

      i.  Select the first member in the pool **10.1.20.11:80**.

      ii. Under the **Configuration** section, change the **Ratio** of the member to 3

          .. image:: ../images/image26.png
             :width: 750px


   e. Select the **Update** button


Ratio LB Verification
---------------------


1. Check the pool statistics by selecting **Statistics** in the top bar, if you are still in **Local Traffic >> Pools** or by going to **Statistics >> Module Statistics >> Local Traffic** and selecting **Pool** from **Statistics Type**.

2. Reset the statistics for your **www_pool** pool by checking the boxes next to the pool members and hitting the **Reset** button

3. Log back into Ubuntu Jumphost and browse to your **www_vs** **(10.1.10.100)** virtual server

4. Refresh the browser screen several times (use “<ctrl>” F5)

5. Select the **Refresh** button on the **Statistics** screen

   a. How many Total connections has each member taken?

   b. Is the ratio of connections correct between the members?

6. Now go back and put the pool back to Round Robin Load Balancing Method

   a. Reset the statistics

   b.  Refresh the virtual server page several times

   c. Refresh the statistics

   d.  Does the ratio setting have any impact now?


Priority Groups Lab
-------------------

Let’s look at priority groups. In this scenario we will treat the
**.13** server as if it was in a disaster recovery site that can be
reached over a backhaul. To maintain at least two members in the pool
for redundancy and load sharing, our customer would like to use it
during maintenance periods or if one of the two other pool members
fails.

1. Go to **Local Traffic >> Pools >> www_pool**

2. Select the **Members** tab.

3. Set the **Load Balancing Method** back to **Round Robin**

4. Set the **Priority Group Activation** to **Less than… 2** Available Members.

   .. image:: ../images/image27.png
      :width: 750px


5. Don’t forget the **Update** button

6. Select the pool members **10.1.20.11** and **10.1.20.12** and set their **Priority Group** to **2**. This will allow you to change the priority on that particular member.

   .. image:: ../images/image28.png
      :width: 750px


Review your settings and let’s see how load balancing reacts now.

1. Select the **Statistics** tab.

2. Reset the pool statistics.

3. In your RDP session to Jumpbox, browse to your virtual server and refresh several times.

4. Refresh you statistics on bigip01.

5. Are all members taking connections?

6. Which member isn’t taking connections?

Let’s simulate a maintenance window, or an outage, by disabling a pool member in the highest priority group. As this will drop the    number of active members below 2, this should cause the low priority group to be activated.

1. Select the member in the Priority Group 2 and Disable that pool member using the **Disable** button

   .. image:: ../images/image29.png
      :width: 750px


   .. note::
      The status indicator will change to black, indicating the member has been disabled


2. Once again, select **Statistics**, reset the pool statistics, browse to the virtual server and see which pool members are taking hits now.

   .. tip:: Once you are done testing re-enable your disabled pool member.


Monitor Labs
------------

Objective:

-  Build a default monitor for nodes

-  Build a content monitor for your pool

Default Monitors

1. Go to **Local Traffic >> Nodes**, note the Status.

   Notice there are Nodes in this table even though we never specifically configured them under the Node portion of the GUI. Each time a unique IP address is placed in a pool, by default, a corresponding node entry is added and assigned the default monitor (if any).

2. Select the **Default Monitor** tab.

   .. image:: ../images/image30.png
      :width: 515px
 

   Notice we have several options, for nodes you want a generic monitor, so we will choose **icmp**.

3. Select **icmp** from the **Available** box and hit the arrows button pointing to the left to place
   it in the **Active** box.

4. Click on the **Update** button to finalize your changes.

5. Select **Node List** or **Statistics** from the top tab.

   * What is the Status of the Nodes?

6. Select **Statistics >> Module Statistics >> Local Traffic**

   * What is the Status of your Nodes, Pool and Virtual Server?


Content Monitors
----------------

The default monitor simply tells us the IP address is accessible, but we really don’t know the status of the particular application the node supports. We are now going to create a monitor to specifically test the application we are interested in. We are going to do a basic content check of our web site to determine if the servers are responding properly.

1. Browse to **http://10.1.10.100**. A number of content items you could use to check the site status.  You could check for text on this page. You could view the source code and check for a text string not normally visible to the user. You can also look in the HTTP header information being returned. 

2. We will be looking for the HTTP status “\ **200 OK**\ ” in the HTTP header information as our receive string to determine availability.

3. Select **Local Traffic >> Monitor** on the side-bar and select the plus (**+**) sign or the **Create** button from the top right of the Monitor list page.

   .. image:: ../images/image32.png
      :width: 650px


4. Now we can create a monitor to check the content of our web page to
   ensure things are running properly.

   a.  **Name**: www_test

   b. **Type**: HTTP

      .. image:: ../images/image33.png
         :width: 350px


5. Once you have selected your parent (Type) monitor, you can access the **Configuration** section

   a. **Send String**: Enter the command to retrieve the page you want **GET /index.php HTTP/1.0\\r\\n\\r\\n**

   b. In the Receive String box put “\ **200 OK**\ ” (no quotes)

      .. image:: ../images/image34.png
         :width: 460px

      .. note::
         The receive string is not case sensitive.

6. Click **Finish** and you will be taken back to **Local Traffic >> Monitors**

7. Where is your new Monitor?

   a. **Hint:** Check the lower right hand corner of the Monitors list,
      here you can go to the next page or view all Monitors

      .. image:: ../images/image35.png
         :width: 120px

   b. You can change the number of records displayed per page in
      **System >> Preferences**

8. Go to **Local Traffic >> Pools >> www_pool** and choose **Properties**
   from the top bar.

   a. Remove the **http** monitor from the Active box.

   b. Select the **www_test** monitor from the Available monitor’s window in the **Configuration** section and move it to the Active        window.

      .. image:: ../images/image36.png
         :width: 500px


9. Hit **Update** to apply the change.

10. Select **Statistics** from the tabs.

    * What is the status of the pool and its members?

11. Go to **Local Traffic >> Virtual Servers**, what is the status of your virtual server?

    a. Browse to your **www_vs** virtual server. Which members are taking traffic?

    b. Just for fun, let's reverse the monitor. Go to Properties of the monitor and change the **Reverse** setting to **Yes**. Update the monitor and browse to your virtual server again.
    
       Now when **200 OK** is returned it indicates the server is not responding successfully. You can
       see where this would be useful if you were looking for a 404 (bad
       page) response. 
       
       Once completed, please go back and **undo** your changes to the monitor. 



Persistence Labs
----------------
       
In this lab we will configure a couple types of persistence and view their behavior. For persistence, profiles will have to be created and attached to our virtual server.
       
Lab Requirements:
       
Prior to beginning the lab verify your **www_pool** has been set to the following parameters:
       
1. **Load Balancing Method**: Round Robin
       
2. **Priority Group Activation**: Disable

   The members **Ratio** and **Priority Group** mean nothing since we aren’t using Ratio load balancing and Priority Groups are disabled.
       
3. Hit **Update**
       
4. Hit your virtual server several times, you should see all servers respond.
       
**Simple (Source Address) Persistence**
       
1. Go to **Local Traffic >> Profiles** and select the **Persistence**
          tab.
       
2. From the **Persistence Profiles** screen select the **Create**
             button.
       
   .. image:: ../images/image37.png
      :width: 750px


3. At the **New Persistence Profile** screen enter:
       
   a. **Name**: my-src-persist
       
   b. **Persistence Type**: Source Address Affinity
       
      .. image:: ../images/image38.png
         :width: 650px


      This will add the **Configuration** section to the **General Properties** section.
       
      * Note the parent profile.
       
   c. In the **Configuration** section, click the checkbox to the right of **Timeout** and set the value to 60 seconds
       
      .. image:: ../images/image38a.png
         :width: 650px

       
   d. Leave the **Prefix Length** as None
       
      1. This is the default, and is a /32 prefix (255.255.255.255 mask).
       
      2. Each new IP address will create a new persistence record.

   e.  Click the **Finished** button.
         
   f. You have just created your first custom Profile.
       
      .. note::
         The check box for your new custom profile isn’t grayed out and can be selected to allow you to delete the profile if desired.


4. Now let’s attach our new profile to the virtual server.
       
5. Go to **Local Traffic >> Virtual Server** and ….
       
6.   Select **www_vs** and the **Resources** tab or ….
       
7. Take the shortcut directly to the **Resources** of the virtual server. (Can you find it?)

   .. note:: 
      When we created the Virtual Server everything was on a single page, we find when we return to modify the Virtual Server the Properties and Resources are on different pages.
      
   
8. Set the **Default Persistence Profile** to **my-src-persist**.
       
   .. image:: ../images/image39.png
      :width: 750px


9. Don’t forget to **Update** before leaving the page. *(Be careful, someday I will quit telling you that.)*
       
Testing Source Address Affinity
-------------------------------

At this point you may want to open a second browser window to the management GUI of bigip01.
       
1. For one management window go to **Statistics >> Module Statistics >> Local Traffic**
       
2. Select **Persistence Records** for the **Statistics Type** menu
       
   .. image:: ../images/image40.png
      :width: 750px

       
   At this point you will see that Persistence Records statistics display has been disabled by default. A TMSH database command is           required to activate it but we will view the records from TMSH.
       
3. SSH to bigip01 or open a Web Shell session from the Access dropdown menu.
       
4. At the prompt enter the following command to show persistence records::
   
     tmsh show ltm persistence persist-records
   

5. Go to RDP session and refresh your virtual server page several times.
       
6. In your other management GUI window go to **www_pool** and clear the member statistics.
       
7. Open a browser session to your virtual server and refresh several
             times.
       
8. How many members are taking traffic?
       
9. Check your Persistence Records again from the bigip01 SSH or Web Shell window::

      tmsh show ltm persistence persist-records


10. Are there any persistence records?
       
              
**Cookie Persistence (Cookie Insert)**
       
1. Go to **Local Traffic >> Profiles >> Persistence** tab and hit **Create**
       
2. Let’s name our profile **my_cookie_insert** (original isn’t it)
       
3. Our **Persistence Type** will be **Cookie**
       
4. This brings us to the **Configuration** section.
       
   .. image:: ../images/image42.png
      :width: 510px


5. As you can see the default **Cookie Method** is **HTTP Cookie Insert**, so we won’t have to modify the Cookie Method
       
    The BIG-IP will also create a cookie name for you using a combination of “\ **BIGipServer**\ ” and the pool name the virtual server     service. We will take this default also.
       
6. We will use a session cookie. Which means the cookie is deleted when the browser is closed.
       
7. Select **Finished**
       
8. Now attach your cookie persistence profile to your virtual server’s **Default Persistence Profile**
       
9. Go to **Local Traffic >> Virtual Server >> www_vs >> Resources** tab
       
10. Set the **Default Persistence Profile** to **my_cookie_insert**
       
11. Hit **Update**
       
    Whoa! Did you just get this error message?
       
    .. image:: ../images/image43.png
       :width: 600px


    Remember what we said earlier about some Profiles requiring prerequisite Profiles? Since we are looking in the HTTP header for           the cookie the prerequisite for the Cookie Profile is the HTTP profile.


We will have to go to the Virtual Server to add the HTTP profile prior to adding the Cookie Persistence profile.
       
1. Select the **Properties** tab on your virtual server
       
2. Go to **HTTP Profile (client)** in the **Configuration** section and select the default HTTP (**http**) profile.
       
   .. image:: ../images/image44.png
      :width: 600px


3. Hit the **Update** button
       
4. Now we can go back to the **Resource** tab and add our cookie
          persistence profile.
       
**Testing Cookie Persistence**
       
1. If you wish you can watch the member statistics to validate your persistence.
       
2. Open a new browser in your RDP session to your virtual server and refresh
             several times.
       
3. Does the page ever change?
       
4. Did you hit a different server?
       
5. Refresh several times. Are you hitting the same server?

Let's take a look at the cookie.
       
1. On the web page of the demo application right click and select **Inspect**. This should work on most browsers.

2. In the **Element** bar of the Inspect window select **Network** and refresh your web page.

3. In the **Name** section of the Inspect window on the content select **f5demo.css** and **Cookies** on the **Name** bar.  Here you will see the BIG-IP cookie that was inserted.

   .. tip::
      **Inspect** is your friend.  Learn to use it.  It will aid in debugging, iRules, monitors and more.
       

   .. image:: ../images/image45.png
      :width: 750px

   
4. Before proceeding to the next lab **please** remove the persistence profile from the virtual server. 
