---
layout: post
date: 2016-02-09
title: 'Getting started with PHPSpec'
description: 'Introduction to the PHPSpec BDD testing framework.'
categories: 'development'
tags: 'development'
---

I'll guide you through the very basics of PHPSpec by using the FizzBuzz code kata.
FizzBuzz is a game in which you count. However, any number divisible by three is
replaced by the word Fizz and any divisible by five by the word Buzz.
Numbers divisible by both become FizzBuzz.

PHPSpec describes itself as:
<blockquote>
    phpspec is a tool which can help you write clean and working PHP code using
    behaviour driven development or BDD. BDD is a technique derived from test-first development.
</blockquote>

### Getting started

First off, use [Composer](https://getcomposer.org/) to install PHPSpec into your project
by creating a `composer.json` file.

{% highlight javascript %}
{
    "require-dev": {
        "phpspec/phpspec": "~2.0"
    },
    "autoload": {
        "psr-0": {
            "": "src"
        }
    }
}
{% endhighlight %}

Once the `composer.json` file is created, run:

{% highlight php %}
$ composer install
{% endhighlight %}

Pro tip: if you want to use `phpspec` instead of `vendor/bin/phpspec`, add the
`vendor/bin` path to your `$PATH`.

### Creating a spec

Once PHPSpec is properly installed, we continue by creating a spec by using the
`phpspec` command.

{% highlight php %}
$ phpspec describe FizzBuzz
{% endhighlight %}

The output should be something like:

{% highlight php %}
Specification for FizzBuzz created in /Users/your-user/Projects/fizz-buzz/spec/FizzBuzzSpec.php.
{% endhighlight %}

As you can see, PHPSpec created both the `spec/` and the `src/` folder in your project.
However, the `src/` folder doesn't contain any files yet. No worries, PHPSpec will
create all that for you. Just run this command and PHPSpec will detect that the implementation
is missing and ask you wether you want to create the the file or not.

{% highlight php %}
$ phpspec run
{% endhighlight %}

Once all files are in place we can continue by implementing our spec. Open the
`FizzBuzzSpec.php` file in your editor of choice and start adding some test cases.

{% highlight php %}
<?php

namespace spec;

use PhpSpec\ObjectBehavior;
use Prophecy\Argument;

class FizzBuzzSpec extends ObjectBehavior
{
    function it_is_initializable()
    {
        $this->shouldHaveType('FizzBuzz');
    }

    function it_returns_1_for_1()
    {
        $this->translate(1)->shouldReturn(1);
    }

    function it_returns_2_for_2()
    {
        $this->translate(2)->shouldReturn(2);
    }

    function it_returns_fizz_for_3()
    {
        $this->translate(3)->shouldReturn('Fizz');
    }

    function it_returns_buzz_for_5()
    {
        $this->translate(5)->shouldReturn('Buzz');
    }

    function it_returns_fizzbuzz_for_15()
    {
        $this->translate(15)->shouldReturn('FizzBuzz');
    }
}
{% endhighlight %}

When we run `phpspec run` again, we'll see our tests fail (except for the first one).
So, we'll have to implement our FizzBuzz class.

{% highlight php %}
<?php

class FizzBuzz
{
    function translate($number)
    {
        if ($number % 15 === 0) {
            return 'FizzBuzz';
        }
        if ($number % 5 === 0) {
            return 'Buzz';
        }
        if ($number % 3 === 0) {
            return 'Fizz';
        }

        return $number;
    }
}
{% endhighlight %}

Run `phpspec run` again and all tests will be green. Such wow, our very first
PHPSpec tests. Of course this is a very simple example, using a simple spec and
thus a simple implementation.

PHPSpec is a very powerful tool and it can do much more than testing this basic
FizzBuzz example. If you want to know more about the options of the `phpspec` command, head over to
the [console docs](http://phpspec.readthedocs.org/en/latest/cookbook/console.html). If you
want to create more extensive tests for your classes, head over to the
[matcher docs](http://phpspec.readthedocs.org/en/latest/cookbook/matchers.html)
to see more test options.
