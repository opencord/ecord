# E-CORD Developer Guid

As for other CORD profiles, E-CORD can also be deployed in environments other than physical PODs. This creates a more convenient environment for developers, using less resources and providing a faster development life-cycle.

Two environments are available, depending on your needs:
* Local Developer Machine: a development environment running directly on your laptop
* CORD-in-a-Box

## Mock/local Machine Development Environment

To understand what a local development environment is, what it can help you with, and how to build it, look at [this page](https://guide.opencord.org/xos/dev/workflow_mock_single.html).

When it’s time to specify the PODCONFIG file, use
* *ecord-mock.yml* for local sites, instead of the default value (rcord-mock.yml)
* *ecord-global-single.yml* to mock global nodes, instead of the default value (rcord-mock.yml)

## CORD-in-a-Box (CiaB) development

To understand what CiaB is and what it can help you with, look [here](https://guide.opencord.org/xos/dev/workflow_pod.html).

To build E-CORD CiaB, follow the steps [here](https://guide.opencord.org/install_virtual.html).

When it’s time to specify the PODCONFIG file, use
* *ecord-virtual.yml* for local sites, instead of the default value (rcord-virtual.yml)
* *ecord-global-single.yml* to deploy global nodes, instead of the default value (rcord-virtual.yml)

**NOTE** When inspecting CiaB you will also need, on the global node to use 

```
export VAGRANT_CWD=~/cord/build/scenarios/single
```

instead of 

```
export VAGRANT_CWD=~/cord/build/scenarios/cord
```

More detailed instructions on how to develop and deploy using CiaB can be found in the troubleshooting [guide](https://guide.opencord.org/troubleshooting.html).

## Run an E-CORD test subscriber
When using local PODs, you can emulate E-CORD test subscribers, doing the following.

* ssh into the the head node
* ```cd /opt/cord/build/platform-install```
* ```ansible-playbook -i inventory/head-localhost --extra-vars "@/opt/cord_profile/genconfig/config.yml" ecord-test-subscriber-playbook.yml```