Disclaimer
==========

| I am a noob
|
| That said...
|
| Slides available at github.com/matthewcroughan/presentations
| website at croughan.sh

Balena does things for you?
==========================

BalenaOS removes many steps to deploying a system, so that you can get on with your life:

|   https://www.balena.io/docs/reference/OS/overview/2.x/
|   https://www.balena.io/engine/

.. container:: progressive

  * Logging is switched to ram (Less writes)
  * Read only root partition prevents corruption
  * RootA + RootB partitions to rollback in case of upgrade failure
  * A small VPN is auto established between devices and OpenBalena/Balena
  * Container deltas allow you to deploy only the changes between two states

Balena lets you do what..?
=========================

.. container:: handout
  
  Pushing updates between devices rather than having to manage them via ssh with something like ansible.
  Tunnelling into devices on specific ports, e.g grafana on port 3000
  The logs of devices are streamed to the Balena backend since it's stored in ram on the device
  Applications are the Dockerfiles or multi-dockerfile (multicontainer) projects you run on the device
  
Balena lets you create machines in the real world from Dockerfiles. Using it, you can:

.. container:: progressive

  - Push updates to devices
  - Tunnel into devices 
  - View the logs of those devices
  - Administrate large swarms of devices
  - Centrally manage and deploy "applications".

A different concept than just using a Linux machine
===================================================

This is great. But I can't SSH in, nothing is persistent, where even am I?

.. container:: handout

  As great as Balena is, it is a new and different way of working with machines and does require some new knowledge and getting used to.

You'll need to learn about:

.. container:: progressive

  - Containers
  - Hypervisor/Supervisor
  - Dockerfiles
  - Git
  - Balena-cli

Different approaches
====================

A way to skin a cat, one of many but a good way indeed.

Other ways to accomplish many of the same goals include:

.. container:: progressive

  * Hypriot - https://blog.hypriot.com/
  * Ansible - https://www.ansible.com/
  * Debootstrap - https://wiki.debian.org/Debootstrap
  * Buildroot - https://buildroot.org/
  * Yocto - https://www.yoctoproject.org/

Different approaches
====================

.. container:: handout

  These are all essentially just languages that can be learned in order to provision, automate and distribute Linux systems, they're all valuable in their own ways

A short summary of what the alternatives do and offer.

.. container:: progressive

  * Hypriot:

    *  Docker, preinstalled and ready to use.

  * Ansible:

    *  Agentless (requires only OpenSSH) automation of remote machines using simple scripts in a custom DSL, essentially much like running bash scripts across many many machines.

  * Debootstrap:
  
    *  Automated debian installation in a bash-like DSL.
  
  * Buildroot:
  
    *  Cross compiling toolchain for embedded linux.
  
  * Yocto:
      
    *  Toolchain and environment for embedded linux, similar to buildroot with different semantics and toolage.

How do you do it?
=================

.. container:: handout

  Balena lets you create a theoretical machine in a Dockerfile, you can test it on your local machine, then deploy it to a Pi-like device or other SOC in the real world.
  
.. container:: progressive

  * Imagine a concept for an embedded machine:

    * "Wouldn't it be nice to make Octoprint into an embedded, stateless pi that I just plug in and can reflash at any point"

  * Make a dockerfile that represents that machine

    * In just a few lines, you can create a functioning system built on a Debian userspace, for example:

.. code:: dockerfile

  FROM debian:buster
  RUN apt-get update && apt install mosquitto -y
  CMD ["/usr/sbin/mosquitto"]

.. container:: progressive

  * Test that concept by building the machine on your local machine or in the "cloud builder"

These 20 lines create a functioning machine running Octoprint
=============================================================

My project, Octobalena, uses a simple Dockerfile that can be built and ran from anywhere and pushed to remote devices via git or the balena-cli.

.. code:: dockerfile

 FROM --platform=linux/arm/v6 nunofgs/octoprint:master-alpine
 
 # Install Raspberry Pi Libs and mDNS Prerequisites for PyBonjour
 RUN apk update && apk add --no-cache avahi-compat-libdns_sd raspberrypi && \
     ln -s /opt/vc/bin/* /usr/bin/
 
 # Install build deps for various plugins, Octolapse for example. Should find a way for these to be conditionally installed in the Future.
 RUN apk add --no-cache --virtual .build-deps \
     zlib-dev \
     jpeg-dev
 # Install PyBonjour and Zeroconf
 RUN pip install --no-cache-dir https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/pybonjour/pybonjour-1.1.1.tar.gz zeroconf
 
 # Install eudev so we can use udevd properly
 RUN apk update && apk add --no-cache eudev
 
 COPY ./config.yaml /data/config.yaml
 COPY ./supervisord.conf /etc/supervisor/supervisord.conf
 COPY ./udevd.sh /app/udevd.sh
 
 CMD ["/app/udevd.sh"]


A few things I've done with it
==============================

.. container:: progressive

  * Octobalena - https://github.com/matthewcroughan/octobalena
  * MING - https://github.com/dynamicdevices/ming
  * Some vinylcutteri thing.. - https://havent-uploaded-it-yet..
  * A buoy in Hull! - More on that later


Demo 00 - Simple Mosquitto Dockerfile
=====================================

.. code:: dockerfile

  FROM debian:buster
  RUN apt-get update && apt install mosquitto -y
  CMD ["/usr/sbin/mosquitto"]

Demo 01 - Octobalena
====================
Let's deploy a 3d printer shall we?

Demo 02 - Graphing a temperature sensor with MING
=================================================

Demo 03 - Vinylcutter
====================

.. container:: handout

  Think about the standard setup for something like the vinylcutter app. Usually you would have to flash raspbian to an sd card, play around and apt install a bunch of things until your desired outcome works, fail a few times, etc. Then, at the end you have something that works, but none of the legwork you did has been recorded and you may not be able to get the device back into that state again since you didn't log what you did. The whole architecture of docker is such that you can't end up in that situation. This is the difference between an embedded system and a general system that you set up once, to be ran once until someone screws it up.


Demo 04 - Tunnels
=================
Let's:
.. container:: progressive
  Browse buoy data
  Control our 3d printers on our local 
  Look at the MING setup at home

Demo 05 - Grey boxes that can become anything at any time
=========================================================

No more reflashing sd cards! My 3d printer can be a MING if it wants to.

Demo 06 - OpenBalena
====================

But what if I don't want to be part of their botnet? I want my own botnet!

Demo 07 - Adding a new container to an existing project
=======================================================

.. container:: handout

  I've used Home Assistant at home and Kapacitor+Chronograf in the Hull buoy project. I should demonstrate how I added those here.

there's a cool component I want to use with MING, let's tack it on.

More on the buoy
================

.. container:: handout
  
  Discuss how much of a nightmare it would be to not have logs of things going wrong, the stability and ease of mind when running with Balena. Show images with feh from images/ directory

.. container:: progressive

  * Signalling via node red
  * Message transport via MQTT
  * Data visualisation by Grafana/Chronograf
