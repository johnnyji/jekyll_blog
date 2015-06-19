---
layout: post
title: "Hack Moments: eval"
date: 2015-04-19
categories: Rails
---

Upon accident, I found the `eval` method in Ruby. To my understanding, this method will be able to take a stringified array and rearrayify it. (Those can't be real words...)

{% highlight ruby %}
original_array = ["SFU", "BCIT", "Lighthouse Labs"]

stringified_array = "[\"Byrne Creek Secondary\", \"Lighthouse Labs\", \"Codecore Bootcamp\"]"

# Now for the magic...

eval(database_string_version) 
# => ["SFU", "BCIT", "Lighthouse Labs"]
{% endhighlight %}
<br>
<strong>Mind = Blown.</strong>