---
published: true
title: Typed vs Untyped languages
layout: post
---

Today my thoughts are for typed versus untyped languages. That said, I know that untyped languages has many advantages, but I really hate to be able to set potatoes to onions. 
Look at this sample code on Javascript:

{% highlight javascript %}
    
    var onion = { name: 'onion', numberOfLayers: 5 };
    document.write("<p>" + onion.name + "</p>");
    
    onion = { name: 'potato', isFried: false };
    document.write("<p>" + onion.name + "</p>");
    
    onion = 0;
    document.write("<p>" + onion + "</p>");
    
{% endhighlight %}
[Live Feedle](http://jsfiddle.net/kappy/dLYYd/)

This is a perfectly valid code. The most obvious matter on this type of assignments is that, we’re assigning completely different things to the same variable witch causes the variable to lose all its meaning. Meaningful variables are the number one rule thumb on my book of trades. This is also one of the most common issues on code that I review.

This is why I love typed languages:
![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/typeLanguage_zps682579fc.png)

Can you see that thin red line? That's the IDE telling you: You're doing it wrong!