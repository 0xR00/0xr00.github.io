---
title: OpenSecret - HTB Challenge
date: 2026-06-12
mermaid: true
categories: [HackTheBox, Challenges, Very Easy]
tags: []
---

Vamos a acceder a la aplicación web:

![](/assets/images/htb/challenges/opensecret/1.png)

Si rellenamos el formulario vemos que se tramita una petición por el método `POST`. Donde el servidor nos responde indicando que no hemos indicando nuestro token de sesión:

![](/assets/images/htb/challenges/opensecret/2.png)

Si vamos a `Debugger` y vemos que se estár renderizando de código JavaScript y lo vamos leyendo encontraremos lo siguiente:

![](/assets/images/htb/challenges/opensecret/3.png)

-- -
