# Containers for SageMath & friends

This repository contains a collection of dockerfiles and supporting
files for building various SageMath containers.

Those containers are in the build; we will upload them soon on
Dockerhub.  Note however that they are experimental; use with caution!

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

They can be run similarly:

    docker run -it sagemath/sagemath gap

    docker run -it sagemath/sagemath gp         # PARI/GP

    docker run -it sagemath/sagemath maxima

    docker run -it sagemath/sagemath R

    docker run -it sagemath/sagemath singular

### Running the Notebook interfaces

To start the Jupyter notebook server you can start the container with the following command:

    docker run -p 127.0.0.1:8888:8888 sagemath/sagemath sage -notebook=jupyter --no-browser

You can then connect your web browser to the printed out address
(typically http://localhost:8888).

Similarly, to run the legacy Sage notebook server:

    docker run --net="host" sagemath/sagemath sage -notebook

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

If you want to have a container already set up for the Jupyter enviroment,
you can use sagemath/sagemath-jupyter. It is based on sagemath/sagemath.

    docker run -p 127.0.0.1:8888:8888 sagemath/sagemath-jupyter

makes the Jupyter notebook accessible via `localhost:8888`, while

    docker run sagemath/sagemath-jupyter

makes it accessible under the containers ip address on port `8888`. You can
see the ip address of the container using `docker inspect`. This is useful
if you want to have more than one notebook server running.

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
