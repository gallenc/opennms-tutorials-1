# Exercise 1-1 

[Main Menu](../README.md) | [Session 1](../session1/README.md) | [Exercise-1-1](../session1/Exercise-1-1.md)

## Getting started with A Docker Compose Project

All of the examples in this set of tutorials use docker compose to create an OpenNMS system and also to simulate networks or servers which we wish to manage.

The first tutorial example uses the docker compose project in the folder [minimal-minion-activemq](../session1/minimal-minion-activemq)

The  [docker-compose.yaml](../session1/minimal-minion-activemq/docker-compose.yaml) file configures an opennms horizon with its database, an opennms minion and six test servers running netsnmp.

The configurations for each of the containers are held in separate folders under the [container-fs](../session1/minimal-minion-activemq/container-fs) folder

We will look at OpenNMS configuration in detail as we move forwards, but In this first example, we simply want to get OpenNMS running and scanning a simple network.

You should have already installed docker and docker compose. 
Open a terminal (or power-shell in windows) and navigate to the first example project folder and issue the following commands.

```
cd minimal-minion-activemq
docker compose up -d
```
You should see a large number of images downloading and eventually all of the services starting.
The `-d` is important as it makes the example run as a daemon in the background once it is started.

Once started, you will see output like:

```
Network minimal-minion-activemq_N000               Created
Network minimal-minion-activemq_N001               Created
Volume "minimal-minion-activemq_data-opennms"      Created
Volume "minimal-minion-activemq_data-opennms-cfg"  Created
Volume "minimal-minion-activemq_data-postgres"     Created
Container database                                 Started
Container horizon                                  Started
Container minion1                                  Started
Container netsnmp_1_2                              Started                                                                                        Container netsnmp_1_1                              Started
Container netsnmp_2_1                              Started                                                                                                                                                                          Container netsnmp_2_2                              Started
```
Since this is the first time running, OpenNMS horizon will create and populate it's database. 
This will take some time but you can watch the process by following the horizon logs as the system boots

```
docker compose logs -f horizon
```
A long listing will happen until you see something like

```
horizon  | [INFO] Invocation start successful for MBean OpenNMS:Name=KarafStartupMonitor
horizon  | [INFO] Invoking start on object OpenNMS:Name=Telemetryd
horizon  | [INFO] Invocation start successful for MBean OpenNMS:Name=Telemetryd
horizon  | [INFO] Invoking start on object OpenNMS:Name=Trapd
horizon  | [INFO] Invocation start successful for MBean OpenNMS:Name=Trapd
horizon  | [INFO] Invoking start on object OpenNMS:Name=PerspectivePoller
horizon  | [INFO] Invocation start successful for MBean OpenNMS:Name=PerspectivePoller
```
At which point, OpenNMS has booted correctly.

You should now be able to browse to the OpenNMS terminal on

http://localhost:8980/opennms

or possibly if running docker desktop in windows with IPv6

http://[::1]:8980/opennms

Login with 
```
username: admin 
password admin
```
(on a production system you should change the admin password on first log in)

## Closing docker and preserving the database

When you have finished, you should shutdown with 
```
docker compose down
```
This will preserve the database volume for when you next boot OpenNMS

If you want to delete the database and start OpenNMS from scratch next time use
```
docker compose down -v
```

## Example network

The docker compose [docker-compose.yaml](../session1/minimal-minion-activemq/docker-compose.yaml) example creates an example network illustrated in the following image

![alt text](../session1/images/examplenetwork1.drawio.png "Figure examplenetwork1.drawio.png")

The ip addresses of all of the nodes have been explicitly defined in this example rather than being randomly allocated by docker.

You will see there are two network segments.

```
 N000: 172.20.0.0/24
 N001: 172.20.2.0/24
```


The OpenNMS horizon server is on N000 and has visibility of two of the servers running NetSNMP.

```
netsnmp_1_1 172.20.0.101
netsnmp_1_2 172.20.0.102
```
Any nodes managed directly and hence on the same subnet as the horizon server are said to be in the `default` location.

You can open a terminal session log into the horizon server and ping the other services on the same subnet using the same service names defined in docker compose. 
Try the following.

```
docker compose exec horizon bash
opennms@horizon:~$ ping netsnmp_1_1

PING netsnmp_1_1 (172.20.0.101): 56 data bytes
64 bytes from 172.20.0.101: icmp_seq=0 ttl=64 time=0.164 ms

# Ctrl-C escapes from the ping

exit # leaves the container
```

An OpenNMS minion, minion1, is defined which  has two network interfaces.

```
172.20.0.25 is able to see OpenNNMS horizon on N000 
172.20.2.25 is able to see the two NetSNMP servers on N002.
```

You can log into the minion and ping the other two servers.

```
docker compose exec minion1 bash
minion@minion1:~$ ping  netsnmp_2_1
PING netsnmp_2_1 (172.20.2.101): 56 data bytes
64 bytes from 172.20.2.101: icmp_seq=0 ttl=64 time=0.056 ms

```

The minion is set up to be in the `minion1-location` location

(In this example the minion1 communicates with the horizon server using ActiveMQ but we will look at this specific configuration later).

## discovering the network with OpenNMS

We will now show how to discover a network over a range of IP addresses

Log into the OpenNMS UI as shown above and navigate to the admin page by selecting the cogs.

![alt text](../session1/images/adminPage.png "Figure adminPage.png")

Select `Configure Discovery` and enter two discovery ranges

```
172.20.0.1  172.20.0.254 with location  default
172.20.2.1  172.20.2.254 with location  minion1-location
```
Then select Save and Restart Discovery

![alt text](../session1/images/DiscoveryConfig.png "Figure DiscoveryConfig.png")

![alt text](../session1/images/DiscoveryRange.png "Figure DiscoveryRange.png")

Discovery will take 30 seconds to start but once started, the system will sequentially poll all the addresses in a given discovery range.
If the location is set to `default`, the horizon server will do a direct scan of the local network.
If the location is set to the location of the minion i.e. `minion1-location`, the minion will do a scan of the selected address range.

Discovery follows the following order

* OpenNMS first ping's all of the addresses in the range. 
* If a ping to an IP address responds, a 'new suspect' event is generated and OpenNMS will do a port scan of the IP address. Any open ports will be associated with known services for that port on that address.
* reverse DNS lookup is used to associate node names with the ip addresses.
* If SNMP is discovered on port 161, then OpenNMS will interrogate the SNMP agent to find out what it  can about the device. 

After a few minutes, OpenNMS should have discovered all of the servers in the range. 

You can examine the discovered nodes on the info/nodes page

![alt text](../session1/images/nodes.png "Figure nodes.png")

If you click on `show interfaces` you will see the interfaces associated with each node.

![alt text](../session1/images/nodesandinterfaces.png "Figure nodesandinterfaces.png")

Note that 172.20.0.1 and 172.20.2.1 are the discovered docker gateway interfaces on the N000 and N001 networks.

172.20.0.25 is the N000 network interface discovered on minion1. 
This has a docker dns entry of minion1.minimal-minion-activemq_N000    

127.20.2.25 is the N001 network interface discovered on minion1. 
THis has no docker dns entry and so is just named after it's ip address

Try adding any other IP addresses of equipment you want to discover.

The discovery configuration seen on the UI is backed by a file discovery-configuration.xml in the OpenNMS /etc directory. 
Examine this using the following steps.

```
docker compose exec horizon bash
opennms@horizon:~$ cd etc
opennms@horizon:~/etc$ cat discovery-configuration.xml

<discovery-configuration xmlns="http://xmlns.opennms.org/xsd/config/discovery" location="Default" packets-per-second="1.0" initial-sleep-time="30000" restart-sleep-time="86400000">
   <specific retries="1" timeout="2000" foreign-source="selfmonitor">127.0.0.1</specific>
   <include-range retries="1" timeout="2000">
      <begin>172.20.0.1</begin>
      <end>172.20.0.254</end>
   </include-range>
   <include-range location="minion1-location" retries="1" timeout="2000">
      <begin>172.20.2.1</begin>
      <end>172.20.2.254</end>
   </include-range>
</discovery-configuration>opennms@horizon:~/etc$

```
You can see that the ranges are configured in the `/etc/discovery-configuration.xml` file


## Conclusion

In this session we have provided a general overview of OpenNMS and run a docker compose example which allows us to illustrate discovering and managing a small network.

In the following sessions we will begin to look more closely at openNMS configuration


