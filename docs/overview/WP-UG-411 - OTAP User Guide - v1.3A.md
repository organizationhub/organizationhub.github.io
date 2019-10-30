![M:\\Cloud Vault\\Latest Searches\\Search Results 27.5.2016 8.40
(2)\\Wirepas\_logo\_2016\_slogan\_RGB (ID
20915).png](./WP-UG-411 - OTAP User Guide - v1.3A/media/image1.png){width="5.815951443569554in"
height="2.3055555555555554in"}


---
layout: default
title: OTAP
parent: APIs
nav_order: 4
---

**Table of Contents**

[1 Introduction 3](#introduction)

[2 OTAP Use Case - Network OTAP 3](#otap-use-case---network-otap)

Introduction
============

Wirepas Mesh provides means for updating the firmware and single-MCU
application running in the devices that constitute a network. This is
called the OTAP (Over-The-Air-Programming) procedure. Updating the
firmware, application or both to the network of devices needs careful
planning and utilization of known working procedures. This document
explains how to perform the OTAP procedure successfully, and what needs
to be considered when preparing for OTAP.

OTAP Use Case - Network OTAP
============================

Network OTAP is the safest way to upgrade a running Wirepas Mesh
network. It can be used to upgrade a Wirepas Mesh network even when a
new stack firmware release is not backward-compatible.

Network OTAP is initialized by uploading a new scratchpad (scratchpad
sequence number is higher than the current one in the network) to all
sinks using Wirepas Mesh Dual-MCU API. First the stack in sink is
stopped, then the new scratchpad is uploaded to the sink, then the sink
is started again. As the scratchpad has higher sequence number compared
to the earlier, the scratchpad gets copied from device to device in the
network. The devices will inform back to the sink when they have
received the new scratchpad. Once all devices have reported that they
have successfully received and stored the new scratchpad, update command
is sent from the sink to the devices in the network and devices will
process the new scratchpad, boot and then start forming the network
again.

User Guide
==========

For more information, check OTAP user guide.
