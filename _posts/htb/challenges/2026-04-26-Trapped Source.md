---
title: Trapped Source - HTB Challenge
date: 2026-04-26
mermaid: true
categories: [HackTheBox, Challenges, Very Easy]
tags: []
---

Si vemos el Debugger:

![](/assets/images/htb/challenges/trappedsource/1.png)

Vemos que hay un endpoint llamado `flag`, vamos a ver que hay:

![](/assets/images/htb/challenges/trappedsource/2.png)

Vemos que solo puede enviarse una solicitud por el método POST, vamos a enviarla:

![](/assets/images/htb/challenges/trappedsource/3.png)

Vemos que requiere un JSON, leyendo el código fuente se ve que quiere que indiquemos el `pin`, vamos a ello:

![](/assets/images/htb/challenges/trappedsource/4.png)

---