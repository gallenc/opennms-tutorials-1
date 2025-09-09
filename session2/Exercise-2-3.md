
# Exercise 2-3 OpenNMS Event Configuration

[Main Menu](../README.md) | [Session 2](../session2/README.md) | [Exercise-2-3](../session2/Exercise-2-3.md)

### OpenNMS Event Configuration

The top level OpenNMS event configuration file is [etc/eventconf.xml](../../main/pristine-opennms-config-files/etc-pristine/eventconf.xml).
This can contain its own event configurations but can also reference a list of event configuration files in the [etc/events](../../main/pristine-opennms-config-files/etc-pristine/events) folder.

You will see that the [etc/events](../../main/pristine-opennms-config-files/etc-pristine/events) folder contains event definition files for the SNMP traps from many vendors with the vendor's prefixing the name of the event definition file.

It also contains a number of definitions for events generated internally by OpenNMS.
These event files are usually named with `opennms` as the file name prefix.
For example [opennms.internal.events.xml](../../main/pristine-opennms-config-files/etc-pristine/events/opennms.internal.events.xml)

The order in which the event definition files are referenced in [etc/eventconf.xml](../../main/pristine-opennms-config-files/etc-pristine/eventconf.xml) is important.
The first matching event definition found when interpreting an event will always be used. 

For this reason, the last file in the list is always [etc/events/opennms.catch-all.events.xml](../../main/pristine-opennms-config-files/etc-pristine/events/opennms.catch-all.events.xml) because this defines the event definition we have already seen for a completely unknown trap.

Normally, we wouldn't want to alter a configuration directly inside a container because we will loose this configuration when the container re-starts.
However as an experiment, we are going to directly modify the eventconf.xml inside our container to include a definition for this new trap.

The netsnmp containers already have the [vi editor](https://devhints.io/vim) installed but and you can use this if you wish

However we can make our lives a bit simpler if we also install the [nano editor](https://www.nano-editor.org/dist/latest/nano.html) because it is easier to use.

(Note you may also need to change the background colour of the powershell to black to see all of the characters when editing xml markup. Use Powershell>properties>colors>screen background)

```
# log into the opennms horizon container as the root user
docker compose exec -u root horizon bash

# install nano
root@horizon:/usr/share/opennms# apt-get install -y nano

# exit as root user 
root@horizon:/usr/share/opennms# exit

```
Now we can use nano to edit our configuration

```
# log into the opennms horizon container as the normal user
docker compose exec horizon bash

# edit the eventconf.xml file
nano etc/eventconf.xml
```
You can now paste the following event definitions into the file between the end of the `</global>` definition and the first `<event-file>` in the list

```
   ...
   </global>

   <event>
      <mask>
         <maskelement>
            <mename>id</mename>
            <mevalue>.1.3.6.1.4.1.52330.6.2</mevalue>
         </maskelement>
         <maskelement>
            <mename>generic</mename>
            <mevalue>6</mevalue>
         </maskelement>
         <maskelement>
            <mename>specific</mename>
            <mevalue>1</mevalue>
         </maskelement>
         <varbind>
             <vbnumber>3</vbnumber>
             <vbvalue>1</vbvalue>
         </varbind>
      </mask>
      <uei>uei.opennms.org/traps/example_trap_definition_RAISE</uei>
      <event-label>example trap definition 1 RAISE</event-label>
      <descr>example trap definition 1 RAISE varbind1=%parm[#1]% varbind2= %parm[#2]% varbind3= %parm[#3]%
      </descr>
      <logmsg dest="logndisplay">example trap definition 1 RAISE varbind1=%parm[#1]% varbind2= %parm[#2]% varfbind3= %parm[#3]%
      </logmsg>
      <severity>Warning</severity>
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%" alarm-type="1" auto-clean="false"/>
   </event>
   <event>
      <mask>
         <maskelement>
            <mename>id</mename>
            <mevalue>.1.3.6.1.4.1.52330.6.2</mevalue>
         </maskelement>
         <maskelement>
            <mename>generic</mename>
            <mevalue>6</mevalue>
         </maskelement>
         <maskelement>
            <mename>specific</mename>
            <mevalue>1</mevalue>
         </maskelement>
         <varbind>
             <vbnumber>3</vbnumber>
             <vbvalue>0</vbvalue>
         </varbind>
      </mask>
      <uei>uei.opennms.org/traps/example_trap_definition_CLEAR</uei>
      <event-label>example trap definition 1 CLEAR</event-label>
      <descr>example trap definition 1 varbind1=%parm[#1]% varbind2= %parm[#2]% varfbind3= %parm[#3]%
      </descr>
      <logmsg dest="logndisplay">example trap definition 1 CLEAR varbind1=%parm[#1]% varbind2= %parm[#2]% varfbind3= %parm[#3]%
      </logmsg>
      <severity>Normal</severity>
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%"
                  alarm-type="2"
                  clear-key="uei.opennms.org/traps/example_trap_definition_RAISE:%dpname%:%nodeid%"
                  auto-clean="false"/>
   </event>

   <event-file>events/opennms.snmp.trap.translator.events.xml</event-file>
   ... 
```
Save eventconf.xml and exit the editor.

There is a pearl command in the `bin` directory to directly send events into OpenNMS.
We can use this to send an event into OpenNMS to force a reload the new event configuration without restarting OpenNMS.

```
bin/send-event.pl uei.opennms.org/internal/reloadDaemonConfig -p 'daemonName Eventd'
```

You will see a reloadDaemonConfigSuccessful event in the event list

Using the mib browser or netsnmp, try sending the traps you previously sent and see how they are presented in the event list.

```
# log into the netsnmp_1_1 container

docker compose exec netsnmp_1_1 bash

# send an example trap definition 1 RAISE trap to horizon port 1162 using netsnmp

snmptrap -v 2c -c public horizon:1162 ""  .1.3.6.1.4.1.52330.6.2.0.1        .1.3.6.1.4.1.52330.6.2.7.0  s xxxx   .1.3.6.1.4.1.52330.6.2.1.0 i 0  .1.3.6.1.4.1.52330.6.2.5.0 i 1

# send an example trap definition 1 CLEAR trap to horizon port 1162 using netsnmp

snmptrap -v 2c -c public horizon:1162 ""  .1.3.6.1.4.1.52330.6.2.0.1        .1.3.6.1.4.1.52330.6.2.7.0  s xxxx   .1.3.6.1.4.1.52330.6.2.1.0 i 0  .1.3.6.1.4.1.52330.6.2.5.0 i 0

```
You will see corresponding WARNING and NORMAL events on the event list

You will also see a corresponding WARNING alarm with an event count of each raise trap sent and a CLEARED alarm if the clear event is sent.

![alt text](../session2/images/reloadEvents.png "Figure reloadEvents.png")

## how does the event definition work

We have created an event configuration for two events which also create and clear an alarm.

All events defined in OpenNMS must have a unique identifier called the `<uei>`

The event definitions contain a `<mask>` element which you can see match the suggested trap masks in the original unformatted events. 

The event definitions can also also contain one or more `<varbind>` elements which define the sequential number of the varbind and the corresponding varbind value to match. 
Remember, OpenNMS ignores the OID of the varbind but uses the `<vbnumber>` order in which the varbind is declared. 

```
         <varbind>
             <vbnumber>3</vbnumber>
             <vbvalue>0</vbvalue>
         </varbind>
```

The `<descr>` and `<logmsg>` fields contain the human readable text for the event.
The varbinds can be included in this text using the excape sequence %parm[#3]% etc.
HTML escape sequences can also be included in the text but because the event file is in XML, we need to use character codes for reserved characters such as &lt; for < characters.

The `<severity>` field can have the values (corresponding to standard ITU X733 definitions)

```
    Critical: Numerous devices are impacted by an event.
    Major: A device is down, or in danger of going down.
    Minor: A part of a device (service, interface, power supply, and so on) has stopped functioning.
    Warning: An event has occurred that may require action.
    Normal: Informational severity message; no action is required.
    Cleared: Indicates that an alarm with a self-clearing error condition has been corrected, and service is restored.
    Indeterminate: No severity could be associated with the event.
```
## how does the alarm definition work

You will see that the `example trap definition 1 RAISE` event also contains an `<alarm-data>` element.
```
      <uei>uei.opennms.org/traps/example_trap_definition_RAISE</uei>
      <severity>Warning</severity>
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%" alarm-type="1" auto-clean="false"/>
```

Event definitions which RAISE alarms are defined as alarm-type="1"

The `reduction-key` is designed to match the unique instance of this alarm.
This matches the alarms UEI and also the distribted poller name  - which broadly corresponds to the minion or core instance receiving the trap and the nodeid which is the unique identifier of the node. 
Whenever a trap matching the mask and varbinds is first received, a new alarm is created.
Subsequent matching taps will just increase the event count on the alarm but will not create a new alarm.

You will see that the `example trap definition 1 CLEAR` event  contains a more complex  `<alarm-data>` element.

```
      <uei>uei.opennms.org/traps/example_trap_definition_CLEAR</uei>
      <event-label>example trap definition 1 CLEAR</event-label>
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%"
                  alarm-type="2"
                  clear-key="uei.opennms.org/traps/example_trap_definition_RAISE:%dpname%:%nodeid%"
                  auto-clean="false"/>
```

Event definitions which CLEAR alarms are defined as alarm-type="2" and also include a `clear-key` which you will see is designed to match the reduction-key of the event which initially raised the alarm.

When an alarm-type="2" event is received, the corresponding alarm is cleared and eventually within a few minutes removed from the alarm list.

Note that a subtle change in alarm behaviour can be set by overriding properties in the [etc/opennms.properties](../../main/pristine-opennms-config-files/etc-pristine/opennms.properties)  file.

```
###### Alarmd Properties ######
#
# Enable this property to force Alarmd to create new alarms when an problem re-occurs and the
# existing Alarm is in a "Cleared" state.
#
# Default: false
#org.opennms.alarmd.newIfClearedAlarmExists = false
```
Setting this property true will always ensure that a new alarm is created for a raise event if a previous instance has cleared. 

We can also change how alarms are displayed and generate a sound on alarm updates

```
###### Alarm List Page Options ######
# Several options are available to change the default behaviour of the Alarm List Page.
# <opennms url>/opennms/alarm/list.htm 
#
# The alarm list page has the ability to generate a sound either on each new alarm
# or (more annoyingly) on each change to an alarm event count on the page.
#  
# Turn on the sound feature. Set true and Alarm List Pages can generate sounds in the web browser.
# opennms.alarmlist.sound.enable=false
#
# Set the default setting for how the Alarm List Pages generates sounds. The default setting can be 
# modified by users for the duration of their logged-in session using a drop down menu . 
#    off = no sounds generated by the page.
#    newalarm = sounds generated for every new alarm in the page
#    newalarmcount = sounds generated for every increase in alarm event count for alarms on the page
#
# opennms.alarmlist.sound.status=off

# By default the alarm list page displays acknowledged and unacknowledged alarms in separate search tabs
# Some users have asked to be able to see both on the same page. This option allows the alarm list page 
# to display acknowleged and unacknowledged alarms on the same list but unacknowledged alarms
# flash until they are acknowledged.
#
# opennms.alarmlist.unackflash=false

```

## Summary
You can see that an important part of designing an event configuration is deciding which events raise alarms and which events clear them.

There are many more options than we have covered for alarm and event definitions. 
A good source of information is the [alarm and event documentation](https://docs.opennms.com/horizon/33/operation/deep-dive/alarms/configuring-alarms.html)

But also looking at the [share/xsds/eventconf.xsd](../../main/pristine-opennms-config-files/xsds/eventconf.xsd) can also reveal more options.

We will look a real alarm definition exercise in much more detail in the next session.

 

