# icinga2 Setup
## Konfiguration der Server
> Aufgrund der Leserlichkeit werden die Codeblöcke einleitend mit den zugehörigen Servernamen begonnen
> 
> * #srvicinga2 (Core-Server)
> * #srvicingadb (Datenbank)
> 
> Das Auslagern der Datenbank ist nicht zwingend nötig, untersützt aber das Skalieren in besonders großen Umgebungen.
> IP-Adressen und Namen der Schnittstellen können abweichen
### System updaten
	#srvicinga2: 
	sudo apt-get update -y && sudo apt-get upgrade -y
### Icinga Packet Repo einbinden
```
apt-get update
apt-get -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
 echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

apt-get update
```
### Netzwerk Konfiguration
Die beiden VMs haben zwei Netzwerkschnittstellen
* LAN-Schnittstelle (enp0s3) 
* Inter-VM-Kommunikation (enp0s8) 

Die Inter-VM-Kommunikations-Schnittstelle erhält eine statische IP-Konfiguration.
```
#srvicinga2:
sudo nano /etc/netplan/00-installer-config.yaml
```
Datei wie folgt anpassen:
```
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.100.2/24

  version: 2
```
```
#srvicingadb:
sudo nano /etc/netplan/00-installer-config.yaml
```
Datei wie folgt anpassen:
```
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.100.3/24

  version: 2
```
```
#srvicingadb:
sudo nano netplan apply
```

### Namesauflösung
Servernamen des jeweils anderen Server zur lokalen Auflösung hinzufügen
```
#srvicinga2:
sudo nano /etc/hosts
```
Hinzufügen zum oberen Block der Datei:
```
192.168.100.3 srvicingadb
```
```
#srvicingadb:
sudo nano /etc/hosts
```
Hinzufügen zum oberen Block der Datei:
```
192.168.100.2 srvicinga2
```

### Icinga-Core installieren
	#srvicinga2: 
	sudo apt-get install icinga2 -y 
### Monitoring Plugins installieren
	#srvicinga2:
	sudo apt-get install monitoring-plugins -y `
### Datenbank installieren
Das Skript "mysql_secure_installation" fodert während der Ausführung auf ein Root-Passwort zu setzen. Das Passwort wird im Verlauf der Installation benötigt.

	#srvicingadb:
	sudo apt-get update -y && sudo apt-get upgrade -y
	sudo apt-get install mysql-server mysql-client -y
	mysql_secure_installation
	#Dialog folgen...
	
<br>
Während der Installation wird ein Deployment Wizard automatisch gestartet. Dieses verwenden wir nicht und konfigurieren die Anbindung manuell.

	#srvicinga2:
	sudo apt-get install icinga2-ido-mysql -y
<br>

	#srvicinga2:
	sudo nano /etc/icinga2/features-available/ido-mysql.conf
<br>
Die Datei sollte wie folgt angepasst werden:

	library "db_ido_mysql"

	object IdoMysqlConnection "ido-mysql" {
	  user = "icinga",
	  password = "STRONG-PASSWORD!",
	  host = "srvicingadb",
	  database = "icinga"
	}
<br>

	#srvicingadb:
	sudo mysql -u root -p

	CREATE DATABASE icinga;
	CREATE USER 'icinga'@'%' IDENTIFIED BY 'STRONG-PASSWORD!';
	GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga.* TO 'icinga'@'%';
	FLUSH PRIVILEGES;
	quit;

Neuste Version des SQL-Schemas runterladen

	#srvicingadb:
	wget https://raw.githubusercontent.com/Icinga/icinga2/master/lib/db_ido_mysql/schema/mysql.sql
	sudo mysql -u root -p icinga < mysql.sql
Anpassen des Listinig Ports von Mysql

	#srvicingadb:
	sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
<br>

Die Zeile `bind-address = 127.0.0.1` ändern zu `bind-address = 192.168.100.3`
	
	#srvicingadb:
	sudo service mysql restart

Anschließend kann die Verbindung zwischen den Server aktiviert werden.

	#srvicinga2:
	sudo icinga2 feature enabnle ido-mysql
	sudo service icinga2 restart

### Icinga API aktivieren
Für das Webfrontend und andere Dienste wird eine API Schnittstelle verwendet.

	#srvicinga2:
	sudo icinga2 api setup
### Icingaweb2 installation
	#srvicinga2:
	sudo apt-get install apache2 -y
### Icingaweb2 API User anlegen
In der Datei ` /etc/icinga2/conf.d/api-users.conf` den folgenden Eintrag unten in der Datei ergänzen:
```
object ApiUser "icingaweb2" {
  password = "SUPER-GEHEIMES-RANDOM-CHAR-PW!"
  permissions = [ "status/query", "actions/*", "objects/modify/*", "objects/query/*" ]
}
```
	#srvicinga2:
	sudo icinga2 restart

	
### Icingaweb2 installieren

	#srvicinga2:
	sudo apt-get install icingaweb2 libapache2-mod-php icingacli
<br>
	
	#srvicingadb:
	CREATE DATABASE icingaweb2;
	CREATE USER 'icingaweb2'@'%' IDENTIFIED BY 'Super-Secret-PW!';
	GRANT ALL ON icingaweb2.* TO 'icingaweb2'@'%';
	FLUSH PRIVILEGES;
	quit;

Für den Zugriff muss noch ein APi Token generiert werden:
	
	icingacli setup token create
Nun kann der Zugriff über das Webinterface getestet werden und die Installation abgeschlossen werden.

	http://192.168.100.2/icingaweb2/setup
Nach eingeben des Access-Tokens werden im Webinterface optionale und nötige Abhängigkeiten angezeigt. Diese sollten je nach Setup installiert werden.


## Icingaweb2 Modul Director installieren
Um Konfigurationsanpassungen direkt über das Webinterface vorzunehmen wird zusätzlich das Modul Director installiert

Datenbank für Director installieren:

	#srvicingadb:
	sudo mysql -u root -p

	CREATE DATABASE director;
	CREATE USER 'director'@'%' IDENTIFIED BY 'SECURE-PASSWORD!';
	GRANT ALL ON director.* TO 'director'@'%';
	FLUSH PRIVILEGES;
	quit;
	
Die Datenbank muss nun als neue Ressource in ICINGAWEB2 hinzugefügt werden.
**Screenshots**

Das Modul kann mit dem folgenden Script heruntergeladen werden.
```

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
ICINGAWEB_MODULEPATH="/usr/share/icingaweb2/modules"
REPO_URL="https://github.com/icinga/icingaweb2-module-director"
TARGET_DIR="${ICINGAWEB_MODULEPATH}/director"
MODULE_VERSION="1.8.0"
git clone "${REPO_URL}" "${TARGET_DIR}" --branch v${MODULE_VERSION}
```
Icinga Director bentötigt einen Api-User in der Datei `/etc/icinga2/conf.d/api-users.conf`

    object ApiUser "director" {
      password = "GEHEIM!GEHEIM!"
      permissions = [ "*" ]
      //client_cn = ""
    }
Master Node Setup 
	
	srvicinga2 node setup


	
 


