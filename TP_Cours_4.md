# TP Avancé : "Mission Ultime : Sauvegarde et Sécurisation"

## Étape 1 : Analyse et nettoyage du serveur


### 1. Lister les tâches cron pour détecter des backdoors :


```powershell
[root@localhost ~]# for user in $(cut -f1 -d: /etc/passwd); do echo -e "\n\n==> $user:"; crontab -u $user -l; done > liste_crontabs.txt
```

```powershell
[root@localhost ~]# cat liste_crontabs.txt

...

==> attacker:
*/10 * * * * /tmp/.hidden_script

```

### 2. Identifier et supprimer les fichiers cachés :

```powershell
[root@localhost tmp]# ls -al


[root@localhost tmp]# rm malicious.sh
[root@localhost tmp]# rm .hidden_file
[root@localhost tmp]# rm .hidden_script

[root@localhost home]# rm -r attacker/

[root@localhost tmp]# rm .nop
```

### 3. Analyser les connexions réseau actives :

```powershell
[root@localhost /]# netstat -an
Active Internet connections (servers and established)
..
```

## Étape 2 : Configuration avancée de LVM

### 1. Créer un snapshot de sécurité pour /mnt/secure_data :

```powershell
[root@localhost secure_data]# lvcreate --size 500M --snapshot --name snap /dev/vg_secure/secure_data
```

### 2. Tester la restauration du snapshot :

```powershell
[root@localhost secure_data]# mount /dev/vg_secure/snap /media/
[root@localhost secure_data]# cd /media/
[root@localhost media]# ls
lost+found  sensitive1.txt  sensitive2.txt
```

### 3. Optimiser l’espace disque :

```powershell

[root@localhost ~]# lvextend -L +2G /dev/mapper/vg_secure-snap

```

## Étape 3 : Automatisation avec un script de sauvegarde

```powershell
[root@localhost backup]# cat backup_secure.sh
#!/bin/bash

# Variables
SOURCE_DIR="/mnt/secure_data"
BACKUP_DIR="/backup"
DATE=$(date +"%Y%m%d")
BACKUP_FILE="$BACKUP_DIR/secure_data_$DATE.tar.gz"
MAX_BACKUPS=7

# Vérifie si le répertoire de destination existe, sinon le crée
if [ ! -d "$BACKUP_DIR" ]; then
  echo "Création du répertoire de sauvegarde : $BACKUP_DIR"
  mkdir -p "$BACKUP_DIR"
fi

# Archive le contenu tout en excluant les fichiers cachés, .tmp, et .log
if [ -d "$SOURCE_DIR" ]; then
  echo "Création de l'archive : $BACKUP_FILE"
  tar --exclude="*.tmp" --exclude="*.log" --exclude=".*" -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .
  echo "Sauvegarde terminée avec succès : $BACKUP_FILE"
else
  echo "Erreur : Le répertoire source $SOURCE_DIR n'existe pas."
  exit 1
fi

# Fonction de rotation des sauvegardes
echo "Rotation des sauvegardes : conservation des $MAX_BACKUPS dernières sauvegardes."
BACKUPS_COUNT=$(ls -1 $BACKUP_DIR/secure_data_*.tar.gz 2>/dev/null | wc -l)

if [ "$BACKUPS_COUNT" -gt "$MAX_BACKUPS" ]; then
  # Supprimer les plus anciennes sauvegardes si le nombre dépasse MAX_BACKUPS
  OLDEST_BACKUPS=$(ls -1t $BACKUP_DIR/secure_data_*.tar.gz | tail -n +$((MAX_BACKUPS + 1)))
  for BACKUP in $OLDEST_BACKUPS; do
    echo "Suppression de l'ancienne sauvegarde : $BACKUP"
    rm -f "$BACKUP"
  done
fi

echo "Processus terminé."

```

```powershell
[root@localhost backup]# chmod +x backup_secure.sh
[root@localhost backup]# ./backup_secure.sh
Création de l'archive : /backup/secure_data_20241125.tar.gz
Sauvegarde terminée avec succès.
```

```powershell
crontab -e
0 3 * * * /backup/backup_secure.sh >> /backup/backupdubackup
```

## Étape 4 : Surveillance avancée avec auditd

```powershell
[root@localhost /]# sudo nano /etc/audit/rules.d/audit.rules
-w /etc/ -p wa -k etc_changes
[root@localhost /]# sudo augenrules --load
[root@localhost /]# sudo auditctl -l
[root@localhost /]# sudo touch /etc/test_audit
[root@localhost /]# sudo ausearch -k etc_changes > /var/log/audit_etc.log
[root@localhost /]# cat /var/log/audit_etc.log
```


## Étape 5 : Sécurisation avec Firewalld

```powershell
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --remove-service=ftp
sudo firewall-cmd --permanent --remove-service=smtp
sudo firewall-cmd --permanent --remove-service=telnet
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```
```powershell
[root@localhost audit]# firewall-cmd --permanent --add-rich-rule="rule family=ipv4 source address=192.168.56.109 reject"
success
```
```powershell
[root@localhost audit]# sudo firewall-cmd --permanent --add-rich-rule="rule family=ipv4 source address=192.168.0.0/22 service name=ssh accept"
success
```

