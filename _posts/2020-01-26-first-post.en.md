---
layout: post
permalink: /posts/2020-01-26-first-post.html
title: "First post! Woohoo! Also, how to serve Jekyll in multiple languages"
date: 2020-01-26 21:30:00 -0800
categories: greeting jekyll i18n translation site
lang: en
---

Alright, it's working! It took a bit of effort, mainly because I just don't know a lot of web development and I wanted things to work in a very specific way... but that's the fun of DIY. I might as well share what I learned.

<!--more-->

The main problem was that I wanted my blog to be both in English and Spanish (and maybe French in the future, if I practice enough). I got it working exactly to my liking! Try it yourself, choose your language from the top navigation bar.

## How to add multi-language support

To achieve this, I used the [Polyglot](https://github.com/untra/polyglot) plugin. It changes the way Jekyll builds sites, creating subfolders for each configured language. Any page that's missing a translation uses the default language instead.

The instructions to set it up are available on the GitHub repo, it's really easy to do! The only issue I ran into is that it doesn't work with Jekyll 4.0.0, so I had to downgrade to 3.8.6.

## How to translate the blog theme

Next, we need to update a few parts of the theme we're using. For that, you need to copy over the relevant files from it. You can find them by using the following command:

{% highlight bash%}$ bundle info --path minima
/home/bozz/gems/gems/minima-2.5.1{% endhighlight %}

You'll want to go look at the `_layouts` and `_includes` folders. In particular, I needed the `footer.html` and `header.html` includes, and the `home.html` layout.

Next, choose the text you want to translate, and put them in a `_data/translated_strings.yml` file. Here's a sample of mine:

{% highlight yaml %}
description:
  en: >-
    Just a little blog to put down my thoughts
  es_CL: >-
    Sólo un pequeño blog para escribir mis pensamientos
subscribe:
  en: Subscribe
  es_CL: Suscríbete
via_rss:
  en: via RSS
  es_CL: por RSS
{% endhighlight %}

Then you can use these snippets like so:

{% highlight ruby %}{% raw %}
<p class="rss-subscribe">{{ site.data.translated_strings.subscribe[site.active_lang] }} <a href="{{ "/feed.xml" | relative_url }}">{{ site.data.translated_strings.via_rss[site.active_lang] }}</a></p>
{% endraw %}{% endhighlight %}

## How to allow users to change languages

The final step is allowing users to choose a language. For that, I added a bunch of links to the nav-bar like this:

{% highlight ruby %}{% raw %}
{% for lang_option in site.languages %}
  {% assign lang_icon = site.data.languages[lang_option].icon %}
  {% assign lang_name = site.data.languages[lang_option].name[site.active_lang] %}
  {% if site.default_lang == lang_option %}
    <a class="page-link" href=" {{ page.permalink }}">
  {% else %}
    <a class="page-link" href="{{ site.url }}/{{ lang_option }}{{ page.permalink }}">
  {% endif %}
    {{ lang_icon }} {{ lang_name }} 
  </a>
{% endfor %}
{% endraw %}{% endhighlight %}

There's two interesting things to note here. First, see that dangling space in the anchor href? That's on purpose. It tells Polyglot to use the default language link. Second, there's the `site.data.languages[lang_option]` snippet. That reads the contents of `./_data/languages.yml`:

{% highlight yaml %}
en:
  name:
    en: English
    es_CL: Inglés
  icon: 🇺🇸
es_CL:
  name:
    en: Spanish
    es_CL: Español
  icon: 🇨🇱
{% endhighlight %}

Yes, I used emojis for the country icons. Sue me. I also added individual translations for the name on each language, which is overkill but I wanted the practice.

And that's it for now! I'm kind of excited about having this little pet project, it's kind of fun. I have a bunch of features I want to implement, but I'll tackle those little by little.
