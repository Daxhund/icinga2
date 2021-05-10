# icinga2 Setupguide 

## Host
* Ubuntu 20.04.2.0 LTS
* 40 GB HDD
* 4 CPU Kerne
* 4096 MB RAM

## Installation icinga2 
### Icinga Repository hinzufügen
```
apt-get -y install apt-transport-https wget gnupg
  
wget -O - https://packages.icinga.com/icinga.key | apt-key add -
. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
/etc/apt/sources.list.d/${DIST}-icinga.list
echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
/etc/apt/sources.list.d/${DIST}-icinga.list

apt-get update
```
### Installation Datenbankserver
Zur Absicherung der Datenbank wird anschließend das Skript `mysql_secure_installation` ausgeführt
```
apt-get install mysql-server mysql-client
mysql_secure_installation
```
### Installation Icinga2 und IDO Konfiguration
Zusätzliche zur icinga2 Engine wird eine Sammelung an bewehrten Check-Commands auf dem host installiert. Icinga2 verwendet diese Skript-Kolletion um verschiedene 
Abfragen(Host/Service-Checks) ausführen zu können. Die Sammlung kann unter `/usr/lib/nagios/plugins` eingesehen und beliebig erweitert werden.
```
apt-get install icinga2 monitoring-plugins 
systemctl enable icinga2
```
Für eine vollfunktionsfähige Installation benötigt icinga2 ein Datenbank Backend zum Speichern aller Objekte und Checkergebnisse.
Diese Datenbank(IDO = Icinga Data Output) wird ebenfalls von Icingaweb2 verwendet. 

```
apt-get install icinga2-ido-mysql
```
Während der Installation wird ein Deployment Wizard automatisch gestartet. Dieses verwenden wir nicht und konfigurieren die Anbindung manuell. 

    nano /etc/icinga2/features-available/ido-mysql.conf
```
    library "db_ido_mysql"
    
    object IdoMysqlConnection "ido-mysql" {
      user = "icinga",
      password = "STRONG-PASSWORD!",
      host = "srvicingadb",
      database = "icinga"
    }
```
Datenbank und Benutzer anlegen:
    mysql -u root -p
    
```
CREATE DATABASE icinga2;
CREATE USER 'icinga2'@'localhost' IDENTIFIED BY 'STRONG-PASSWORD!';
GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga2.* TO 'icinga2'@'localhost';
FLUSH PRIVILEGES;
quit;
```
Sql-Schema aufspielen:
```
wget https://raw.githubusercontent.com/Icinga/icinga2/master/lib/db_ido_mysql/schema/mysql.sql
sudo mysql -u root -p icinga2 < mysql.sql
```
Anschließend kann die Verbindung mit der Datenbank hergestellt werden.
```
icinga2 feature enable ido-mysql
systemctl restart icinga2
icinga2 feature list
icinga2 daemon -C
```
### Icinga API aktivieren
Für das Webfrontend und andere Dienste wird eine API Schnittstelle verwendet.
```
icinga2 api setup
systemctl restart icinga2
```
## Icingaweb2 Installation
### API User anlegen
In der Datei `/etc/icinga2/conf.d/api-users.conf` den folgenden Eintrag unten in der Datei ergänzen:
```
object ApiUser "icingaweb2" {
  password = "SUPER-GEHEIMES-RANDOM-CHAR-PW!"
  permissions = [ "status/query", "actions/*", "objects/modify/*", "objects/query/*" ]
}
```
### Webserver und icingaweb2 installieren
```
apt-get install apache2 icingaweb2 libapache2-mod-php icingacli
```
### Icingaweb2 Datenbank anlegen
Icingaweb2 benötigt ebenfalls ein eigenes Datenbank-backend.

    mysql -u root -p  
```
CREATE DATABASE icingaweb2;
CREATE USER 'icingaweb2'@'localhost' IDENTIFIED BY 'Super-Secret-PW!';
GRANT ALL ON icingaweb2.* TO 'icingaweb2'@'localhost';
FLUSH PRIVILEGES;
quit;
```
Über das Terminal muss nun noch ein Token generiert werden (Der Token wird zu Authentifizierung am Webinterface benötigt) 
um die Konifiguration im Webinterface abzuschließen.
```
icingacli setup token create
systemctl restart icinga2 
systemctl restart apache2
```
Über einen Webbrowser wird die Konfiguration abgeschlossen. 
    http://<IP>/icngaweb2/setup 

## Icingaweb2 Modul Director installieren

Um Konfigurationsanpassungen direkt über das Webinterface vorzunehmen wird zusätzlich das Modul Director installiert

### Datenbank für Director installieren:
``` 
sudo mysql -u root -p

CREATE DATABASE director;
CREATE USER 'director'@'localhost' IDENTIFIED BY 'SECURE-PASSWORD!';
GRANT ALL ON director.* TO 'director'@'localhost';
FLUSH PRIVILEGES;
quit;
```
Die Datenbank muss nun als neue Ressource in ICINGAWEB2 hinzugefügt werden. Screenshots

Die Modul-Abhägikeiten können mit dem folgenden Script heruntergeladen werden.
```
MODULE_NAME=reactbundle
MODULE_VERSION=v0.9.0
REPO="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}"
MODULES_PATH="/usr/share/icingaweb2/modules"
git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}"
icingacli module enable "${MODULE_NAME}"

MODULE_NAME=incubator
MODULE_VERSION=v0.6.0
REPO="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}"
MODULES_PATH="/usr/share/icingaweb2/modules"
git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}"
icingacli module enable "${MODULE_NAME}"

MODULE_NAME=ipl
MODULE_VERSION=v0.5.0
REPO="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}"
MODULES_PATH="/usr/share/icingaweb2/modules"
git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}"
icingacli module enable "${MODULE_NAME}"
```
Das eigentliche Modul kann mit dem folgenden Script heruntergeladen werden.
```
ICINGAWEB_MODULEPATH="/usr/share/icingaweb2/modules"
REPO_URL="https://github.com/icinga/icingaweb2-module-director"
TARGET_DIR="${ICINGAWEB_MODULEPATH}/director"
MODULE_VERSION="1.8.0"
git clone "${REPO_URL}" "${TARGET_DIR}" --branch v${MODULE_VERSION}
```
Icinga Director bentötigt einen Api-User in der Datei /etc/icinga2/conf.d/api-users.conf
```
object ApiUser "director" {
  password = "GEHEIM!GEHEIM!"
  permissions = [ "*" ]
  //client_cn = ""
}
```
```
icinga2 node wizard #masternode setup!
icingacli module enable director
systemctl restart icinga2
systemctl restart apache2
useradd -r -g icingaweb2 -d /var/lib/icingadirector -s /bin/false icingadirector
install -d -o icingadirector -g icingaweb2 -m 0750 /var/lib/icingadirector

MODULE_PATH=/usr/share/icingaweb2/modules/director
cp "${MODULE_PATH}/contrib/systemd/icinga-director.service" /etc/systemd/system/
systemctl daemon-reload
systemctl enable icinga-director.service
systemctl start icinga-director.service
```
## Performance Grafiken
### Installation InfluxDB
https://docs.influxdata.com/influxdb/v2.0/get-started/?t=Linux
TODO
### Installation Grafana
https://grafana.com/docs/grafana/latest/installation/debian/
TODO
### Installation Icingaweb2 Plugin
https://grafana.com/docs/grafana/latest/installation/debian/
TODO
