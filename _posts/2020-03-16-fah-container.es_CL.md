---
layout: post
permalink: /posts/2020-03-16-fah-container.html
title: "Cómo correr Folding@Home con soporte para CUDA en un contenedor de Docker"
date: 2020-03-16 20:20:00 -0800
tags: [post, coding, blog]
lang: es_CL
---

Es fácil sentirse impotente ante crisis globales como la que estamos viviendo hoy. Quedarse encerrado, esperando a que el coronavirus se termine... ¿de verdad no hay nada que podamos hacer? Pues no, hay algunas formas en que podemos ayudar. Un ejemplo es [Folding@Home](https://foldingathome.org/). F@H es una iniciativa que usa los recursos de tu computador para simular cómo funcionan las proteínas y otros químicos complejos. Ahora mismo, han publicado [órdenes de trabajo de alta prioridad para analizar el virus 2019-nCoV](ttps://foldingathome.org/2020/02/27/foldinghome-takes-up-the-fight-against-covid-19-2019-ncov/), con la esperanza de ayudar a encontrar anticuerpos compatibles.

Si quieres ayudar, es tan fácil como [descargar y ejecutar el programa de F@H](https://foldingathome.org/start-folding/). Ahí es donde me quedé atascado: no pude instalar el programa en Ubuntu. Decidí tratar de resolverlo por mi cuenta.

<!--more-->

## ¿Cuál es el problema?

Estoy usando [Pop!\_OS 19.10](https://system76.com/pop), una distribución de Linux basada en Ubuntu. Cuando intenté instalar los paquetes de F@H, `apt` reclamó que no podía resolver `python-gnome2` y otras dependencias relacionadas.

## Primer intento: forzar la instalación

Encontré algunas guías que intentan descargar e instalar todas las dependencias faltantes a la fuerza. Tienes que bajar los DEBs e instalarlos tú mismo, y luego arreglar todos los errores emergentes.

No me sorprendió mucho que esto no funcionara. Las instalaciones fallaron, hubo conflictos de archivos entre paquetes... la verdad es que me dio lata siquiera tratar de arreglarlo.

## Segundo intento: usar un contenedor de Docker

Hablando con algunos amigos en medios sociales sobre mi problema, uno sugerió una idea genial: ¿por qué no usar un contenedor de Docker? Hay un montón de contenedores pre-fabricados que ya contienen todas las dependencias necesarias. Casi seguro alguien ya lo ha hecho.

Efectivamente, [encontré varias alternativas](https://hub.docker.com/search?q=folding-at-home&type=image). Decidí usar el contenedor creado por [yurinnick](https://hub.docker.com/r/yurinnick/folding-at-home), que fue actualizado hace poco tiempo. El contenedor además es liviano y simple.

Lamentablemte, algo más faltó. El contenedor corrió sin problemas, pero no pudo detectar mi tarjeta de video y en consecuencia usar CUDA u OpenCL. Las órdenes de trabajo de F@H funcionan mejor en tarjetas de video porque son buenas para paralelizar masivamente pequeñas unidades de cómputo.

## Tercer intento: arreglar el soporte para CUDA y OpenCL en el contenedor

Afortunadamente, tengo algo de experiencia trabajando con contenedores de nVidia gracias a mi tiempo en AWS SageMaker. No mucho, siendo honesto, pero lo suficiente como para resolver esto por mi cuenta.

Estos fueron los pasos que seguí:
1. Actualizar los drivers de la tarjeta de video.
2. Opcional, pero podría ser útil para hacer debugging: instalar las [herramientas para CUDA de nVidia](https://developer.nvidia.com/cuda-downloads). Cuando está instalado correctamente, deberías poder ejecutar `nvidia-smi` para ver la versión actual de CUDA en el sistema.
3. Instalar las [herramientas para contenedores de nVidia](https://github.com/NVIDIA/nvidia-docker). Esto permite que tus contenedores accedan a la tarjeta de video, así que es vital. Puedes confirmar si funciona corriendo `docker run --gpus all nvidia/cuda:10.0-base nvidia-smi`. Si la información de sistema que el contenedor puede ver es correcta, todo está en orden.
4. Finalmente, creé un fork del [repositorio de yurinnick](https://github.com/yurinnick/folding-at-home-docker/) y cambié la base del `Dockerfile` a `nvidia/opencl`. Este contenedor tiene todas las dependencias que F@H necesita para usar la tarjeta de video. He publicado mi resultado final a [este repositorio de Docker Hub](https://hub.docker.com/r/thebozzcl/folding-at-home).

Con estas preparaciones listas, y tras haber arreglado la imagen del contenedor, pude ejecutar órdenes de trabajo en la GPU usando el siguiente comando:

```
docker run --gpus all \
  --name folding-at-home \
  -p 7396:7396 \
  -p 36330:36330 \
  -e USER=Anonymous \
  -e TEAM=0 \
  -e ENABLE_GPU=true \
  -e ENABLE_SMP=true \
  --restart unless-stopped \
  thebozzcl/folding-at-home \
  --allow 0/0 \
  --web-allow 0/0
```

## Ejecutando el contenedor en otras configuraciones

No he hecho muchas pruebas, sólo en mi computador, así que no puedo comentar mucho. Hay un par de cosas de las que estoy seguro:
* Este contenedor debería funcionar sin problemas en otros computadores con una GPU de nVidia, siempre y cuando las herramientas de nVidia estén instaladas.
* Este contenedor también va a funcionar en computadores sin una tarjeta de video dedicada... pero es mucho más pesado que otros contenedores que funcionarían igual de bien.
* No sé si este contenedor funcionará en un computador con una tarjeta de video AMD. No sé cuál es el proceso para hacerlas funcionar... pero si lo logras, ¡deja un comentario!

## Siguientes pasos... o mejor no

Lamentablemente, no quiero compromenterme con este proyecto a largo plazo. Estaría dispuesto a arreglar pequeños problemas y revisar merge requests, pero estoy tapado en obligaciones así como estoy y tengo poco tiempo para dedicarle.

Como sea, este contenedor es tremendamente simple y en buena parte no es mi trabajo... así que siéntete libre de crear un fork y cambiarlo a gusto!
