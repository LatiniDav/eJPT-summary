---
hidden: true
---

# Tools

## db\_autopwn&#x20;

VULN ASSESSMENT x identificare tutti gli exploit module relativi alle porte aperte



## autoblue-ms17-010

```bash
//download tool
https://github.com/3ndG4me/AutoBlue-MS17-010
//install dipendenze python
PYTHON2
    pip2.7 install -r requirements.txt
PYTHON3
    pip install -r requirements.txt

cd shellcode
chmod +x shell_prep.sh
./shell_prep.sh
```



NESSUS

Guida al download e installazione su Vuln Assessment&#x20;

***

### <mark style="color:$primary;">Armitage GUI</mark>

{% code title="msf" %}
```shell
service postgresql start && msfconsole -q
    db_status
```
{% endcode %}

Apri altro terminale

{% code title="kali" %}
```shell
armitage
#Connect, yes e vai    
```
{% endcode %}































































