# DLC Ultime : "Le Défi du SysAdmin Suprême"

## Étape 1 : Analyse et nettoyage d’un fichier de logs

```powershell
[root@localhost ~]# grep -Ev "DEBUG|TRACE" /var/log/messy_logs.log > /var/log/temp_logs.log
```
```powershell
awk -v date="$(date -d '3 days ago' '+%Y-%m-%d')" '$0 >= date' /var/log/temp_logs.log > /var/log/temp_logs_filtered.log
```
```powershell
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" /var/log/temp_logs_filtered.log | sort | uniq -c | sort -nr > /var/log/ip_count.txt
```
```powershell
sort /var/log/temp_logs_filtered.log | head -n 100 > /var/log/cleaned_logs.log
```

## Étape 2 : Retrouver des données dissimulées


```powershell
grep -E -o "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,7}\b" /home/hidden_data/* > emails_found.txt
```
```powershell
[root@localhost ~]# grep -l "IMPORTANT" /home/hidden_data/* | xargs grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" > ip_addresses.txt
```
```powershell
grep -h "base64" /home/hidden_data/* | base64 --decode > decoded_data.txt
```
```powershell
grep -ril "SECRET" /home/hidden_data/ > secret_files.txt
```

## Étape 3 : Automatisation avancée avec un script

```powershell
#!/bin/bash

LOG_FILE="/var/log/system_cleaner.log"
TMP_DIR="/tmp"
SYSLOG="/var/log/syslog"

# 1. Supprimer les fichiers vieux de plus de 7 jours dans /tmp
echo "$(date): Nettoyage des fichiers temporaires." >> $LOG_FILE
find $TMP_DIR -type f -mtime +7 -exec rm -f {} \;

# 2. Analyser les logs pour détecter les échecs SSH
echo "$(date): Analyse des échecs SSH." >> $LOG_FILE
grep "Failed password" $SYSLOG | awk '{print $(NF-3)}' | sort | uniq | while read ip; do
    sudo firewall-cmd --permanent --add-rich-rule="rule family=ipv4 source address=$ip reject"
done
sudo firewall-cmd --reload

# 3. Rapport des modifications dans /etc
echo "$(date): Génération du rapport des modifications dans /etc." >> $LOG_FILE
find /etc -type f -mtime -1 > /var/log/etc_changes_report.txt

```

```powershell
crontab -e
0 * * * * /home/system_cleaner.sh
```

## Étape 4 : Le puzzle final
```powershell
[root@localhost ~]# grep "FRAGMENT" /opt/challenge/puzzle_data.txt | sort -k2,2 > reconstructed.txt
```

```powershell
[root@localhost ~]# grep -o "0x[0-9a-fA-F]\+" reconstructed.txt | while read hex; do
    echo $hex | xxd -r -p
done
Hello!!!world
```