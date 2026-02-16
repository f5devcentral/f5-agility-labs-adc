Lab 2 – Creating custom scanners

Overview: This lab we will create a custom GenAI scanner

Task 1 – GenAI Scanner

1. Click on Playground on the left navigation panel

2. Click on *Build a custom scanner* button in the upper right corner

   .. image:: ../_static/lab2-custom-scanner.png
      :alt: Custom scanner dialog
      :align: center 


3. Click on *GenAI scanner*

4. The GenAI scanner dialog is displayed

   a. Enter Internal Financial Forecast - <First Initial><LastName>

   b. In the description field enter *Detect any mention of financial
      forecasts or budget data* and click the Save button

   .. image:: ../_static/lab2-custom-scanner.png
      :alt: Custom scanner dialog
      :align: center 

   c. A Save new version dialog is displayed. Here you can optionally
      change the version string and enter a comment. We will have you click
      the Save version button.

   .. image:: ../_static/lab2-save-version.png
      :alt: Save version dialog
      :align: center 

   d. We will want to test the new scanner we just created. On the
      playground page the new scanner will be located on the right side of
      the page. Click on the test button to enable testing of the scanner.

   e. In the dialog box at the bottom of the page. Enter the following
      prompt:
         “Here’sthe internal Q4 financial forecast: Total projected revenue is
         $12.5M, operating expenses are budgeted at $8.3M, and marketing is
         allocated $1.2M. Please summarize this for an executive
         presentation.”*

   f. Click the up arrow to send it. The outcome should be that the prompt
      was blocked.

   .. image:: ../_static/lab2-blocked-outcome.png
      :alt: Image of a blocked prompt in the playground.
      incorrect.
      :align: center

   g. To be able to use this scanner, the scanner version will need to be
      published. Hover your mouse over the just created version of the
      scanner and the publish button will appear. Click the Publish button.

   .. image:: ../_static/lab2-publish-scanner.png
      :alt: Publish scanner dialog
      incorrect.
      :align: center

   h. In order to use this newly published scanner, we will need to add it
   to our project. Navigate back to your project page and click the Add
   scanners button.

   .. image:: ../_static/lab2-add-scanner.png
      :alt: Add scanner to project
      be incorrect.
      :align: center


   i. Your newly published scanner will be displayed in the listing of
   configured custom scanners. Click the Add button to the right of your
   scanner.

   .. image:: ../_static/lab2-add2project-genai-scanner.png
      :alt: Add selected scanner to project
      :align: center

   j. Click on your project name at the top of the page to return to the
   project view. Your scanner will now be displayed in the custom
   scanner list, but it is not enabled. Click the Enable button.

   .. image:: ../_static/lab2-enabled-genai-scanner.png
      :alt: Enabled GenAI scanner
      :align: center


   k. Verify the scanner is enabled by clicking the Chat button on the left
      navigation and enter the prompt you used in the playground: *“Here’s
      the internal Q4 financial forecast: Total projected revenue is
      $12.5M, operating expenses are budgeted at $8.3M, and marketing is
      allocated $1.2M. Please summarize this for an executive
      presentation.”* The outcome should be that the prompt was blocked.

   .. image:: ../_static/lab2-blocked-outcome.png
      :alt: Chat with blocked GenAI prompt
      :align: center


Task 2 – Keyword scanners

1. Click Playground from the left navigation

2. Click on Build a Custom Scanner and then click on Keyword Scanner

+----------------------------------+-----------------------------------+
| Name                             | Keyword-<First Initial><Last      |
|                                  | Name>                             |
+==================================+===================================+
| Keywords                         | Enter five words or strings you   |
|                                  | want the scanner to trigger on    |
+----------------------------------+-----------------------------------+

3. Save your scanner

4. Test the scanner by toggling the Test button

5. Once satisfied it is working correctly, publish the scanner



Task 3 – RegEx Scanner

1. Click on Build a custom scanner button and click on RegEx Scanner

+----------------------------------+-----------------------------------+
| Name                             | RegEx-<First Initial><Last Name>  |
+==================================+===================================+
| Regular Expression               | bnz-\d{8}$                        |
+----------------------------------+-----------------------------------+

2. Test the RegEx in the test string box. If the string you enter
   matches it will be highlighted.

3. Save and publish the scanner



Task 4 – Changing a scanner’s mode

Up until now we’ve only used scanners that would block a prompt or
response. A scanner can be set in two other modes (Audit and Redact).

Audit – Allows the prompt to proceed while flagging it for review later.
It does not interrupt the workflow.

Redact – Mask sensitive data at the edge. The original data is
discarded, with the masked data being stored.

1. Click on Scanners on the left navigation

2. Enable one of your previously created scanners

3. The mode background should change from gray to red. Click on the
   scanners mode to see the three options.

.. image:: ../_static/lab2-scanner-modes.png
   :alt: Image showing the three scanner modes (Block, Audit, Redact)
   :align: center

4. Test the different modes through the Chat Tab to observe the
   outcomes.

5. Look at the Log messages to view the behavior of the scanner
