![M:\\Cloud Vault\\Latest Searches\\Search Results 27.5.2016 8.40
(2)\\Wirepas\_logo\_2016\_slogan\_RGB (ID
20915).png](./WP-UG-411 - OTAP User Guide - v1.3A/media/image1.png){width="5.815951443569554in"
height="2.3055555555555554in"}
---
layout: default
title: OTAP
parent: User Guides
nav_order: 1
---

**Table of Contents**

[3 Network OTAP Procedure 3](#network-otap-procedure)

[3.1 Preparing The Scratchpad 3](#preparing-the-scratchpad)

[3.2 Checking the OTAP Status In The Network
4](#checking-the-otap-status-in-the-network)

[3.3 Check Battery Levels (Optional) 4](#check-battery-levels-optional)

[3.4 Uploading The Scratchpad To The Sink
4](#uploading-the-scratchpad-to-the-sink)

[3.5 Tracking The Scratchpad Deployment To The Network
4](#tracking-the-scratchpad-deployment-to-the-network)

[3.6 Decision For Taking The Scratchpad Into Use
5](#decision-for-taking-the-scratchpad-into-use)

[3.7 Follow-up To Know When Scratchpad Is Taken Into Use
5](#follow-up-to-know-when-scratchpad-is-taken-into-use)

[3.8 Take The Scratchpad In Use In The Sinks
5](#take-the-scratchpad-in-use-in-the-sinks)

[4 Tools for OTAP 6](#_Toc5632489)

[4.1 Wirepas Mesh API Host Library (Python)
6](#wirepas-mesh-api-host-library-python)

[4.1.1 Using The *MeshApiDevice* From *meshapi.py*
6](#using-the-meshapidevice-from-meshapi.py)

[4.1.2 Using The load\_otap\_image*.py*, find\_all\_nodes.py and
get\_network\_otap\_status.py scripts for doing the OTAP
7](#using-the-load_otap_image.py-find_all_nodes.py-and-get_network_otap_status.py-scripts-for-doing-the-otap)

[4.1.3 Using The otap\_nw.py and otap\_nw.ini scripts for doing the OTAP
7](#using-the-otap_nw.py-and-otap_nw.ini-scripts-for-doing-the-otap)

[4.2 Wirepas Terminal 8](#wirepas-terminal)

[5 Additional Tips 9](#additional-tips)

[6 References 10](#references)


Network OTAP Procedure
======================

Preparing The Scratchpad
------------------------

The OTAP scratchpad is a container file that can have one or multiple
firmware and application images. Most common use cases are a single
stack firmware image, a single application image and both the stack
firmware and an application image. If updating to a new stack firmware
version the safest OTAP scratchpad image type contains both the new
stack firmware and the application compiled with a matching SDK version.

The OTAP image with just the stack firmware is provided in the Wirepas
Mesh firmware release as a file *wirepas-firmware-*xxxx*-v*y.z*.otap*.
This can be used as is for updating just the stack firmware. If an
application is already present in the device, it is not affected.

To generate an OTAP scratchpad containing the stack firmware and
application or just the application the Python script file
*genscratchpad.py* from the Wirepas Mesh SDK is used. WP-RM-108 -- OTAP
reference Manual \[1\] Appendix A gives a description on how to use the
Python script.

Each of the image contained in the scratchpad contains a hardware ID and
a memory area ID that are also set by the *genscratchpad.py*. Those need
to match to the software running in the device for the images to be
taken into use during device boot, but the device will still store the
scratchpad and upload it to its neighbors even if itself cannot use it.
In this way the hardware IDs and memory area IDs can be used to update a
heterogenous network having devices with different hardware and
application profiles.

The OTAP scratchpads also contain a sequence number that is used to
decide which one of the scratchpad is the newer one that should be
written over the old one. The sequence number is a one byte cyclical
value that has two special values. 0 indicates that the OTAP is turned
off in this device, i.e. the device will not accept new scratchpads from
the neighboring nodes. It can still be written over through Wirepas Mesh
Dual-MCU API. Value 255 means that the scratchpad will be overwritten by
any other scratchpad that is available in the network.

Since the scratchpad sequence is cyclical it means that it wraps around
by doing the comparison in reverse if the difference of sequence numbers
is more than 128. In practice this means that for example sequence
number 1 is higher that sequence number 200. Please refer to WP-RM-100
-- Wirepas Mesh Dual-MCU API \[3\] chapter 2.7 as to how the comparison
is done in this case.

While the otap scratchpad file has its own scratchpad sequence that can
be used as a default value by the tools, it is still separately defined
during the scratchpad upload.

Checking the OTAP Status In The Network
---------------------------------------

It is a good idea to check the OTAP status in the network so that the
scratchpad sequence numbers that are stored in the devices are known
before OTAP. This is done through the Wirepas Mesh Dual-MCU API 's
MSAP-REMOTE\_STATUS.request. The Wirepas tools and APIs generally
support this. Besides the scratchpad sequence number, the remote status
responses contain a lot more information about the current running stack
firmware. See \[3\] chapter 2.3.16 for further details.

Check Battery Levels (Optional)
-------------------------------

The OTAP process consumes significantly more energy than regular network
operation. In case devices in the network are battery powered, it is a
good idea to check battery voltage levels beforehand.

The battery levels are contained in the node diagnostics coming from the
devices. See \[4\] chapter 2.3 for further details.

Uploading The Scratchpad To The Sink
------------------------------------

First step in the actual OTAP process is uploading the scratchpad to a
device in the network. In theory, this can be any device, but it is
usually done through the sinks that are used to control the OTAP
procedure for the whole network.

The scratchpad is uploaded to a sink by using the MSAP-SCRATCHPAD
services in the Wirepas Mesh Dual-MCU API. See \[3\] chapter 2.3.15 for
further details on the primitives used.

In simplified form this is done in steps:

1.  Stop the stack in the sink with MSAP-STACK\_STOP. See \[3\] chapter
    2.3.3

2.  Upload the scratchpad using the MSAP-SCRATCHPAD services mentioned
    above

3.  Start the stack in the sink with MSAP-STACK\_START. See \[3\]
    chapter 2.3.3

This should be done to all the sinks in the network to make sure that
the whole network is updated.

After the sinks have restarted, they will start uploading the new
scratchpad to their immediate neighbors that in turn will upload it to
their neighbors. In this way, all the devices in the network will get
the latest scratchpad.

Tracking The Scratchpad Deployment To The Network
-------------------------------------------------

To track the deployment of the new scratchpad the devices will send OTAP
remote status messages to sinks. These are the same primitives that the
OTAP remote status query uses. See \[3\] chapter 2.3.14.3 for details.

Due to constant reformation of the network while scratchpad is being
deployed, some of the status messages might be lost on their way to
sink. To make sure that all the nodes have the correct scratchpad, the
remote status request from chapter 3.2 can be issued again.

Decision For Taking The Scratchpad Into Use
-------------------------------------------

After all the devices in the network have received the new scratchpad,
the new scratchpad can be taken into use. To do this, an update request
command is send to all the nodes using MSAP-REMOTE\_UPDATE.request. The
request contains the sequence number for the scratchpad to be taken into
use and a delay in seconds how long the device will wait until it
reboots. Delay of 0 seconds will cancel the ongoing update request. See
\[3\] chapter 2.3.14.4 for details.

Please note that in the case that the sinks are not supposed to be
updated with the scratchpad that has been deployed to the network, a
scratchpad with the scratchpad sequence number zero needs to be saved to
the sinks before the remote update requests are sent. Otherwise the
sinks might receive remote update requests from each other and update to
the firmware that should only be running in the nodes.

Follow-up To Know When Scratchpad Is Taken Into Use
---------------------------------------------------

After the scratchpad is taken into use in the devices, the sinks should
receive boot info diagnostics messages (See \[4\] chapter 2.4) from the
nodes.

Again, due to boots in the network, some of the boot messages are
potentially lost, so it is a good idea to query the remote scratchpad
status as was done in chapters 3.2 and 3.5.

If the scratchpad contained a matching stack firmware, the "processed
scratchpad sequence number" in the scratchpad status response should
contain the new scratchpad sequence number.

In the case the scratchpad contained only an application or multiple
applications, the "processed scratchpad sequence number" will not
change. The application update then needs to be checked through
application functionality.

Take The Scratchpad In Use In The Sinks
---------------------------------------

If the sinks are also going to be using the same scratchpad then also a
local update request is send to the sink using
MSAP-SCRATCPAD\_UPDATE.request (see \[3\] chapter 2.13.15.7) and then
stopping and starting the stack through MSAP-STACK\_STOP and
MSAP-STACK\_START (see \[3\] chapter 2.3.3) to reboot the sink and allow
bootloader to take the new image into use.

[]{#_Toc5632489 .anchor}

Tools for OTAP
==============

Wirepas Mesh API Host Library (Python)
--------------------------------------

Wirepas Mesh API Host Library (Python), later shortened to Python API
library, has all the necessary functions to update a network with OTAP.
It is possible to easily write a script to do an OTAP with the functions
found in the *MeshApiDevice* class.

### Using The *MeshApiDevice* From *meshapi.py*

The *MeshApiDevice* class has all the required functions to do an OTAP.
Following the steps from chapter 3 of this document a script can be
created to do this from one or multiple sinks.

1.  Check the battery levels from node diagnostics by starting the
    diagnostics listener using *MeshApiDevice.start\_node\_diag\_rx* and
    then receiving the diagnostics with
    *MeshApiDevice.get\_node\_diag\_rx* which returns a dictionary
    containing the value "voltage" for each diagnostics message. See
    \[4\] chapter 2.3 on how to convert the values to voltages.

2.  Get the scratchpad information from the network by sending the
    remote status query with *MeshApiDevice.otap\_remote\_status* then
    starting the listening with
    *MeshApiDevice.wait\_remote\_status\_indication\_rx* and receiving
    the status messages with
    *MeshApiDevice.get\_remote\_status\_indication\_rx.* If the network
    is large then multiple status request might be needed to get all the
    answers.

3.  Load the OTAP image from a file by creating an instance of
    *otapimage.OTAPImage* and then loading the file to the instance with
    *OTAPImage.load\_file* function.

4.  Upload the OTAP image to the sink by first stopping the stack with
    *MeshApiDevice.stack\_stop* and then uploading the image with
    *MeshApiDevice.load\_otap\_image*. The OTAP sequence number can then
    be redefined with the *otap\_sequence* function parameter. Then
    start the stack.

5.  Use methods from step 2 to check the remote status from the nodes.
    The remote status indications come automatically when the nodes
    receive scratchpad but resending the status query is probably
    required.

6.  When all the nodes have received the new image then send the update
    query using *MeshApiDevice.otap\_remote\_update\_request.* The
    responses for this can be also read using the
    *MeshApiDevice.get\_remote\_status\_indication\_rx.* Note that a
    scratchpad image with a sequence number zero needs to be loaded to
    the sinks before sending the remote update requests in case that the
    sinks are not supposed to take the image in use.

7.  The boot diagnostics messages from the devices can be received using
    the *MeshApiDevice.start\_boot\_rx* and
    *MeshApiDevice.get\_boot\_rx* functions.

8.  Use the same method as in step 2 to check the status of the network
    after the update.

9.  Update the sink by stopping the sink with
    *MeshApiDevice.stack\_stop,* setting the image bootable in the sink
    by *MeshApiDevice.set\_otap\_image\_bootable*, the stopping the
    stack again to reboot the device and then starting the stack with
    *MeshApiDevice.stack\_start.*

### Using The load\_otap\_image*.py*, find\_all\_nodes.py and get\_network\_otap\_status.py scripts for doing the OTAP

> Listed scripts are delivered along with Python API 2.1.8, following
> steps describes the process

-   Check the current OTAP status of the NW to see current processed
    scratchpad:

    -   python get\_network\_otap\_status.py -c (-f config.ini)

        -   -c parameter is used to compress the list of nodes into more
            readable format. Very usable in NWs, having high amount of
            nodes

-   Load new scratchpad to NW. Sequence id is current sequence id + 1

    -   python load\_otap\_image.py \--otapfile \<path to file\>\*.otap
        \--sequence (id+1) \--noupdate \--startstack (\--configfile
        config.ini)

        -   \--noupdate used so that sink would not take nodes' FW into
            use

-   Make sure that all nodes come alive after scratchpad transfer, in
    case there are more than 1 sink, use \'-sa\' or \'\--sinkamount\'
    parameter and use the expected value for sinkamount

    -   python find\_all\_nodes.py -c (-f config.ini) (-sa \<desired
        amount\>)

-   Check that all nodes have stored the new scratchpad

    -   python get\_network\_otap\_status.py -c (-f config.ini)

-   Take new scrathpad into use for all nodes (remember to check
    sequence id used with parameter -u):

    -   python get\_network\_otap\_status.py -c -u (desired id) -d 120
        (-f config.ini)

        -   -d parameter stands for delay, which is the desired time
            that node would wait before taking the action

    -   Stop the command execution after all nodes has responded,
        although they are still using the old version

-   Check that all nodes have taken the new version into use.

    -   python get\_network\_otap\_status.py -c (-f config.ini)

-   Load new fw into sink with sequence number 0, this works for one or
    more sinks connected into GW

    -   python load\_otap\_image.py \--otapfile \<path to file\>\*.otap
        \--sequence 0 \--startstack (\--configfile config.ini) (-sa)

-   Verify that all the whole NW is up and running, not needed if
    verified already before sink update

    -   python get\_network\_otap\_status.py -c (-f config.ini)

### Using The otap\_nw.py and otap\_nw.ini scripts for doing the OTAP

This is the most automatized way for doing the OTAP, introduced along
with Python API 2.1.8 ,all you need to do is:

-   Edit the content of otap\_nw.ini to correspond your NW setup

-   Run script

    -   Python otap\_nw.py

Following things are defined in otap\_nw.ini: OTAP files for nodes and
sinks, number of nodes expected to be in the network, different timeout
values and repeat count for getting OTAP status from the network.

If node amount in network is not known, or if it can vary during the
OTAP, then nbr\_of\_nodes can be set to value 0. In this case, the
script will wait in each phase until defined timeout is reached, before
continuing to next phase, so the script execution might take a long
time. When node amount is known (nbr\_of\_nodes is set to non-zero
value), the script will continue to next phase once defined criteria
(e.g. all nodes has stored the new scratchpad) is met.

Some command line arguments can be used when executing the otap\_nw.py
script:

-   \--configfile or -f: load custom configuration .ini file

-   \--interval or -i: change default otap status polling interval

-   \--repeats or -r: define how many times update request broadcast is
    repeated

-   \--ignorelist: otap status of listed nodes is ignored

-   \--nosink: sink is not updated

Example commands:

python otap\_nw.py -f custom\_config.ini --nosink

-   if script cannot connect to sink, then custom configuration may be
    needed

-   with this -f argument, custom configuration is given. It e.g.
    defines serial connection towards sink

-   with argument --nosink, sink update is skipped

python -m wirepas.tools.otap\_nw.ini

-   otap is done with default argument values and parameter values
    defined in otap\_nw.ini file

NOTE: otap\_nw.ini needs to be located in the same directory from which
the script is started

otap\_nw.ini file itself has more information about its parameters.

There is also some information about command line arguments in the
script help

-   python otap\_nw.py -h

Wirepas Terminal
----------------

It is also possible to do an OTAP with Wirepas Terminal in Windows, but
this is mostly feasible for small networks since it is hard to keep
track on what is the status of the network and you have to connect all
the sinks to separate Wirepas Terminals.

Connect to each sink with Wirepas Terminal by serial connection or using
a serial-to-TCP/IP bridge. Check that the stack is running from the left
side bottom of the window. Choose Tools-\> Start stack if it isn't. Then
follow these steps:

1.  To get the current scratchpad sequences in the network open the OTAP
    window from Tools -\> OTAP operations.. and then press "Get status"
    from the Remote operations. Now the OTAP status messages should
    appear in the Terminal main window.

2.  To load the OTAP image in the sink stop the stack from Tools -\>
    Stop stack then select Tools -\> OTAP operations.. Select the file
    with Browse.. button and then check the Sequence number in the File
    column of the table below. Edit the sequence number if needed and
    press "Save to node". Now the OTAP image is being uploaded to the
    sink. Check from the main window that it succeeds.

3.  To deploy the scratchpad into the network, simply start the stack
    from Tools -\> Start stack. You can now follow the OTAP status
    messages from the main window. Wait 5-10 minutes on 10-20 node
    networks and up to 30 minutes on 100 node networks.

4.  Open the Tools -\> OTAP operations window again. From Remote
    operations, check that the Sequence number matches to the saved
    sequence number and edit if needed. Then check Reboot delay. 300
    seconds should be enough for most networks with the size of 10-100
    nodes. Press Update from the Remote operations frame (not the Local
    operations) and check the main window to see the update status
    messages.

5.  To update the sink after the reboot time has elapsed stop the stack
    again with Tools-\>Stack stop and select Tools -\> OTAP operations
    and press Update from the Local operations frame this time. Select
    Tools-\>Stack stop again and then Tools-\>Stack start to bring the
    sink back online. Now the network should be running the new
    software.

Additional Tips
===============

> The scratchpad deployment time depends on the size of the network
> under each sink. Very small networks take about 5 minutes and larger
> ones may take up to 30 minutes to distribute the new scratchpad to all
> devices.
>
> The reboot time should be dependent on the size of the network. Use a
> longer reboot time for larger networks.
>
> If an application needs to be removed through OTAP, it is done by
> sending an "empty" application image to the network.
>
> ![](./WP-UG-411 - OTAP User Guide - v1.3A/media/image2.png){width="0.3055555555555556in"
> height="0.3055555555555556in"}
>
> Wirepas Mesh has an OTAP failsafe mechanism which activates in the
> following situation:

-   There is a valid, new, unprocessed scratchpad stored in memory, and

-   Remote update request has not been issued yet, i.e. the update delay
    > countdown is not running, and

-   The device loses its connection to the network (no route to sink)
    and no connection can be re-established within one hour

> After having no route to sink for one hour, Wirepas Mesh stack
> firmware will automatically mark the scratchpad for processing and
> reboots the device.
>
> This failsafe mechanism is to make sure that the device will process
> the new scratchpad, even if it leaves the network for an extended
> period of time, before it can receive the remote update request. The
> expectation is, that while the node was away from the network, the
> rest of the network updated to a new stack firmware version. In rare
> cases, old and new firmware versions might not be able to communicate
> with each other. This failsafe mechanism makes sure that the stack
> firmware is updated even in such cases.

References
==========

\[1\] WP-RM-108 -- OTAP Reference Manual

\[2\] WP-RM-109 -- OTAP Migration Appendix

\[3\] WP-RM-100 -- Wirepas Mesh Dual-MCU API

\[4\] WP-RM-104 -- Wirepas Mesh Diagnostics Reference Manual
