---
layout: post
title: "CSS Showdown: REM vs. EM"
date: 2015-06-11
categories: CSS
---

So up until now I've always tried to use `em` whenever possible for sizing fonts and other things alike. However, I've never taken the time to fully understand the difference between `em` and `rem`. 

After taking some time to read upon this subject. I may actually never use `em` again... (at least when it comes to fonts). Here's why.

`rem` stands for <strong>Root EM</strong>. Whenever your referring to a unit of measurement as `em` in css, it's referring to it's closest <strong>DIRECT PARENT</strong>.

So `1em` of a direct child component is 100% of it's parent, `0.73em` is 73% of it's parent etc...

I find the problem with this approach is that the CSS isn't quite elegantly scaled. If I change an `em` somewhere in my code, I have no idea how many other styles are affected down the chain, and in a bigger application, this could become troublesome quick.

So enter `rem`, which is relative not to it's DIRECT parent component, but the root parent component (body, html). Sizing in `rem` is alot easier because I can be rest assured that changing the sizing on a particular style will have no effect on other elements, and that whenever I'm specifiying sizing, I only have to specify it to the body or html versus whatever that component's direct parent is.

So <strong>TL;DR</strong> answer: Use `rem` over `em` because rem is relative to the body and html rather than direct parent, which is both mentally easier to comprehend and prevents against unintended sizing changes.