---
layout: post
date: 2013-10-17
title: 'Media Capture and Streams API'
description: 'Build a photobooth with the JavaScript Media Capture and Streams API'
categories: 'development'
tags: 'development'
---

I like playing around with JavaScript. Recently I discovered the Media Capture and Streams API,
which is part of the navigator object. The MDN provides some information on the [Media Capture and Streams API](https://developer.mozilla.org/en-US/docs/Web/Guide/API/Camera).
I also couldn't resist creating a small demo. The Media Capture and Streams API still needs vendor prefixing,
so my demo will only work in Chrome since I was to lazy to get it to work in all browsers.
But hey, it's a demo! Go check it out on [JSFiddle](http://jsfiddle.net/vanbosse/uBrSA/) and make sure you allow the use of
your camera by hitting "Allow" in the upper right corner of your browser.
Once the stream is all set, hit the button to take a picture. Even multiple times!

### Creating the HTML
Creating the HTML is fairly easy. All we need is an HTML video tag to stream the feed,
a canvas to be able to "print" the picture and of course a button to create the picture.

{% highlight html %}
<video height="300" width="400" autoplay>Video is not supported by your browser</video>
<div class="buttonHolder">
    <a href="#" class="button" id="pictureButton">Cheese!</a>
</div>
<canvas height="300" width="400"></canvas>
{% endhighlight %}

### JavaScript
In order to display the camera feed, we need to fetch the user media. This can
easily be done by calling the vendor prefixed "getUserMedia" function. All it
takes is an object saying which kind of media we'd like to fetch, which, in this example
 are video and audio and a success function. If you'd like you could also add an
error handler. Now the only thing that is left, is creating a click event on the
button drawing an image of the video onto the canvas. Check out the code below.

{% highlight javascript %}
{
    emitStream: function() {
        // Asking permission to get the user media.
        // If permission granted, assign the stream to the HTML 5 video element.
        navigator.webkitGetUserMedia({video: true, audio: true}, function (stream) {
            that._video = document.querySelector('video');
            that._video.src = window.URL.createObjectURL(stream);
        });
    },

    takePicture: function() {
        // Assigning the video stream to the canvas to create a picture.
        that._canvas = document.querySelector('canvas');
        var context = that._canvas.getContext('2d');
        context.clearRect(0, 0, 0, 0);
        context.drawImage(that._video, 0, 0, 400, 300);
    }
}
{% endhighlight %}
