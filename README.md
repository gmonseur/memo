# Mémo

- [Unix](#unix)
- [Drush](#drush)
- [Ngrok](#ngork)

## <a name="unix"></a>Unix

**Synchroniser l'heure**
```
/usr/sbin/ntpdate ntp.ovh.net
```

**Process apache en tmp réel**
```
watch -n 1 "echo -n 'Apache Processes: ' && ps -C httpd --no-headers | wc -l && free -m"
```

**Nombre de connexions simultanées**<br>
- *Web*
```
netstat -tanpu | grep :80 | awk '{print $5}' | cut -f 1 -d ":" | sort |uniq -c
```
- *Apache*
```
netstat -tanp | grep ESTABLISHED | grep http | wc -l
```
- *Test IP*
```
netstat -tanp | grep ESTABLISHED | grep http | awk {'print $5'} | sort -n 
netstat -an | grep SYN_RECV |wc -l
```
- *Test IP SYN Flood*
```
for i in ` netstat -tanpu | grep "SYN_RECV" | awk {'print $5'} | cut -f 1 -d ":" | sort | uniq -c | sort -n | awk {'if ($1 > 3) print $2'}` ; do echo $i; done 
```
- *Max client*
```
cat /var/log/httpd/error_log | grep MaxClient
```

**Bloquer les ip à l'origine d'une attaque SYN Flood**
```
for i in ` netstat -tanpu | grep "SYN_RECV" | awk {'print $5'} | cut -f 1 -d ":" | sort | uniq -c | sort -n | awk {'if ($1 > 3) print $2'}` ; do echo $i; iptables -A INPUT -s $i/24 -j DROP; done 
```

**Iptables Firewall**
```
iptables -L
```
**Flush/Reset iptables**
```
iptables -F INPUT
```

**Logs en temps réél**
```
tail -f /var/log/messages
```
- CTRL+Z pour stopper
- bg pour tache en arrière plan
- fg pour rétabli
- CTRL+C pour quitter

**Supprimer les archives qui date de plus de 30 jours dans le dossier “/var/log”**
```
find /var/log/ -maxdepth 3 -name "*gz*" -ctime +30 -exec rm -f {} \;
```

**Liste tout les Vhosts actuellement actif dans Apache**
```
httpd -S
```

**Liste tout les Vhosts actuellement actif dans Nginx**
```
grep server_name /etc/nginx/sites-enabled/* -RiI

starting from version 1.9.2 you can do:
nginx -T
```

**Check la syntaxe d'une zone DNS**
```
named-checkzone example.com /var/named/example.com.db
```

**Liste des partitions**
```
df -h 
```

**Espace disque utilisé contenu dans le répertoire courant**
```
du -hc --max-depth=1 
```

**Compte le nbre de fichiers**
```
ls -al | wc -l
```

**Modifier les user et group récursivement**
```
chown -R user:group folder
```

**Copie récursive (+merge)**
```
rsync --recursive source/ destination/
```

**Copie dossier de serveur => serveur**
```
scp -r -P 5022 your_folder your_username@domain.com:/some/remote/directory 
scp -r -P 2221 your_username@domain.com:/some/remote/directory . // => le point indique le répertoire local courrant
```

**Relancer Apache**
```
/etc/init.d/httpd stop
/etc/init.d/httpd start
/etc/init.d/httpd restart
```

**Test cfg nginx restart**
```
nginx -t
service nginx restart
```

**Lien symbolique nginx**
```
ln -s /etc/nginx/sites-available/MYVHOST /etc/nginx/sites-enabled/MYLINK
```

**Relancer mysql**
```
service mysql stop
service mysql start
OU
/etc/init.d/mysql zap
/etc/init.d/mysql stop
/etc/init.d/mysql start
```

**Nombre anormalement élevé de processus MySQL**
```
ps ax | grep mysql
```

**Import/Export SQL**
- *Import*
```
mysqldump -u USERNAME -pPASSWORD DATABASE | gzip -c > ~/dump_2010-06-14.sql.gz
mysqldump -u USERNAME -pPASSWORD DATABASE > ~/dump_2010-06-14.sql
```
- *Export*
```
mysql -u USERNAME -pPASSWORD DATABASE < database.sql

// Force the import without warnings or errors:	
mysql -f -u USERNAME -pPASSWORD DATABASE < database.sql
```

**Mysqlcheck**
```
mysqlcheck -o --all-databases -u root -ppassword
```

**Optimisation images (jpg et png)**<br>
Besoin des packages *jpegoptim* et *optipng*.<br>
Récursivement suivant le répertoire courant :
```
find . -type f -name "*.jpg" -exec jpegoptim -t -m90 --all-progressive --strip-all {} \;
find . -type f -name "*.png" -exec optipng -o5 -strip all {} \;
```

**Test envoi e-mail**
```
echo "This is the mail body" | mail example@example.com
```

**Antivirus ClamAV**
```
clamscan -i -r /path
```

**Recherche les fichiers modifiés depuis les 30 derniers jours dans le répertoire courant**
```
find ./ -type f -mtime -30
```

**Recherche uniquement sur un utilisateur dans le répertoire courant**
```
find . -user "apache"
```

**Recherche tous les fichiers .jpg qui ne contiennent pas le mot "color" dans le répertoire courant**
```
find . -name "*.jpg" ! -name "*color*"
```

**Screen**
```
screen
```
- CTRL+A et ? = paramètres screen
- CTRL+A et d = detached
- CTRL+A et :quit = quit

```
// Liste
	screen -ls 
// Re-attach
	screen -r pid
```

**Supprimer le ^M venant de Windows dans les fichiers txt**
- *Pour vérifier*
```
cat -A fichier.sh
```
- *Pour supprimer*
```
sed -ie 's/\r//' fichier.sh
```

## <a name="drush"></a>Drush

**Activer/Désactiver le mode maintenance**
```
drush sset system.maintenance_mode 1
drush sset system.maintenance_mode 0
```

**Désactive l'aggregation des fichiers CSS/JS**
```
drush -y config-set system.performance css.preprocess 0
drush -y config-set system.performance js.preprocess 0
```

**Supprimer tous les noeuds ou type de noeuds (devel doit être installé)**
```
drush genc --kill 0 0 
drush genc --kill --types=article 0 0
```

**Lister les modules activées/désactivés/non installés**
```
drush pml --no-core --type=module --status=Enabled
drush pml --no-core --type=module --status=Disabled
drush pml --no-core --type=module --status="Not installed"
```

**Lister les modules à mettre à jour (update status)**
```
drush ups
```

**Mettre à jour Drupal et tout les modules**
```
drush up
```

**Mettre à jour certains modules (séparer par un espace)**
```
drush up module1 module2
```

**Import/Export/Delete de config**
```
drush cex
drush cim
drush cdel
```

**Update db/entity**
```
drush updb
drush entup
```

**Génère un lien de connexion unique pour l'utilisateur madmax**
```
drush uli madmax
```

## <a name="ngork"></a>Ngork
```
ngrok http -subdomain=customsubdomain -host-header=rewrite home.dev:80
```






