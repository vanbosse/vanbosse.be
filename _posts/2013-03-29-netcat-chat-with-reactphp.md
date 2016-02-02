---
layout: post
date: 2013-03-29
title: 'Netcat chat with ReactPHP'
description: 'Build a netcat chat client with the ReactPHP framework'
categories: 'development'
tags: 'development'
---

We'll be creating a chat application but, not in your browser, but built with PHP.
What we will be using is [ReactPHP](http://reactphp.org/) and the netcat command line tool.
We'll create a server, running on PHP and use netcat as our client.
Check out the repository of the demo here: [https://github.com/vanbosse/react-netcat-chat](https://github.com/vanbosse/react-netcat-chat).

### ReactPHP
ReactPHP bescribes itself as:
<blockquote>
Event-driven, non-blocking I/O with PHP.
</blockquote>
Make sure you visit the [http://reactphp.org](http://reactphp.org) website, it might look familiar.

React is a low-level library for event-driven programming in PHP.
At its core is an event loop, on top of which it provides low-level utilities, such as:
Streams abstraction, async dns resolver, network client/server, http client/server, interaction with processes.
Third-party libraries can use these components to create async network clients/servers and more.

### Getting started
If you'd like to try React yourself, just get started, installing ReactPHP is easy.
Start off with installing [Composer](https://getcomposer.org).

{% highlight php %}
$ curl -sS https://getcomposer.org/installer | php
{% endhighlight %}

If you'd like Composer to be available system wide, issue:

{% highlight php %}
$ sudo mv composer.phar /usr/local/bin/composer
{% endhighlight %}

When you like Composer, but it feels kinda new. Make sure you visit their documentation.
So, with Composer installed, we're ready to download ReactPHP.
Just issue the following commands and wait for composer to do its job.

{% highlight php %}
$ composer init --require=react/http:0.2.* -n
$ composer install
{% endhighlight %}

Now everything is in place, let's get cracking!

### Netcat chat application
In the example I created, I've built a simple server script. What it does is,
it creates a loop and a socket server. If anyone knocks on our server door,
we accept the connection, ask for a nickname and store the connection.
When one of these connected clients send data, the server will prepend the correct
nickname and output the correct message. I did not validate anything, so just
use the application as you're supposed to. To run the server issue:

{% highlight php %}
$ php server.php
{% endhighlight %}

And to connect a client to the server, just netcat into the correct port.

{% highlight php %}
$ nc localhost 1337
{% endhighlight %}

If you'd like to collect the repository and check the demo, follow the
instructions in the `README` file: [https://github.com/vanbosse/react-netcat-chat](https://github.com/vanbosse/react-netcat-chat).

Igor Wiedler, the main man behind ReactPHP even has a ChatRoulette demo.
Definitely worth checking out! You can find it here: [https://github.com/reactphp/chatroulette](https://github.com/reactphp/chatroulette).
