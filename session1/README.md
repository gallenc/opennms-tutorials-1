# Session 1 Introduction to OpenNMS

[Main Menu](../README.md) | [Session 1](../session1/README.md)

## Contents

1. What is OpenNMS
2. Getting Started Tutorial
* Installation
* Running docker compose example 1
* Importing a network - discovery
* SNMP community strings
* Importing a network - requisitions

[Session 1 Video](https://youtu.be/NYE9KmMtooY)

## What is OpenNMS

### Overview

OpenNMS is the world first enterprise and service provider grade service assurance platform created using the open source model.
OpenNMS was first published in the [sourceforge opennms project](https://sourceforge.net/projects/opennms/) over 20 years ago. 
More recently, it has migrated and is now maintained maintained in the official [github opennms project](https://github.com/OpenNMS/opennms)
Since that time the number of users and contributors has grown into a world wide community of people who use OpenNMS for a variety of purposes.

* Service Providers - use multiple instances of OpenNMS as a service assurance components within in their operations support stack
* Large Enterprises - rely on on OpenNMS as their primary netowrk monitoring platform for  very large scale enterprise networks
* Small Enterprises - significnalty reduce the work load for their IT team by monitoring their networks and services using OpenNMS
* OEM Vendors - package OpenNMS as an IP mediation layer within their own Network or Element management solution set.
* Researchers and Educators - leaverage OpenNMS for researching or teaching topics in Service Assurance for next generation networks.

An overview of the major capabilities of OpenNMS is provided in the following slide presentation [OpenNMS_Overview-v1.0.pdf](../session1/OpenNMS_Overview-v1.0.pdf)

### Documentation
Comprehensive documentation on using the OpenNMS project is provided through the [OpenNMS Documentation Site](https://docs.opennms.com/start-page/1.0.0/index.html).

You will see that documentation is available for both the Meridian and Horizon distributions of OpenNMS.

Horizon is the cutting edge, fast moving (but unsupported) distribution of OpenNMS.
New major releases of Horizon come out every 3-6 months.
Horizon will always contain the latest features as it is the OpenNMS development stream.
However, there is no guarantee of backwards compatibility of API's or configurations between Horizon releases.

Meridian is the hardened long term supported version of OpenNMS. 
New major releases of Meridian come out once a year and each major release is supported for 3 years through point releases providing bug fixes or security patches.
API's and configurations devloped for a given Meridian release are guaranteed not to require updating over the 3 year life of the platform.
This makes Meridian a more suitable choice for a production environment.

This tutorial and it's examples have currently been designed to work with [Horizon 33](https://docs.opennms.com/horizon/33/index.html) but the principles covered will also apply to [Meridian 2024](https://docs.opennms.com/meridian/2024/index.html).

As you work through these tutorials you should also familiarise yourself with the relevant documentation of each feature covered as the documentation will provide more details of options which aren't included in the examples and also any minor differences between Meridian and Horizon.

### Other Sources of Information

Over the years, many contributors have added insights or examples and these are often available by searching the [OpenNMS Discourse Server](https://opennms.discourse.group/).

In addition, the [OpenNMS Mattermost chat](https://chat.opennms.com/) channels are a great place to ask questions of the community.

## Getting Started Tutorial

The OpenNMS documentation covers in some detail the process of [Setting up a production Horizon Instance](https://docs.opennms.com/horizon/33/deployment/core/introduction.html).
This can be quite daunting for a newcomer and in particular for people just wanting a system for this set of tutorials.
Fortunately, help is at hand in the form of simple installation scripts and docker-compose examples.

If you want to get started quickly with OpenNMS Horizon on a bare metal system or a virtual machine, you use the [opennms-forge quick install scripts](https://github.com/opennms-forge/opennms-install). These convenient scripts install a basic Horizon instance with its depencencies (Java, Postgresql etc) on Debian or Red Hat compatible systems, running the steps documented in the deployment guide. 
Use this quick starting point to evaluate and explore how to configure Horizon before moving to more complex distributed environments.

For more complex examples, you can also try the various docker-compose examples provided in the [OpenNMS Forge Stack Play repository](https://github.com/opennms-forge/stack-play).
A variety of examples are provided there which illustrate configurations for quite complex OpenNMS deployments. 
The Stack Play exmaples are kept up to date with the latest OpenNMS horizon releases and also with the latest supported releases of grafana, postgres, kafka etc.

The tutorials in this repo are in part derived from the stack play examples and will give you a good starting point for understanding both traditional and containerised OpenNMS deployments.

We are now going to start our first OpenNMS example and begin to find our way around the OpenNMS system.

Please proceed to [Exercise-1-1](../session1/Exercise-1-1.md)






