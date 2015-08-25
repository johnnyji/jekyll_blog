---
layout: post
title: 'The Ultimate Sticky Footer!'
date: 2015-08-24
categories: CSS
---

One of the troubles that I'm sure every developer has run into is how to get the perfect sticky footer on your website. I've used alot of different variations but my favourite one by far is one I <strike>came up with</strike> (I'm sure someone did this before I did, and if so I'm sorry but I'm taking your credit!).

This sticky footer implementation will allow:

<blockquote>
  <li>The footer to stick to the bottom of the page</li>
  <li>The footer will extend as the page does</li>
  <li>There will be whitespace between the footer and the page content</li>
  <li>The implementation is simple!</li>
</blockquote>
<br>

Enough talk! Here's how it's done:

{% highlight css %}
/* Our HTML setup

<body>

  <div class='app-wrapper'>
    All of your app contents go here...
  </div>

  <footer>This is our footer!</footer>

</body>

*/


/*Here's the CSS*/

body {
  height: 100%;
  margin: 0;
}

.app-wrapper {
  min-height: 100%; /*page will be minimum 100% of viewport*/
  height: auto; /*page will grow as content expands*/

  /*IMPORTANT: the padding should be 2x the footer's height so the footer stay at the bottom and at the same time still have top room for white space between itself and the app content as the app grows, the padding acts as the space between the content and the footer, as well as the footers alloted height*/
  padding-bottom: 200px; 
}

footer {
  position: relative;
  width: 100%;
  height: 100px;

  margin-top: -100px; /*Because the footer is right below the app-wrapper, which is at it's minimum, the whole page height, we need to make the margin-top the same as the footer's height so that it can shift up by the amount of it's height. Note that this is where the padding: 200px on app-wrapper comes in. After the footer has been shifted up 100px, there will still be white space between the footer and the app content due to the extra padding we gave to the app-wrapper*/
}
{% endhighlight %}
<br>