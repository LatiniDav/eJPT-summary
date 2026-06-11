---
icon: ubuntu
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

# Linux cmd

```shell
shell

sessions -u <NUMBER>


cat
ls -al
cd
```

## <mark style="color:$primary;">Users e privilegi</mark>

```shell
id                                         #Chi sono?
id                                         #Che privilegi ho? (UID)
sudo -l                                    #Che privilegi ho? (UID)
id                                         #In che gruppi sono?

cat /etc/group
groups                                     #In che gruppi sono?
groups root                                #In che gruppi sta root?
groups <USER>                              #In che gruppi sta l'user USER?

cat /etc/passwd                            #Quali altri users ci sono?

getent group sudo                          #Chi è admin?
wheel                                      #Chi è admin?

su <USER>                                  #Cambia user
```

## <mark style="color:$primary;">Network</mark>

```bash
ifconfig                                   #Leggi interfacce network
ip a                                       #Leggi interfacce network
ip a s                                     #Routing 

cat /etc/networks                          #Guarda tutti i network configurati
cat /etc/hosts                             #Trova se ci sono dispositivi conosciuti

ss -tulnp                                  #Leggi servizi e porte aperte

netstat -tulnp                    #Se hai già shell su victim
nc 127.0.0.1 <PORT>               #Se trovi port su local dopo netstat
```

## <mark style="color:$primary;">File</mark>

```bash
dir C:\                                    #Leggi disco C


wget                        


#Prendi, unzip e leggi
get <FILE>.tar.gz
tar -xvzf <FILE>.tar.gz
cat <FILE>.txt  

#Cerca file
find / -name "*.txt" -type f 2>/dev/null
```

## <mark style="color:$primary;">Password</mark>

```shell
cat /etc/shadow

echo "root:$......:::" > hash.txt
cat -xvzf rockyou.tar.gz                   #Unzip                           
sudo gunzip /usr/share/wordlists/rockyou.txt

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

## <mark style="color:$primary;">Process List</mark>

```shellscript
ps
ps -ef                                     #Dimmi tutti i processi

crontab -l 
crontab
cat /etc/cron.d
```







