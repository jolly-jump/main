.. _migration-label:

===============================
 Migration auf linuxmuster 7.0
===============================

.. sectionauthor:: `@jeffbeck <https://ask.linuxmuster.net/u/jeffbeck>`_,
		   `@Tobias <https://ask.linuxmuster.net/u/Tobias>`_,

Es wird eine Migration der Benutzerinformationen (Namen, Passwort,
Projekte), Computerinformationen (``workstations``), der Benutzerdaten
(``/home``), Tausch- und Projektverzeichnisse und der Geräte-Abbilder
(``/var/linbo``) unterstützt.

Nicht migriert werden Beschreibungen von Projekten, Quota-Tabellen und
Rollen, die Geräte bekommen. Diese müssen von Hand angepasst werden.

Ebenso werden die Dienste mrbs, openSchulPortfolio und der Mail-Server
nicht migriert, da diese - wenn benötigt - zur Installation in einem
Dockercontainer vorgesehen sind.

Voraussetzungen
===============

Es muss als Quellsystem linuxmuster.net in der Version 6.2 installiert
sein. Es ist möglich, dass auch ab Version 6.1 und 6.0
eine Migration funktioniert. Dies wurde nicht offiziell
getestet. (Stand: Dez. 2018)

Wer eine alte paedML Linux (zwischen Version 4.0.6 und 5.1.0) besitzt,
für den kann der `Upgradepfad
<http://docs.linuxmuster.net/de/v62/systemadministration/migration/index.html>`_
über eine Migration zu einer linuxmuster.net 6.2 eine Option sein.

- Die Migration wird in die Standard-Schulinstanz `default-school` vorgenommen.
- Das Zielsystem darf außer den standardmäßig angelegten Benutzern des
  Setups keine zusätzlichen Benutzer, Gruppen oder Projekte haben.
  
Vorgehen
--------

1. Zunächst installiert man auf dem Quellsystem (Version 6.x) das
   Paket `sophomorix-dump` und exportiert die Daten  (ca. 15MByte).
    
2. Danach importiert man diese Daten auf einem Zielsystem (Version
   7.x) und rekonstruiert dort Benutzer, Passwörter, Projekte und
   Geräte, etc.

3. Es müssen manuell die Verzeichnisse ``/home/share``,
   ``/home/teachers`` und ``/home/students`` im Zielsystem gemountet
   werden (z.B. über eine externe Festplatte und bind-mount,
   Netzwerk-mount, etc.) und importiert werden.

4. Die Daten von LINBO können ebenso wie Benutzerdaten synchronisiert
   werden.
 
Export der Daten unter linuxmuster.net 6.x
==========================================

Der Server 6.x muss sich in einem synchronisierten Zustand befinden,
d.h. der Befehl auf der Konsole ``sophomorix-check`` darf keine
hinzuzufügenden oder zu verändernden Benutzer anzeigen.
Dafür führt man folgende Schritte als `root` nacheinander aus:

.. code-block:: console

   # sophomorix-check
   ...
   # sophomorix-add
   ...
   # sophomorix-move
   ...
   # sophomorix-kill
   ...

Jetzt sollte ein ``sophomorix-check`` keine Benutzer mehr verändern
wollen.

sophomorix-dump installieren
----------------------------

Installiere jetzt ``sophomorix-dump`` aus dem babo-Repository oder
lade das entsprechende Debian-Paket von der Webseite herunter

.. code-block:: console

   server ~ # apt-get update
   server ~ # apt-get install sophomorix-dump
   ...
   sophomorix-dump (3.63.2-1) wird eingerichtet ...

Alternativ kannst du (z.B. wenn du das babo-Repository nicht
einbinden kannst) unter http://pkg.linuxmuster.net/babo/ die
neueste Version `sophomorix-dump_u.v.w-z_all.deb` herausfinden,
herunterladen und installieren:

.. code-block:: console

   server ~ # wget http://pkg.linuxmuster.net/babo/sophomorix-dump_3.63.2-1_all.deb
   server ~ # dpkg -i sophomorix-dump_3.63.2-1_all.deb
   ...
   sophomorix-dump (3.63.2-1) wird eingerichtet ...

Daten exportieren
-----------------

Führe das Skript ``sophomorix-dump`` aus

.. code-block:: console

   server ~ # sophomorix-dump
   ...
       * Dump OK: /root/sophomorix-dump/data/etc/linuxmuster/subnets
   ########### End: Results of dump ##########
   WARNINGs in Results of dump are OK:
   
     /etc/sophomorix/virusscan/sophomorix-virusscan-excludes.conf
     /etc/sophomorix/virusscan/sophomorix-virusscan.conf
     /var/lib/sophomorix/virusscan/penalty.db
       are only needed, if you had configured sophomorix for scanning viruses

Die Zusammenfassung zeigt Fehler und Warnungen an. Warnungen und der folgende Fehler:
``ERROR dumping: /root/sophomorix-dump/data/etc/sophomorix/user/mail/*`` können ignoriert werden.

Die exportierten Daten (bis zu 15MByte) liegen jetzt in
``/root/sophomorix-dump``. Kopiere dieses Verzeichnis auf den Server
mit Version 7.x. Um die exportierten Daten wieder zu löschen, führe ``sophomorix-dump --clean`` aus.


Import der Daten unter linuxmuster.net 7.x
==========================================

Installiere die ``sophomorix-vampire``-Skripte über

.. code-block:: console

   server ~ # apt update
   server ~ # apt install sophomorix-vampire
   ...

Das Skript ``sophomorix-vampire -h`` zeigt Optionen und Schritte an,
die im folgenden durchgeführt werden. 

Kompletter Import mit sophomorix-vampire-example
------------------------------------------------

Beispielhaft führt das Skript ``sophomorix-vampire-example`` alle
Schritte für eine typische Schule durch. Es empfiehlt sich das Skript
in den übertragenen Ordner ``sophomorix-dump`` zu kopieren und an die
eigenen Bedürfnisse anzupassen. Besonders der Import der Nutzerdaten
sollte in der folgenden Schritt-für-Schritt Anleitung genau geprüft
werden.

1. Analyse der exportierten Daten
---------------------------------

Die folgende Analyse zeigt

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --analyze

``ERROR``:
  z.B. fehlende Dateien (``/etc/sophomorix/user/mail/*`` wird dagegen
  nicht in jeder Installation verwendet)

``INFO``:
  z.B. Gruppen, die während der Migration umbenannt werden

``WARNING``:
  z.B. Warnungen, welche Dateien überschrieben werden

2. Migration der Klassen
------------------------

Alle Klassen werden vor den Benutzern migriert, inklusive eventueller
Umbenennungen der Klassennamen wie in der Analyse angezeigt. Dafür
erstellt man zunächst das Klassenskript und führt es danach aus

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --create-class-script
   server ~ # /root/sophomorix-vampire/sophomorix-vampire-classes.sh

Jetzt können die neu erstellten Klassen überprüft werden, beispielsweise

.. code-block:: console

   server ~ # sophomorix-class -i
   server ~ # sophomorix-class -i --class teachers

3. Migration der Benutzer
-------------------------

Zunächst muss die Passwortlängen und -komplexitätsüberprüfung von
Samba 4 so eingestellt werden, dass bisherige einfache Passwörter
erlaubt sind.

.. code-block:: console

   server ~ # samba-tool domain passwordsettings set --complexity=off
   server ~ # samba-tool domain passwordsettings set --min-pwd-length=1

Jetzt wird aus den exportierten Daten eine Datei ``sophomorix.add``
erzeugt, die an die richtige Stelle im System kopiert werden muss, um
danach die Benutzer regulär aufzunehmen.

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --create-add-file
   server ~ # cp /root/sophomorix-vampire/sophomorix.add /var/lib/sophomorix/check-result/sophomorix.add

Folgender Schritt informiert vorab mit ``ERRORS`` und ``WARNINGS``
über mögliche Fehlermeldungen bei der geplanten Aufnahme. Diese Fehler
sollten manuell in der Datei
``/var/lib/sophomorix/check-result/sophomorix.add`` korrigiert werden.

.. code-block:: console

   server ~ # sophomorix-add -i
   ...
   WARNING:
   ERROR:
   ...

Die Aufnahme der Benutzer wird ca. 1 Sekunde Zeit pro Benutzer in
Anspruch nehmen, Zeit einen Tee zu trinken.

.. code-block:: console

   server ~ # sophomorix-add 
   ...

Die Aufnahme

- nimmt die Benutzer mit ihren Erstpasswörtern auf, dies kann mit

  .. code-block:: console

     server ~ # sophomorix-passwd --test-firstpassword
     ...

  getestet werden, was hier zu 100% funktionieren sollte. Im nächsten
  Schritt folgt der Import der aktuellen Passworthashes.

- gibt den Benutzern zunächst keine Rechte für die WebUI/Schulkonsole. Dies folgt
  in einem späteren Schritt.


4. Passworthashes importieren
-----------------------------

Die mit Hash codierten Passwörter werde mit folgendem Befehl
importiert und sollte keine Fehler erzeugen

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --import-user-password-hashes
   ...
   0 ERRORS:

Jetzt müssen die standardmäßig komplexen Passwörter wieder aktiviert werden

.. code-block:: console

   server ~ # samba-tool domain passwordsettings set --complexity=default
   server ~ # samba-tool domain passwordsettings set --min-pwd-length=default

Tests
~~~~~

Jetzt sollten für Konten bei denen nicht mehr das Erstpasswort gilt,
der folgende Test fehlschlagen. Für alle Konten mit Erstpasswörtern
sollte er noch funktionieren.

.. code-block:: console

   server ~ # sophomorix-passwd --test-firstpassword

Zeige einen oder mehrere Benutzer an

.. code-block:: console

   server ~ # sophomorix-user -i
   server ~ # sophomorix-user -i --user name
   server ~ # sophomorix-user -i --user na*

5. Klassenadministratoren importieren
-------------------------------------

Wie bisher

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --create-class-adminadd-script
   server ~ # /root/sophomorix-vampire/sophomorix-vampire-classes-adminadd.sh

6. Projekte importieren
-----------------------

Im nachfolgenden Schritt werden alle Projekte importiert.

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --create-project-script
   server ~ # /root/sophomorix-vampire/sophomorix-vampire-projects.sh

Tests
~~~~~

Zeige ein oder mehrere Projekte an

.. code-block:: console

   server ~ # sophomorix-project -i
   server ~ # sophomorix-project -i -p name | p_name
   server ~ # sophomorix-project -i -p p_na*

7. Konfigurationsdateien importieren
------------------------------------

Mit folgendem Schritt werden wichtige Konfigurationsdateien verändert. 

Das Skript muss zwei Mal ausgeführt werden.

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --restore-config-files
   ...
   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --restore-config-files

.. hint::

   Jetzt solltest du noch die Datei ``school.conf`` bearbeiten, denn das
   wird nicht automatisch gemacht.

8. Updates diverser Einstellungen
---------------------------------

Grundsätzlicher Durchlauf von ``sophomorix-check`` muss funktionieren:

.. code-block:: console

   server ~ # sophomorix-check

Stelle sicher, dass keine weiteren Benutzer hinzugefügt werden müssen:

.. code-block:: console

   server ~ # sophomorix-add -i

Mit folgendem Schritt werden

- Benutzernamen in UTF-8 konvertiert (ab jetzt sind Umlaute und Sonderzeichen in Namen möglich),
- Zugriffsrechte in der Schulkonsole gesetzt

.. code-block:: console

   server ~ # sophomorix-update

Lösche die Benutzer, die nach deinen Einstellungen in ``school.conf`` fällig werden.

.. code-block:: console

   server ~ # sophomorix-kill

Tests
~~~~~

So kann man überprüfen, ob Sonderzeichen in ``students.csv`` oder ``teachers.csv`` in das System übernommen wurden:

.. code-block:: console

   server ~ # sophomorix-user -i -u <user_with_umlaut>

9. Rechner importieren
----------------------

.. code-block:: console

   server ~ # linuxmuster-import-devices --dry-run
   server ~ # linuxmuster-import-devices

Tests
~~~~~

Überprüfe, ob einzelne Rechner vorhanden sind:

.. code-block:: console

   server ~ # sophomorix-device -d firewall -i
   server ~ # sophomorix-device -r no-pxe -i

Überprüfe ob die Namensauflösung funktioniert:

.. code-block:: console

   server ~ # sophomorix-device --dns-test

10. Überprüfung von Benutzern und Gruppen
-----------------------------------------

Benutzer und Gruppen können mit folgendem Skript getestet werden:

.. code-block:: console

   server ~ # sophomorix-vampire --datadir /path/to/dir/sophomorix-dump --verify-uid

11. Synchronisiere Benutzerdaten
--------------------------------

Zunächst müssen über irgendein Verfahren die Verzeichnisse
``/home/share``, ``/home/teachers`` und ``/home/students`` vom
Quellsystem im Zielsystem unter einem Pfad (hier im Beispiel:
``/mnt``) erscheinen.

.. code-block:: console

   /mnt/home/share
   /mnt/home/students
   /mnt/home/teachers

Der Pfad im Zielsystem wird über das Kommandozeilenargument
``--path-oldserver /mnt`` an nachfolgende Skripte übergeben und
erwartet dann die obige Ordnerstruktur unterhalb von ``/mnt``.

Für einzelne Schüler, Lehrer, Klassen und Projekte sollte man ein
Synchronisieren testen: 

.. code-block:: console

   server ~ # sophomorix-vampire --rsync-student-home <studentname> --path-oldserver /mnt
   server ~ # sophomorix-vampire --rsync-teacher-home <teachername> --path-oldserver /mnt
   server ~ # sophomorix-vampire --rsync-class-share <classname> --path-oldserver /mnt
   server ~ # sophomorix-vampire --rsync-project-share <projectname> --path-oldserver /mnt

Jetzt können alle Schüler, Lehrer, Klassen und Projekte in einem Schritt importiert werden

.. code-block:: console

   server ~ # sophomorix-vampire --rsync-all-student-homes --path-oldserver /mnt
   server ~ # sophomorix-vampire --rsync-all-teacher-homes --path-oldserver /mnt
   server ~ # sophomorix-vampire --rsync-all-class-shares --path-oldserver /mnt
   server ~ # sophomorix-vampire --rsync-all-project-shares --path-oldserver /mnt

12. Synchronisiere LINBO-Daten
------------------------------

Alle Daten von LINBO können ebenso wie die Benutzerdaten aus dem
früheren Verzeichnis ``/var/linbo`` importiert werden. 

.. code-block:: console

   /mnt/var/linbo

Auch hier wird beispielsweise der Inhalt von ``/var/linbo`` in das
Zielsystem nach ``/mnt`` eingebunden. Das Skript erwartet dann die
obige Ordnerstruktur unterhalb von ``/mnt``.

.. code-block:: console

   server ~ # sophomorix-vampire --rsync-linbo --path-oldserver /mnt

Jetzt muss LINBO erneut installiert werden, um Änderungen,
die nur unter linuxmuster.net v7 existieren, importiert werden

.. code-block:: console

   server ~ # apt-get --reinstall install linuxmuster-linbo7 linuxmuster-linbo-common7

13. Dinge, die manuell gemacht werden müssen
--------------------------------------------

- Beschreibungen zu Projekten hinzufügen
- Die Rolle von Geräten festlegen
- Quota für die Benutzer (neu) festlegen
- Bei migrierten Subnetzen: Es muss in
  ``/etc/linuxmuster/subnets.csv`` das Gateway für das Servernetz
  eingetragen werden, z.B. 10.0.0.253 für einen L3-Switch. Danach muss
  ``linuxmuster-import-subnets`` ausgeführt werden.
