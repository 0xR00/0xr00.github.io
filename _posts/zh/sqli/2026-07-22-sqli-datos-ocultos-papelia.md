---
title: Recupera los libros ocultos del catálogo de Papelia - ZH
date: 2026-07-22
mermaid: true
categories: [ZHacksLabs, "SQL Injection"]
tags: []
---

Vamos a ver la aplicación:

![](/assets/images/zh/sqli/sqli-datos-ocultos-papelia/1.png)

Vemos que se trata de una librería, donde tenemos varios botones para filtrar entre lirbos si le damos a uno:

![](/assets/images/zh/sqli/sqli-datos-ocultos-papelia/2.png)

Vemos que se está tramitando un parámetro por el método `GET` llamado `category`. Podemos pensar que esto posiblemente se esté enviando a una consulta SQL como la siguiente:

```sql
SELECT * FROM libros WHERE category = 'NUESTRO_INPUT'
```

Vamos a probar a inyectar una `'` para ver si ocasionamos un fallo en la consulta:

![](/assets/images/zh/sqli/sqli-datos-ocultos-papelia/3.png)

Vemos que si estamos pudiendo afectar a la query, donde desvela un poco más de la consulta que se realiza donde nos da que finaliza con `AND published = 1` entonces en este caso para hacer un volcado de todos los datos de la tabla podremos usar `' or 1=1--` comentando el `AND published = 1` para así que nos muestre todos los libros en este caso:

![](/assets/images/zh/sqli/sqli-datos-ocultos-papelia/4.png)

---