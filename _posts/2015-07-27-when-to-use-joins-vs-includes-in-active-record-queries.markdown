---
layout: post
title: "When to use Joins vs. Includes in ActiveRecord Queries"
date: 2015-07-27
categories: ActiveRecord
---

The difference between `includes` and `joins` has always eluded me a little bit when it came to ActiveRecord queries. I knew that `includes` performs a `LEFT OUTER JOIN` whereas `joins` performs an `INNER JOIN`, but the real difference between them comes into play depending on the type of data you need to retrieve.

If you need to access a table and it's association, it's better to use `include`, such as:

{% highlight ruby %}
@result = Post.includes(:tags, comments: [:user, :tags]).find(3)

# This query will return the post with an id of 3, and it's associations which include it's tags, comments, each comment's user and tags. This query preloads all the information.

@result.user #=> this will load the user info without another additional query
{% endhighlight %}
<br>

However, if you use a `joins` method, this will only create the association between the two tables but it won't actually get the data unless you manually specify through a `select`. Therefore, when you use `joins` and you try to access attributes on the joined table, you are still making excessive queries that are not necessary.

{% highlight ruby %}
@result = Post.joins(:tags, comments: [:user, :tags]).find(3)

# This will make the association between all the tables, but it will not preload any information.

@result.user #=> this will be a seperate SQL query.
{% endhighlight %}
<br>

`joins` do have their handiness. For example, if you ever need to select only some columsn from a table, a join is preferred because `includes` will not allow you to specify the attributes you select.

{% highlight ruby %}
@result = Post.joins(:comments).select("comments.title AS comment_title").find(3)

# This will only select the comment titles for this post.
{% endhighlight %}
<br>