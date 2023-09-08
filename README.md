# Use Benthos.dev to source data from one RabbitMQ cluster to another

The following Docker Compose file creates a RabbitMQ 3.7.9 container and a Rabbit 3.11.18 container.

The image shows RabbitMQ 3.7.9 on the left and RabbitMQ 3.11.18 on the right after using Benthos to move a few messages from source to destination

![RabbitMQ source and destination screenshot](./images/rabbitmq_screenshot.png)

Install Benthos to copy data

```
curl -Lsf https://sh.benthos.dev | bash
```

Send some sample data in to a source Rabbit MQ exchange (create the exchange and queue first)

```
curl -u guest:guest -X POST http://localhost:15672/api/exchanges/%2f/example_exchange/publish -d'{"properties":{},"routing_key":"my_key","payload":"Hello, World!","payload_encoding":"string"}'
```

Create a Benthos config file

```
input:
  amqp_0_9:
    urls: ["amqp://guest:guest@localhost:5672/"]
    queue: example_queue
    prefetch_count: 10
    consumer_tag: benthos_consumer

output:
  amqp_0_9:
    urls: ["amqp://guest:guest@localhost:5673/"]
    exchange: destination_exchange
```

Run Benthos

```
benthos -c ./config.yaml
```

You'll see an output like this:

```
INFO Running main config from specified file       @service=benthos path=./config.yaml
INFO Listening for HTTP requests at: http://0.0.0.0:4195  @service=benthos
INFO Launching a benthos instance, use CTRL+C to close  @service=benthos
INFO Sending AMQP messages to exchange: destination_exchange  @service=benthos label="" path=root.output
INFO Receiving AMQP 0.9 messages from queue: example_queue  @service=benthos label="" path=root.input
```
