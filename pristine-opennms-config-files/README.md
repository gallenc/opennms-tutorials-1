# Pristine OpenNMS Configuration Files

For convenience in these tutorials, the following folders are provided with the un-modified configuration files shipped with OpenNMS Horizon 33.1.6

[etc-pristine](../pristine-opennms-config-files/etc-pristine)

[xsds](../pristine-opennms-config-files/xsds)

These files are provided to help you create or modify new configurations for the tutorials.

Within any OpenNMS installation, including containers, the running configuration is always stored in the directory

```
<opennms-home>/etc
```

An unmodified configuration is also provided in the following folders: 

```
<opennms-home>/share/etc-pristine
```
Contains the OpenNMS configuration as shipped and before it has been modified.

```
<opennms-home>/share/xsds
```

Contains the xsd definition files for all of the OpenNMS XML configuration files. 
These files provide additional documentation covering all of the fields in each XML configuration. 


You can also create your own local copies from inside any horizon image using

```
docker compose cp horizon:/usr/share/opennms/share/etc-pristine ./opennms-config-files/etc-pristine
docker compose cp horizon:/usr/share/opennms/share/xsds ./opennms-config-files/xsds

```
