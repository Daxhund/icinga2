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
```
Für eine vollfunktionsfähige Installation benötigt icinga2 ein Datenbank Backend zum Speichern aller Objekte und checkergebnisse.
Die Datenbank(IDO = Icinga Data Output) wird ebenfalls von Icingaweb2 verwendet.

```
apt-get install icinga2-ido-mysql
```
Während der Installation wird ein Deployment Wizard automatisch gestartet. Dieses verwenden wir nicht und konfigurieren die Anbindung manuell. 

    sudo nano /etc/icinga2/features-available/ido-mysql.conf
```
    library "db_ido_mysql"
    
    object IdoMysqlConnection "ido-mysql" {
      user = "icinga",
      password = "STRONG-PASSWORD!",
      host = "srvicingadb",
      database = "icinga"
    }
```

