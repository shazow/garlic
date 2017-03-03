# garlic

Garlic is a language agnostic task scheduler.

**Status**: README-driven development, no functionality.


## Overview

Tasks are scheduled as webhooks that get delivered to a worker over HTTP. Workers
can be implemented on any platform that speaks HTTP. All benefits from existing
HTTP infrastructure and tooling applies, such as load balancing.

Garlic is written in Go, and it is also a thing that gophers like to eat.


## Goals

Garlic is the task or job scheduler and queue; a client is anything that creates new jobs; workers handle job execution.

* Garlic, clients, and workers speak HTTP.
* Workers are language and implementation agnostic. Anything that speaks HTTP can be a worker. You can have many workers, each implemented differently. The scheduler does not care.
* Garlic is an incremental addition that is easily replaceable.
* Garlic can be directly replaced with a worker and things continue to work (without the extra features like scheduling).

In effect, any garlic-compliant worker is a basic garlic substitute.


## Usage


Start garlic:

```shell
$ garlic --bind=127.0.0.1:6060
```

Start a worker, this can be any HTTP-speaking server:

```shell
$ nc -l 127.0.0.1 12345 << EOF
HTTP/1.0 200 OK
EOF
```

Schedule a job:

```shell
$ curl http://127.0.0.1:6060/ -d '{"target": "http://127.0.0.1:12345/myendpoint", "delay": "5min", "hello": "world"}'
```

Only one parameter is required: `target`. Some parameters have special meaning, like `delay` will delay the job. Other parameters are just passed on to the webhook wholesale. When the webhook request is made, the original job request is the payload.

Five minutes later, the worker will see an HTTP request come in that looks like this:

```
POST /myendpoint HTTP/1.1
Host: 127.0.0.1:12345
User-Agent: garlic/1.0
Accept: */*
Content-Length: 85
Content-Type: application/json

{"target": "http://127.0.0.1:12345/myendpoint", "delay": "5min", "hello": "world"}
```

*In fact*, the request that the worker receives through garlic is equivalent to the worker getting called directly the same way garlic would be called (but without the extra scheduling features). This makes it very convenient for development environments where you might not want to run a garlic daemon, but simply change the garlic endpoint to be your worker endpoint.


## Suggested scheduler features

- Scheduled tasks should return a response to the client as soon as possible. The service should feel asynchronous.
- Endpoint for listing jobs and stats.
- Failure reporting (email, webhook).

Suggested special key behaviour:

- `delay` will parse the value and delay the job appropriately. Example: "5min", "2pm", "tuesday"
- `cron` will parse cron-like syntax and schedule recurring jobs.
- `id` allows to specify a concrete identifier for the job. If the id already exists, it will replace the existing job.
- `delete` will delete the value id if it exists (other behaviours like delay also stack).

Also maybe:

- Resource throttling.
- Timeouts.


## Security

* Bind garlic only locally-accessible hosts. Avoid exposing your garlic
  scheduler to the public.

If garlic must be exposed to the public,

* Use HTTPS.
* Use Basic-Auth to authorize clients.
* Include and verify a secret token in each job payload for workers.


## License

MIT
