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

### Virtual Pod (CiaB) development

To understand what a Virtual Pod is and what it can help you with, look
[here](/xos/dev/workflow_pod.md).

To build E-CORD Virtual Pod, follow the steps [here](/install_virtual.md).

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

When using local PODs, you can emulate E-CORD test subscribers by running:

```shell
$ make ecord-test-local-subscriber

PLAY [Create E-CORD local test subscriber config and run test] ************************

TASK [Gathering Facts] ****************************************************************
ok: [head1]

TASK [test-ecord-local-subscriber : Create test-ecord-local-subsc**********************
ok: [head1]

TASK [test-ecord-local-subscriber : Read test-ecord-local-subscri**********************
ok: [head1]

TASK [test-ecord-local-subscriber : Run TOSCA to add E-CORD test-**********************
ok: [head1]

TASK [test-ecord-local-subscriber : Wait for vEG VM to come up] ***********************
FAILED - RETRYING: Wait for vEG VM to come up (10 retries left).
changed: [head1]

TASK [test-ecord-local-subscriber : Get ID of VM] *************************************
changed: [head1]

TASK [test-ecord-local-subscriber : Get Management IP of VM] **************************
changed: [head1]

PLAY RECAP ****************************************************************************
head1                      : ok=7    changed=3    unreachable=0    failed=0
```

If this completes successfully (no make error) then you've successfully created
a test subscriber vEG VM.

