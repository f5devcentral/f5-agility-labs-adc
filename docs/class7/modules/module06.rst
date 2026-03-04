Bonus Lab 6: Support and Troubleshooting
========================================

In this lab you will review basic troubleshooting commands on the BIG-IP

Objective:

-  Perform a TCPDump to watch traffic flow.

-  Obtain web page information via Curl.

Troubleshoot using TCPDump or Curl.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to your **www_vs (10.1.10.100)** virtual server and set **Source Address Translation** to **None** and click on Update.

   a. Now go to your RDP session and browse the web site. You will not be able to access it even though
      the status of the virtual is available.

      i. Because BIG-IP is not the server’s default gateway the server's response goes around the BIG-IP.

   b. The web administrator tells you everything is fine as far as he
      can see and thinks the issue is with the BIG-IP, because they
      ALWAYS think the issue is with the BIG-IP.

   c. You begin by debugging the client connections to the web servers
      through the BIG-IP using TCPDump.

2. SSH to the management port of your BIG-IP. Remember the BIG-IP is a
   full proxy. You will need two TCP dumps and therefore two SSH windows for
   the client-side connection and the server-side connection.

   a. First let’s see what if we are hitting the virtual server. At the
      Linux CLI prompt:

      i. **tcpdump -i client_vlan -X -s 128 'host 10.1.10.100 and port 80'**

         1. This is a little overkill, but a good example of syntax. We
            will only look at traffic headed for the virtual server, we
            will see the first 128 bytes (-s128) in ASCII (-X).

   b. Go to your browser and attempt to access the virtual server. You
      should see something like this:

.. code::

   19:06:26.078221 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59300 > 10.1.10.100.http: Flags [S], seq 2550232388, win 64240, options [mss 1460,sackOK,TS val 1039826130 ecr 0,nop,wscale 7], length 0
           0x0000:  4500 003c c103 4000 4006 514d 0a01 0a06  E..<..@.@.QM....
           0x0010:  0a01 0a64 e7a4 0050 9801 7544 0000 0000  ...d...P..uD....
           0x0020:  a002 faf0 289a 0000 0204 05b4 0402 080a  ....(...........
           0x0030:  3dfa 7cd2 0000 0000 0103 0307            =.|.........
   19:06:26.078243 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59302 > 10.1.10.100.http: Flags [S], seq 808627591, win 64240, options [mss 1460,sackOK,TS val 1039826130 ecr 0,nop,wscale 7], length 0
           0x0000:  4500 003c b4ea 4000 4006 5d66 0a01 0a06  E..<..@.@.]f....
           0x0010:  0a01 0a64 e7a6 0050 3032 ad87 0000 0000  ...d...P02......
           0x0020:  a002 faf0 289a 0000 0204 05b4 0402 080a  ....(...........
           0x0030:  3dfa 7cd2 0000 0000 0103 0307            =.|.........
   19:06:26.078268 IP 10.1.10.100.http > d95aecf4-7016-4b32-b8db-65d7f7af42d8.59302: Flags [S.], seq 1779535623, ack 808627592, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 993460125 ecr 1039826130], length 0
           0x0000:  4500 003c 735e 4000 ff06 dff1 0a01 0a64  E..<s^@........d
           0x0010:  0a01 0a06 0050 e7a6 6a11 8f07 3032 ad88  .....P..j...02..
           0x0020:  a012 3908 289a 0000 0204 05b4 0103 0300  ..9.(...........
           0x0030:  0402 080a 3b36 ff9d 3dfa 7cd2            ....;6..=.|.
   19:06:26.078865 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59302 > 10.1.10.100.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 1039826131 ecr 993460125], length 0
           0x0000:  4500 0034 b4eb 4000 4006 5d6d 0a01 0a06  E..4..@.@.]m....
           0x0010:  0a01 0a64 e7a6 0050 3032 ad88 6a11 8f08  ...d...P02..j...
           0x0020:  8010 01f6 2892 0000 0101 080a 3dfa 7cd3  ....(.......=.|.
           0x0030:  3b36 ff9d                                ;6..
   19:06:26.078281 IP 10.1.10.100.http > d95aecf4-7016-4b32-b8db-65d7f7af42d8.59300: Flags [S.], seq 652435236, ack 2550232389, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 993460125 ecr 1039826130], length 0
           0x0000:  4500 003c 79f1 4000 ff06 d95e 0a01 0a64  E..<y.@....^...d
           0x0010:  0a01 0a06 0050 e7a4 26e3 5f24 9801 7545  .....P..&._$..uE
           0x0020:  a012 3908 289a 0000 0204 05b4 0103 0300  ..9.(...........
           0x0030:  0402 080a 3b36 ff9d 3dfa 7cd2            ....;6..=.|.
   19:06:26.079558 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59300 > 10.1.10.100.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 1039826131 ecr 993460125], length 0
           0x0000:  4500 0034 c104 4000 4006 5154 0a01 0a06  E..4..@.@.QT....
           0x0010:  0a01 0a64 e7a4 0050 9801 7545 26e3 5f25  ...d...P..uE&._%
           0x0020:  8010 01f6 2892 0000 0101 080a 3dfa 7cd3  ....(.......=.|.
           0x0030:  3b36 ff9d                                ;6..
   19:06:26.079568 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59300 > 10.1.10.100.http: Flags [P.], seq 1:466, ack 1, win 502, options [nop,nop,TS val 1039826132 ecr 993460125], length 465: HTTP: GET / HTTP/1.1
           0x0000:  4500 0205 c105 4000 4006 4f82 0a01 0a06  E.....@.@.O.....
           0x0010:  0a01 0a64 e7a4 0050 9801 7545 26e3 5f25  ...d...P..uE&._%
           0x0020:  8018 01f6 2a63 0000 0101 080a 3dfa 7cd4  ....*c......=.|.
           0x0030:  3b36 ff9d 4745 5420 2f20 4854 5450 2f31  ;6..GET./.HTTP/1
           0x0040:  2e31 0d0a 486f 7374 3a20 3130 2e31 2e31  .1..Host:.10.1.1
           0x0050:  302e 3130 300d 0a43 6f6e 6e65 6374 696f  0.100..Connectio
           0x0060:  6e3a 206b 6565 702d 616c 6976 650d 0a43  n:.keep-alive..C
      
c. From the dump you can see you are are hitting the virtual server.  Your original client IP is in the first line of
   the dump **<client_ip>.59300** > **10.1.10.100.http:** going to the virtual server.  The dump clip shows the TCP three-way handshake between the client and the virtual server and the initial part of the request **GET/ HTTP/1.1 Host: 10.1.10.100**

3. In the second SSH window we will do an expanded **tcpdump** for the
   sake of interest.

   a. **tcpdump –i server_vlan -X –s 128 'host 10.1.10.6'**

   b. Hit your virtual server again. You see packets with your client IP heading for the servers. They just aren’t responding. So we could reasonably suspect a server issue.

.. code::

   19:23:24.394987 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59358 > 10.1.20.12.http: Flags [S], seq 792828689, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 994478442 ecr 0], length 0
           0x0000:  4500 003c 846e 4000 ff06 c539 0a01 0a06  E..<.n@....9....
           0x0010:  0a01 140c e7de 0050 2f41 9b11 0000 0000  .......P/A......
           0x0020:  a002 3908 3242 0000 0204 05b4 0103 0300  ..9.2B..........
           0x0030:  0402 080a 3b46 896a 0000 0000            ....;F.j....
   19:23:25.394816 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59358 > 10.1.20.12.http: Flags [S], seq 792828689, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 994479442 ecr 0], length 0
           0x0000:  4500 003c 8473 4000 ff06 c534 0a01 0a06  E..<.s@....4....
           0x0010:  0a01 140c e7de 0050 2f41 9b11 0000 0000  .......P/A......
           0x0020:  a002 3908 3242 0000 0204 05b4 0103 0300  ..9.2B..........
           0x0030:  0402 080a 3b46 8d52 0000 0000            ....;F.R....
   19:23:26.394885 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59358 > 10.1.20.12.http: Flags [S], seq 792828689, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 994480442 ecr 0], length 0
           0x0000:  4500 003c 8477 4000 ff06 c530 0a01 0a06  E..<.w@....0....
           0x0010:  0a01 140c e7de 0050 2f41 9b11 0000 0000  .......P/A......
           0x0020:  a002 3908 3242 0000 0204 05b4 0103 0300  ..9.2B..........
           0x0030:  0402 080a 3b46 913a 0000 0000            ....;F.:....
   19:23:27.394897 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59358 > 10.1.20.12.http: Flags [S], seq 792828689, win 14600, options [mss 1460,sackOK,eol], length 0
           0x0000:  4500 0030 847c 4000 ff06 c537 0a01 0a06  E..0.|@....7....
           0x0010:  0a01 140c e7de 0050 2f41 9b11 0000 0000  .......P/A......
           0x0020:  7002 3908 3236 0000 0204 05b4 0402 0000  p.9.26..........
   19:23:28.394998 IP d95aecf4-7016-4b32-b8db-65d7f7af42d8.59358 > 10.1.20.12.http: Flags [R.], seq 792828690, ack 0, win 0, length 0
           0x0000:  4500 0028 8480 4000 ff06 c53b 0a01 0a06  E..(..@....;....
           0x0010:  0a01 140c e7de 0050 2f41 9b12 0000 0000  .......P/A......
           0x0020:  5014 0000 322e 0000                      P...2...
1
4. First, let’s check to see if the server is responding to HTTP on port 80. On the BIG-IP in an SSH window:

   a. Do a **<ctrl-c>** to escape out of **tcpdump**, if you are still
      in it, and use **curl** to test the server.  You should get output similar to to whats below.

.. code::
   
   example: 
   
   curl -i 10.1.20.11

You should get output similar to what is below. The **-i** switch tells **curl** to output the HTTP header information also.

.. code::

   [root@bigip01:Active:Standalone] config # curl -i 10.1.20.11
   HTTP/1.1 200 OK
   Date: Sun, 08 Feb 2026 03:27:25 GMT
   Server: Apache/2.4.10 (Debian)
   X-Powered-By: PHP/7.0.17
   Cache-Control: no-cache
   Vary: Accept-Encoding
   X-COLOR: (null)
   Content-Length: 7819
   Content-Type: text/html; charset=UTF-8

   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="utf-8">
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
       <meta name="viewport" content="width=device-width, initial-scale=1">
       <!-- The above 3 meta tags *must* come first in the head; any other head content must come *after* these tags -->
       <title>F5 vLab | Home</title>

       <!-- Bootstrap -->
       <link href="/css/f5demo.css" rel="stylesheet">
       <link href="/css/bootstrap.min.css" rel="stylesheet">
       <link href="/dyncss/f5header.css" rel="stylesheet">
       <link href="/dyncss/f5footer.css" rel="stylesheet">
       <style>
         svg, g, path {
         fill: #000;
         }
         .f5main {
         border-style:solid;
         border-color:#a0bf37;
         background-color:#fff;
         border-width:10px;
         }
         .f5table .table td {
         text-align: right;
         background-color: #000;
         }
             </style>
       <script src="/js/f5demo.js"></script>
     </head>
     <body>   

b. The server is responding to the BIG-IP when directly connected, but
   not through the virtual server. Sounds like the server is routing
   around the BIG-IP, which means the BIG-IP is not the default gateway.

Turn **SNAT Automap** back on the **www_vs** virtual server
