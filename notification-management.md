# Benachrichtigungsmanagement 
Um eine Überwachung und im Falle eines Ausfalls eine Alarmierung zu ermöglichen kann Icinga2 Für verschiedene Events Benachritugung, bzw. Emails verschicken.

## Konfiguration
Icinga2 bringt breits zwei Skripte für die Email-Benachrichtigung(Host-Events/Service-Events) mit.
Die Skripte liegen unter ´´´ etc/icinga2/scripts/ ´´´

Die beide Skripte müssen im Director als "Notification-Command" eingebunden werden.
### Host-Notification-Command
![grafik](https://user-images.githubusercontent.com/64025827/120031897-bc30aa80-bff9-11eb-8fab-8c178f1b0e9a.png)
![grafik](https://user-images.githubusercontent.com/64025827/120031969-d10d3e00-bff9-11eb-85d3-ac471cf97e9e.png)
### Service-Notification-Command
![grafik](https://user-images.githubusercontent.com/64025827/120032211-25182280-bffa-11eb-9f7f-6c987c022dba.png)
![grafik](https://user-images.githubusercontent.com/64025827/120032229-2d705d80-bffa-11eb-89ef-56d17790ffa1.png)

