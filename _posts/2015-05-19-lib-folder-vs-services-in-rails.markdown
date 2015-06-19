---
layout: post
title: "Lib Folder vs. Services in Rails"
date: 2015-05-19
categories: Rails
---

It's by convention, awful practice to be performing business logic within your controllers.

Logic should always be stored in some type of class or model. The sole purpose of the controller is to direct the flow of the application to different places.Logic pertaining to any specific dataset or object should belong in the corresponding model. <em>But where does all my other logic go?</em>

Well all other logic can be be created into their own classes and be put either in the `lib` folder or be created as service objects in the `services` folder. So under which condition should we use which?
<br><br>

<strong>Service objects</strong>

Service objects are classes that perform business logic for your application. These classes should be stored in a Services folder in your app directory. An example is retrieving hashtags out of a post body and reembedding them as links in the post creation process. In this case, `EmbedHashtags` should be a service object and in order to initiate it. You would need to call:

{% highlight ruby %}
# this service object lives in app/services/embed_hashtags.rb
EmbedHashtags.new(@post.body).call
{% endhighlight %}
<br>

<strong>Classes in the lib folder</strong>

Whenever a task is generic and can be used independently regardless of the paticular app it's in. It should be stored in the lib folder as a class. A good example is different Oauth login procedures such as classes like `FacebookAuth` or `GithubAuth`. These classes could be exported to other apps and would work just as fine.