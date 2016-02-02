---
layout: post
date: 2012-10-17
title: 'Pub-sub with RabbitMQ and WebSocket'
description: 'Use Publish/Subscribe for realtime interaction with PHP, RabbitMQ and WebSocket'
categories: 'development'
tags: 'development'
---

Aren't realtime updates great? Here's a way of implementing realtime into apps.
Using [RabbitMQ](http://rabbitmq.com), PHP, [Node.js](http://nodejs.org) and [Socket.io](http://socket.io).
In this blog post I'll be using an example. Imagine you are at a train station,
waiting for the next train heading towards Berserkistan. Wouldn't it be great if
you could see updates on when the train is going to arrive?
A demo application is available at: [https://github.com/vanbosse/rabbitmq-demo](https://github.com/vanbosse/rabbitmq-demo).
Follow the instructions in the `README` file.

### RabbitMQ
First of all, we need RabbitMQ. RabbitMQ is a message broker that implements the
[Advanced Message Queuing Procotol (AMQP)](https://github.com/vanbosse/rabbitmq-demo).
It's a server, written in Erlang. To install the RabbitMQ server, visit [http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/download.html)
and follow the instructions for your OS. Once the server is installed, start it!

### Publish/subscribe
The publish/subcribe pattern basically says that subscribers receive all updates
a publisher sends out. In this case, we'll have a PHP script functioning as
the publisher and a Node.js server functioning as a subscriber.
Every update the script sends out, will be received by the server.

The publisher needs to create a connection with the RabbitMQ server.
When a connection is established, an exchange can be created.
An exchange handles all incoming updates and redistributes them to the available queues.
When an exchange is created, messages can be sent. Sending updates if fairly easy:

{% highlight php %}
<?php

require_once __DIR__ . '/lib/php-amqplib/amqp.inc';

// Create a connection with RabbitMQ server.
$connection = new AMQPConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

// Create a fanout exchange.
// A fanout exchange broadcasts to all known queues.
$channel->exchange_declare('updates', 'fanout', false, false, false);

// Create and publish the message to the exchange.
while (true)
{
    $data = array(
        'type' => 'update',
        'data' => array(
            'minutes' => rand(0, 60),
            'seconds' => rand(0, 60)
        )
    );
    $message = new AMQPMessage(json_encode($data));
    $channel->basic_publish($message, 'updates');
    sleep(3);
}

// Close connection.
$channel->close();
$connection->close();
{% endhighlight %}

The Node.js server is a subscriber. It also needs to connect with the RabbitMQ server.
Once the server is connected to the RabbitMQ server it can subscribe to the exchange. It can now receive messages.

So, for our story this means, if a train sends out location updates to the RabbitMQ
server on a regular basis, the Node.js server will receive these updates since it's a subscriber.

### WebSocket
When the Node.js server notices a message has been added, it can send out that
message to all of its connected websockets. A websocket connected to the Node.js
server can for example be someone who's using his browser to see the webpage containing the updates of the train.

{% highlight javascript %}
// Create context using rabbit.js (cfr ZMQ),
// io and the subscriber socket.
var context = require('rabbit.js').createContext(),
    io = require('socket.io').listen(8080),
    sub = context.socket('SUB');

// Set correct encoding.
sub.setEncoding('utf8');

// A websocket is connected (eg: browser).
io.sockets.on('connection', function(socket) {

    // Connect socket to updates exchange.
    sub.connect('updates');

    // Register handler that hanles incoming data when the socket
    // detects new data on our queues.
    // When receiving data, it gets pushed to the connectec websocket.
    sub.on('data', function(data) {
        var message = JSON.parse(data);
        socket.emit(message.type, message.data);
    });
});
{% endhighlight %}

Again, for our story this means that, when an update is received, an update
can be sent out to all of the waiting passengers.

### Demo
In: [https://github.com/vanbosse/rabbitmq-demo](https://github.com/vanbosse/rabbitmq-demo)
lives a demo on these realtime updates. Follow the instructions in the `README` file and have fun.
The code is well documented, make sure to check out the comments in the code.

More info and tutorials on RabbitMQ: [http://www.rabbitmq.com/getstarted.html](http://www.rabbitmq.com/getstarted.html)
