---
layout: post
permalink: /posts/2020-05-19-pihole-wireguard.html
title: "Cómo usar WireGuard para llevar tu Pi-hole a todos lados"
date: 2020-05-19 19:00:00 -0800
tags: [post, coding, privacy]
lang: es_CL
---

Llevo varios años usando [Pi-hole](https://pi-hole.net/). Es una herramienta fantásica para mejorar tu experiencia en linea y proteger tu privacidad... excepto por un problema: no es portable... o normalmente no lo es. ¡Arreglemos eso con una VPN casera!

> :warning: **Actualización, 2021/03/11**: En la guía original, me faltaron algunos pasos para configurar Dynamic DNS. Ahora sí debería estar completo.

<!--more-->

## ¿Por qué querría hacer esto?

Personalmente, lo que quiero es bloquear publicidad. Si me conoces, sabes que desprecio la publicidad en linea y los trackers de actividad. Tomo medidas extremas para bloquearlos, al punto que casi no hay publicidad en mi vida. Usar Pi-hole desde cualquier lado es una herramienta fantástica.

Si eso no es suficiente, hay otras razones que te pueden interesar:
* Acceder a tu red casera remotamente
* Compartir un servidor casero con amigos
* Proteger tu privacidad en conexiones desconocidas

Es muy fácil de implementar y completamente gratis fuera del hardware y un dominio.

## Requisitos

Para partir, vas a necesitar algunas cosas:
* Una Raspberry Pi. Estoy usando una [RPi 4 modelo B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/), aunque creo que modelos más viejos también deberían servir.
* Un nombre de dominio.
* Un router que soporte port forwarding (creo que la mayoría puede, hoy en día).
* Una cuenta en un hosting de DNS que soporte Dynamic DNS.
  * Yo uso [Hurricane Electric](https://dns.he.net/). No dejes que la página anticuada te engañe... funciona bien, y es completamente gratis (al menos para lo que necesitamos en este proyecto).
* La dirección IP pública de tu router.

## Primero: configurar tu dominio de WireGuard

Cuando tengas tu dominio y te hayas registrado en un servicio de DNS, tienes que:
1. Agregar tu dominio al servicio de DNS, siguiendo todas las instrucciones que te den. Cuando estés listo, deberías tener al menos los siguientes registros:
   * SOA (start of authority)
   * NS (nameserver)
2. Con esto listo, agrega un registro A (mapping de dominio a dirección IP) con esta configuración:
   * Nombre: tu dominio... o un subdominio, si quieres usar el dominio principal para otra cosas
   * Dirección IP: tu dirección IP pública
   * TTL: no más de 300 (5 minutos)
   * Dynamic DNS habilitado. Esto va a permitir que tu VPN funcione incluso si tu dirección IP pública cambia.
3. Crea una contraseña para tu registro de Dynamic DNS, y anótala. La necesitamos para que tu Raspberry Pi o router pueda actualizar el registro A.
4. Configura las actualizaciones de Dynamic DNS:
   * Si quieres hacer esto desde tu Raspberry Pi, puedes usar `ddclient`.
   * Yo lo hice desde mi router (que probablemente también usa `ddclient` por debajo).
   * Para Hurricane Electric, ésta es la configuración que usé:
     * Proveedor: custom
     * Servidor de Dynamic DNS: `dyn.dns.he.net`
     * Request: `/nic/update`
     * Dominio: tu dominio o subdominio
     * Usuario: tu dominio o subdominio
     * Contraseña: la que creaste en el paso `3.`

Esto te va a permitir conectarte a tu router remotamente. **¡Asegúrate de desactivar el acceso a la UI de tu router desde la WAN!**

## Luego: configurar Pi-hole

**NOTA**: casi todas estas instrucciones son links a páginas de fabricantes o del software involucrado. Si cualquiera de estos links se rompe, házmelo saber y los arreglo apenas pueda.

¿Tienes tu Raspberry Pi lista? Configurar Pi-hole y WireGuard es muy fácil:
1. [Instala tu sistema operativo](https://www.raspberrypi.org/documentation/installation/installing-images/). Te recomiendo que [habilites SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/) para seguir la instalación.
2. Conéctate a tu Raspberry Pi via SSH.
3. En la configuración de DHCP de tu router, dale una dirección IP estática a tu Raspberry Pi. Anótala.
   * Reinicia la Raspberry Pi (o al menos su servicio de redes) para aplicar la nueva IP.
4. [Instala Pi-hole](https://github.com/pi-hole/pi-hole/#one-step-automated-install).
   * Te recomiendo la instalación automatizada: `curl -sSL https://install.pi-hole.net | bash`
   * Durante la instalación, vas a poder escoger tus DNS de respaldo. Algunas de las opciones que te dan no son buenas para la privacidad, así que ten cuidado.
   * Durante la instalación, te van a ofrecer configurar la dirección IP actual como estática. Hazlo.
5. Cuando hayas confirmado que tu Pi-hole está funcionando (prueba abriendo el dashboard en http://pi.hole/admin), configura tu router para que use tu Raspberry Pi como su *único* servidor de DNS.
   * Si quieres, puedes agregar un DNS secundario por si tu Raspberry Pi falla.

Listo, ahora los dispositivos en tu red casera pueden usar Pi-hole para bloquear publicidad.

## Finalmente: configurar WireGuard

Por supuesto que esto no se puede usar remotamente, así que tenemos que instalar WireGuard:
1. [Instala PiVPN](https://www.pivpn.io/)
   * PiVPN también tiene un instalador: `curl -L https://install.pivpn.io | bash`
   * Durante la instalación, escoge WireGuard como tu protocolo de VPN. Puedes usar cualquier puerto, pero toma nota para después.
   * Cuando el instalador te pida que escojas entre una IP pública y un DNS público, escoge el DNS. En la pantalla siguiene, escribe el nombre del (sub)dominio que configuraste antes.
   * El instalador debería detectar automáticamente tu instalación de Pi-hole, y preguntarte si quieres usarla en tu VPN. ¡Acepta!
2. En tu router, habilita port forwarding hacia tu Raspberry Pi para el puerto de WireGuard. Por defecto, es `51820`.

¡Y eso es todo! Ahora sólo tienes que probar tu configuración de WireGuard. Puedes usar tu smartphone:
1. En tu Raspberry Pi, crea un nuevo perfil de usuario con el comando `pivpn -a`.
2. Cuando termines, puedes copiar el perfil a otros dispositivos de dos formas:
 * Copiando el archivo de configuración.
 * **Recomendado**: usando un código QR generado por el comando `pivpn -qr`. Vamos a usar este método en los pasos siguientes.
3. Toma tu teléfono e instala el cliente de WireGuard:
 * [Android](https://play.google.com/store/apps/details?id=com.wireguard.android&hl=en_US)
 * [iOS](https://apps.apple.com/us/app/wireguard/id1441195209)
4. Abre la aplicación y agrega un nuevo túnel. Si seguiste las instrucciones, sólo tienes que apuntar la cámara de tu smartphone hacia el código QR y la aplicación se encargará del resto.

Para probarlo, desconecta tu teléfono de tu red WiFi, y trata de abrir http://pi.hole/ de nuevo. Si lo configuraste bien, deberías poder abrir el dashboard sin problemas.

## ¿Y si no quiero hacer todo esto?

O sea... sigo creyendo que la guía entera es fácil de seguir, pero está bien. Hay servicios de VPN pagados que ofrecen DNS anti-publicidad, como [Windscribe](https://windscribe.com/). Otra opción es usar una VPN local portable, como [DNS66](https://f-droid.org/en/packages/org.jak_linux.dns66/). Hay otras opciones, pero éstas son las que he usado.

Ambas funcionan bien, pero tienen sus desventajas:
* Sólo pueden funcionar en un número limitado de dispositivos a la vez (puedes configurar tu router para conectarlo a tu VPN, pero va a ralentizar tu conexión).
* En el caso de la VPN, estás confiando que el proveedor no va a registrar, compartir o vender tu actividad. Esto es un riesgo que yo no estaba dispuesto a tomar.

No importa qué opción escojas, la internet cambia drásticamente cuando eliminas la mayoría del ruido. ¡Disfruta!

## Problemas comunes

Éstos son algunos de los problemas que he tenido al re-configurar my VPN de WireGuard:

* Asegúrate que tu Dynamic DNS esté actualizándose correctamente. Esto podría ser por una contraseña incorrecta o por la configuración de Dynamic DNS.
* Asegúrate que el port forwarding esté configurado correctamente. No tienes idea de cuántas veces usé el puerto equivocado.
* Si no puedes acceder a los dispositivos de tu red remotamente, prueba lo siguiente:
  1. Abre tu cliente de WireGuard en tu dispositivo.
  2. Edita el túnel de WireGuard.
  3. En "IPs Permitidas", agrega el rango de IPs que corresponda a la red de tu casa. Algunos valores típicos son `192.168.0.0/24` y `192.168.1.0/24`.
  4. Guarda los cambios e intenta de nuevo.