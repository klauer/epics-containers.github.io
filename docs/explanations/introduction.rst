Essential Concepts
==================

Overview
--------

.. include:: ../overview.rst

See below for more detail on each of these.

Concepts
--------

Images and Containers
~~~~~~~~~~~~~~~~~~~~~
Containers provide the means to package up IOC software and execute it
in a lightweight virtual environment. This includes saving the packages
into public or private image registries such as DockerHub or Google Cloud
Registry.

The Open Container Initiative (OCI) defines the set of container services
and their APIs such that container images can be interchanged between
different frameworks.

Thus, in this project we develop, build and test our container images
using docker or podman but the images can be run under Kubernetes' own
container runtime.

This article does a good job of explaining the relationship between docker /
containers and Kubernetes https://semaphoreci.com/blog/kubernetes-vs-docker

An important outcome of using containers is that you can alter the
environment inside the container to suit the IOC code, instead of altering the
code to suit your infrastructure. At DLS, this means that we are able to use
vanilla EPICS base and support modules. We no longer require our own
forks of these repositories.

Generic IOCs and instances
""""""""""""""""""""""""""

An important principal of the approach presented here is that an IOC container
image represents a 'generic' IOC. The generic IOC image is used for all
IOC instances that connect to a given class of device.

An IOC instance runs in a container that bases its
filesystem on a generic IOC image.
In addition the instance has configuration mapped into the
container that will bootstrap the unique properties of that instance.
In most cases the configuration need only be a single IOC boot script.

This approach reduces the number of images required and saves disk. It also
makes for simple configuration management.

Throughout this documentation we will use the terms Generic IOC and
IOC Instance. The word IOC without this context is ambiguous.


Kubernetes
~~~~~~~~~~
https://kubernetes.io/

Kubernetes easily and efficiently manages containers across clusters of hosts.​

It builds upon 15 years of experience of running production workloads at
Google, combined with best-of-breed ideas and practices from the community​,
since it was open-sourced in 2014.

Today it is by far the dominant orchestration technology for containers.

In this project we use Kubernetes to provide a standard way of implementing
these features:

- Auto start IOCs when servers come up​
- Manually Start and Stop IOCs​
- Monitor IOC status and versions​
- Deploy versioned IOCs to the beamline​
- Rollback to a previous IOC version​
- Allocate the server which runs an IOC​
- View the current log ​
- View historical logs (via graylog)​
- Connect to an IOC and interact with its shell
- debug an ioc by starting a bash shell inside it's container
- etc.


Helm
~~~~
https://helm.sh/

Helm is the most popular package manager for Kubernetes applications​.

The packages are called Helm Charts​. They contain templated YAML files to
define a set of resources to apply to a Kubernetes cluster​.

Helm has functions to deploy Charts to a cluster and manage multiple versions
of the chart within the cluster​.

It also supports registries for storing version history of charts,
much like docker.

In this project we use Helm Charts to define and deploy IOC instances. Each
beamline has its own Helm Repository which stores current and historical
version of its IOC instances.

Repositories
~~~~~~~~~~~~

All of the assets required to manage a
set of IOCs for a beamline are held in repositories.

Thus all version control is done
via these repositories and no special locations in
a shared filesystem are required
(The legacy approach at DLS relied heavily on
know locations in a shared filesystem).

In the epics-containers examples all repositories are held in the same
github organization. This is nicely contained and means that only one set
of credentials is required to access all the resources.

There are many alternative services for storing these repositories, both
in the cloud and on premises. Below we list the choices we have tested
during the POC.

The 3 classes of repository are as follows:

:Source Repository:
  - Holds the source code but also provides the
    Continuous Integration actions for testing, building and publishing to
    the image / helm repositories. These have been tested:

    - github
    - gitlab (on prem)

:An image repository:
  - Holds the generic IOC container images and their
    dependencies. The following have been tested:

    - github packages
    - dockerhub
    - Google Cloud Container Registry

:A helm chart repository:
  - This is where the definitions of IOC instances
    are stored. They are in the form of a helm chart which describes to
    Kubernetes the resources needed to spin up the IOC.
    These have been tested:

    - github packages
    - Google Cloud Artifact Registry

Continuous Integration
~~~~~~~~~~~~~~~~~~~~~~

Our examples all use continuous integration to get from pushed source
to the published images / helm charts.

This allows us to maintain a clean code base that is continually tested for
integrity and also to maintain a direct relationship between source code tags
and the tags of their built resources.

There are these types of CI:

:Generic IOC source:
    - builds a generic IOC container image
    - runs some tests against that image
    - publishes the image to github packages (only if the commit is tagged)

:beamline definition source:
    - builds a helm chart from each ioc definition
      (TODO ibek will do this - at present the charts are hand coded)
    - TODO: IOCs which are unchanged should not be published again
    - publishes the charts to github packages (only if the commit is tagged)

:helm library source:
    - builds the helm chart
    - publishes it to github packages (only if the commit is tagged)

:documentation source:
    - builds the sphinx docs
    - publishes it to github.io pages

Scope
-----
This project initially targets the low hanging fruit of x86_64 Linux Soft
IOCs only.

Other linux architectures could be added to the Kubernetes cluster.

In future it is entirely possible that Windows IOCs could also be supported
since Kubernetes supports mixed OS clusters. This would be needed
because windows containers require a windows host.

Hard IOCs will not be supported in their current form. Perhaps there is a
future possibility of turning hard IOCs remote devices.

Note that OPI is also out of scope for this initial phase. See
`no_opi`

Additional Tools
----------------

k8s-epics-utils
~~~~~~~~~~~~~~~
The project k9s-epics utils contains a script to add simple command
line functions for deploying and monitoring IOCs.

See `CLI` for details.

It also provides a Dockerfile for building a personal developer image
allowing a developer to work on support modules or IOCs anywhere.

TODO provide documentation for the developer image.


Ibek
~~~~
IOC Builder for EPICS and Kubernetes provides a way to generate an IOC
helm chart from a YAML description of the IOC.

TODO: this is in early development and not yet included in the epics-containers
Organization.