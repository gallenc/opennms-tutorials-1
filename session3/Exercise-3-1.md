# Exercise 3-1 

[Main Menu](../README.md) | [Session 3](../session3/README.md) | [Exercise-3-1](../session3/Exercise-3-1.md)

In this exercise we will start with the basic event configurations generated from the camera mib and add appropriate alarm behaviour.

A set of example trap definitions to use in tests of your configuration are provided in [CHUBB SNMP Example Traps](../session3/ExampleTrapsCHUBB-TVBS-CAMERA-MIB.md). 
You should be able to create traps for all of the possible event combinations described in the mib.

A set of example configurations as a result of completing this exercise are provided in [Example Configurations](../session3/minimal-minion-activemq/example-configurations/) but try the exercise first before looking at the examples.

## Step 1 - check you can send traps and generate events with the initial config

You should have already copied your initially generated  `eventconf.xml` and `CHUBB-TVBS-CAMERA-MIB.events.xml` files into your docker compose example and restarted OpenNMS.

Check that you can generate events by sending a variety of traps from the `chubb_camera_01` container.

## Step 2 - Identify how raise and clear traps are identified and create raise and clear events.

As an example, you will see that `healthChange` traps all use the following mask

```
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
      </mask>
```

and that varbind2 is either 0(Clear) or 1(Triggered)

So we can create TWO events with different UEIs matching Cleared or Triggered traps by adding a `varbind` section to the mask

```
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
            <vbnumber>2</vbnumber>
            <vbvalue>1</vbvalue>
         </varbind>
      </mask>
      <uei>uei.opennms.org/traps/CHUBB-TVBS-CAMERA-MIB/healthChangeRAISE</uei>  
...
      <severity>Warning</severity>

```

and

```
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
            <vbnumber>2</vbnumber>
            <vbvalue>0</vbvalue>
         </varbind>
      </mask>
      <uei>uei.opennms.org/traps/CHUBB-TVBS-CAMERA-MIB/healthChangeCLEAR</uei>  
...
      <severity>Normal</severity>

```
 
Remember that the severity field can have the values  Critical, Major, Minor, Warning, Normal, Cleared, Indeterminate.
You need to decide if the same severity applies to all triggering event traps.

The restored state events should be set to `Normal`.

Test your events once you have made the changes.

## Step 3 Add a reduction key to the alarm triggeing events

For all of the raise events, turn them into type 1 alarms by adding a reduction key

```
...
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%" alarm-type="1" auto-clean="false" />
   </event>
```

## Step 4 Add a clear key to all of the clearing events

For all the clearing events, you need to turn them into type 2 alarms by adding a `clear-key`.

```
...
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%" 
         alarm-type="2"
         clear-key="uei.opennms.org/traps/CHUBB-TVBS-CAMERA-MIB/healthChangeRAISE:%dpname%:%nodeid%" 
         auto-clean="false" />
   </event>
```
Note that the `clear-key` must reference the fully expanded uei of the raise event which it will be clearing.

## Step 5 Check that the raise and clear traps create and clear alarms
Having created your configuration, load it into OpenNMS and start sending some new traps to see that you are now getting alarms which raise and clear as expected.

# Summary
Once we have completed this exercise we have grasped a good understanding of event processing in OpenNMS.

In [Example Configurations](../session3/minimal-minion-activemq/example-configurations/) you will see two approaches to decoding the alarms.

The simple approach merely adds a varbind substitution  to the reduction and clear keys.

If varbind1 determines the cause of the alarm, we need to use this to uniquely distinguish which alarm is being raised or cleared on a device.

In a type 1 alarm (raise), this is appended to the reduction-key using the substitution `%parm[#1]%`

```
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%:%parm[#1]%" alarm-type="1" auto-clean="false" />
```

In  type 2 alarm (clear), this is also appended to the clear key using the substitution `%parm[#1]%`

```      
      <alarm-data reduction-key="%uei%:%dpname%:%nodeid%:%parm[#1]%" alarm-type="2"
         clear-key="uei.opennms.org/traps/CHUBB-TVBS-CAMERA-MIB/logicInputChange:%dpname%:%nodeid%:%parm[#1]%" auto-clean="false" />

```

This simple approach works well but it has the significant disadvantage that all of the events and alarms defined this way will have the same severity.
They also still have to show substantially the same text (albeit with text substitution).

A more complete approach fully decodes all of the problem causes into separate events. 
This is more work but it allows full flexibility as to the text content and the severity of each configured event and alarm.

[Session 4](../session4/README.md)  will look at some more advanced event and alarm processing techniques.

