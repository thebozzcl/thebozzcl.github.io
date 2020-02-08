---
layout: post
permalink: /posts/2020-02-03-added-comments.html
title: "Now I have comments!"
date: 2020-02-03 22:30:00 -0800
tags: [post, coding, blog]
lang: en
---

It's been a busy week, between going back to work and my other personal project (expect posts about it soon!). I still found some time to get some work on my blog done, and now I have comments enabled! I'm gonna go on and blabber about the process, as usual.

<!--more-->

When writing this blog, I stick to three design principles:
* Keep it light and simple
* Keep it cheap
* Do cool stuff anyway

From the start, I knew comments would complicate things a bit. You need to store comments somewhere! You need to save them, and load them and show them! That's a load of stuff to do, it's not simple! You can always use a third-party service for comments, but that's not cheap! I decided to take some time and look at my options carefully. I ended choosing [FastComments](https://fastcomments.com/).

## So why FastComments?

FastComment's tagline is "At 7.27 kB with no dependencies, it's the fastest comment service around". That covers the simple and light part. How about cheap? Well, their cheapest tier costs $5.00 with a load limit of "just" 1 million page loads per month, so I'll manage. A self-hosted solution in the cloud might be cheaper, to be honest, but that's more work than I'm willing to do. All in all, FastComments seemed worth a try.

## How to set it up

After creating an account, setup is very easy: you just need to paste their code snippet into your site. I did it the Jekyll way and created a new include for comments, and then used that in the site layouts. Then I customized the snippet slightly:

{% highlight ruby %}{% raw %}
<script src="https://cdn.fastcomments.com/js/embed.min.js"></script>
<div id="fastcomments-widget"></div>
<script>
  window.FastCommentsUI(document.getElementById('fastcomments-widget'), {
    tenantId: "{{ site.fastcomments.tenantId }}",
    urlId: "{{ site.url }}{{ page.permalink | post.permalink }}",
    commentCountFormat: "[count] comments on {{ page.title | post.title }}",
    headerHTML: "<h1>{{ site.data.translated_strings.leave_a_comment[site.active_lang] }}</h1>",
  });
</script>
{% endraw %}{% endhighlight %}

The first field, `tenantId`, is just the customer identifier.

FastComments uses the `urlId` field to aggregate comments. If left unset, it defaults to the URL of the current page. Instead, I hard-coded the default language URL for the page so the comments section is shared across all languages!

The last two fields, `commentCountFormat` and `headerHTML`, allow you to override some of the strings in the comments widget. I used this, plus the translation scheme I set up in [my previous post](/posts/2020-01-26-first-post.html), to partially translate it!

## Thoughts and feature wishlist

FastComments is a very new service, so it's still a WIP. There's a bunch of improvements I'd like to see:
* Better localization or string overrides: ~~I talked to their support team, and they're planning to improve this in the next few weeks.~~ **UPDATE 02/08/2020**: localization is now supported either by auto-detect or by passing the `locale` attribute to the widget. It supports `en_us`, `es_es` and `fr_fr`, which should do for my purposes!
* Better documentation: I struggled to get things working my way a little bit because the documentation for the code snippet is lacking. I had to search through the FastComments blog to [figure out the available arguments](https://blog.fastcomments.com/(1-24-2020)-how-to-make-a-comment-system-like-hackaday.com.html). I'd prefer dedicated documentation.

So while it's pretty bare-bones right now, FastComments matches my design philosophy well and it's very cheap. I can't confirm them being the lightest or the fastest, but it's more than good enough for what I wanted. I'm pretty happy so far, and I'm looking forward to what they offer in the future.

## Runner-up comments system: can you do comments with just a static site and nothing else?

You can! If we kept everything that way, the only way to add comments is by changing the code of my site. This is exactly what [Staticman](https://staticman.net/) does: it pushes a change to the code repository for each comment. It's such a delightfully odd solution, and it works pretty well! There's a catch, though: it takes a while for comments to show up because the site needs to be rebuilt each time. I opted to keep the site responsive and do it some other way.
