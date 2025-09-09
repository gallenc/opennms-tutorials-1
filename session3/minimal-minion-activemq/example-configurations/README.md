# Example Configurations

In this folder you will find example configuration files. 
Try the exercises first and then compare your work with these example.

The files are always called the following names, but the contents vary `CHUBB-TVBS-CAMERA-MIB.events.xml` `eventconf.xml`

## Events generated directly from the CHUBB-TVBS-CAMERA.mib

[events-generated-from-mib](../example-configurations/events-generated-from-mib/) These files are the result of following the steps to simply parse the CHUBB-TVBS-CAMERA.mib

To load into the running container do the following 

```
cd minimal-minion-activemq
docker compose cp ./example-configurations/events-generated-from-mib/eventconf.xml horizon:/usr/share/opennms/etc/
docker compose cp ./example-configurations/events-generated-from-mib/CHUBB-TVBS-CAMERA-MIB.events.xml horizon:/usr/share/opennms/etc/events/

# send an event to reload the daemon
docker compose exec horizon /usr/share/opennms/bin/send-event.pl uei.opennms.org/internal/reloadDaemonConfig -p 'daemonName Eventd' 
```

Note Perl is not installed by default in opennms containers but curl can be used instead (substitute --user username:password as appropriate and note \" escape characters used in powershell)

```
docker compose exec horizon curl --user admin:admin -X POST http://localhost:8980/opennms/rest/events -H 'Content-Type: application/json' -d '{\"uei\": \"uei.opennms.org/internal/reloadDaemonConfig\", \"severity\": \"NORMAL\", \"parms\": [{\"parmName\": \"daemonName\", \"value\": \"Eventd\" }]}' 
```

## Simple set of alarms created from the CHUBB-TVBS-CAMERA.mib

[simple-alarms-from-mib](../example-configurations/simple-alarms-from-mib/) These files contain a first cut at alarm definitions 

---
**NOTE**

 `varbind 1` %param[#1]% is used in the `reduction-key` and `clear-key` to differentiate alarm causes.
This simplifies event definitions but prevents different severities and text for different alarm types.

---

To load into the running container do the following 

```
cd minimal-minion-activemq
docker compose cp ./example-configurations/simple-alarms-from-mib/eventconf.xml horizon:/usr/share/opennms/etc/
docker compose cp ./example-configurations/simple-alarms-from-mib/CHUBB-TVBS-CAMERA-MIB.events.xml horizon:/usr/share/opennms/etc/events/

# send an event to reload the daemon
docker compose exec horizon /usr/share/opennms/bin/send-event.pl uei.opennms.org/internal/reloadDaemonConfig -p 'daemonName Eventd' 
```

Note Perl is not installed by default in opennms containers but curl can be used instead (substitute --user username:password as appropriate and note \" escape characters used in powershell)

```
docker compose exec horizon curl --user admin:admin -X POST http://localhost:8980/opennms/rest/events -H 'Content-Type: application/json' -d '{\"uei\": \"uei.opennms.org/internal/reloadDaemonConfig\", \"severity\": \"NORMAL\", \"parms\": [{\"parmName\": \"daemonName\", \"value\": \"Eventd\" }]}' 
```


## Fully Decoded set of alarms created from the CHUBB-TVBS-CAMERA.mib

[full-alarms-from-mib](../example-configurations/full-alarms-from-mib/) These files contain fully decoded alarm definitions 

---
**NOTE**

This example has slightly different UEI definitions because it it taken from an historical proof of concept where the alarms were fully decoded. 

In this case the alarms are fully decoded except for the logicInputChange events where varbind 1 is the logic input id and we don't know what this value means so we cannot assign a unique event.

---

To load into the running container do the following 

```
cd minimal-minion-activemq
docker compose cp ./example-configurations/full-alarms-from-mib/eventconf.xml horizon:/usr/share/opennms/etc/
docker compose cp ./example-configurations/full-alarms-from-mib/CHUBB-TVBS-CAMERA-MIB.events.xml horizon:/usr/share/opennms/etc/events/

# send an event to reload the daemon
docker compose exec horizon /usr/share/opennms/bin/send-event.pl uei.opennms.org/internal/reloadDaemonConfig -p 'daemonName Eventd' 
```
