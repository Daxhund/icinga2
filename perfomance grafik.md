# Installation 
## Installation Influxdb
Influxdb ist eine Datenbank optimiert auf viele kleine Datenpunkt und stellt damit eine optimales Backend für das Sammeln der Checkergebnisse über einen längeren Zeitraum. 
```
wget -qO- https://repos.influxdata.com/influxdb.key | gpg --dearmor > /etc/apt/trusted.gpg.d/influxdb.gpg
export DISTRIB_ID=$(lsb_release -si); export DISTRIB_CODENAME=$(lsb_release -sc)
echo "deb [signed-by=/etc/apt/trusted.gpg.d/influxdb.gpg] https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" > /etc/apt/sources.list.d/influxdb.list

apt-get update && sudo apt-get install influxdb
systemctl unmask influxdb.service
systemctl start influxdb
systemctl enable influxdb
```

## Installation Grafana 
Um die Datenpunkte auszuwerten und übersichtliche Grafiken aus der Datenbank zu erstellen, wird zusätzlich grafana installiert.
```
wget -O - https://packages.grafana.com/gpg.key | apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" > /etc/apt/sources.list.d/grafana.list

apt-get update
apt-get install grafana
systemctl enable grafana-server.service
systemctl start grafana-server.service

influxdb
CREATE DATABASE icinga2;
CREATE USER icinga2 WITH PASSWORD 'its-a-secret-can-u-keep-it';
quit
```
## icinga2 konfigurieren 
icinga2 muss ein zusätzliches Modul für das schreiben in die influxdb konfiguriern 
```
icinga2 feature enable influxdb
nano /etc/icinga2/features-enabled/influxdb.conf
```
```
object InfluxdbWriter "influxdb" {
  host = "127.0.0.1"
  port = 8086
  database = "icinga2"
  flush_threshold = 1024
  flush_interval = 10s
  host_template = {
    measurement = "$host.check_command$"
    tags = {
      hostname = "$host.name$"
    }
  }
  service_template = {
    measurement = "$service.check_command$"
    tags = {
      hostname = "$host.name$"
      service = "$service.name$"
    }
  }
}

```
## InfluxDB in grafana einbinden:
Über Webinterface http://Host-IP:3000/datasources/new?gettingstarted
```
Name: InfluxDB
Type: InfluxDB
Default: Yes

Url: http://127.0.0.1:8086
Access: Server (Default)

Database: icinga2
User: icinga2
Password: its-a-secret-can-u-keep-it
```
## Icingaweb2 Modul hinzufügen
```
ICINGAWEB_MODULEPATH="/usr/share/icingaweb2/modules"
REPO_URL="https://github.com/Mikesch-mp/icingaweb2-module-grafana"
TARGET_DIR="${ICINGAWEB_MODULEPATH}/grafana"
git clone "${REPO_URL}" "${TARGET_DIR}"
icingacli module enable grafana
```
Im Grafana Webinterface ein neues Dashboard importieren
JSON.config liegt im Git-Repo