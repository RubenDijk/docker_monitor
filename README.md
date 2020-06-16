# Custom components for Home Assistant

[![maintainer](https://img.shields.io/badge/maintainer-Sander%20Huisman%20-blue.svg?style=for-the-badge)](https://github.com/Sanderhuisman)

## About

This repository contains the docker monitor component I developed for my own [Home-Assistant](https://www.home-assistant.io) setup. Feel free to use the component and report bugs if you find them. If you want to contribute, please report a bug or pull request and I will reply as soon as possible. Please star & watch my project such I can see how many people like my components and for you to stay in the loop as updates come along.

## Docker Monitor

The Docker monitor allows you to monitor statistics and turn on/off containers. The monitor can connected to a daemon through the url parameter. When home assistant is used within a Docker container, the daemon can be mounted as follows `-v /var/run/docker.sock:/var/run/docker.sock`. The monitor is based on [Glances](https://github.com/nicolargo/glances) and [ha-dockermon](https://github.com/philhawthorne/ha-dockermon) and combines (in my opinion the best of both integrated in HA :)).

## Access Docker API Remote

In case that you run Homeassistant ith an alternative installation with supervisor you will have to make the Docker API available remotely to access from inside the Home-Assistant Docker Container to the host Docker Instance.

Here is a small How-To make the Docker Api available vie remote:

How do I access the Docker REST API remotely?

Warning: After this setup your Docker REST API port (in this case 1111) is exposed to remote access.

Here is how I enabled it on Ubuntu 16.04 (Xenial Xerus).

Edit the docker service file (it is better to avoid directly editing /lib/systemd/system/docker.service as it will be replaced on upgrades)
```sudo systemctl edit docker.service```
Add the following content

```
[Service]
ExecStart=
ExecStart=/usr/bin/docker daemon -H fd:// -H tcp://0.0.0.0:1111
```

For docker 18+, the content is a bit different:

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:1111
```
Save the modified file. Here I used port 1111, but any free port can be used.

Make sure the Docker service notices the modified configuration:

```systemctl daemon-reload```

Restart the Docker service:

```sudo service docker restart```

Test

```curl http://localhost:1111/version```

See the result


```{"Version":"17.05.0-ce","ApiVersion":"1.29","MinAPIVersion":"1.12","GitCommit":"89658be","GoVersion":"go1.7.5","Os":"linux","Arch":"amd64","KernelVersion":"4.15.0-20-generic","BuildTime":"2017-05-04T22:10:54.638119411+00:00"}```

Now you can use the REST API.

How do I access the Docker REST API through a socket (from localhost)?

Connect the internal Unix socket somewhat like this,

Using curl

```curl --unix-socket /var/run/docker.sock http:/localhost/version```

And here is how to do it using PHP

```$fs = fsockopen('/var/run/docker.sock');

fwrite($fs, "GET / HTTP/1.1\r\nHOST: http:/images/json\r\n\r\n");

while (!feof($fs)) {
    print fread($fs,256);
}
```
In PHP 7 you can use curl_setopt with the CURLOPT_UNIX_SOCKET_PATH option.


### Events

The monitor can listen for events on the Docker event bus and can fire an event on the Home Assistant Bus. The monitor will use the following event:

* `{name}_container_event` with name the same set in the configuration.

The event will contain the following data:

* `Container`: Container name
* `Image`: Container image
* `Status`: Container satus
* `Id`: Container ID (long)

### Configuration

To use the `docker_monitor` in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
docker_monitor:
  hosts:
    - url: unix://var/run/docker.sock
      name: Docker
      event: true
      monitored_conditions:
        - version
      containers:
        homeassistant_homeassistant_1:
          switch: False
          sensors:
            - status
            - uptime
            - cpu_percentage_usage
            - memory_usage
            - memory_percentage_usage
            - network_total_up
            - network_total_down
        homeassistant_database_1:
          switch: False
          sensors:
            - status
            - uptime
            - cpu_percentage_usage
            - memory_usage
            - memory_percentage_usage
            - network_total_up
            - network_total_down
        homeassistant_mosquitto_1:
          switch: True
          sensors:
            - status
            - uptime
            - cpu_percentage_usage
            - memory_usage
            - memory_percentage_usage
            - network_total_up
            - network_total_down
```

#### Configuration variables

| Parameter            | Type                     | Description                                                           |
| -------------------- | ------------------------ | --------------------------------------------------------------------- |
| name                 | string       (Required)  | Client name of Docker daemon. Defaults to `Docker`.                   |
| url                  | string       (Required)  | Host URL of Docker daemon. Defaults to `unix://var/run/docker.sock`.  |
| event                | boolean      (Optional)  | Listen for events from Docker. Defaults to false.                     |
| scan_interval        | time_period  (Optional)  | Update interval. Defaults to 10 seconds.                              |
| monitored_conditions | list         (Optional)  | Array of conditions to be monitored. Defaults to all conditions       |
| containers           | list         (Required)  | Array of containers to monitor. Defaults to all containers.           |

| Monitored Conditions              | Description                     | Unit  |
| --------------------------------- | ------------------------------- | ----- |
| version                           | Docker version                  | -     |
| containers_total                  | Total number of containers      | -     |
| containers_paused                 | Number of paused containers     | -     |
| containers_running                | Number of running containers    | -     |
| containers_stopped                | Number of stopped containers    | -     |
| images_total                      | Total number of images          | -     |

| Container Conditions              | Description                     | Unit  |
| --------------------------------- | ------------------------------- | ----- |
| status                            | Container status                | -     |
| uptime                            | Container start time            | -     |
| image                             | Container image                 | -     |
| cpu_percentage_usage              | CPU usage                       | %     |
| memory_usage                      | Memory usage                    | MB    |
| memory_percentage_usage           | Memory usage                    | %     |
| network_total_up                  | Network total upstream          | MB    |
| network_total_down                | Network total downstream        | MB    |

## Credits

* [frenck](https://github.com/frenck/home-assistant-config)
* [robmarkcole](https://github.com/robmarkcole/Hue-sensors-HASS)
