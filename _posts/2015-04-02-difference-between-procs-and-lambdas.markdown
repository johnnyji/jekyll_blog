---
layout: post
title: "Difference Between Procs and Lambdas"
date: 2015-04-02
categories: Ruby
---

<h3><strong>Lambda</strong></h3>

When you instantiate a `lambda` within a method, the lambda will perform it's function in coherence with the method. So if you define a lambda:

{% highlight ruby %}
def some_method
  my_lambda = lamda { return "Hello!" }.call
  "Goodbye"
end

some_method #=> "Goodbye"
{% endhighlight %}

Even though the lambda has a return statement, it will not exit the entire method but only return itself, and continue on with the method it exists in.
<br><br>

<h3><strong>Proc</strong></h3>

When you instantiate a `Proc.new` within a method, the proc becomes integrated as a part of the method:

{% highlight ruby %}
def some_method
  my_proc = Proc.new { return "Hello!" }.call
  "Goodbye"
end

some_method #=> "Hello!"
{% endhighlight %}

Now when the method runs, once the proc is hit. The entire method will stop and return. 
<br><br>