# Swarm Dashboard

A simple monitoring dashboard for Docker in Swarm Mode.

![Example Dashboard](./swarm.gif)

## About

Swarm dashboard shows you all the tasks running on a Docker Swarm organised
by service and node. It provides a visualisation that's space efficient
and works well at a glance.

You can use it as a simple live dashboard of the state of your Swarm.

The Dashboard has a node.js server which streams swarm updates to an Elm client
over a websocket.

### Prior art

* Heavily inspired by [Docker Swarm Visualiser](https://github.com/dockersamples/docker-swarm-visualizer)

## Running

At the moment, the dashboard needs to be deployed on one of the swarm managers.
You can configure it with the following Docker compose file:

```yml
# compose.yml
version: "3"

services:
  dashboard:
    image: charypar/swarm-dashboard
    volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
    ports:
    - 8080:8080
    environment:
      PORT: 8080
      # DOCKER_SOCKET: /var/run/docker.sock # Optionally customise where the docker socket is
      # DOCKER_HOST: 127.0.0.1 # Or provide a tcp connection to a docker proxy process.
      # DOCKER_PORT: 2375
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
```

and deploy with

```
$ docker stack deploy -c compose.yml svc
```

## Production use

There are two considerations for any serious deployment of the dashboard:

1. Security - the dashboard node.js server has access to the docker daemon unix socket
   and runs on the manager, which makes it a significant attack surface (i.e. compromising
   the dashboard's node server would give an attacker full control of the swarm)
1. The interaction with docker API is a fairly rough implementation and
   is not very optimised. The server polls the API every 500 ms, publishing the
   response data to all open websockets if it changed since last time. There
   is probably a better way to look for changes in the Swarm that could be used
   in the future.

## Rough roadmap

* Show more service details (published port, image name and version)
* Node / Service / Task details panel
* Show node / task resources (CPU & Memory)
* Improve security for potential production use

Both feature requests and pull requests are welcome

## Contributors

* Viktor Charypar (owner, BDFL) - code, docs
* Clementine Brown - design
