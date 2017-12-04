# Configuring CFM
This guide describes how to configure CFM monitoring for an E-Line generated in a local E-CORD POD. This monitoring will then be viewable in the “Delay and Jitter” link in the CORD UI. It assumes that your POD already has a global E-CORD node and at least 2 local E-CORD instances, with each local E-CORD having a CFM capable device (e.g. Microsemi EA1000) connected.It is assumed that these devices are already detected and connected in ONOS as per [the E-CORD installation guide](https://guide.opencord.org/profiles/ecord/installation_guide.html]).

The guide acceses an ONOS_CORD instance at `onos-cord`, which has the ssh port 8102 and the web port 8182. Change the IP adddresses to your own, and if these ports have been forwarded for some reason on your setup, make sure to accommodate as necessary.

This guide describes a minimal CFM setup, with a single MD (maintenance domain) and MA (maintenance association), two MEPs (measurement endpoints), and one DM (delay measurement).

You can also find a [Postman](https://www.getpostman.com/) JSON collection that contains the JSON necessary to setup CFM under `cord/orchestration/profiles/ecord/examples/`.

Detailed information on the CFM application itself can be found on the [ONOS Wiki](https://wiki.onosproject.org/display/ONOS/Layer+2+Monitoring+with+CFM+and+Services+OAM).

## Creating an MD
Post the following JSON file to `onos-cord:8182/onos/cfm/md/` on your local instances. Make sure you have a header with basic authorization set to username “onos” and password “rocks”, as this will be required for this and all future JSON requests in this guide.

NOTE: MDs and MAs are stored in a distributed store within the same cluster. In our testing setup, ONOS_CORD and ONOS_Fabric exist on two separate clusters. If you see the MD on the ONOS_Fabric (i.e. the MD shows up after running cfm-md-list) after instantiating it on the first local node, do not push the JSON on the second node. Otherwise, feel free to push on the second node.

If you are creating more than one MD, they must have different `mdNumericId`s. Be aware that devices will only be able to take delay measurements from others on the same MD.

```json
{"md": {
    "mdName": "onf",
    "mdNameType": "CHARACTERSTRING",
    "mdLevel": "LEVEL3",
    "mdNumericId": 1
   }
}
```

## Creating an MA
Created MAs should be a part of a pre-defined MD, and are associated with a particular VLAN ID, defined in the `vid-list`. We defined an MD called `onf`, so we post the following JSON to `onos-cord:8182/onos/cfm/md/onf/`.

NOTE: MAs are also stored in a distributed store. See the previous note in “Setting up MDs” for possible consequences.

If you are creating more than one MA, they must have different `maNumericId`s and `vid`s, while `component-id`s may be reused. The `rmep-list` defines the list of MEP IDs that can be associated with this vlan. We only have two MEPs here since we are monitoring an Eline. All MEPs an MA might use must be defined in the `rmep-list`, as changes made to the `rmep-list` after the MA has already been pushed to a device will not be reflected in that device's copy of the MA.

```json
{
  "ma": {
    "maName": "ma-vlan-1",
    "maNameType": "CHARACTERSTRING",
    "maNumericId": 1,
    "ccm-interval": "INTERVAL_1S",
    "component-list": [
      { "component": {
        "component-id":"1",
        "tag-type": "VLAN_STAG",
        "vid-list": [
          {"vid":1}
        ]
        }
      }
    ],
    "rmep-list": [
      { "rmep":10 },
      { "rmep":20 }
    ]
  }
}
```

## Creating a MEP
A measurement endpoint (MEP) is associated with a NETCONF device, and one must be created for each device. A MEP can only be created for the device connected to the current local node, and must be associated with an MA. The MEP ID must already exist on that MA’s `rmep-list` and must not already be in use by another device.

With our MD set to onf, and MA set to ma-vlan-1, we post the following JSON to `onos-cord:8182/onos/cfm/md/onf/ma/ma-vlan-1/mep`

```json
{
  "mep": {
    "mepId": 10,
    "deviceId": "netconf:10.6.0.160:830",
    "port": 0,
    "direction": "DOWN_MEP",
    "mdName": "onf",
    "maName": "ma-vlan-1",
    "primary-vid": 1,
    "administrative-state": true,
    "ccm-ltm-priority": 4,
    "cci-enabled" :true
  }
}
```

Since we have 2 Microsemi devices, we will also need to post MEP creation JSON to our second local node, making sure to change the `mepId` and `deviceId`. In our example, we defined two MEP IDs, 10 and 20, to be used with our first and second local nodes. Both MEPs must be set up correctly in order to create a delay measurement (DM).

## Creating a DM
Check the `remote_mep_state` of your created meps to ensure that they have a value of `RMEP_OK` before creating delay measurements. You can do so by performing a GET; in our case, we would perform a GET on `onos-cord:8182/onos/cfm/md/onf/ma/ma-vlan-1/mep/10` on our first local node, and on `onos-cord:8182/onos/cfm/md/onf/ma/ma-vlan-1/mep/20` on our second local node.

DMs are associated with a single MEP, and a maximum of 2 can exist on a single MEP.

IMPORTANT: The `measurementsEnabled` in the sample JSON below are required for the CFM stat UI to function properly.

To create a DM on our first local instance, the following JSON is submitted by POST to `onos-cord:8182/onos/cfm/md/onf/ma/ma-vlan-1/mep/10`. Notice that the remoteMepId is set to 20 because we are trying to get the delay measurement to 20 from 10.

```json
{
  "dm": {
        "remoteMepId":20,
        "dmCfgType": "DMDMM",
        "version": "Y17312008",
        "priority": "PRIO1",
        "messagePeriodMs": 100,
        "startTime": {
          "immediate":true
        },
        "stopTime": {
          "none":true
        },
        "frameSize": 1500,
        "measurementIntervalMins": 1,
        "measurementsEnabled": [
            "INTER_FRAME_DELAY_VARIATION_TWO_WAY_AVERAGE",
            "INTER_FRAME_DELAY_VARIATION_TWO_WAY_MAX",
            "INTER_FRAME_DELAY_VARIATION_TWO_WAY_MIN",
            "INTER_FRAME_DELAY_VARIATION_TWO_WAY_BINS",
            "FRAME_DELAY_TWO_WAY_AVERAGE",
            "FRAME_DELAY_TWO_WAY_MAX",
            "FRAME_DELAY_TWO_WAY_MIN",
            "FRAME_DELAY_TWO_WAY_BINS"
        ]
  }
}
```

We don’t need to create a DM on the second local instance for this DM to work. Make sure that you aren’t setting the `remoteMepId` equal to the mepId you are trying to create the DM on.

## Viewing Delay and Jitter

At this point, we are done configuring CFM and can now view delay and jitter statistics in the CORD UI under “Delay and Jitter”. Login into the CORD UI at `onos-cord/xos` with the username `xosadmin@opencord.org`, and the password generated in the local node under `/opt/credentials/xosadmin@opencord.org`. The "Delay and Jitter" link will be on the left navigation panel once you have successfully logged in.
