---
layout: post
date: 2016-03-06
title: 'Elastic Series Part 1: Install, config & CRUD'
description: 'Getting started with Elasticsearch. Install it, configure it and perform basic CRUD.'
categories: 'development'
tags: 'development'
---

In the "Elastic Series" we'll cover some basics on Elasticsearch. The first part
will cover the installation of Elasticsearch, the configuration of Elasticsearch
and some basic CRUD operations. But first of all...

### What is Elasticsearch

Elasticsearch is a search server based on Lucene. It provides a distributed,
multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents.

### Get started

Get started by [downloading Elasticsearch](https://www.elastic.co/downloads/elasticsearch). Once it's downloaded,
untar/unzip it into a folder of choice and run the ```bin/elasticsearch``` command to start a node.

### Configuration

Actually Elasticsearch doesn't need any configuration. They made sure everything has a good default value.
So, if you're fine with the default settings, you can just fire up a node. However, it's best practice to
change some settings. Head over to the ```config``` folder in which you'll find the ```elasticsearch.yml``` file.

You'll want to change the ```cluster.name``` setting to something unique for your setup, for example ```elastiscsearch-<yourproject>-<dev|prod>```
or something like that. Because, when you fire up a node, nodes will try to join this cluster.
So having a more unique name than "elasticsearch" is recommended. You might also want to
set the ```node.name``` setting to something more generic than a Marvel Comic Name, which
is what you'll get when you do not specify a name.

Set the ```bootstrap.mlockall``` to ```true``` to prevent swapping.
Set ```network.host``` to ```127.0.0.1``` to prevent access from anything else than the localhost.

We're all set, fire up your node by running ```bin/elasticsearch``` and verify the health of your cluster by running:
{% highlight php %}
$ curl -XGET localhost:9200/_cluster/health?pretty
{% endhighlight %}

### How Elasticsearch works

Everything in Elastichsearch is a JSON document. You basically use HTTP verbs to perform your actions.
So for CRUD this would mean:
{% highlight php %}
Create => PUT or POST
Read => GET
Update => PUT or POST
Delete => DELETE
{% endhighlight %}

The way Elasticsearch stores data is with ```indices``` and ```types```. And it's structured like this:
```index/type/identifier```. For example one can have: ```library/books/1```. So the JSON document
with ID 1 is of type book and resides in the library index.

### CRUD

So, say we want to add a book with ID = 1 to the library, we can do this by sending JSON data over PUT.
You'll get a JSON response in which the field ```created``` should be set to ```true```.
{% highlight php %}
# Request
$ curl -XPUT localhost:9200/library/books/1?pretty -d '{
    "title": "My superawesome book.",
    "author": {
        "first_name": "John",
        "last_name": "Doe"
    },
    "price": 19.95
}'
{% endhighlight %}

{% highlight php %}
# Response
{
    "_index" : "library",
    "_type" : "books",
    "_id" : "1",
    "_version" : 1,
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "created" : true
}
{% endhighlight %}

To verify the book was created you can try to collect the book. You'll receive a JSON response again
in which the book you've added should be under the ```_source``` field. The ```found``` field can be checked
in order if anything was found for the specific ID. And you'll get a set of meta data as well.
{% highlight php %}
# Request
$ curl -XGET localhost:9200/library/books/1?pretty
{% endhighlight %}

{% highlight php %}
# Response
{
    "_index" : "library",
    "_type" : "books",
    "_id" : "1",
    "_version" : 1,
    "found" : true,
    "_source" : {
        "title" : "My superawesome book.",
        "author" : {
            "first_name" : "John",
            "last_name" : "Doe"
        },
        "price" : 19.95
    }
}
{% endhighlight %}

However, if you don't want to specify the identifier of the document, you can create a document over POST
instead of PUT. You'll get a JSON response in which the auto-generated ID will be present in the meta data.
{% highlight php %}
# Request
$ curl -XPOST localhost:9200/library/books?pretty -d '{
    "title": "My ID-less book",
    "author": {
        "first_name": "John",
        "last_name": "Doe"
    },
    "price": 24.95
}'
{% endhighlight %}

{% highlight php %}
# Response
{
    "_index" : "library",
    "_type" : "books",
    "_id" : "AVNMeI_hHHrKVbiXKFAX",
    "_version" : 1,
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "created" : true
}
{% endhighlight %}

If you want to update a book, the simplest way is to just overwrite it. In the JSON response, you'll see that
the ```created``` field will be ```false``` and that the ```version``` meta data will be bumped one version.
{% highlight php %}
# Request
$ curl -XPUT localhost:9200/library/books/1?pretty -d '{
    "title": "My superawesome comicbook.",
    "author": {
        "first_name": "John",
        "last_name": "Doe"
    },
    "price": 19.95
}'
{% endhighlight %}

{% highlight php %}
# Response
{
    "_index" : "library",
    "_type" : "books",
    "_id" : "1",
    "_version" : 2,
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    },
    "created" : false
}
{% endhighlight %}

You can also perform a partial update if you don't want to overwrite all the data. Append ```/_update```
to your POST request and add the document data you want to update. Note that the reponse will again return the updated
version meta data.
{% highlight php %}
# Request
$ curl -XPOST localhost:9200/library/books/1/_update?pretty -d '{
    "doc": {
        "price": 9.95
    }
}'
{% endhighlight %}

{% highlight php %}
# Response
{
    "_index" : "library",
    "_type" : "books",
    "_id" : "1",
    "_version" : 3,
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    }
}
{% endhighlight %}

In order to delete data, just perform a DELETE request to the document you want to delete. The JSON response
will contain wether the document was found and if it was successfully deleted. You can still verify by
trying to collect the document afterwards. You should see that the ```found``` field is now set to ```false```
{% highlight php %}
# Request
$ curl -XDELETE localhost:9200/library/books/1?pretty
{% endhighlight %}

{% highlight php %}
# Response
{
    "found" : true,
    "_index" : "library",
    "_type" : "books",
    "_id" : "1",
    "_version" : 4,
    "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
    }
}
{% endhighlight %}

Verify with a GET request:

{% highlight php %}
# Request
$ curl -XGET localhost:9200/library/books/1?pretty
{% endhighlight %}

{% highlight php %}
# Response
{
    "_index" : "library",
    "_type" : "books",
    "_id" : "1",
    "found" : false
}
{% endhighlight %}

### What's next?

I'll try to keep up the series and do some posts on Kibana and Marvel, as well as on querying/filtering/searching and
diving deeper into it with aggregated searches. Best of luck!
