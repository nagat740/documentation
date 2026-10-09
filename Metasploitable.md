### B3 Act05
# machine :
- kali linux 2025
- machine Metasploitable

- Ouverture console metasploit
```bash
msfconsole
``` 

- Cherche des modules avec des mot préçis
```bash
search <module>
``` 

- Permet de voir les info d'un modules
```bash
info <chemin_vers_le_module>
```

- Recherche de service / port sur une machine
```bash
nmap -sV <ip>
```

- Utiliser un exploit
```bash
use <id de l'exploit>
```

- Définir une variable de l'exploit
```bash
set <variable> <valeur>
```

## Attaque sur serveur samba

- use auxiliary/scanner/smb/smb_version

- search samba <version_samba>

- Voir les page web d'un site 
dirb http://<ip ou nom de domaine>

- Voir les vulnérabilité d'un module pour une certaine année
search <module> cve:<année>

