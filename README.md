# confluent-docker

Run the OSS Confluent stack locally with Docker.

This repo is a 'just-enough' repo for running [Confluent OSS Stack](https://www.confluent.io/product/confluent-open-source/) on Docker without either being too minimalistic or including the whole suite of services. This is also the baseline Kafka configuration for developing Guthega and related modules.

To start with, it's based on the ubiquitous Docker toolchain (for [Mac](https://docs.docker.com/docker-for-mac/) or [Windows](https://docs.docker.com/docker-for-windows/)), but will be extended to include Kubernetes in due course.

## Running the Stack

```
docker-compose up
```

This will start the following containers:

* Zookeeper - bound to host port 2181
* Kafka - bound to host port 9092
* Schema Registry
* Rest Proxy
* Nginx reverse proxy to the Rest Proxy - bound to host port 80

## Interacting with the Stack

### Configure Your hosts File

If you'd like to connect to the Kafka and/or Zookeeper service from an application running on your local machine, you'll need to make a minor modification to your `hosts` file:

```
...
127.0.0.1	localhost kafka zookeeper
...
```

### Via Kafka CLI Tools

Once the stack is up and running, you can use the standard Kafka command line tools to perform administrative functions. For example:

```
kafka-topics --create --zookeeper kafka:2181 --replication-factor=1 --partitions=1 test
```

Refer to the [Kafka Quickstart](https://kafka.apache.org/quickstart) documentation for examples of the types of interactions you can have on the command line.

### Via API

An Nginx container front-ends the standard Confluent rest-proxy container. You can interact with the API through port 80:

```
http :80/topics
```

To post a message:

```
http POST :80/topics/test \
  "Content-Type:application/vnd.kafka.json.v1+json" \
  < test.json
```

Refer to the [API documentation](https://docs.confluent.io/current/kafka-rest/docs/api.html) for a full list of API endpoints.

### Via Docker Compose

To allow your Dockerized application to connect to the Kafka services, you need to ensure that they're running on the same Docker network on your machine. Add the following to the `networks` configuration in your `docker-compose.yml` file:

```
networks:
  kafka:
    external:
      name: confluent-docker_kafka
```

Then, in each of your application services, specify the `network` it should run in:

```
...
services:
...
  myservice:
    build: .
    ...
    networks:
      - kafka
...
```
