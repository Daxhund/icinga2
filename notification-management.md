# Benachrichtigungsmanagement 
Um eine Überwachung und im Falle eines Ausfalls eine Alarmierung zu ermöglichen kann Icinga2 Für verschiedene Events Benachritugung, bzw. Emails verschicken.

## Email-Benachrichtigung Konfiguration
Icinga2 bringt breits zwei Skripte für die Email-Benachrichtigung(Host-Events/Service-Events) mit.
Die Skripte liegen unter ´´´ etc/icinga2/scripts/ ´´´

Die beide Skripte müssen im Director als "Notification-Command" eingebunden werden.
### Host-Notification-Command
![grafik](https://user-images.githubusercontent.com/64025827/120031897-bc30aa80-bff9-11eb-8fab-8c178f1b0e9a.png)
![grafik](https://user-images.githubusercontent.com/64025827/120031969-d10d3e00-bff9-11eb-85d3-ac471cf97e9e.png)
### Service-Notification-Command
![grafik](https://user-images.githubusercontent.com/64025827/120032211-25182280-bffa-11eb-9f7f-6c987c022dba.png)
![grafik](https://user-images.githubusercontent.com/64025827/120032229-2d705d80-bffa-11eb-89ef-56d17790ffa1.png)

## Benutzer und Benutzergruppen anlegen
![grafik](https://user-images.githubusercontent.com/64025827/120070535-50971d80-c08b-11eb-9285-ab8f96a04484.png)
![grafik](https://user-images.githubusercontent.com/64025827/120070576-81775280-c08b-11eb-9fed-09daebb9d0b8.png)
![grafik](https://user-images.githubusercontent.com/64025827/120070681-082c2f80-c08c-11eb-858f-90402b2b0524.png)


![grafik](https://user-images.githubusercontent.com/64025827/120806027-a9146200-c546-11eb-8a1e-0c09314512dd.png)

![grafik](https://user-images.githubusercontent.com/64025827/120806924-b5e58580-c547-11eb-87cb-37c4799bdd33.png)
