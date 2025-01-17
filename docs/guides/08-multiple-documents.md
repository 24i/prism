# Serving Multiple OpenAPI Documents

A single Prism instance serves one OpenAPI document, but serving multiple documents is possible by running multiple Prism instances on different documents, then serving their content using a reverse Proxy. 

You can use any HTTP proxy along with a process management tool to achieve this. The following example explains how to serve multiple OpenAPI documents using [Docker Compose](https://docs.docker.com/compose/).

## Preparing configuration files

Depending on your needs, you might have a `docker-compose.yaml` file like this:

```yaml
version: '3'
services:
  proxy:
    image: caddy
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
    ports:
      - '8080:80'
    depends_on:
      - prism_1
      - prism_2
  prism_1:
    image: stoplight/prism:4
    command: >
      mock -p 4010 --host 0.0.0.0
      https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v2.0/yaml/petstore.yaml
  prism_2:
    image: stoplight/prism:4
    command: >
      mock -p 4010 --host 0.0.0.0
      https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore.yaml
```

And the corresponding `Caddyfile` file:

```
http://localhost

route /app_1/* {
	uri strip_prefix /app_1
	reverse_proxy prism_1:4010
}

route /app_2/* {
	uri strip_prefix /app_2
	reverse_proxy prism_2:4010
}
```

This configuration will allow access to the first Prism instance on `localhost:8080/app_1` and the second instance on `localhost:8080/app_2` endpoint.
Also, please mind that the provided `Caddyfile` only serves illustrative purposes and hence should not be considered complete nor production-ready.

## Running the cluster

Let's first place the aforementioned files under the same directory - here `prism-multi-example`.

```
prism-multi-example
├── docker-compose.yaml
└── Caddyfile
```

With the files in place, you can run (from under `prism-multi-example/`): `docker-compose up`.
After a second, you should be set up with two Prism instances serving different OAS documents.

## Testing the cluster

If you used `Caddyfile` configuration provided in this document, you should now have two endpoints available:

1. `http://localhost:8080/app_1`
2. `http://localhost:8080/app_2`

You can now try these out by issuing:

```bash
curl http://localhost:8080/app_1/pets
curl http://localhost:8080/app_2/pets
```

At this point, the responses should be given back without issues.

## Adding more documents

You can easily extend the cluster by adding more nodes to `docker-compose.yaml`.
Under `services`, you can add:

```yaml
prism_3:
  image: stoplight/prism:4
  command: >
    mock -p 4010 --host 0.0.0.0
    https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore.yaml
```

To make the new node accessible from the proxy, please also adjust `Caddyfile`:

```
route /app_3/* {
	uri strip_prefix /app_3
	reverse_proxy prism_3:4010
}

```
