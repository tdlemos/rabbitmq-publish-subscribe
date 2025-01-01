# .NET 8 RabbitMQ - Publish & Subscribe

In this exemple we'll deliver a message to multiple consumers. This pattern is known as "publish/subscribe".

To illustrate the pattern, we're going to build a simple logging system. It will consist of two programs -- the first will emit log messages and the second will receive and print them.

In our logging system every running copy of the receiver program will get the messages. That way we'll be able to run one receiver and direct the logs to disk; and at the same time we'll be able to run another receiver and see the logs on the screen.

Essentially, published log messages are going to be broadcast to all the receivers.

## Exchanges
Now it's time to introduce the full messaging model in Rabbit.

Let's quickly go over what we covered in Queue pattern:

- A producer is a user application that sends messages.
- A queue is a buffer that stores messages.
- A consumer is a user application that receives messages.

The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Actually, quite often the producer doesn't even know if a message will be delivered to any queue at all.

Instead, the producer can only send messages to an exchange. An exchange is a very simple thing. On one side it receives messages from producers and the other side it pushes them to queues. The exchange must know exactly what to do with a message it receives. Should it be appended to a particular queue? Should it be appended to many queues? Or should it get discarded. The rules for that are defined by the exchange type.
![](images/flow1.jpg)

There are a few exchange types available: **__direct__**, **__topic__**, **__headers__** and **__fanout__**. We'll focus on the last one -- the fanout. Let's create an exchange of this type, and call it logs:


```CSharp
await channel.ExchangeDeclareAsync(exchange: "logs", type: ExchangeType.Fanout);
```

The fanout exchange is very simple. As you can probably guess from the name, it just broadcasts all the messages it receives to all the queues it knows. And that's exactly what we need for our logger.

> ### Listing exchanges
> To list the exchanges on the server you can run the ever useful rabbitmqctl:
> 
> ````linux
> sudo rabbitmqctl list_exchanges
> ````
>In this list there will be some amq.* exchanges and the default (unnamed) exchange. These are created by default, but it is unlikely ?you'll need to use them at the moment.

> ### The dafult exchange