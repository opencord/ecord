# E-CORD Developer Guide

This page describes general guidelines for developers who want to download and
work on the E-CORD source code, or need to mock special development
environments.

## Download the E-CORD source code

E-CORD is part of the default CORD code base. To know how you can download the
CORD source code, see [Building and Installing CORD](/install.md).  Each E-CORD
service lives in a specific repository. A list of E-CORD services and links to
their repositories is available in the [main page of this
guide](/profiles/ecord/).

> NOTE: The E-CORD source code is available from the 4.1 release (branch) of
> CORD.

## Developer Environments

As for other CORD profiles, E-CORD can also be deployed in environments other
than physical PODs. This creates a more convenient environment for developers,
using less resources and providing a faster development life-cycle.

Two environments are available, depending on your needs:

* Local Developer Machine: a development environment running directly on your laptop
* CORD-in-a-Box

### Mock/local Machine Development Environment

To understand what a local development environment is, what it can help you
with, and how to build it, look at [this
page](/xos/dev/workflow_mock_single.md).

When it’s time to specify the PODCONFIG file, use

* *ecord-mock.yml* for local sites, instead of the default value (rcord-mock.yml)
* *ecord-global-single.yml* to mock global nodes, instead of the default value (rcord-mock.yml)

### CORD-in-a-Box (CiaB) development

To understand what CiaB is and what it can help you with, look
[here](/xos/dev/workflow_pod.md).

To build E-CORD CiaB, follow the steps [here](/install_virtual.md).

When it’s time to specify the PODCONFIG file, use

* *ecord-virtual.yml* for local sites, instead of the default value
  (rcord-virtual.yml)
* *ecord-global-single.yml* to deploy global nodes, instead of the default
  value (rcord-virtual.yml)

> NOTE: When inspecting CiaB you will also need, on the global node to use.

```shell
export VAGRANT_CWD=~/cord/build/scenarios/single
```

instead of

```shell
export VAGRANT_CWD=~/cord/build/scenarios/cord
```

More detailed instructions on how to develop and deploy using CiaB can be found
in the [troubleshooting guide](/troubleshooting.md).

## Run an E-CORD test subscriber

When using local PODs, you can emulate E-CORD test subscribers, doing the
following.

1. ssh into the the head node

2. `cd /opt/cord/build/platform-install`

3. Deploy a test subscriber:

    ```
    ansible-playbook -i inventory/head-localhost --extra-vars "@/opt/cord_profile/genconfig/config.yml" ecord-test-subscriber-playbook.yml
    ```

