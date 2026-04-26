---
title: TryHeartMe - THM
date: 2026-04-26
mermaid: true
categories: [TryHackMe, Linux, Easy]
image: 
  path: /assets/images/thm/tryheartme/logo.png
tags: [JWT]
---

---

## Enumeration

Vamos a empezar con un escaneo `nmap`:
```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/THM]
└─$ nmap -p- --open -sS -n -Pn 10.66.178.241 -vvv
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-15 00:00 +0100
Initiating SYN Stealth Scan at 00:00
Scanning 10.66.178.241 [65535 ports]
Discovered open port 22/tcp on 10.66.178.241
Discovered open port 5000/tcp on 10.66.178.241
Completed SYN Stealth Scan at 00:01, 31.17s elapsed (65535 total ports)
Nmap scan report for 10.66.178.241
Host is up, received user-set (0.10s latency).
Scanned at 2026-02-15 00:00:42 CET for 31s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 62
5000/tcp open  upnp    syn-ack ttl 62

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 31.26 seconds
           Raw packets sent: 68134 (2.998MB) | Rcvd: 67787 (2.711MB)
```
Vamos a realizar un segundo escaneo para obtener información más detallada:
```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/THM]
└─$ nmap -p22,5000 -sCV 10.66.178.241
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-15 00:01 +0100
Nmap scan report for 10.66.178.241
Host is up (0.10s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 82:df:f4:71:a9:25:07:c5:ff:4b:f7:15:2a:05:60:e8 (ECDSA)
|_  256 fa:8a:b6:df:17:98:98:85:fa:fd:0a:28:06:0d:f5:aa (ED25519)
5000/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
|_http-title: TryHeartMe \xE2\x80\x94 Shop
|_http-server-header: Werkzeug/3.0.1 Python/3.12.3
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.91 seconds
```

| PUERTO     | DESCRIPCIÓN            |
| ---------- | ---------------------- |
| `22/TCP`   | `OpenSSH 9.6p1`        |
| `5000/TCP` | `Werkzeug httpd 3.0.1` |

Vamos a ver la aplicación web alojada en el puerto `5000`:

![](/assets/images/thm/tryheartme/1.png)

### Directory & Files Fuzzing

Voy a emplear `ffuf`:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/THM]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u "http://10.66.178.241:5000/FUZZ" -e .php,.html,.css,.js,.txt -c

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.66.178.241:5000/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Extensions       : .php .html .css .js .txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 3351, Words: 617, Lines: 101, Duration: 130ms]
login                   [Status: 200, Size: 1461, Words: 210, Lines: 49, Duration: 126ms]
register                [Status: 200, Size: 1517, Words: 220, Lines: 49, Duration: 105ms]
admin                   [Status: 302, Size: 223, Words: 18, Lines: 6, Duration: 106ms]
account                 [Status: 302, Size: 227, Words: 18, Lines: 6, Duration: 105ms]
logout                  [Status: 302, Size: 189, Words: 18, Lines: 6, Duration: 104ms]
```

### Website - TCP 5000
Viendo la aplicación podemos iniciar sesión:

![](/assets/images/thm/tryheartme/2.png)

Probando credenciales predeterminadas ninguna con éxito...
![](/assets/images/thm/tryheartme/3.png)

Vamos a crearnos una cuenta:

![](/assets/images/thm/tryheartme/4.png)
![](/assets/images/thm/tryheartme/5.png)

Si vamos a la sección `Account` podremos ver lo siguiente:

![](/assets/images/thm/tryheartme/6.png)

Vemos que tenemos unos "créditos". Vamos a investigar la tienda:

![](/assets/images/thm/tryheartme/7.png)

Vemos que hay varios productos, vamos a intentar comprar alguno a pesar de no tener créditos:

![](/assets/images/thm/tryheartme/8.png)

Vamos a ver como se tramita:

![](/assets/images/thm/tryheartme/9.png)

#### JWT

Vemos que se tramita por el método `POST`, en la solicitud de compra vemos que usa un JWT *(JSON Web Token)*. Vamos a descodificar el JWT para ver que datos indica:

![](/assets/images/thm/tryheartme/10.png)

Vemos cosas interesantes, vamos a probar a interceptar la compra y modificar el JWT para indicar como que tuviéramos 1000 créditos:

![](/assets/images/thm/tryheartme/11.png)

![](/assets/images/thm/tryheartme/12.png)

![](/assets/images/thm/tryheartme/13.png)

Vemos que hemos podido realizar la compra modificando la cantidad de puntos en el JWT!! Vimos también que tenemos el `rol`. En el fuzzing encontramos el directorio `/admin`:

![](/assets/images/thm/tryheartme/14.png)

Vamos a interceptar esta solicitud cambiando el valor del JWT en `role`:

![](/assets/images/thm/tryheartme/15.png)

![](/assets/images/thm/tryheartme/16.png)

![](/assets/images/thm/tryheartme/17.png)

Lo logramos!! Estamos en el Admin Portal. Vamos a darle `Open ValenFlag` pero vamos a interceptar y añadir otra ver la JWT con el `role` de `admin`:

![](/assets/images/thm/tryheartme/18.png)

Vamos a darle a comprar:

![](/assets/images/thm/tryheartme/19.png)

---