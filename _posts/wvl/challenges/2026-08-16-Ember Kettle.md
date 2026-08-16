---
title: Ember Kettle - WVL
date: 2026-08-16
mermaid: true
categories: [WebVerseLabs, "XSS"]
tags: ["XSS","Reflected XSS"]
---

Vamos a ver la aplicación:

![](/assets/images/wvl/challenges/ember_kettle/1.png)

Navegando por la aplicación me encuentro la siguiente funcionalidad:

![](/assets/images/wvl/challenges/ember_kettle/2.png)

Vamos a poner algo al azar para ver como se refleja y en donde se introduce:

![](/assets/images/wvl/challenges/ember_kettle/3.png)

Vamos a ver el DOM a ver donde se refleja:

![](/assets/images/wvl/challenges/ember_kettle/4.png)

Vemos que se refleja en el atributo `value` de la etiqueta `input`, vamos a intentar escapar de ese atributo, para ello usaré este payload simple:

```html
a"abc123
```

![](/assets/images/wvl/challenges/ember_kettle/5.png)

Hemos podido escapar del atributo, ahora tenemos 2 opciones para intentar un XSS:

1. Usar otros atributos para automatizar la ejecución del código JavaScript
2. Salir de la etiqueta `input` y ejecutar código JavaScript

## Opción 1
Para este caso podremos usar la función `autofocus` y el evento `onfocus`, el payload quedaría algo así:

```html
a" autofocus onfocus="alert()
```

> Se añade una comilla doble al principio para que cierre correctamente el atributo y no haya conflictos.
{: .prompt-info }

![](/assets/images/wvl/challenges/ember_kettle/6.png)

## Opción 2
Para salir de la etiqueta necesitaremos salir del atributo y luego cerrar la etiqueta `input`, el payload queda así:

```
"><script>alert()</script>
```

![](/assets/images/wvl/challenges/ember_kettle/7.png)

---