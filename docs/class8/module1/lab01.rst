tcpdump Switches
~~~~~~~~~~~~~~~~

Every BIG-IP has tcpdump utility natively running within the managemet plane.  The tcpdump command has several switches with different purposes.
The following are some of the most commonly used.

You can run these commands from the Jumpbox to see the output in our lab environment or you can just read through the information as a reference guide
To launch the SSH connection to the BIG-IP double click on the Putty shortcut on the desktop.  Then connect to the BIGIP01 instance.  The credentials are:

user: 'root'
password: 'admin.F5demo.com'

#. **tcpdump -D**

   To list the available interfaces for packet capture use tcpdump -D

   The 'any' interface will be taken by TMM and made into the interface '0.0'

#. **tcpdump -i**

   To capture traffic on a specific interface use tcpdump -i <interface name>. i.e. 'tcpdump -i 0.0'

   When using 0.0 for the interface on a capture make sure to use a capture filter or you will get too much information and may impact performance on the F5.

#. **tcpdump -n**

   Use tcpdump -n to disable name resolution of host names

#. **tcpdump -nn**

   Use tcpdump -nn to disable name resolution of both host names and port names

#. **tcpdump -X**

   Use tcpdump -X to show output including ASCII and hex.  This will making reading screen output easier.

#. **tcpdump -w**

   Use tcpdump -w to write the packet capture to a capture file that is readable in an application such as Wireshark.

#. **tcpdump -s**

   Use 'tcpdump -s0' to capture the full data packet.  The number following the 's' indicates the number of bits to capture of each packet.  0 indicates all.

#. **tcpdump -W 4 -C 50 -w /var/tmp/file.pcap**

   Use the -W switch to write a number of files.  In combination with the -C of filesize.  In this example we will get a rotation of 4 files each 50 Mb in size.  After the fourth file is finished the first will be deleted and a new file written and keep going until the capture is stopped.

   This is a good option for circular logging when it is unclear when an event will happen and you can stop shortly after the even happens.
