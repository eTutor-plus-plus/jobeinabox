# JobeInABox

The [Moodle CodeRunner question type plugin](https://moodle.org/plugins/qtype_coderunner) requires a [Jobe server](https://github.com/trampgeek) on which to run student-submitted jobs. [JobeInABox](https://hub.docker.com/r/trampgeek/jobeinabox/) is a Docker image that provides a basic Jobe server that runs all the standard languages. For full information on Jobe servers, see the 
[full Jobe documentation](https://github.com/trampgeek/jobe)

*NOTE: for security and performance reasons it is strongly recommended to run Jobe on a 
dedicated standalone server, even when running it in a container.*

## Building and running your own image locally (strongly recommended)

There are several ways to build and run a JobeInABox container, for example:

* [Podman](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)
* [Docker](https://docs.docker.com/)

For production use you should build your own image using the local timezone. In this example we use Docker as follows:

Pull [this repo from Github](https://github.com/eTutor-plus-plus/jobeinabox), cd into the jobeinabox directory and type a command
of the form

    sudo docker build . -t my/jobeinabox --build-arg TZ="Europe/Amsterdam"

You can then run your newly-built image with the command

    sudo docker run -d -p 4000:80 --name jobe my/jobeinabox

This will give you a jobe server running on port 4000, which can then be
tested locally and used by Moodle as explained in the section "Using jobeinabox" below.

## Setting the number of jobe users

By default, Jobe will run up to 16 jobs simultaneously. To change the number, exec a shell in the container with a command of the form

    docker exec -it jobe bash

 and then:

    apt update; apt install nano
    nano  /var/www/html/jobe/application/config/config.php

Find the line

    $config['jobe_max_users'] = 8;

and change the value from 8 to the number of cores on your machine, or to a higher
value if you expect to run mainly I/O-bound jobs.

Then re-install Jobe (within the container) with the commands:

    cd /var/www/html/jobe
    ./install --purge

## Checking performance

You can check the performance of the container with the command

    docker exec -it jobe /var/www/html/jobe/testsubmit.py --perf


### Warnings:

1.  The image is over 1 GB, so may take a long time to start the first
    time, depending on your download bandwidth.

## Using jobeinabox

Having started a jobeinabox container by either of the above methods, you
can check it's running OK by browsing to

     http://[host_running_docker]:4000/jobe/index.php/restapi/languages

and you should get a JSON-encoded list of the supported languages, namely

    [["c","7.3.0"],["cpp","7.3.0"],["java","10.0.2"],["nodejs","8.10.0"],["octave","4.2.2"],["pascal","3.0.4"],["php","7.2.7"],["python3","3.6.5"]]

If you wish to run the test suite within the container, use the command

    sudo docker exec -t jobe /usr/bin/python3 /var/www/html/jobe/testsubmit.py

To set your Moodle/CodeRunner plugin to use this dockerised Jobe server, set the Jobe server field in the CodeRunner admin settings (Site Administration > Plugins > Question types > CodeRunner) to

    [host_running_docker]:4000

Do not put http:// at the start.

To stop the running server, enter the command:

    sudo docker stop jobe

To remove the running server, enter the command:

    sudo docker rm jobe

To check if there is anything left, enter the command

    sudo docker ps -a

## Notes on security:

1.  Note that while the container in which this Jobe runs should be secure, the
    container's network is currently just bridged across to the host's network.
    This means that Jobe can be accessed from anywhere that can access the host
    and can access any URI that the host can access. Firewalling of the host is
    essential for production use.

1.  Rebuild the container regularly to ensures that it is running
    with the latest jobe version and security updates.


