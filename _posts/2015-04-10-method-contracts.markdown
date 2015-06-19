---
layout: post
title: "Method contracts"
date: 2015-04-10
categories: Theory
---

This post will be somewhat conceptual but equally as important as any of the other ones. It's 2 weeks into the [Lighthouse Labs][lighthouse] course right now and one of the instructors [David VanDusen][david] has dawned on me an extremely important concept. <strong>Method Contracts.</strong>

Methods/functions in programming have all have a specific purpose, and their method contracts define their functionality. When writing any method, you should be able to clearly state it's method contract in one sentence WITHOUT using the word <em>and</em>.

<br>
<strong><em>The blueprint for a method contract is as follows:</em></strong>
<blockquote>
  <ol>
    <li>What arguments are the method parameters taking, and what are the argument types?</li>
    <li>What functionality is the method responsible for?</li>
    <li>What are the values the method returns and their value types?</li>
    <li>What exceptions do the method raise under what circumstances?</li>
  </ol>
</blockquote>
<br>
  
A method is definitive and well designed when you can answer those 4 questions without a shadow of a doubt.

Invaluable expertise; I'll never look at methods the same again.

[lighthouse]: http://lighthouselabs.ca
[david]: http://davidvandusen.com