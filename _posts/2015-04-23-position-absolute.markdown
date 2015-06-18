---
layout: post
title: "Position: Absolute"
date: 2015-04-23
categories: CSS
---

So I've been using `position: absolute;` for about as long as I can rememeber, but today I can say I fully <strong>fully</strong> understand it. Here's the golden rule:

<br>
  `position: absolute;` <strong>will position itself relative to the FIRST parent element that is NOT</strong> `position: static;`  
<br>

This is very important because by default elements are going to be `position: static;`. Therefore if you want to position divs absolutely inside other divs, MAKE SURE those parent divs have their positions defined as anything besides static!

<br><br>
<h2>YES I FINALLY GOT IT. THIS IS THE BEST FEELING EVER.</h2>

(Thanks to [Murat][murat])

[murat]: http://muratayfer.com/