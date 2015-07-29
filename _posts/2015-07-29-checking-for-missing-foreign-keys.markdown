---
layout: post
title: "Checking for Missing Foreign Keys"
date: 2015-07-29
categories: Rails
---

Recently I've been getting into optimizing database structure and queries for performance in Rails. One major thing that's gone over my head was add indexes.

Thankfully I found an amazing piece of script at [Tom Ward's][tomward] site. Just run `spring rails c` and paste the following code into the console. This will output every place where there should be an index on a foreign key, but isnt.

{% highlight ruby %}
c = ActiveRecord::Base.connection
c.tables.collect do |t|  
  columns = c.columns(t).collect(&:name).select {|x| x.ends_with?("_id" || x.ends_with("_type"))}
  indexed_columns = c.indexes(t).collect(&:columns).flatten.uniq
  unindexed = columns - indexed_columns
  unless unindexed.empty?
    puts "#{t}: #{unindexed.join(", ")}"
  end
end
{% endhighlight %}

[tomward] https://tomafro.net/2009/09/quickly-list-missing-foreign-key-indexes