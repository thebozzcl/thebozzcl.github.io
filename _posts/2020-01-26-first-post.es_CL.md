---
layout: post
permalink: /posts/2020-01-26-first-post.html
title: "¡Primer post, woohoo! Y cómo tener un blog en Jekyll en varios idiomas"
date: 2020-01-26 21:30:00 -0800
tags: [post, coding, blog]
lang: es_CL
---

¡Por fin está funcionando! Me tomó un poco de esfuerzo, más que nada porque no sé mucho de desarrollo web y quería que esto funcionara de una forma muy específica... pero eso es lo entretenido del DIY. Voy a aprovechar de compartir las cosas que aprendí.

<!--more-->

El principal problema que tuve es que quería que mi blog estuviera disponible en Inglés y Español (a lo mejor Francés en el futuro). ¡Lo dejé funcionando exactamente como quería! Pruébalo escogiendo un idioma de la barra superior.

## Cómo añadir soporte para múltiples idiomas

Para añadir soporte de múltiples idiomas, usé el plugin [Polyglot](https://github.com/untra/polyglot). Polyglot cambia la forma en que Jekyll construye los sitios, creando sub-directiorios para cada idioma configurado. Si una página no está traducida, Polyglot usa la versión del idioma por defecto.

Las instrucciones para configurar Polyglot están disponibles en su repositorio de GitHub, ¡es muy fácil de hacer! El único problema que me encontré es que no funciona con Jekyll 4.0.0, así que tuve que bajar la versión a 3.8.6.

## Cómo traducir el tema del blog

El siguiente paso es actualizar algunas partes del tema que estamos usando. Para eso, tienes que copiar los archivos relevantes del tema. Puedes encontarlos usando el siguiente comando:

{% highlight bash%}$ bundle info --path minima
/home/bozz/gems/gems/minima-2.5.1{% endhighlight %}

Vas a querer mirar el contenido de los directorios `_layouts` y `_includes`. En mi caso, necesitaba los includes `footer.html` y `header.html`, y el layout `home.html`.

Luego tienes que escoger el texto que quieres traducir, y ponlas en un nuevo archivo llamado `_data/translated_strings.yml`. Acá hay un ejemplo del mío:

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

Luego puedes usar las líneas traducidas de esta forma:

{% highlight ruby %}{% raw %}
<p class="rss-subscribe">{{ site.data.translated_strings.subscribe[site.active_lang] }} <a href="{{ "/feed.xml" | relative_url }}">{{ site.data.translated_strings.via_rss[site.active_lang] }}</a></p>
{% endraw %}{% endhighlight %}

## Cómo permitir que los usuarios escojan su idioma

El paso final es darle a los usuarios una forma de escoger el idioma. Para eso, le agregué un par de links al nav-bar:

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

Hay dos cosas interesantes que notar. Primero, ¿ves ese espacio al principio del href en el anchor? Eso es a propósito. Le dice a Polyglot que use el link del idioma por defecto. Segundo, tenemos la parte que dice `site.data.languages[lang_option]`. Eso lee los contenidos de `./_data/languages.yml`:

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

Sí, usé emojis para los íconos de los países. ¿Algún problema? También le puse traducciones individuales al nombre de cada idioma. Es innecesario, pero me sirvió de práctica.

¡Y eso es por ahora! Estoy un poco emocionado por este pequeño proyecto, ha sido entretenido. Hay algunas otras funcionalidades que quiero implementar, pero voy a atacarlas de a poco.
