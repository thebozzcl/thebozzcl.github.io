---
layout: post
permalink: /posts/2020-02-03-added-comments.html
title: "¡Ahora tengo comentarios!"
date: 2020-02-03 22:30:00 -0800
categories: site comments fastcomments
lang: es_CL
---

Ha sido una semana ocupada, entre volver al trabajo y mi otro proyecto personal (¡se vienen posts pronto!). Aun así me tomé el tiempo de trabajar un poco en mi blog, ¡y ahora tengo comentarios! Como siempre, voy a chacharear sobre el proceso.

<!--more-->

Cuando escribo este blog, mantengo tres reglas en mente:
* Que sea liviano y simple
* Que sea barato
* Que haga cosas entretenidas igual

Supe desde el comienzo que agregar comentarios me iba a complicar un poco. ¡Tienes que guardar los cometarios, cargarlos y mostrarlos! ¡Tienes que tener un lugar donde ponerlos! Es harto trabajo, ¡no es fácil! Puedes usar un servicio externo, ¡pero eso no es barato! Decidí tomármelo con calma y evaluar mis opciones con cuidado. Finalmente, escogí [FastComments](https://fastcomments.com/).

## ¿Por qué FastComments?

El slogan de FastComments es "Con 7,27 KB y sin dependencias, es el servicio de comentarios más rápido disponible". Eso cubre el requisito de simpleza y ligereza. ¿Qué tal el precio? Su subscripción más barata cuesta USD$5,00 con un límite de "sólo" 1 millón de cargas al mes, así que creo que voy a estar bien. Una solución hosteada por mí en la nube hubiera sido más barato... pero es más trabajo del que estoy dispuesto a hacer. Considerando todo esto, probar FastComments no parece una mala idea.

## Cómo configurarlo

Después de crear una cuenta, configurarlo es bien fácil: sólo tienes que copiar su snippet de código en tu sitio. Yo lo hice a la manera de Jekyll, creando un nuevo include para comentarios, y luego usándolo en los layouts del sitio. Luego personalicé el snippet un poco:

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

El primer atributo, `tenantId`, es sólo el identificador del cliente.

FastComments usa el atributo `urlId` para agregar comentarios. Si no lo configuras, usa la URL de tu sitio por defecto. En vez de eso, hard-codée la URL del idioma por defecto. ¡Todas las versiones de un post comparten los mismos comentarios!

Los últimos dos atributos, `commentCountFormat` y `headerHTML`, te permiten cambiar algunos de los strings en el widget de comentarios. ¡Usé esto, más el esquema de traducciones que configuré en [mi post anterior](/posts/2020-01-26-first-post.html), para traducirlo parcialmente!

## Comentarios y wishlist

FastComments es un servicio muy nuevo, así que todavía está en progreso. Hay algunas mejoras que me gustaría ver:
* Mejor localización o remplazo de strings: hablé con el equipo de soporte, y quieren lanzar una mejora en las próximas semanas.
* Mejor documentación: me costó un poco hacer esto funcionar a mi manera porque la documentación del snippet no es muy buena. Tuve que buscar en el blog de FastComments hasta que encontré [una lista completa de argumentos](https://blog.fastcomments.com/(1-24-2020)-how-to-make-a-comment-system-like-hackaday.com.html). Preferiría documentación dedicada.

Aun si es un poco limitado ahora mismo, FastComments calza bien con mi filosofía de diseño y es muy barato. No puedo confirmar si es el servicio más liviano o rápido, pero los resultados son suficientemente buenos para mí. Estoy contento por ahora, y espero con ganas lo que ofrezcan en el futuro.

## Segundo finalista: ¿puedes tener comentarios con sólo un sitio estático, y nada más?

¡Resulta que sí! Si mantuviéramos el sitio tal como está, la única forma de agregar comentarios es cambiando el código del sitio. Esto es exactamente lo que hace [Staticman](https://staticman.net/): empuja un cambio al repositorio del código para cada comentario. ¡Es una solución tan bizarra, pero que funciona tan bien! Tien un problema, eso sí: como cada comentario necesita una reconstrucción del sitio para mostrarse, los comentarios toman mucho tiempo en publicarse. Decidí mantener el sitio más dinámico y optar por otra solución.