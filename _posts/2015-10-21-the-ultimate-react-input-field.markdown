---
layout: post
title: 'The Ultimate React Input Field!'
date: 2015-10-21
categories: React
---

How complicated could an input field be? As it turns out, really fucking complicated; seriously. I recently started my new job at [Cumul8][cumul8] and man is it awesome. The people here are wickedly smart, ESPECIALLY my lead dev.

I've been following a lot of really good practices since I've been here and one of my first tasks was to improve the input field components. I've learnt so much in these first couple weeks that I thought I'd do a brief overview of the way I now build input field components.

### Here's what our input field component(s) should accomplish:
  - Ability to inherit it's values from a wrapper (parent) component
  - Ability to notify to the wrapper when the input data changes, so it can change it's state containing the data
  - Ability to update the data state kept on the wrapper no matter how nested the state object is (This is the one that blew my mind.)
  - Be very easy to drop in anywhere and use (That's the point!)

### Here what we'll be using:
  - React (Duh)
  - Immutable.js (I'll also do a version not using Immutable)
  - ES6/ES7 (All the young kids are doing it!)

[cumul8]: http://cumul8.com/