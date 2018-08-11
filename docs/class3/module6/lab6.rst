Lab 6: BIG-IP Policies and iRules
=================================

When clients attempt to access your **secure_vs**, you don’t want them
to have to remember to type HTTPS before the web site, but you also
don’t want to open port 80 (HTTP) on your web servers as that is just
asking for trouble. To avoid this issue, you will be creating an HTTP
virtual server that will redirect HTTP to HTTPS and the **secure_vs**.
Also, you will write an iRule and a BIG-IP policy that will retrieve
images from a different pool of servers than the default pool attached
to the virtual server. This will give you a simple comparison between
the two methods. You will use a policy on the HTTP server and an iRule
on the HTTPS virtual server.

Using the Built-in https_redirect iRule
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. While it would be easy to write your own redirect iRule, note that F5
   has one prebuilt that you can use

   #. Example of simple redirect iRule:

      .. code-block:: tcl

         when HTTP_REQUEST {
            HTTP::redirect https://[HTTP::host][HTTP::uri]
         }

#. Go to **Local Traffic >> iRules**

   #. In the search box at the top of the list of iRules, type
      **redirect** and hit **Search.**

      |image0|

   #. Open the iRule and take a quick look. This is an F5 Verified and
      supported iRule.

#. Create your HTTP-to-HTTPS redirect virtual server.

   #. Go to **Local Traffic >> Virtual Servers** and create a new
      virtual server.

      #. **Name:** redirect_to_secure_vs

      #. **Destination:** <same IP as secure_vs>

      #. **Service Port:** 80 (HTTP)

      #. **Source Address Translation:** None <you don’t need this as
         this traffic is going nowhere>

      #. **iRule:** **\_**\ sys_https_redirect

      #. Hit **Finished**

         #. **WOW!** That didn’t go too far did it. You just got an
            error. If you are going to redirect the HTTP request, you
            need the HOST and URI information and that requires the
            HTTP protocol

   #. In the **Configuration** section make sure the default **http**
      profile is added to the virtual server

   #. HTTP Profile: **http**

   #. Select **Finished**

#. Test your policy by going to **http://<ip address of your virtual>**

   #. You should be redirected to the HTTPS virtual server

   #. As you can see, very small iRules can make a very big difference

Use a BIG-IP Policy to retrieve images from a different pool 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. **Create** a new pool named **image_pool**, use the **http** monitor
   for status, and add one member **10.1.20.14:80**

#. First you will create your policy container and set your match
   strategy

   #. Try to do this using the instructions, but a screen shot of the
      policy is available in the **Appendix** at the end of the lab
      guide if you would like it

#. Go to **Local Traffic >> Policies >> Policy List** and select **Create**

   #. **Policy_Name:** access_image_pool

   #. **Strategy:** Execute **first** matching rule

   #. **Create Policy**

      |image1|

#. Now you can create/view policy rules by selecting **Create**

   #. **Name:** get_jpegs

   #. In the box under **Match all of the following conditions:** select
      the |image2| to the right of **All Traffic**

      #. Use the drop-down menus to look at the **HTTP URI** and
         check if it **ends_with** an image type

      #. Look for JPEGs by adding **jpg** in the box under the **any
         of** box and selecting **Add**

         |image3|

   #. Under **Do the following when the traffic is** matched, build the
      following operation.

      #. **Forward Traffic** to the **pool** named **image_pool**

   #. **Save**

      |image4|

#. The policy is saved in **Draft** form and is not available until
   **Published**. To publish the policy:
   
   #. Select the **Save Draft Policy** drop-down menu and select **Save and Publish Policy**

      |image5|

#. Go to the **Resources** section of your **www_vs** virtual server and
   select **Managed** over the **Policies** box

   #. Move **access_image_pool** for the **Available** box to the
      **Enabled** box

      |image6|

#. Now test your change by browsing to http://10.1.10.100

   #. If your policy is working correctly, all of the images under **F5 Platform List** should be from **NODE #4**

   #. Other images are PNG images and have a different extension

      |image7|

Use an iRule to Retrieve Images From a Different Pool 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Now you will use an iRule to perform the same image retrieval. Your
   **image_pool** is already created

#. Go to **Local Traffic >> iRules** and select **Create**

   #. **Name:** access_image_pool
   
   #. In the **Definition** section enter the following:

      |image8|

   #. This activity is not meant to be “cut and paste”. We want you to get
      comfortable and familiar with typing iRules inside the GUI.

      #. Try hovering the cursor over a command or event, such as,
         **HTTP_REQUEST** or **HTTP:uri**. You will see a definition of the
         item. For example:

         |image9|

#. Save your iRule and go to the **Resources** section of your
   **secure_vs** and select **iRules >>** Manage

   #. Move your **access_image_pool** iRule into the **Enabled** box

#. Test your **secure_vs** virtual by going to **https://10.1.10.105**

   #. The results should be the same as before

#. **Extra Credit!** Change both the policy and iRule to access the
   **image_pool** for **png** file types

   #. You should notice one is easier to update than the other
   
.. |image0| image:: images/image1.png
   :width: 7.78919in
   :height: 0.85714in
.. |image1| image:: images/image2.png
   :width: 3.71849in
   :height: 1.89076in
.. |image2| image:: images/image3.png
   :width: 0.25836in
   :height: 0.21669in
.. |image3| image:: images/image4.png
   :width: 7.5in
   :height: 0.55208in
.. |image4| image:: images/image5.png
   :width: 4.19936in
   :height: 2.52101in
.. |image5| image:: images/image6.png
   :width: 1.5276in
   :height: 0.9916in
.. |image6| image:: images/image7.png
   :width: 4.50581in
   :height: 2.0084in
.. |image7| image:: images/image8.png
   :width: 4.42946in
   :height: 3.80833in
.. |image8| image:: images/image9.png
   :width: 6.48958in
   :height: 2.20843in
.. |image9| image:: images/image10.png
   :width: 4.98783in
   :height: 2.08333in
