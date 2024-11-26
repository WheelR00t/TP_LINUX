# DLC : "L'ultime défi du SysAdmin"

## Étape 1 : Analyse avancée et suppression des traces suspectes

### 1. Rechercher des utilisateurs récemment ajoutés :

```powershell
[root@localhost audit]# grep -a "new user" /var/log/secure
Nov 24 18:11:10 vbox useradd[1450]: new user: name=attacker, UID=1000, GID=1000, home=/home/attacker, shell=/bin/bash, from=/dev/pts/0
Nov 25 20:24:14 localhost sudo[4365]:    root : TTY=pts/0 ; PWD=/var/log/audit ; USER=root ; COMMAND=/bin/grep 'new user' /var/log/secure
Nov 25 20:24:32 localhost sudo[4378]:    root : TTY=pts/0 ; PWD=/var/log/audit ; USER=root ; COMMAND=/bin/grep 'new user' /var/log/secure
Nov 25 20:25:02 localhost sudo[4381]:    root : TTY=pts/0 ; PWD=/var/log/audit ; USER=root ; COMMAND=/bin/grep -a 'new user' /var/log/secure
```

### 2. Trouver les fichiers récemment modifiés dans des répertoires critiques :


```powershell
[root@localhost audit]# find /etc /usr/local/bin /var -type f -mtime -7
```

### 3. Lister les services suspects activés :
```powershell
[root@localhost audit]# sudo systemctl list-unit-files --state=enabled
...
28 unit files listed.
```

### 4. Supprimer une tâche cron suspecte :
```powershell
[root@localhost audit]# crontab -u attacker -r
```

## Étape 2 : Configuration avancée de LVM

...

## Étape 3 : 

### 1. Bloquer les attaques par force brute :

```powershell
[root@localhost audit]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" service name="ssh" log prefix="SSH_ATTEMPT" level="info" limit value="1/m" drop'
```

### 2. Restreindre l’accès SSH à une plage IP spécifique :
```powershell
firewall-cmd --permanent --zone=trusted --add-source=192.168.0.0/22
firewall-cmd --reload
```

### 3. Créer une zone sécurisée pour un service web :


```powershell
[root@localhost audit]# sudo firewall-cmd --permanent --new-zone=web_secure
[root@localhost audit]#sudo firewall-cmd --permanent --zone=web_secure --add-service=http --add-service=https
[root@localhost audit]#sudo firewall-cmd --permanent --zone=web_secure --change-interface=enp0s3
[root@localhost audit]#sudo firewall-cmd --reload

```

## Étape 4 : Création d'un script de surveillance avancé


### Écrivez un script monitor.sh :
```powershell
#!/bin/bash
LOG_FILE="/var/log/monitor.log"
echo "$(date): Surveillance démarrée." >> $LOG_FILE

# Connexions actives
ss -tunap >> $LOG_FILE

# Fichiers modifiés
inotifywait -r /etc -e modify -e create -e delete >> $LOG_FILE &
```

### Ajoutez une alerte par e-mail :

```powershell
echo "Modification détectée" | mail -s "Alerte de modification" wheelroot@root.fr
```

### Automatisez le script :
```powershell

*/5 * * * * /home/monitor.sh
```


## Étape 5 : Mise en place d’un IDS (Intrusion Detection System)

### Installer et configurer AIDE :
```powershell
[root@localhost home]# yum install aide
[root@localhost home]# sudo aide -i
```
```powershell
[root@localhost etc]# nano passwd
[root@localhost etc]# sudo aide --check
Changed entries:
---------------------------------------------------

...

f   ...    .C... : /etc/passwd

...
```