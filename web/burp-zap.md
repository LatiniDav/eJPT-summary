---
icon: globe-pointer
---

# BURP / ZAP

## <mark style="color:$primary;">Burp Suite</mark>

```shell
# 1. Open browser e set FoxyProxy su BURP
# 2. Open Burp, select "proxy" tab e click su "intercept is on"
# 3. Premi invio sul browser per cercare l'IP/URL
# 4. Leggi richiesta su Burp
```

***

## <mark style="color:$primary;">OWASP ZAP</mark>

```shell
# 1. Apri OWASP ZAP
# 2. Fai refresh (su browser) della pagina che vuoi analizzare
# 3. Vai su ZAP. 
    # Tools -> 
    # Spider -> 
    # Select uno degli URL che ti serve -> 
    # Recurse eanbled -> 
    # Advanced (Max depth 5, Max children 0, Max duration 0) e Aspetta ->
    # Avrai trovato tutti i link e le risorse relative al target IP
```





















