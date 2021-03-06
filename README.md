# (Experimental) containers for SageMath & friends

This repository contains a collection of dockerfiles and supporting
files for building various containers for SageMath and its components
(GAP, Singular, PARI/GP, ...).

The containers are available on [dockerhub](https://hub.docker.com/u/sagemath/).
At this stage, they are still experimental; *use with caution*.
The intention is to superseed the other existing SageMath containers.

## [sagemath/sagemath](sagemath/Dockerfile) (roughly 7GB)

This container contains a basic installation of the latest version of
SageMath, built from sources on the latest Ubuntu. Commands are run as
the Sage user. The SageMath distribution includes several programs
like GAP, Singular, PARI/GP, R, ... which are available in the path.

### Installation

    docker pull sagemath/sagemath

or simply continue to the next step.

### Running Sage & co with a text interface

To run Sage:

    docker run -it sagemath/sagemath

Other software included in this image can be run similarly:

    docker run -it sagemath/sagemath gap

    docker run -it sagemath/sagemath gp         # PARI/GP

    docker run -it sagemath/sagemath maxima

    docker run -it sagemath/sagemath R

    docker run -it sagemath/sagemath singular

### Running the notebook interfaces

To run the Jupyter Notebook interface (for Sage, ...):

    docker run -p 8888:8888 sagemath/sagemath-jupyter

Alternatively, to run the legacy Sage notebook server:

    docker run -p 8080:8080 sagemath/sagemath sage -notebook

You can then connect your web browser to the printed out typically, namely http://localhost:8888 for the Jupyter notebook and http://localhost:8080 for the legacy notebook. For the legacy notebook the webbrowser will ask for a login and password which are respectively `admin` and `sage`.

**Note** Running the sagemath-jupyter container is equivalent to running the `sagemath/sagemath` base docker container with the following command:

    docker run -p 127.0.0.1:8888:8888 sagemath/sagemath sage -notebook=jupyter --no-browser --ip='*' --port=8888

The `--ip` option is required by the Jupyter notebook to allow connections to
the notebook through the Docker network.

**Note for Windows:** An additional step is required on Windows in order to
connect to the notebook via localhost.  This is because Docker on Windows
involves two layers of virtualization: It runs a small Linux VM in VirtualBox
in order to run the main Docker engine, which does not run natively on Windows.
Then Docker containers are run within that VM.

Because of this, even when using port forwarding as above, this only forwards
ports on the Linux VM "host", and not on your physical host OS.  Therefore
connecting via `localhost`/`127.0.0.1` does not immediately work.  In order for
this to work we also need to set up port forwarding from your physical host OS
to the Linux VM.  To do this open a command prompt and run:

    "C:\Program Files\Oracle\VirtualBox\VBoxManage" modifyvm "default" --natfp1 "sagemath-jupyter,tcp,,8888,,8888"
    
Or if the VM is already running you can modify the running VM too.  To just replace `modifyvm "default" --natfp1`
with `controlvm "default" natfp1` (yes, the command line interfaces are slightly inconsistent).

This change should be persistent and need not be repeated unless the Docker VM
is uninstalled and reinstalled (or possibly if it is upgraded--TBD).

### Rebuilding the container

Prequisites: network access to download Sage (http/https)

    docker build --tag="sagemath/sagemath" sagemath

## [sagemath/sagemath-develop](sagemath-develop/Dockerfile)

This container is similar to the previous one, except that SageMath is
built from the latest unstable release version of Sage, retrieved by
cloning the develop branch from github.

TODO: include git-trac

To download and start it:

    docker run -it sagemath/sagemath-develop

### Rebuilding the container

    docker build --tag="sagemath/sagemath-develop" sagemath-develop

## [sagemath/sagemath-jupyter](sagemath-jupyter/Dockerfile)

TODO: fix the duplicated documentation for running the Jupyter notebook

If you want to have a container already set up for the Jupyter enviroment,
you can use sagemath/sagemath-jupyter. It is based on sagemath/sagemath.

    docker run -p 8888:8888 sagemath/sagemath-jupyter

makes the Jupyter notebook accessible via `localhost:8888`, while

    docker run sagemath/sagemath-jupyter

makes it accessible under the container's ip address on port `8888`. You can
see the ip address of the container using `docker inspect`. This is useful if
you want to have more than one notebook server running.  Typically this will
be something like:

    172.17.0.1

where the fourth field may be incremented depending on the number of running
containers on the host.

### Rebuilding the container

    docker build --tag="sagemath/sagemath-jupyter" sagemath-jupyter

## [sagemath/sagemath-patchbot](sagemath-patchbot/Dockerfile)

This container, built on top of sagemath-develop, is meant to run
instances of the [Sage patchbot](http://patchbot.sagemath.org/)
running securely in a sandbox, to ensure that the untrusted code it
fetches and run cannot harm the host machine.

### Starting the patchbot:

    docker run -t --name="patchbot" -d sagemath/sagemath-patchbot
    pid=$(docker inspect -f '{{.State.Pid}}' patchbot )
    ip=$(docker inspect -f '{{.NetworkSettings.IPAddress}}' patchbot )
    trac_ip=$(getent hosts trac.sagemath.org | awk '{ print $1 }')
    patchbot_ip=$(getent hosts patchbot.sagemath.org | awk '{ print $1 }')
    nsenter -t $pid -n iptables -A FORWARD -s ${ip} -d ${trac_ip} -j ACCEPT
    nsenter -t $pid -n iptables -A FORWARD -s ${ip} -d ${patchbot_ip} -j ACCEPT
    nsenter -t $pid -n iptables -A FORWARD -s ${ip} -j REJECT --reject-with icmp-host-prohibited

### Rebuilding the container:

    docker build --tag="sagemath/sagemath-patchbot" sagemath-patchbot

## sagemath/sagemath-fat (in the plan)

sagemath/sagemath with latex, the commonly used (GAP) packages, etc.

## sagemath/sagemath-fat-jupyter (in the plan)

Same as sagemath-jupyter, but based on sagemath-fat
