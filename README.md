[![Build Status](https://travis-ci.org/elastic/beats-docker.svg?branch=master)](https://travis-ci.org/elastic/beats-docker)

## Description

This repository contains the official [Beats][beats] Docker images from
[Elastic][elastic].

These images are currently under development, and should be considered alpha-quality.

Please do not run these images in production, but feel free to experiment.

[beats]: https://www.elastic.co/products/beats
[elastic]: https://www.elastic.co/

## Prerequisites

1. [docker-engine](https://docs.docker.com/engine/installation/)
2. python3.5, python3.5-pip and python3.5 development libraries
3. virtualenv compatible with python3.5

## Quick demo
Run `make demo` to build, test and run the Beats.

Elasticsearch and Kibana containers are also provided. Once the
containers are all running, point a browser at
[http://localhost:5601](http://localhost:5601) to find Kibana, and log in
with `elastic`/`changeme` as the username and password.

## Operational notes
### All Beats
#### Configuration file
Each Beat has a YAML configuration file at
`/usr/share/[BEAT]/[BEAT].yml`. A simple default file is provided, but
you will probably want to override it by bind-mounting your own
configuration like this:

``` bash
docker run -v metricbeat.yml:/usr/share/metricbeat/metricbeat.yml docker.elastic.co/beats/metricbeat:5.3.1
```

Alternatively, you could extend the image like this:

``` dockerfile
FROM docker.elastic.co/beats/metricbeat:5.2.1
COPY metricbeat.yml /usr/share/metricbeat/metricbeat.yml
```

### Metricbeat
Normally, container isolation prevents Metricbeat from seeing
information about the host system and/or other
containers. See [_Running Metricbeat in a Container_][mbcontainer] for
details.

In the demo, Metricbeat is configured to monitor processes on the host
system.

[mbcontainer]: https://www.elastic.co/guide/en/beats/metricbeat/current/running-in-container.html

### Filebeat
A common use for a Filebeat container is to monitor logs on the Docker
host system. By default, Filebeat is configured to watch all files
matching `/mnt/log/*.log`. Thus, a quick way to get started is to
mount the host system's log directory:

``` bash
docker run -v /var/log:/mnt/log docker.elastic.co/beats/filebeat:5.3.1
```

This mount is configured in the demo, so Filebeat will ship the logs
of the host system.

### Packetbeat
Packetbeat runs as a non-root user, but requires some network
capabilities to operate correctly. Ensure that the `NET_ADMIN`
capability is available to the container. Like so:

``` bash
docker run --cap-add=NET_ADMIN docker.elastic.co/beats/packetbeat:5.3.1
```

You may also wish to connect the Packetbeat container to the host
network to see traffic for the host system:

``` bash
docker run --cap-add=NET_ADMIN --network=host docker.elastic.co/beats/packetbeat:5.3.1
```

## Supported Docker versions

The images have been tested on Docker 1.13.1.

## Contributing, issues and testing

Acceptance tests for the images are located in the `test` directory,
and can be invoked with `make test`. Python 3.5 is required to run the
tests. They are based on the
excellent [testinfra](http://testinfra.readthedocs.io/en/latest/),
which is itself based on
wonderful [pytest](http://doc.pytest.org/en/latest/).

`beats-docker` is developed under a test-driven
workflow, so please refrain from submitting patches without test
coverage. If you are not familiar with testing in Python, please
raise an issue instead.

This image is built on [Ubuntu 16.04][ubuntu-1604].

[ubuntu-1604]: https://github.com/tianon/docker-brew-ubuntu-core/blob/188bcceb999c0c465b3053efefd4e1a03d3fc47e/xenial/Dockerfile
