---
layout: post
title: "Lighthouse Labs W1D3: The Power of Symbols"
date: 2015-04-01
categories: Ruby
---

After programming in Ruby for about 2 good months I am ashamed to say that I had no idea just how powerful symbols were!

When you create a string called `"Johnny"`, it'll have an `object_id` of <strong>1</strong>. However once I create another identical string called `"Johnny"`, that second string will have an `object_id` of <strong>2</strong>. 

This happens because no matter if the two strings are identical or not, they're both recognized in the project as two different objects, causing them to have different Object IDs.

However, when you create a symbol called `:johnny`, i'll have an `object_id` of <strong>1</strong>. Yet, anywhere else in the program if I write another `:johnny` symbol, it'll also have an `object_id` of <strong>1</strong>. Each symbol is UNIQUE and cannot be changed.
<br><br>

<h3><strong>So why should you care?</strong></h3>

1. This saves ALOT of memory. By using symbols in hashes as keys instead of strings, you're not having to set a new object every time you change a key-pair value.

2. Symbols can act as ID holders. Because symbols with the same name have the same `object_id`, you could use symbols to match truthiness. For example. When users input their hometown, I can call `to_sym` on that string. This would change `"vancouver"` to `:vancouver`. Then I can call `Users.all(where hometown: :vancouver)`. Even though this symbol holds no real value, it is finding all the users that contains it's object_id.

3. Symbols themselves only contain a name and an Object ID and holds no real value unless assigned a value as a key-value pair in a hash. Therefore setting a variable such as `city = :vancouver` could mean that :vancouver is acting as an ID holder to match somewhere else OR `:vancouver` is actually the key to a value set elsewhere in your program!
<br><br><br>

Symbols are fucking amazing. I'm going to abuse their power for my own gain from now on and so should you.

