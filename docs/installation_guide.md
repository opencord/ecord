# Installation guide

The paragraph describes how to install and configure the E-CORD global node and the local sites on a phsyical POD.

## Hardware requirements (BOM)
Following is a list of hardware needed to create a typical E-CORD deployment. References will be often made to the generic CORD BOM.
 
**NOTE**: The hardware suggested is a reference implementation. Hardware listed have been usually used by ONF and its community for lab trials and deployments to validate the platform and demonstrate it's capabilities. You’re very welcome to replace any of the components and bring in yours into the ecosystem. As a community, we would be happy to acknowledge your contribution and put new tested devices in the BOM as well.

Following the suggested hardware list is reported.

### Global node
* 1x development machine - same model used for a [generic CORD POD](https://guide.opencord.org/install_physical.html#bill-of-materials-bom--hardware-requirements)
* 1x compute node (server) - same model used for a [generic CORD POD](https://guide.opencord.org/install_physical.html#bill-of-materials-bom--hardware-requirements)

### Local site (POD)
The hardware listed is needed for each POD.

* Everything listed in the BOM of a generic CORD POD
**WARNING** Currently E-CORD does not support more than 1 fabric switch per POD. While soon will be possible to use more than one fabric switch, it is useless now to buy more than a fabric switch per local site.
* 1x Centec v350, used as “Ethernet Edge switch”
* 1x CPE, composed by
    * 1x 2-port TP-Link Gigabit SFP Media converter, model MC220L(UN)
    * 1x Microsemi EA1000 programmable SFP
    * 1x fiber cable (or DAC) to connect the CPE to the Ethernet Edge switch
* 1x 40G to 4x10G  QSFP+ module to connect the Ethernet Edge switch to the access leaf fabric switch (EdgeCore QSFP to 4x SFP+ DAC, model ET6402-10DAC-3M, part M0OEC6402T06Z)
 
**NOTE**: The role of the CPE is to get users’ traffic, tag it with a VLAN id, and forward it to the Ethernet Edge switch. Additionally, the CPE sends and receives OAM probes to let CORD monitor the status of the network. For lab trials, a combination of two components has been used to emulate the CPE functionalities: a media converter, used to collect users’ traffic from an Ethernet CAT5/6 interface (where a traditional host, like a laptop, is connected) and send it out from its other SFP interface; a programmable SFP (plugged into the SFP port of the media converter), that a) tags the traffic with a specific VLAN id and forwards it to the Ethernet Edge switch; b) sends and receives OEM probes to let CORD monitor the network. The programmable SFP is currently configured through NETCONF, using the ONOS Flow Rule abstraction translated into NETCONF XML for the drivers, and the ONOS-based CarrierEthernet application to generate the Flow Rules based on requests.

## Installing the global node
To install the global orchestrator on a physical node, you should follow the steps described in the main [physical POD installation](https://guide.opencord.org/install_physical.html).

At a high level, bootstrap the development machine and download the code.

### Local POD configuration file
When it’s time to write your POD configuration, use the [physical-example.yml](https://github.com/opencord/cord/blob/master/podconfig/physical-example.yml) file as a template. Either modify it or make a copy of it in the same directory.
Fill in the configuration with your own head node data.

As cord_scenario, use *single*. This won’t install OpenStack and other software, which are not needed on the global node.

As cord_profile, use *ecord-global*.

### POD Build
Continue the installation as described in the guide by running the make build target:

```
make build
```

### DNS Services restart
As soon as the procedure is finished you need to restart two services on the pod.

```
sudo service nsd restart
sudo service unbound restart
```

Now your global node is ready to be connected to the local sites.

## Installing an E-CORD local site
To install the local node you should follow the steps described in the main [physical POD installation](https://guide.opencord.org/install_physical.html). Bootstrap the development machine and download the code.
When it’s time to write your pod configuration, use the [physical-example.yml](https://github.com/opencord/cord/blob/master/podconfig/physical-example.yml) file as a template. Either modify it or make a copy of it in the same directory.
Fill in the configuration with your own head node data.

As cord_scenario use *cord*.

As cord_profile use *ecord*.

## Configure the Global node
It’s essential to configure the global node properly, in order to instruct it about the existing local sites, have them connected and coordinated.
Configuring the global node consists of two parts, an XOS/Tosca configuration and an ONOS/JSON.

### Configuring XOS (Tosca Configuration)
The first part consists of instructing the XOS running on the global node about the other XOS instances, orchestrating the local sites.
 
To configure your XOS instance, do the following:
* Create your TOSCA file, using as template the file on your development/management machine, under *CORD_ROOT/orchestration/profiles/ecord/examples/vnaasglobal-service-reference.yaml* (available also online, [here](https://github.com/opencord/ecord/blob/master/examples/vnaasglobal-service-reference.yaml))
* Save it on the global node.
* SSH into your global node
* On the global node, run the following command

```
python /opt/cord/build/platform-install/scripts/run_tosca.py 9000 xosadmin@opencord.org YOUR_XOS_PASSWORD PATH_TO_YOUR_TOSCA FILE
```

**NOTE**: If the XOS password has been auto-generated, you can find it on the global node, in 
```
/opt/credentials/xosadmin@opencord.org
```

### Configuring ONOS (JSON configuration)
To configure ONOS on the global node:
* SSH into the global node
* Login to ONOS_CORD: *ssh -p 8102 onos@onos-cord*
* In the ONOS CLI (*onos>*) verify that apps are loaded by executing: *apps -a -s*
    The following applications should be enabled:
    ```
    org.onosproject.drivers
    org.opencord.ce-api
    org.opencord.ce.global
    ```
    If one or more apps mentioned above are not present in the list, they can be activated with *app activate APP-NAME*
* Logout from ONOS (CTRL+D or exit)
* Anywhere, either on the global node itself, or on any machine able to reach the global node, write your ONOS configuration file. The following is an example configuration for a global node that communicates with two local sites, with domain names site1 and site2.
    
    ```
    {
      "apps" : {
        "org.opencord.ce.global.vprovider" : {
          "xos" : {
            "username" : "xosadmin@opencord.org",
            "password" : "YOUR_XOS_PASSWORD (see note below)",
            "address" : "YOUR_GLOBAL_NODE_IP",
            "resource" : "/xosapi/v1/vnaas/usernetworkinterfaces/"
          }
        },
        "org.opencord.ce.global.channel.http" : {
          "endPoints" : {
            "port" : "8182",
            "topics" : [
              "ecord-domains-topic-one"
            ],
            "domains" :
            [
              {
                "domainId" : "YOUR-SITE1-EXTERNAL-IP-fabric-onos",
                "publicIp" : "YOUR-SITE1-EXTERNAL-IP",
                "port" : "8181",
                "username" : "onos",
                "password" : "rocks",
                "topic" : "ecord-domains-topic-one"
              },
              {
                "domainId" : "YOUR-SITE1-EXTERNAL-IP-cord-onos",
                "publicIp" : "YOUR-SITE1-EXTERNAL-IP",
                "port" : "8182",
                "username" : "onos",
                "password" : "rocks",
                "topic" : "ecord-domains-topic-one"
              },
              {
                "domainId" : "YOUR-SITE2-EXTERNAL-IP-fabric-onos",
                "publicIp" : "YOUR-SITE2-EXTERNAL-IP",
                "port" : "8181",
                "username" : "onos",
                "password" : "rocks",
                "topic" : "ecord-domains-topic-one"
              },
              {
                "domainId" : "YOUR-SITE2-EXTERNAL-IP-cord-onos",
                "publicIp" : "YOUR-SITE2-EXTERNAL-IP",
                "port" : "8182",
                "username" : "onos",
                "password" : "rocks",
                "topic" : "ecord-domains-topic-one"
              }
            ]
          }
        }
      }
    }
    ```
    
    **NOTE** Under the key “topics” you can specify as many topics (string of your choice) as you prefer for sake of load balancing the communication with the underlying (local-site) controllers among the instance of the global ONOS cluster. For each domain you also have to specify one of these topics.
    
    **NOTE**: If the XOS password has been auto-generated, you can find it on the global node, in /opt/credentials/xosadmin@opencord.org

* Use CURL to push your file:
    ```
    curl -X POST -H "content-type:application/json"  http://YOUR-GLOBAL-NODE-IP:8182/onos/v1/network/configuration -d @YOUR-JSON-FILE.json --user onos:rocks
    ```

## Configure the local sites
Local sites configuration consists of two parts:
* ONOS_Fabric configuration
* The ONOS_CORD configuration
 
The local site configurations explained below need to happen on the head nodes of all local sites.

### ONOS_fabric configuration
ONOS_Fabric manages the underlay network (the connectivity between the fabric switches).

To configure ONOS_Fabric do the following:
* SSH on the local site head node
* Log into ONOS_Fabric: *ssh -p 8101 onos@onos-fabric*
* In the ONOS CLI (*onos>*) verify that apps are loaded: *apps -a -s*
    The list of applications enabled should include:
    ```
    org.onosproject.segmentrouting
    org.opencord.ce.api
    org.opencord.ce.local.bigswitch
    org.opencord.ce.local.channel.http
    org.opencord.ce.local.fabric
    ```
    If one or more apps mentioned above are not present in the list, they can be activated with *app activate APP-NAME*
    * Check that the site domain id is correctly set in ONOS, running
    ```
    cfg get org.opencord.ce.local.bigswitch.BigSwitchManager
    ```
    The value should be *YOUR-HEAD-NODE-IP-fabric-onos*
* Check that the fabric switch is connected to onos typing devices. If no devices are there, make sure your fabric switch is connected to ONOS and go through the [Fabric configuration guide](https://guide.opencord.org/appendix_basic_config.html#connect-the-fabric-switches-to-onos).
* Logout from ONOS (CTRL+D or exit)
* Anywhere, either on the head node itself, or on any machine able to reach the head node, write your ONOS configuration file.
    Following, is an example configuration for a local site.
    
    ```
    {
      "apps" : {
        "org.opencord.ce.local.fabric" : {
          "segmentrouting_ctl": {
            "publicIp": "YOUR-HEAD-NODE-IP",
            "port": "8181",
            "username": "onos",
            "password": "rocks",
            "deviceId": "of:YOUR-FABRIC-SW-DPID"
          }
        },
        "org.opencord.ce.local.bigswitch" : {
          "mefPorts" :
          [
            {
              "mefPortType" : "INNI",
              "connectPoint" : "of:YOUR-FABRIC-SW-DPID/PORT-ON-FABRIC-CONNECTING-TO-EE",
              "interlinkId" : "EE-1-to-fabric"
            },
            {
              "mefPortType" : "ENNI",
              "connectPoint" : "of:YOUR-FABRIC-SW-TO-UPSTREAM-DPID/PORT (duplicate as many time as needed)",
              "interlinkId" : "fabric-1-to-fabric-2"
            }
          ]
        },
        "org.opencord.ce.local.channel.http" : {
          "global" : {
            "publicIp" : "YOUR-GLOBAL-NODE-IP",
            "port" : "8182",
            "username" : "onos",
            "password" : "rocks",
            "topic" : "ecord-domains-topic-one"
          }
        }
      }
    }
    ```

* Use CURL to push your file
    ```
    curl -X POST -H "content-type:application/json"  http://YOUR-LOCAL-HEAD-NODE-IP:8181/onos/v1/network/configuration -d @YOUR-JSON-FILE.json --user onos:rocks
    ```

#### The MEF ports
Under the key “mefPorts” there is the list of physical ports that have to be exposed to the global node. These ports represent MEF ports and can belong to different physical devices, but they will be part of a single abstract “bigswitch” in the topology of the global ONOS (see [E-CORD topology abstraction](https://guide.opencord.org/profiles/ecord/overview.html#e-cord-topology-abstraction)). These ports represent also the boundary between physical topologies controlled by different ONOS controllers.

In the Json above:
* *of:YOUR-FABRIC-SW-DPID/PORT-ON-FABRIC-CONNECTING-TO-EE* is the DPID and the port of the fabric device where the ethernet edge is connected to. 
* *of:YOUR-FABRIC-SW-TO-UPSTREAM-DPID/PORT* is the DPID and the port of the fabric device where the transport network is connected to (in the example fabric switches are connected together). 

This is hinted through the interlinkId.

#### The topic attribute
The *topic* attribute under the *org.opencord.ce.local.channel.http* is a string of your choice. It is used to run an election in an ONOS cluster to choose the ONOS instance that interacts with the global node.

### ONOS_CORD configuration
ONOS_CORD manages the overlay network. It controls both the users' CPEs and the Ethernet Edge devices.
The programmable microsemi SFP -part of your emulated CPE- is configured and managed through Netconf (more information about NETCONF and ONOS can be found [here](https://wiki.onosproject.org/display/ONOS/NETCONF)).  
The Ethernet Edge device is managed through OpenFlow.  
Both the devices, at the end of the configuration procedure should show up in ONOS_CORD, which can be confirmed by typing *devices* in the ONOS CLI (*onos>*).

To configure ONOS_CORD do the following:
* SSH into the head node
* Login to onos-cord: *ssh -p 8102 onos@onos-cord*
* At the ONOS CLI verify that apps are loaded: *apps -a -s*
    The following applications should be enabled:
    ```
    org.onosproject.drivers.microsemi
    org.onosproject.cfm
    org.opencord.ce.api
    org.opencord.ce.local.bigswitch
    org.opencord.ce.local.channel.http
    org.opencord.ce.local.vee
    ```
    If one or more apps mentioned above are not present in the list, they can be activated with app activate APP-NAME
* Create a new JSON file and write your configuration. As a template, you can use the JSON below:
    ```
    {
     "devices": {
      "netconf:YOUR-CPE-IP:830": {
       "netconf": {
         "username": "admin",
         "password": "admin",
         "ip": "YOUR-CPE-IP",
         "port": "830"
       },
       "basic": {
        "driver": "microsemi-netconf",
        "type": "SWITCH",
        "manufacturer": "Microsemi",
        "hwVersion": "EA1000"
       }
      }
     },
     "links": {
      "netconf:YOUR-CPE-IP:830/0-of:WHERE-YOUR-CPE-IS-CONNECTED (EE DPID/PORT)" : {
       "basic" : {
        "type" : "DIRECT"
       }
      },
      "of:WHERE-YOUR-CPE-IS-CONNECTED (EE DPID/PORT)-netconf:YOUR-CPE-IP:830/0" : {
       "basic" : {
        "type" : "DIRECT"
       }
      }
     },
     "apps" : {
      "org.opencord.ce.local.bigswitch" : {
       "mefPorts" :
        [
         {
          "mefPortType" : "UNI",
          "connectPoint" : "netconf:YOUR-CPE-IP:830/0"
         },
         {
          "mefPortType" : "INNI",
          "connectPoint" : "of:DPID-AND-PORT-OF-EE-CONNECTING-TO-FABRIC",
          "interlinkId" : "EE-2-fabric"
         }
        ]
      },
      "org.opencord.ce.local.channel.http" : {
       "global" : {
        "publicIp" : "YOUR-GLOBAL-NODE-IP",
        "port" : "8182",
        "username" : "onos",
        "password" : "rocks",
        "topic" : "ecord-domains-topic-one"
       }
      }
     }
    }
    ```
* Load the following the Json file just created using curl:
    ```
    curl -X POST -H "content-type:application/json" http://YOUR-LOCAL-SITE-HEAD-IP:8182/onos/v1/network/configuration -d @YOUR-JSON-FILE.json --user onos:rocks
    ```

**Warning** The Json above tries to congiure devices and links at the same time. It may happen that ONOS denies your request of link creation, since it does not find devices present (because their creation is still in progress). If this happens, just wait few seconds and try to push again the same configuration, using the *curl* command above.

## Configure the Ethernet Edge device (Centec v350)
The steps below assume that
* The Centec device to an A(Access)-leaf fabric switch
* ONOS_CORD is running

Follow the steps below to assign an IP address to the Ethernet Edge device and connect it to ONOS_CORD.

### Set a management IP address on the switch OOB interface
The switch management interface should be set with a static IP address (DHCP not supported yet), in the same subnet of the POD internal/management network (by default 10.6.0.0/24).

**NOTE**: Please, use high values for the IP last octet, since lower values are usually allocated first by the MAAS DHCP server running on the head node.

To configure the static IP address, do the following:
* Log into the CLI of the Centec switch (through SSH or console cable)
* *configure terminal*
* *management ip address YOUR_MGMT_ADDRESS netmask YOUR_NETMASK*
* *end*
* *show management ip address*

### Set ONOS-CORD as the Openflow controller
To set ONOS-CORD as the default switch OpenFlow controller and verify the configuration:
* Log into the CLI of the Centec switch (through SSH or console cable)
* *configure terminal*
* *openflow set controller tcp YOUR-LOCAL-SITE-HEAD-IP 6654*
* *end*
* *show openflow controller status*

# Done!
After the global node and the local sites are properly configured, the global node should maintain an abstract view of the topology and you should see UNIs distributed on the map of the XoS GUI. You can start requests to setup Ethernet Virtual Circuit (EVC) from XoS.
