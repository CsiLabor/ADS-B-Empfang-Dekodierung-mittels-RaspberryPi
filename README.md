# Howto: Installation von Dump1090 auf einem RaspberryPi

## Einleitung:

Die Anleitung erklärt, wie man auf einem RaspberryPi mittels eines bestimmten USB DVB-T Sticks und den Software-Paketen rtl-sdr und dump1090 die ADS-B Daten von Flugzeugen zu empfangen und ggf. für andere Anwendungen dekodieren/auszuwerten. 


## Was wird benötigt?

- RaspberryPi (Modell B mit 512MB Speicher) inklusive USB-Netzteil 
- SD-Karte (8GB oder mehr) 
- USB Tastatur und (optional) USB Maus
- USB DVB-T Empfänger mit RTL2832U/E4000 oder (neuer) RTL2832U/R820T als Chipsatz/Tuner
- Ein Windows/Mac/Linux-Rechner

## Installations-Anleitung:


### Basis-Installation des Raspberry Pi: 

#### Teil 1: Auf Ihrem Computer 

##### Windows/Mac:
- SDCardFormatter (für [Windows](https://www.sdcard.org/downloads/formatter_4/eula_windows/") bzw. [Mac](https://www.sdcard.org/downloads/formatter_4/eula_mac/)) und [NOOBS Distribution](http://www.raspberrypi.org/downloads/) herunterladen
- SDCard mit SDCardFormatter formatieren. ``` Wichtig: Die Option „Format Size Adjustment“ aktivieren. ``` 
- NOOBS Distribution entpacken und auf SDCard kopieren

##### oder Linux:
- Partition auf SDCard anlegen und formatieren mittels *mkfs.vfat*. 
- NOOBS Distribution entpacken und auf SDCard kopieren

Eine Schritt-für-Schritt Anleitung (englisch) finden Sie [hier](http://qdosmsq.dunbar-it.co.uk/blog/2013/06/noobs-for-raspberry-pi/)


#### Teil 2: Auf Ihrem RaspberryPi:
- SD-Karte in den RaspberryPi stecken
- RaspberryPi mit der Tastautur, einem Monitor und dem Netzteil in Betrieb nehmen.
- Im Installer auf Sprache „Deutsch“ stellen und „Raspbian“ + „Data Partiton“ auswählen
- Auswahl installieren (~2GB werden heruntergeladen)
- Einen Kaffee oder Tee holen, denn der Vorgang kann abhängig vom Internetzugang lange dauern
- Nach Abschluss RaspberryPi neu starten

### Teil 3: NOOBS installieren

##### Betriebsystem-Auswahl (Pre-Installation):

- Nach dem initialen Bootvorgang wird der _easy operating system install manager_ gestartet.
- Im Install-Manager wählen Sie als Betriebssystem „Raspbian [RECOMMENDED]“ aus
- Zusätzlich wählen Sie bei „Language“ und „Keyboard“ jeweils die Einstellungen für Deutschland aus.
- (Optional) Über „Edit config“ kann noch die config.txt Konfigurationsdatei des RaspberryPi angepasst werden
- Danach gehen Sie auf "Install" um das gewählte Betriebssystem zu installieren
- Am Abschluss der Installation wird der RaspberryPi neu gestartet

##### Konfiguration von Raspbian (Post-Installation):

- Nach dem ersten Bootvorgang des Systems Raspbian wird automatisch Raspi-config gestartet (Hinweis: Der Startvorgang kann beim ersten mal etwas länger dauern)
- In Raspi-config folgendes einstellen:
	- Mit „2 Change User Password“ ein Passwort setzen
	- Bei „4 Internationalisation Options“ / „I2 Change Timezone“ die eigene Zeitzone einstellen (z.B. Europa / Berlin)
	- (Empfohlen) Den SSH Server über „8 Advanced Options“ / „A4 SSH“ einschalten
	- (Optional) „7 Overclock“ den RaspberryPi etwas schneller machen. Bitte die Risiken beachten ([Details](http://elinux.org/RPiconfig#Overclocking))
- Mit „Finish“ das Programm verlassen (Reboot)
	
### Teil 4: Benötigte zusätzliche Pakete installieren

- Auf dem RaspberryPi anmelden mittels Benutzernamen _pi_ und Ihrem gewählten Passwort an
- Um die nachfolgenden Programme installieren zu können, werden zusätzliche Pakete auf dem Raspberry-Pi benötigt (Crosscompiler und libusb Entwickler-Dateien). Zusätzlich wird das nicht benötigte Mathematica inklusive deren Konfigurationsdateien entfernt um ein nicht benötigten Dienst zu beenden und Speicherplatz zu sparen.

	```
	sudo aptitude install cmake libusb-1.0-0-dev
	sudo aptitude purge wolfram-engine
	```

### Teil 5: Installation von rtl-sdr

- Git Repository für  rtl-sdr von OsmoSDR (<http://sdr.osmocom.org/trac/wiki/rtl-sdr>) herunterladen

	```
	git clone git://git.osmocom.org/rtl-sdr.git
	```
	
- In Sourcecode-Verzeichnis wechseln und Anwendung kompilieren

	```
	cd rtl-sdr/
	mkdir build; cd build
	cmake ../ -DINSTALL_UDEV_RULES=ON
	make
	sudo make install
	```
	
- Hier ist eventuell Zeit für z.B. dem aufräumen des Schreibtisch ;-)

- Update den Cache der Shared Libraries

	```
	sudo ldconfig
	```
	
- rtl-sdr udev-Rules kopieren

	```
	sudo cp ../rtl-sdr.rules /etc/udev/rules.d/
	```

- Bestehende DVB-T Kernel Module blacklisten (hierzu no-rtl.conf anlegen)

	```
	sudo nano /etc/modprobe.d/no-rtl.conf
	```
	
	und mit folgendem Inhalt versehen:
	
	```
	blacklist dvb_usb_rtl28xxu
	blacklist rtl2832
	blacklist rtl2830	
	```
	
- System neu starten

	```
	sudo reboot
	```
	
- Optional: prüfe ob der DVB-T Stick erkannt wird

	```
	rtl_test -t
	```
	
	Es sollte eine Ausgabe wie folgt kommen:
				
	>	Found 1 device(s):
	>	0:  Generic RTL2832U OEM
	>
	> Using device 0: Generic RTL2832U OEM
	> Found Rafael Micro R820T tuner
	
	Falls nachfolgende Text erscheint, dann muss das erwähnte Kernel-Modul ebenfalls in die oben angelegte Blacklist aufgenommen und später mittels rmmod oder Reboot entladen werden.
	
	> Kernel driver is active, or device is claimed by second instance of librtlsdr.
	> In the first case, please either detach or blacklist the kernel module
	> (dvb_usb_rtl28xxu), or enable automatic detaching at compile time.	
	
	Sollte die folgende Meldung _No E4000 tuner found, aborting._ bekommen, so stellt das **kein Problem** 

### Teil 6: Installation von dump1090

-	Dump1090 (<https://github.com/MalcolmRobb/dump1090>) herunterladen

	```
	cd ~
	git clone git://github.com/MalcolmRobb/dump1090.git
	```
	
-	Dump1090 kompilieren:

	```
	cd dump1090
	make	
	```
	

- Voilá - Installation ist schon fast fertig und wir können prüfen ob Daten empfangen werden

	```
	./dump1090 --interactive --net --metric --enable-agc
	``` 

	Es sollte dabei eine Ausgabe wie die folgende nach einigen Sekunden zu sehen sein:
	
	![linuxkonsole.png](https://github.com/CsiLabor/ADS-B-Empfang-Dekodierung-mittels-RaspberryPi/blob/master/linuxkonsole.png?raw=true)
	
### Schritt 5: Zuletzt das Init-Script für automatischen Start einrichten

- Das mitgelieferte das Init-Script in den init.d Ordner kopieren
	```
	cp dump1090.sh /etc/init.d
	```

-  Das Init-Script öffnen um die Startparameter anzupassen:

	```
	sudo nano /etc/init.d/dump1090.sh
	```
	
- Die Zeile mit *PROG_ARGS="..."* suchen und durch folgende Zeile ersetzen:
	
	```
	PROG_ARGS="--quiet --net --net-ro-size 500 --net-ro-rate 5 --aggressive --enable-agc"
	```	

	Die gewählten Parameter wurden erweitert um Auto-Gain und dem optionalen Aggressive-Mode (siehe Hinweise).
	
- Kontrollieren Sie auch, ob der unter *PROG_PATH* gewählte Pfad korrekt ist (sollte */home/pi/dump1090* sein).

-	Das Start-Script nun noch ausführbar machen und die Runlevel-Links erzeugen
	```
	sudo chmod +x /etc/init.d/dump1090.sh
	sudo update-rc.d dump1090.sh defaults
	```

### Abschluss:

Welche Möglichkeiten habe ich nach dieser Installation? Ohne die Installation weiterer Anwendungen können Sie sich im Browser auf einer Google Maps bzw. OpenStreetMap-Karte die empfangenen Flugdaten [anzeigen lassen](https://github.com/CsiLabor/ADS-B-Empfang-Dekodierung-mittels-RaspberryPi/blob/master/browserscreenshot.png?raw=true). Hierzu rufen Sie im Browser die URL _http://ip_vom_raspberry:8080/_ auf. 

Selbstverständlich besteht auch die Möglichkeit die Flugdaten mittels alternative Programme wie [Virtual Radar](http://www.virtualradarserver.co.uk) (für Windows/Linux) auszuwerten. Hierzu wird z.B. der von dump1090 bereitgestellte _Beast Raw Feed_ über den Port 30002 verwendet. Die Möglichkeiten sind hierbei beinahe unendlich.

---

### Noch ein paar Hinweise

1. Sollten Sie die dump1090 im Hintergrund starten möchten, so können Sie die Software wie folgt aufrufen:

	```
	./dump1090 --quiet --net --enable-agc &
	```
 
2. Falls der RaspberryPi überlastet sein sollte (z.B. beim Empfang vieler Flugzeuge z.B. mittels einer etwas besseren Antenne), dann kann die Systemlast durch folgenden Programmaufruf reduziert werden. Sollten Sie Ihren RaspberryPi nicht übertaktet haben, kann dies ebenfalls Sinn ergeben:

	```
	./dump1090 --quiet --net --enable-agc --net-ro-size 500 --net-ro-rate 5 &
	```
	
3. Mittels des Parameters _--aggressive_ kann versucht werden die Flugdaten mittels einer alternativen Methode dekodiert werden. Dies *kann* zum empfangen von mehr Flugdaten führen. Dies wird unter anderem mehr einer höheren Fehlerrate "erkauft". Der Einsatz empfiehlt sich laut Entwickler an Orten, bei denen wenig Flugverkehr stattfindet. Es erhöht sich dabei die Chance mehr Daten von den Flugzeugen zu empfangen. Weitere Details finden sich auf der Homepage von Dump1090 unter <https://github.com/MalcolmRobb/dump1090>.

	Nach meiner Erfahrung empfiehlt sich der Einsatz des Parameters auch, wenn man z.B. die beim USB-Stick mitgelieferte Antenne einsetzen möchte. 
	
--

###### Lizenz-Information:

Copyright Jens Dutzi 2014 / Stand: 14.06.2014

![Lizenz](http://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png).

Dieses Werk ist lizenziert unter einer [Creative Commons Namensnennung - Nicht-kommerziell - Weitergabe unter gleichen Bedingungen 4.0 International Lizenz.](http://creativecommons.org/licenses/by-nc-sa/4.0/)
