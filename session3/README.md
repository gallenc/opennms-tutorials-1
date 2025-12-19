
[Main Menu](../README.md) | [Session 3](../session3/README.md)

# Session 3 OpenNMS Events And Alarms

## Session 3 Videos

[Session 3 Video Events And Alarms ](https://youtu.be/cEY3EKy5AFU)

## Events, Alarms and Traps

OpenNMS is an event driven system.
This means that the many processes running in OpenNMS primarily communicate with each other using internal OpenNMS events. 
OpenNMS internal events correspond to changes of state within the system. 
You will have already seen examples of these in events surrounding the discovery of new devices or the import of requisitions.
Other examples would be node down events where OpenNMS cannot communicate with a device or threshold crossing events where the system has detected that a collected value has crossed a user set threshold.

Events may also be externally generated from devices using standard event protocols communicating with OpenNMS through the network; for example SNMP traps or SYSLOGS.
Finally, it is also possible to directly inject events into OpenNMS using the ReST API.

The majority of events are also persisted in the OpenNMS event table and may then be searched and viewed through the UI.
However it is also possible to not persist certain events when they are only used for inter-process communication.

Events tell us something happened at a certain point of time but they don't record the current state of a system.
Often devices will repeatedly send multiple events (traps or logs) when they have detected a problem. 
This can lead to an 'Event Storm' where it is very hard for a user to deal with so many incoming events.

OpenNMS uses Alarms to correlate events into a current state which makes it much easier to see what is the current status of a device, service or network. 
Some events may raise an alarm and some events may cause an alarm to clear. 
Each alarm will maintain a count and a list of events contributing to the alarm state. 

In [Exercise-3-1](../session3/Exercise-3-1.md) we will cover some simple examples to inject traps into OpenNMS.

In [Exercise-3-2](../session3/Exercise-3-2.md) we will cover some simple examples to help explain how OpenNMS traps and alarms are configured.

In [Exercise-3-3](../session3/Exercise-3-3.md) we will look at parsing OpenNMS events from a MIB file.

In [Exercise-3-4](../session3/Exercise-3-4.md) we will look at a case study for monitoring motorway cameras





