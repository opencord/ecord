# Troubleshooting

Having troubles deploying or running E-CORD? You have come to the right place.

## Reset the nodes

Through these steps we will bring all your pods back to a clean state.  
You will need the reset scripts under:
```
CORD_ROOT/orchestration/profiles/ecord/examples/
```
### Global node
Let's start with the global node.

#### Reset Step
* SSH into the global node
* execute these commands
    ```
    cd CORD_ROOT/orchestration/profiles/ecord/examples/
    chmod +x reset-global.sh 
    ./reset-global.sh YOUR_GLOBAL_NODE_IP YOUR_PROPER_PATH_TO_THE_GLOBAL_JSON_CONFIG_FILE
    ```
    an example of the last command is:
    ```
    ./reset-global.sh 127.0.0.1 ~/global-onos-fabric.json
    ```

#### Check Step
Once the reset step is finished we can check if the state is what we expect.

##### XOS
* Login into the XOS UI at:
    ```
    http://<YOUR_GLOBAL_NODE_IP>/xos
    ```
* Click on the *vNaaS* tab on the left
* Click on *Ethernet Virtual Lines* and make sure nothing is there
* Click on *Bandwidth Profiles* and make sure there is at least one bandwidth profile
* Click on *User Network Interfaces* and make sure there that your UNIs are present

##### ONOS
* Log into ONOS_CORD: `ssh -p 8102 onos@onos-cord`
* In the ONOS CLI (onos>) verify that apps are loaded: `apps -a -s`  
  The list of applications enabled should include:
    ```
    org.opencord.ce.api
    org.opencord.ce.global
    ```


### Local Node(s)
Let's continue with all your local pods.

#### Reset Step
* SSH into the local head node
* exectue these commands
    ```
    cd CORD_ROOT/orchestration/profiles/ecord/examples/
    chmod +x reset-local.sh 
    ./reset-local.sh YOUR_LOCAL_NODE_IP YOUR_PATH_TO_ONOS_FABRIC_JSON_CONFIG_FILE YOUR_PATH_TO_THE_ONOS_CORD_JSON_CONFIG_FILE
    ```
    an example of the last one is:
    ```
    ./reset-local.sh 127.0.0.1 ~/local-1-onos-fabric.json ~/local-1-onos-cord.json
    ```

#### Check Step
Once the reset step is finished we can check if the state is what we expect.

##### ONOS Fabric
Let's start with the ONOS_CORD.
* Log into ONOS_Fabric: `ssh -p 8101 onos@onos-fabric`
* In the ONOS CLI (onos>) verify that apps are loaded: `apps -a -s`  
  The list of applications enabled should include:
    ```
    org.onosproject.segmentrouting
    org.opencord.ce.api
    org.opencord.ce.local.bigswitch
    org.opencord.ce.local.channel.http
    org.opencord.ce.local.fabric
    ```
* Check that the BigSwitchManager configuration is properly set by typing
    ```
    cfg get org.opencord.ce.local.bigswitch.BigSwitchManager
    ```
    The output should be:
    ```
    org.opencord.ce.local.bigswitch.BigSwitchManager
    name=domainId, type=string, value=<YOUR_LOCAL_NODE_IP>-fabric-onos, defaultValue=local-domain, description=Domain ID where this ONOS is running
    ```

##### ONOS CORD
Let's now take a look at the ONOS_CORD.

* Log into ONOS_CORD: `ssh -p 8102 onos@onos-cord`
* Verify that apps are loaded: `apps -a -s`  
  The list of applications enabled should include:
    ```
    org.onosproject.drivers.microsemi
    org.onosproject.cfm
    org.opencord.ce.api
    org.opencord.ce.local.bigswitch
    org.opencord.ce.local.channel.http
    org.opencord.ce.local.vee
    ```

* Verify that at least 3 devices, a netconf one and two openflow are presentbyt typing `devices` in the ONOS CLI.  
    An example is:
    ```
    onos> devices
    id=netconf:10.6.0.160:830, available=true, local-status=connected 34s ago, role=MASTER, type=SWITCH, mfr=Microsemi, hw=EA1000, sw=2.6.33-arm1-MSEA1000--00392-ge9ee017.1.0.0.96-dirty, serial=SJC162300029    , driver=microsemi-netconf, ipaddress=10.6.0.160, locType=geo, name=netconf:10.6.0.160:830, port=830, protocol=NETCONF
    id=of:0000001e08095936, available=true, local-status=connected 1m53s ago, role=MASTER, type=SWITCH, mfr=2004-2015 Centec Networks Inc, hw=48T4X, sw=3.1.16.17.alpha, serial=E101ZB141004, driver=default, channelId=10.6.0.150:48073, managementAddress=10.6.0.150, protocol=OF_13
    id=of:00003cfdfe9df039, available=true, local-status=connected 1m53s ago, role=MASTER, type=SWITCH, mfr=Nicira, Inc., hw=Open vSwitch, sw=2.3.2, serial=None, driver=ovs, channelId=10.6.0.14:34793, managementAddress=10.6.0.14, protocol=OF_13
    ```
* Check that the BigswitchManagerConfiguration is properly set by typing
    ```
    cfg get org.opencord.ce.local.bigswitch.BigSwitchManager
    ```
    The output should be:
    ```
    org.opencord.ce.local.bigswitch.BigSwitchManager
    name=domainId, type=string, value=<YOUR_LOCAL_NODE_IP>-fabric-onos, defaultValue=local-domain, description=Domain ID where this ONOS is running
    ```
* Check that the BigswitchManagerConfiguration is properly set by typing
    ```
    cfg get org.onosproject.netconf.ctl.impl.NetconfControllerImpl
    ```
    The output should be:
    ```
    org.onosproject.netconf.ctl.impl.NetconfControllerImpl
        name=sshLibrary, type=string, value=apache_mina, defaultValue=apache_mina, description=Ssh Llbrary instead of Apache Mina (i.e. ethz-ssh2
        name=netconfConnectTimeout, type=integer, value=120, defaultValue=5, description=Time (in seconds) to wait for a NETCONF connect.
        name=netconfReplyTimeout, type=integer, value=120, defaultValue=5, description=Time (in seconds) waiting for a NetConf reply
    ```

### UNI Configuration assurance
Before pushing the UNI  there is one last set of checks to make to be sure the state is properly configured:
* SSH into the global node
* Log in the global ONOS: `ssh -p 8102 onos@onos-cord`
* type `bigswitch-ports`
* find the port where *mefPortType=UNI* and write down its port number.   
    An example is
    ```
    onos> bigswitch-ports
    DefaultPortDescription{number=1, isEnabled=true, type=FIBER, portSpeed=1000, annotations={interlinkId=EE-1-to-fabric, portName=eth-0-50, mefPortType=INNI, portMac=00:1e:08:09:59:69, domainId=10.90.1.30-cord-onos}}
    DefaultPortDescription{number=2, isEnabled=true, type=FIBER, portSpeed=1000, annotations={portName=Optics, mefPortType=UNI, domainId=10.90.1.30-cord-onos}}
    ```
    Where UNI port is number 2
* log into the XOS UI. 
    ```
    http://<YOUR_GLOBAL_NODE_IP>/xos
    ```
* Click on the vNaaS tab on the left
* Click on User Network Interfaces
* Please make sure the port number (the number after the / in the Cpe id) matches what you wrote down. 
  If not, press the hour glass at the far right and edit the number after the / making it match yours. Then on the bottom right click the save and you are done.
  
## RESET Successful
At this point you should be again in a clean state with all the configuration properly pushed.

## Data Plane Debugging
So you have successfully set up the EVC in the CORD UI. And yet when
you try to ping, traffic does not seem to go through. Here we will
walk you through the different steps to check if and where the data
path is failing.

1. First you want to check the flow rules on the MicroSemi SFP.
To do that, log into the ONOS CORD instance on the pod. You can use
the UI (port 8182) or the CLI (port 8102). You should see a flow rule
that matches on the c-tag (you specified it when creating the EVC) and
pushes the s-tag (this is automatically allocated) on top. A second rule
reverses this process: match on s-tag and pop it off. This is all we
can verify for this part of the data path, as the Microsemi driver
does not support port or flow stats (thus we cannot confirm if the
packets are actually hitting the device).

2. The next step is to verify the Ethernet edge (Centec in our case)
   switch. This one is also controlled by the same ONOS CORD
   instance from the previous step. First, check if a meter has been
   installed on the switch; the parameters should match the bandwidth
   profile you selected when provisioning the EVC. Next, you should
   also see a flow rule that matches on the s-tag and outputs to the
   fabric port. A second flow rule does the same for the reverse path.
   You can also look at the port counters of the Ethernet edge and
   confirm that both the input port (where the SFP is connected) and
   the output port (where the fabric is connected) are steadily incrementing
   as you'd expect from your ping.

3. To check the fabric data path, start by connecting to the ONOS
   FABRIC instance. This time you will need to connect to the CLI
   (port 8101). First check if the VLAN cross connect is properly
   installed, by checking the output of the ```netcfg```
   command. Search for the *xconnect* configuration for
   *org.onosproject.segmentrouting*, then verify (1) the given vlan
   corresponds to the s-tag, and (2) the given ports connect the
   fabric to the Ethernet edge and to the transport/next
   CO. Afterwards, check the output of the ```flows``` command, where
   you can see how the VLAN cross connect gets instantiated in the
   switch's tables. Again, check the input/output ports and the VLAN
   ID. You can also relate this to the output of the ```groups```
   command. An example output is given below.

```
onos> flows
deviceId=of:0000cc37ab7cc2fa, flowRuleCount=7
    id=7300003f50706b, state=ADDED, bytes=0, packets=0, duration=14, liveType=UNKNOWN, priority=32768, tableId=10, appId=org.onosproject.segmentrouting, payLoad=null, selector=[IN_PORT:37, VLAN_VID:1], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:20, meter=None, cleared=false, metadata=null}
    id=7300004524fea3, state=ADDED, bytes=0, packets=0, duration=14, liveType=UNKNOWN, priority=32768, tableId=10, appId=org.onosproject.segmentrouting, payLoad=null, selector=[IN_PORT:30, VLAN_VID:1], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:20, meter=None, cleared=false, metadata=null}
    id=730000009feee7, state=ADDED, bytes=0, packets=0, duration=13, liveType=UNKNOWN, priority=1000, tableId=50, appId=org.onosproject.segmentrouting, payLoad=null, selector=[VLAN_VID:1], treatment=DefaultTrafficTreatment{immediate=[], deferred=[GROUP:0x40010000], transition=TABLE:60, meter=None, cleared=false, metadata=null}
    id=1000082927ca9, state=ADDED, bytes=283894, packets=3336, duration=4911, liveType=UNKNOWN, priority=40000, tableId=60, appId=org.onosproject.core, payLoad=null, selector=[ETH_TYPE:lldp], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:CONTROLLER], deferred=[], transition=None, meter=None, cleared=true, metadata=null}
    id=10000a168723c, state=ADDED, bytes=269365, packets=3169, duration=4911, liveType=UNKNOWN, priority=40000, tableId=60, appId=org.onosproject.core, payLoad=null, selector=[ETH_TYPE:bddp], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:CONTROLLER], deferred=[], transition=None, meter=None, cleared=true, metadata=null}
    id=73000030665163, state=ADDED, bytes=0, packets=0, duration=4911, liveType=UNKNOWN, priority=40000, tableId=60, appId=org.onosproject.segmentrouting, payLoad=null, selector=[ETH_TYPE:ipv6, IP_PROTO:58], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:CONTROLLER], deferred=[], transition=None, meter=None, cleared=false, metadata=null}
    id=73000030db30ed, state=ADDED, bytes=0, packets=0, duration=4911, liveType=UNKNOWN, priority=40000, tableId=60, appId=org.onosproject.segmentrouting, payLoad=null, selector=[ETH_TYPE:arp], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:CONTROLLER], deferred=[], transition=None, meter=None, cleared=false, metadata=null}
onos> groups
deviceId=of:0000cc37ab7cc2fa, groupCount=3
   id=0x1001e, state=ADDED, type=INDIRECT, bytes=0, packets=0, appId=org.onosproject.segmentrouting
   id=0x1001e, bucket=1, bytes=0, packets=0, actions=[OUTPUT:30]
   id=0x10025, state=ADDED, type=INDIRECT, bytes=0, packets=0, appId=org.onosproject.segmentrouting
   id=0x10025, bucket=1, bytes=0, packets=0, actions=[OUTPUT:37]
   id=0x40010000, state=ADDED, type=ALL, bytes=0, packets=0, appId=org.onosproject.segmentrouting
   id=0x40010000, bucket=1, bytes=0, packets=0, actions=[GROUP:0x10025]
   id=0x40010000, bucket=2, bytes=0, packets=0, actions=[GROUP:0x1001e]
```
