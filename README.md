# Webhook job queue
The webhook job queue is designed to receive notifications from services and
store the JSON as a dictionary in a `python-rq` redis queue. It is designed
to work with a [Webhook Relay](https://github.com/lscsoft/webhook-relay) that
validates and relays webhooks from known services such as DockerHub, Docker
registries, GitHub, and GitLab. However, this is not required and the receiver
may listen directly to these services.

A worker must be spawned separately to read from the queue and perform tasks in
response to the event. The worker must have a function named `webhook.job`.

## Running

The job queue requires [docker-compose](https://docs.docker.com/compose/install/)
and, in its simplest form, can be invoked with `docker-compose up`. By default,
it will bind to `localhost:8080` but allow clients from all IP addresses. This
may appear odd, but on MacOS and Windows, traffic to the containers will appear
as though it's coming from the gateway of the network created by
Docker's linux virtualization.

In a production environment without the networking restrictions imposed by
MacOS/Windows, you might elect to provide different defaults through the
the shell environment. _e.g._
```
ALLOWED_IPS=A.B.C.D LISTEN_IP=0.0.0.0 docker-compose up
```
where `A.B.C.D` is an IP address (or CIDR range) from which your webhooks will
be sent.

A [worker must be spawned](#example-worker) to perform tasks by removing the
notification data from the redis queue. The redis keystore is configured to
listen only to clients on the `localhost`.

## Example worker
To run jobs using the webhooks as input:

1. Create a file named `webhook.py`
2. Define a function within named `job` that takes a `dict` as its lone argument
3. Install `python-rq`
    * _e.g._ `pip install rq`
4. Run `rq worker` from within that directory

See the [CVMFS-to-Docker converter](https://github.com/lscsoft/cvmfs-docker-worker)
for a real world example.
