---
title: XXE en la subida de avatar (Portra)
date: 2026-10-21
mermaid: true
categories: [ZHacksLabs, "XML External Entity"]
tags: ["XXE","File upload"]
---

---

Vamos a ver la aplicación web:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/1.png)

Si vamos a `Entrar` veremos un formulario de inicio de sesión donde nos darán las siguientes credenciales:

- `m.rivas`:`Portra-2026`

Vamos a iniciar sesión:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/2.png)

Si investigamos la aplicación web podremos encontrar que nos ofrecen la funcionalidad de subir una imagen como foto de perfil:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/3.png)

Como primeras vamos a intentar subir una imagen `.svg` para ver si la procesa y la acepta:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/4.png)

Vemos que no lo permite. Vamos a subir un `.png` normal a ver cual es el proceso:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/5.png)

Vemos que un `.png` si lo acepta y procesa. Vamos a enviar esta solicitud al `Repeater` e intentar modificar la extensión a `.svg` para ver lo acepta:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/6.png)

Vemos que no lo bloquea, esto puede ser debido a:

1. Content-Type
2. Magic Bytes

Ahora vamos a intentar insertar el contenido de un SVG verídico e intentar meter en él el siguiente payload XXE:

*Payload:*

```xml
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd">]>
```

Y para mostrarlo en el contenido del SVG:
```xml
<text x="0" y="16" witdh="100">&data;</text>
```

Vamos a probarlo:

![](/assets/images/zh/xxe/xxe-en-la-subida-de-avatar-portra/7.png)

Vemos que la funcionalidad de foto de perfil es vulnerable a un XXE via SVG file upload.

---