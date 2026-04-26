---
title: Phantom Script - HTB Challenge
date: 2026-04-26
mermaid: true
categories: [HackTheBox, Challenges, Very Easy]
tags: [XSS]
---

Vamos a acceder a la aplicación web:

![](/assets/images/htb/challenges/phantomscript/1.png)

Vemos una zona de búsqueda vamos a probarlo:

![](/assets/images/htb/challenges/phantomscript/2.png)

Vemos que nuestro input se refleja, vamos a probar un HTML Injection:

![](/assets/images/htb/challenges/phantomscript/3.png)

Vemos que es vulnerable a un HTML Injection, vamos a probar un XSS:

![](/assets/images/htb/challenges/phantomscript/4.png)

Después de la pestaña de alerta:

![](/assets/images/htb/challenges/phantomscript/5.png)

---