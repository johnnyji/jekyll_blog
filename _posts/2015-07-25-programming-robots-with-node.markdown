---
layout: post
title: "My Experience Programming Robots in NodeJS"
date: 2015-07-25
categories: Node
---

Today I attended [NodeSchoolYVR][nodeschool] over at the [Zillow][zillow] headquarters in Yaletown, Vancouver; and man was it the best programming experience I've had in a while. The last time I was this amazed was when I first learnt React.

The topic of interest was programming hardware with JavaScript (basically building robots to take over the world), using an amazing JavaScript robotics library called <strong>[johnny-five][johnnyfive]</strong>.

The whole experience was fantastic and it's amazing how one simple snippet of JavaScript code such as the one below can make hardware do such amazing things:

{% highlight javascript %}
var sys = require("sys");
var exec = require('child_process').exec;
var five = require('johnny-five');
var board = new five.Board();

function puts(err, stdout, stderr) { sys.puts(stdout) };

board.on('ready', function() {
  console.log("board ready");
  var button = new five.Button(5);
  var led = new five.Led(9);

  button.on("press", function() {
    exec("say its party time", puts);
    led.strobe(100);
  });
});
{% endhighlight %}
<br>
 
Here's a quick highlight of the awesome stuff we built in just 3 hours of learning the Johnny-Five library:
<br><br>

<strong>A robot fan for a hot summer day:</strong>
<blockquote class="instagram-media" data-instgrm-captioned data-instgrm-version="4" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); margin: 1px; max-width:658px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px); width:calc(100% - 2px);"><div style="padding:8px;"> <div style=" background:#F8F8F8; line-height:0; margin-top:40px; padding:50% 0; text-align:center; width:100%;"> <div style=" background:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACwAAAAsCAMAAAApWqozAAAAGFBMVEUiIiI9PT0eHh4gIB4hIBkcHBwcHBwcHBydr+JQAAAACHRSTlMABA4YHyQsM5jtaMwAAADfSURBVDjL7ZVBEgMhCAQBAf//42xcNbpAqakcM0ftUmFAAIBE81IqBJdS3lS6zs3bIpB9WED3YYXFPmHRfT8sgyrCP1x8uEUxLMzNWElFOYCV6mHWWwMzdPEKHlhLw7NWJqkHc4uIZphavDzA2JPzUDsBZziNae2S6owH8xPmX8G7zzgKEOPUoYHvGz1TBCxMkd3kwNVbU0gKHkx+iZILf77IofhrY1nYFnB/lQPb79drWOyJVa/DAvg9B/rLB4cC+Nqgdz/TvBbBnr6GBReqn/nRmDgaQEej7WhonozjF+Y2I/fZou/qAAAAAElFTkSuQmCC); display:block; height:44px; margin:0 auto -44px; position:relative; top:-22px; width:44px;"></div></div> <p style=" margin:8px 0 0 0; padding:0 4px;"> <a href="https://instagram.com/p/5kuQ0KhZTF/" style=" color:#000; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:normal; line-height:17px; text-decoration:none; word-wrap:break-word;" target="_top">It got too hot so I programmed a fan to keep us cool #nodeschoolinternational #nodeschool #robots #javascript</a></p> <p style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; line-height:17px; margin-bottom:0; margin-top:8px; overflow:hidden; padding:8px 0 7px; text-align:center; text-overflow:ellipsis; white-space:nowrap;">A video posted by Johnny Ji (@johnnyisji) on <time style=" font-family:Arial,sans-serif; font-size:14px; line-height:17px;" datetime="2015-07-25T21:13:24+00:00">Jul 25, 2015 at 2:13pm PDT</time></p></div></blockquote>
<script async defer src="//platform.instagram.com/en_US/embeds.js"></script>

<br><br>

<strong>Building a party robot:</strong>
<iframe 
  width="400px"
  height="300px"
  frameborder="0"
  src="https://instagram.com/p/5kuQ0KhZTF/?taken-by=johnnyisji" 
  allowfullscreen></iframe>

<br><br>

Not a day goes by where I'm not absolutely amazed by the power of JavaScript! Until next time.

[zillow]: http://www.zillow.com/
[nodeschool]: http://nodeschool.io/
[johnnyfive]: https://github.com/rwaldron/johnny-five