# OpenNMS Tutorials

Git Repository of OpenNMS tutorial examples

This is tutorial is version 2025-Horizon33 targeting OpenNMS Horizon 33 / Meridian 2025.

**Note** Some of the videos with each session are from a previous version of this tutorial. 
  They are still mostly aligned with the content but new videos will be created for this version as it is fully updated.

For previous versions of this tutorial see [Version History](../main/versionHistory.md)

The repository contains a number of example projects, exercises and an area called [myPracticeCourseWork](../main/myPracticeCourseWork/) for you to create and store your own example work during the course. 
As you work through the sessions, you should copy each example session into your own [myPracticeCourseWork](../main/myPracticeCourseWork/) space and work on it there. 
That way you commit save your work in your own repository while still being able to receive updates from the upstream master repository.

Please read and follow the Getting Started instructions below before doing anything else.

Enjoy &#128512;

## Contents
[session1](../main/session1/) Introduction to OpenNMS

[session2](../main/session2/) OpenNMS Configuration, Provisioning, Service and Web Site Monitoring

[session3](../main/session3/) Introduction to Events, Alarms and Traps

[session4](../main/session4/) Database Event and Alarm Enrichment with the Event Translator and Jboss Rules (Drools)

[session5](../main/session5/) Performance Data Collection using SNMP

[session6](../main/session6/) Full Worked Example, Summary and Wrap Up

[Extra Examples](../main/extraExamples/) Additional Examples in support of the tutorials

[myPracticeCourseWork](../main/myPracticeCourseWork) This is where you should create and save your own practice projects.

[pristine-opennms-config-files](../main/pristine-opennms-config-files/) Untouched OpenNMS configuration files for reference.

# Getting Started
Before doing anything else you will need to follow these getting started instructions to install the required software.

## Prerequisites

Please see the [Prerequisites Video](https://www.youtube.com/watch?v=M-FuNBkso4M) to help you get started.

In order to complete these exercises you will need docker and docker-compose for the containerised exercises. 
The easiest way to get these packages is to install [docker-desktop](https://www.docker.com/products/docker-desktop/) which is available for Windows, Mac or Linux machines. 

All of these examples have been tried on [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on Windows 11.
They should also work on Docker installed in linux  (e.g. Ubuntu 22 running in VirtualBox).

---
**NOTE**

When running the examples in [docker directly on Windows System for Linux (WSL)](https://daniel.es/blog/how-to-install-docker-in-wsl-without-docker-desktop/) rather than on  [Docker Desktop](https://www.docker.com/products/docker-desktop/), some problems have been reported when using mibbrowser to snmp walk or send snmp traps to docker containers from the host PC. 
If this is the case, you can still generate traps or do snmp walks using the Netsnmp methods described in the tutorials.

---

The following software is also needed:

[Git for windows](https://git-scm.com/download/win)

[Ireasoning Mib Browser](https://www.ireasoning.com/mibbrowser.shtml)

For certain exercises you may also need Java Open JDK (minimum version 17), An IDE (such as Eclipse, Intelij or  VS-code. I use Eclipse) and Mave installed on your machine. 

[Eclipse for Enterprise Java developers](https://www.eclipse.org/downloads/packages/release/2023-12/r/eclipse-ide-enterprise-java-and-web-developers)

[Apache Maven](https://maven.apache.org/)

## Forking this repository

You should fork this repository into your own github account and synchronise your fork with this repo. 
Instructions on how to do this are provided in [Getting Started with Git](../main/gettingStartedWithGit.md)
