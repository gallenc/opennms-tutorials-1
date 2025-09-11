# Session 4 - Database Event and Alarm Enrichment

[Main Menu](../README.md) | [Session 4](../session4/README.md)

[Session 4 Video](https://youtu.be/DzmvuQRQfXQ)  (first part reviews answer to session 3, at 50 min we look at Event Translator)

## Introduction

We have looked at various techniques for creating event and alarm configurations using SNMP Mibs.

Often in a real world network management situation it is necessary to modify or enhance these management objects. 
In this session we will see how to enhance or modify events and alarms to provide more meaningful service impact information to users. 

OpenNMS uses [Postgresql](https://www.postgresql.org/) as the underlying database for storing it's dynamic configurations and processing alarms and events.

Most access to the database in OpenNMS is performed through the java persistence layer which is based on the [Java Persistence API (JPA)](https://en.wikipedia.org/wiki/Jakarta_Persistence) implemented in [Hibernate](https://en.wikipedia.org/wiki/Hibernate_(framework).
More recent alarm and correlation processes such as Drools use this persistence layer.

However it still remains useful in certain circumstances for users to be able to modify the database directly using SQL queries. 

Two configurable daemons can use SQL queries in order to manipulate the tables during event processing. 
* The [Event Translator](https://docs.opennms.com/horizon/33/operation/deep-dive/events/event-translator.html) can create a new event based on an incoming event enriched with data from the database.
* [Vacuumd](https://docs.opennms.com/horizon/33/reference/daemons/daemon-config-files/vacuumd.html) is a daemon which can also process database tables on events and on a schedule. 
   In earlier versions of OpenNMS this was used to process events into alarms but that functionality is now performed by the drools rules integration.
   Although Vacuumd is mostly deprecated, it still has an important role in scheduling the removal of very old event records from the database. 
   The configuration for Vaccumd is held in [/etc/vacuumd-configuration.xml](../pristine-opennms-config-files/etc-pristine/vacuumd-configuration.xml)

We will not look further at Vacuumd in this training but will instead concentrate on the Event Translator.
Firstly, however, we will begin by examining the OpenNMS database.

## Viewing the OpenNMS database

The [session4/minimal-minion-activemq](../session4/minimal-minion-activemq) docker compose project includes a docker image of the  [pgAdmin4](https://www.pgadmin.org/) Postgresql web maintenance tool.

* https://github.com/pgadmin-org/pgadmin4
* https://hub.docker.com/r/dpage/pgadmin4/

To examine the database, you will need to start the project and wait for OpenNMS to start up.
(If the database does not exist or has been deleted, OpenNMS will recreate the `opennms` database in Postgresql before starting up).

```
cd minimal-minion-activemq

docker compose up -d

```

Once the database is created, you can view it on the pgAdmin4 database viewer at http://localhost:8888/

* username: user-name@domain-name.com
* password: minad1234

Once logged into pgAdmin4, select the `servers` tree entry.

You will be asked to enter the password for the user `postgres` to connect to the server - `database`
* user: postgres
* password:  postgres

Then under `databases` you will see the database `opennms` if it has been created.

Under `opennms` you will see an entry `schemas` and under that you will see `tables` where all 115 or so OpenNMS tables are listed.

In the top bar, under `tools` you can select the `Query Tool` which will allow you to enter queries.

THe following diagram shows the query tool open.

![alt text](../session4/images/pgadmin-1.png "Figure pgadmin-1.png")

Try entering a simple SQL command to see all the nodes in the database.

```
select * from node;
```

or to see only the node named `localhost`

```
select * from node where nodelabel = 'localhost';
```

Note that in OpenNMS, most of the model objects (Entities) are backed by a database table (although some objects are created from table joins).

Most model class names begin with `Onms...` e.g. `OnmsAlarm.java`.

The table definition in each entity class will point to the database table backing the object.
For instance [OnmsAlarm.java](https://github.com/OpenNMS/opennms/blob/opennms-33.1.6-1/opennms-model/src/main/java/org/opennms/netmgt/model/OnmsAlarm.java)

```
@Entity
@Table(name="alarms")
public class OnmsAlarm implements Acknowledgeable, Serializable {
...
}

```
You can see the JPA definitions of all of the java objects in the OpenNMS model classes folder on github.
* [/org/opennms/netmgt/model]( https://github.com/OpenNMS/opennms/tree/opennms-33.1.6-1/opennms-model/src/main/java/org/opennms/netmgt/model)

You can browse through the other entities and matching tables but for our purposes, the most important tables are `events`, `alarms` and  `node`

|Model Class | database table |
|------------|----------------|
| [OnmsAlarm.java](https://github.com/OpenNMS/opennms/blob/opennms-33.1.6-1/opennms-model/src/main/java/org/opennms/netmgt/model/OnmsAlarm.java) | alarms |
| [OnmsEvent.java](https://github.com/OpenNMS/opennms/blob/opennms-33.1.6-1/opennms-model/src/main/java/org/opennms/netmgt/model/OnmsEvent.java) | events |
| [OnmsNode.java](https://github.com/OpenNMS/opennms/blob/opennms-33.1.6-1/opennms-model/src/main/java/org/opennms/netmgt/model/OnmsNode.java)   | node  |

Spend a little time browsing the entity objects and seeing how they relate to the database.

## Event Translator

Often, not all of the information required to provide useful reporting is in a given trap and so the event presented to the user needs enriched from additional data sources.
The [Event Translator](https://docs.opennms.com/horizon/33/operation/deep-dive/events/event-translator.html) is a tool which can create a new event from an existing event which is enriched with data from the database.

The unmodified [translator-configuration.xml](/../../pristine-opennms-config-files/etc-pristine/translator-configuration.xml) runs by default in OpenNMS.

We will consider the `Improved LinkDown/LinkUp events` translator definition which improves the SNMP link events.

By themselves SNMP link events only tell us the `ifindex` of the interface which has gone down.
The name and type of this interface is not supplied in the trap.
However the name (eth0 eth1 etc) is very useful to the user.

OpenNMS regularly scans the IF table of a device and will already have the additional information for a given ifindex so we need to extract the ifindex from the trap and look up the information in the interface table before adding it into the event which the user will see.

You will see from the following mib browser walk if the trap has given us the ifindex oid, we can find the other information from the SNMP interface table previously read by OpenNMS. 

![alt text](../session4/images/ifindex.png "Figure ifindex.png")


The definition which turns an SNMP LinkDown trap into an event is in the file [etc/events/opennms.snmp.trap.translator.events.xml](/../../pristine-opennms-config-files/etc-pristine/events/opennms.snmp.trap.translator.events.xml).

```
   <event>
      <mask>
         <maskelement>
            <mename>generic</mename>
            <mevalue>2</mevalue>
         </maskelement>
      </mask>
      <uei>uei.opennms.org/generic/traps/SNMP_Link_Down</uei>
      <event-label>OpenNMS-defined trap event: SNMP_Link_Down</event-label>
      <descr>&lt;p>A linkDown trap signifies that the sending protocol entity recognizes a failure in one of the
            communication link represented in the agent's
            configuration. The data passed with the event are 1) The name and value of the ifIndex instance for the
            affected interface. The name of the
            interface can be retrieved via an snmpget of .1.3.6.1.2.1.2.2.1.2.INST, where INST is the instance returned
            with the trap.&lt;/p></descr>
      <logmsg dest="donotpersist">Agent Interface Down (linkDown Trap)
        </logmsg>
      <severity>Minor</severity>
      <!-- Removed this alarm in preference to the translated version -->
      <!-- <alarm-data reduction-key="%uei%:%dpname%:%nodeid%:%interface%" alarm-type="1" /> -->
   </event>
```

This generates an event from the SNMP link down trap with the uei `uei.opennms.org/generic/traps/SNMP_Link_Down`. 
The Event Translator should listen for this event.

Note that `<logmsg dest="donotpersist">` means that this event is never persisted to the database so we should only ever see the new event from the translator and not this original one.

All of the SNMP trap varbinds become parameters in an event and we will look at these parameters in the translator.

Remember that OpenNMS events only usually  care about the position of a varbind, not its name. 
But in this case the name is very important as the name changes with the ifindex.

The link up and down events from devices are a bit complicated to interpret, particularly since the ifIndex of a port can move around depending on the configuration of the device. 

The SNMP trap definition for a link down event will always have a varbind parameter named after the oid of the ifindex of the link which has gone down.

So we are looking for a paramater (varbind) with the name .1.3.6.1.2.1.2.2.1.1.IFINDEX where IFINDEX is the number of the interface and will tell us which row in the OpenNMS interface table we are looking for.

The following excerpt from [translator-configuration.xml](/../../pristine-opennms-config-files/etc-pristine/translator-configuration.xml) shows how this event is processed.

This shows how the `uei.opennms.org/generic/traps/SNMP_Link_Down` event is enriched with node data to create two new events  `uei.opennms.org/translator/traps/SNMP_Link_Down` and `uei.opennms.org/internal/topology/linkDown`.

```
  <!-- Improved LinkDown/LinkUp events.  Uses translator to add DB information to link status traps events -->
  <translation>
    <event-translation-spec uei="uei.opennms.org/generic/traps/SNMP_Link_Down">
      <mappings>
      
        <mapping>
          <assignment name="uei" type="field" >
            <value type="constant" result="uei.opennms.org/internal/topology/linkDown" />
          </assignment>
        </mapping>
      
        <mapping>
          <assignment name="uei" type="field" >
            <value type="constant" result="uei.opennms.org/translator/traps/SNMP_Link_Down" />
          </assignment>
          <assignment name="ifDescr" type="parameter" default="Unknown">
            <value type="sql" result="SELECT snmp.snmpIfDescr FROM snmpInterface snmp WHERE snmp.nodeid = ?::integer AND snmp.snmpifindex = ?::integer" >
              <value type="field" name="nodeid" matches=".*" result="${0}" />
              <value type="parameter" name="~^\.1\.3\.6\.1\.2\.1\.2\.2\.1\.1\.([0-9]*)$" matches=".*" result="${0}" />
            </value>
          </assignment>
          <assignment name="ifName" type="parameter" default="Unknown">
            <value type="sql" result="SELECT snmp.snmpIfName FROM snmpInterface snmp WHERE snmp.nodeid = ?::integer AND snmp.snmpifindex = ?::integer" >
              <value type="field" name="nodeid" matches=".*" result="${0}" />
              <value type="parameter" name="~^\.1\.3\.6\.1\.2\.1\.2\.2\.1\.1\.([0-9]*)$" matches=".*" result="${0}" />
            </value>
          </assignment>
          <assignment name="ifAlias" type="parameter" default="Unknown">
            <value type="sql" result="SELECT snmp.snmpIfAlias FROM snmpInterface snmp WHERE snmp.nodeid = ?::integer AND snmp.snmpifindex = ?::integer" >
              <value type="field" name="nodeid" matches=".*" result="${0}" />
              <value type="parameter" name="~^\.1\.3\.6\.1\.2\.1\.2\.2\.1\.1\.([0-9]*)$" matches=".*" result="${0}" />
            </value>
          </assignment>
        </mapping>
        
      </mappings>
      
    </event-translation-spec>
```

`<event-translation-spec uei="uei.opennms.org/generic/traps/SNMP_Link_Down">` specifies that this translation runs every time an event with a uei `uei.opennms.org/generic/traps/SNMP_Link_Down` occurs.

The are two mappings, meaning that two new events will be created.

The simplest mapping will create a duplicate of the original event including all of its parameters (varbinds) but with a new uei `uei.opennms.org/internal/topology/linkDown`

```
        <mapping>
          <assignment name="uei" type="field" >
            <value type="constant" result="uei.opennms.org/internal/topology/linkDown" />
          </assignment>
        </mapping>
```

The more complicated mapping will create a new  event `uei.opennms.org/translator/traps/SNMP_Link_Down` with all of its varbinds copied.

The `snmpInterfaces` table is populated for a node if it has an SNMP agent which has successfully been read by OpenNMS.
This table can contain useful additional information which is not in the original trap to identify the description, name and alias of the interface.
The new event we create will also have these fields populated. 

In this code snippet, we can see that we are adding a new parameter called `ifDescr`.

```
          <assignment name="ifDescr" type="parameter" default="Unknown">
            <value type="sql" result="SELECT snmp.snmpIfDescr FROM snmpInterface snmp WHERE snmp.nodeid = ?::integer AND snmp.snmpifindex = ?::integer" >
              <value type="field" name="nodeid" matches=".*" result="${0}" />
              <value type="parameter" name="~^\.1\.3\.6\.1\.2\.1\.2\.2\.1\.1\.([0-9]*)$" matches=".*" result="${0}" />
            </value>
          </assignment>
```

If the sql statement returns no result, the `ifDescr` parameter will be given the default value `Unknown`.

The SQL is actually a JDBC query where each `?` mark is substituted with a value.

The first value is the `nodeid` of the node creating the event. 
The `matches=".*"` allows us to use a regular expression to only match certain values.
In this case all values of nodeid are matched so we are looking for the interface on this node.

The second value will be the contents of a varbind which gives the ifindex of the link that has gone down

In this case we are using a regular expression to extract the last number characters from the .1.3.6.1.2.1.2.2.1.1.nn oid
The last number will be the ifindex we can use to look up the interface

Armed with the nodeid and interfce ifindex, we can look up the interface in the snmpInteface table and extract the snmpIfdescr i.e. the interface descriptor field. 
This is repeatec for al l3 values we want to include in the new event.

If this appear complicated, dont worry, it is but it shows us the power of the event translator to usefully enhance the date in events and alarms.

 but in most cases we don't need to use such complex configurations and the configuration for the next use case will be much simpler.


[Exercise-4-1](../session4/Exercise-4-1.md) walks through the process of creating an example event translator for a real use case.




