[Main Menu](../README.md) | [Session 2](../session2/README.md)

# Session 2 OpenNMS Configuration, Provisioning, Service and Web Site Monitoring

## Contents
1. OpenNMS Configuration Overview
* opennms configuration directories
* configuring a docker image
2. OpenNMS Events and Alarms and Traps
* basic events alarms and traps
* parsing a mib and creating an event configuration

## Videos
[OpenNMS Tutorial Session 2 Exercise 1](https://youtu.be/XwcCRQ2W5fU)

OpenNMS Tutorial Session 2 Exercise 2 - TBD

## OpenNMS Configuration Overview

In older OpenNMS courses, we encouraged people to edit the configuration files directly in an OpenNMS system running on a virtual machine.
This worked well but it was inconvenient to have to modify the configuration for each example exercise.

In this course, it is much more convenient to provide examples using docker where the modified configuration files are simply overlaid on the default configuration files provided in the docker image. 
Please note, however that all of the examples will work equally well in a standard installation or a container installation of OpenNMS.
The configuration files being modified are exactly the same.

Before we proceed it is important to understand the main directory locations in an OpenNMS installation and how these are mapped to the example docker compose projects which use OpenNMS docker containers.
The following figure shows the folders in a typical OpenNMS installation installed on a Linux system using a package manager.
 
![alt text](../session2/images/opennmsFolders.drawio.png "Figure opennmsFolders.drawio.png")

An RPM based installation  (RHEL, Rocky Linux, Centos etc) will follow this exact pattern .

An APT based installation (Debian, Ubuntu etc)  follows the same pattern but instead of real folders, it follows the Debian directory practices of using symbolic links to the etc, logs and share directories.

All of the configuration files are held in the `/opt/opennms/etc/` directory.

The `/opt/opennms/share/etc-pristine` directory always holds the untouched original configuration files shipped with any particular OpenNMS distribution. 
This always allows you to compare any changes with the default state.

In a non-containerised install, we advise users to version control the `/opt/opennms/etc/` directory so that you can track all changes made locally. 
A simple but effective approach to this is to turn the `/opt/opennms/etc/` directory into a git repository and base line on the untouched files. 
You can do this using the following commands:

```
cd /opt/opennms/etc/
git init
git tag -a v1.0 -m 'Initial base configuration of OpenNMS 33.0.6'
```
After this, you can commit, tag and annotate any further changes you make.

In containerised installs, you should version control the configurations injected into the container.
You will need to make sure that the files you are overlaying are baselined against the version of OpenNMS in the selected container otherwise the system may not start.
We will talk about updating an installation in a later module.

The `/opt/opennms/share/xsds` directory contains the XML Schema Definitions for the xml files. 
These xsd definitions are generated from the jaxb annotated code during the build of OpenNMS and provide detailed documentation for all of the fields in the xml configuration files.

---
**NOTE**

For these tutorials, we have provided a folder of untouched xml configurations and associated xsds in [pristine-opennms-config-files](../../main/pristine-opennms-config-files/)
You can copy and modify these as you need to in the docker compose configuration overlays.

---

## Docker Container File Mapping

OpenNMS Docker containers follow a similar pattern to the standard Linux installations, with earlier containers based on Centos and later containers using Ubuntu as the based image. 

However it is not enough just to overwrite the default container directories as the containers have been designed for easy configuration using environment variables and you need to follow additional conventions when injecting configuration files into an OpenNMS container.

### Environment variables and confd

OpenNMS containers use [confd](https://github.com/kelseyhightower/confd/tree/master) templates to create some etc configurations on startup based on injected environment variables and templates. 

The following environment variables can be injected to the container, but if they are not specified in the docker-compose.yml scripts, the shown defaults are used.
(This is why the variables are not always specified in the examples).

```
    environment:
      TZ: 'America/New_York'(4)
      POSTGRES_HOST: 'database'(5)
      POSTGRES_PORT: 5432
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      OPENNMS_DBNAME: 'opennms'
      OPENNMS_DBUSER: 'opennms'
      OPENNMS_DBPASS: 'opennms'
```
These particular database settings set values in the file [etc/opennms-datasources.xml](../../main/pristine-opennms-config-files/etc-pristine/opennms-datasources.xml)

If you overlay `opennms-datasources.xml`, the environment variables will not be applied on container start.

The minion containers use an injected configuration file [/opt/minion/minion-config.yaml](../session2/minimal-minion-activemq/container-fs/minion1/opt/minion/minion-config.yaml) to set up the internal minion /etc/*.cfg properties.

### Overlay Configuration Files 

Secondly, on startup, OpenNMS core containers copy any files in `/opt/opennms-overlay/` and replace the default files in `/opt/opennms/etc/`

So in all of the examples, the relevant files are modified in the docker compose project and injected into the container `/opt/opennms-overlay/` so that they will be copied to the `/opt/opennms/etc` folder before OpenNMS starts.

Finally, you should note that any OpenNMS Plugins can be injected into an OpenNMS instance before start up by copying the plugin files into the `/container-fs/horizon/opt/opennms/deploy` directory which maps to the karaf deploy directory within OpenNMS.

We needed to cover this introduction to configuration file locations so that you understand how the examples relate to your production OpenNMS installation. 
We will have a lot more to say about configuration as we proceed with the course.

### modifying configuration files through the UI

It is possible for a user to modify configuration files through the [OpenNMS UI File Editor](https://docs.opennms.com/horizon/33/reference/configuration/file-editor.html).
To do this, navigate to `info>FileEditor`.

![alt text](../session2/images/onmsFileEditor.png "Figure onmsFileEditor.png")

Only users with `ROLE_FILESYSTEM_EDITOR` can access this page.
The user permissions are set per user under `admin > Configure Users, Groups and On-Call Roles`

![alt text](../session2/images/onmsFileEditorRole.png "Figure onmsFileEditorRole.png")

Saved changes will be permanent in a virtual machine or bare metal install of OpenNMS but please note that in a container, these configuration changes are ephemeral and may be lost on shutdown or overridden on startup.

### Editing files directly

The OpenNMS containers already have the [vi editor](https://devhints.io/vim) installed and you can use this if you wish.

However we can make our lives a bit simpler if we also install the [nano editor](https://www.nano-editor.org/dist/latest/nano.html) because it is easier to use.

(Note you may also need to change the background colour of the powershell to black to see all of the characters when editing xml markup. Use `Powershell>properties>colors>screen background`)

```
# log into the opennms horizon container as the root user
docker compose exec -u root horizon bash

# install nano 
root@horizon:/usr/share/opennms# microdnf -y install nano

# exit as root user 
root@horizon:/usr/share/opennms# exit

```
Now we can use nano to edit our configuration

## Editing externally and copying files in or out of container

You can copy files into and out of the container if you want to preserve them using the `docker compose cp` command as illustrated below.

```
# copy out of the container to the local directory `.`

docker compose cp horizon:/usr/share/opennms/etc/eventconf.xml .

# or copy from the local directory into the container

docker compose cp ./eventconf.xml:horizon:/usr/share/opennms/etc/
```

## Reloading configurations

Normally, configuration changes will be read when the system restarts however for a number of daemons, a daemon reload event can be sent which will restart the daemon in a running system. 

For instance, to reload the event daemon, `eventd` you can use

```
docker compose exec horizon /usr/share/opennms/bin/send-event.pl uei.opennms.org/internal/reloadDaemonConfig -p 'daemonName Eventd' 
```

Note Perl is not installed by default in opennms containers but curl can be used instead to post an event to the system (substitute --user username:password as appropriate and note \" escape characters used in powershell)

```
docker compose exec horizon curl --user admin:admin -X POST http://localhost:8980/opennms/rest/events -H 'Content-Type: application/json' -d '{\"uei\": \"uei.opennms.org/internal/reloadDaemonConfig\", \"severity\": \"NORMAL\", \"parms\": [{\"parmName\": \"daemonName\", \"value\": \"Eventd\" }]}' 
```

Finally it is also possible to reload the `Eventd` daemon from the `Karaf Shell` using:
```
ssh admin@localhost -o UserKnownHostsFile=/dev/null -p 8101 reload-daemon Eventd
```

## Provisioning Requisitions

In [Session 1](../session1/README.md) we looked at how OpenNMS can scan a network and add any devices it discovers. 

OpenNMS uses reverse DNS to make a good guess for each node name and sets up the device to be monitored in a reasonable way.
If a node has SNMP running on port 162, OpenNMS use the default SNMP `public` community strings to request information from the node. 

In many cases, however, users already know what devices and service they have and it is important to name and apply metadata or categories to these devices in a consistent way which aligns with their network inventory. 
Often it is also important for security to use secret SNMP community strings to communicate with the nodes.

In [Exercise-2-1](../session2/Exercise-2-1.md) we will look at how device information and SNMP community strings can be provisioned in OpenNMS.

## Service and Web Site Monitoring

A primary function of OpenNMS is to detect and monitor services by regularly polling them and measuring the response time.

With it's simplest configuration, OpenNMS can and monitor detect services through attempting TCP connection to well known ports such as Mysql port 3306.

OpenNMS can also make HTTP queries to known urls to detect if a service is up. This can be use to good effect with services such as a load balanced WordPress service as in the next example.

To understand this further, have a try at the [Wordpress Service Monitoring Exercise-2-2](../session2/Exercise-2-2-service-monitoring1.md)


The Business Service Monitoring feature of OpenNMS can combine discrete service monitoring actions to build up a service graph which indicates the business impact of a service failure. 

Try the [Business Service Monitoring Exercise-2-3](../session2/Exercise2-3-business-service-monitoring.md) exercise.


## Summary

In this session we have looked at provisioning OpenNMS through discovering a network, provisioning through requisitions and basic service monitoring.

In the next [Session 3](../session3/README.md) we will begin looking at SNMP Traps, Events and Alarms.
