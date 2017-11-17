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
    ./reset-local.sh YOUR_GLOBAL_NODE_IP YOUR_PATH_TO_ONOS_FABRIC_JSON_CONFIG_FILE YOUR_PATH_TO_THE_ONOS_CORD_JSON_CONFIG_FILE
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
