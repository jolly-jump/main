.. include:: /guided_inst.subst

.. _prerequisites-label:

==================
 Vorüberlegungen
==================

.. sectionauthor:: `@cweikl <https://ask.linuxmuster.net/u/cweikl>`_,
		   `@Tobias <https://ask.linuxmuster.net/u/Tobias>`_

Linuxmuster.net wird als Zwei-Server-Lösung (Firewall und
linuxmuster.net-Server) betrieben. Optional können weitere Server wie
z. B. ein Docker-Host eingesetzt werden. Daneben gibt es mindestens
eine Trennung in zwei logische Netzwerke, meist sind aber drei oder
mehr davon gefordert (WLAN, DMZ, Lehrernetz). Zu guter Letzt kann
die linuxmuster.net bequem virtualisiert oder ohne Virtualisierung
betrieben werden.

Daraus leiten sich Voraussetzungen an Hardware, Netzwerkstrukturen und
Software ab, die in diesem Kapitel beleuchtet werden.

Hardware
========

OPNsense
--------

OPNSense ist für x86-32 und x86-64 Bit Architekturen verfügbar und
kann auf SD-Karte, SSD oder HDDs installiert werden. Es wird empfohlen
folgende Hardware-Anforderungen zu erfüllen, um die Mehrzahl der
Einsatzszenarien abzudecken:

+---------------------+-------------------------------------+
| Prozessor           | >= 1 GHz Multi-Core CPU (64 Bit)    |
+---------------------+-------------------------------------+
| RAM                 | >= 1 GB                             |
+---------------------+-------------------------------------+
|Installationsmethode | Video (VGA)                         |
+---------------------+-------------------------------------+
|Festplatte           | mind. 20GB, z.B. 120 GB SSD         |
+---------------------+-------------------------------------+
|NIC                  | mind. 2 (intern + extern),          |
|                     | oder  3 (intern + extern + WLAN)    |
+---------------------+-------------------------------------+

Weitere Hinweise zu möglichen Hardwareanforderungen bei
unterschiedlichen Einsatzszenarien finden sich `hier
<https://wiki.opnsense.org/manual/hardware.html#hardware-requirements>`_.

Als Basis nutzt OPNsense das Betriebssystem FreeBSD.  Hinweise zu den
Anforderungen von FreeBSD bzw. zur Kompatibilität mit eingesetzten
Hardware-Komponenten finden sich unter der `HCL - Hardware
Compatibility List
<https://www.freebsd.org/releases/11.1R/hardware.html>`_


Server linuxmuster v7
---------------------

Für linuxmuster.net v7 wird als Basis ein Ubuntu Server 18.04 LTS
eingesetzt. Es wird empfohlen folgende Hardware-Mindestanforderungen zu
erfüllen:

+---------------------+-------------------------------------+
| Prozessor           | >= 1 GHz Multi-Core CPU (64 Bit)    |
+---------------------+-------------------------------------+
| RAM                 | >= 4 GB                             |
+---------------------+-------------------------------------+
|Festplatte System +  | - vorkonfiguriert sind 25GB + 100GB | 
|Daten                | - mind. 500GB für Daten und Backup  |
|                     | - empfohlen >= 1TB                  |
+---------------------+-------------------------------------+

Docker-Host bzw. OPSI-Host auf Basis eines Ubuntu-Servers
---------------------------------------------------------

Es wird empfohlen, je Server folgende Hardware-Anforderungen zu
erfüllen:

+---------------------+-------------------------------------+
| Prozessor           | >= 1 GHz Multi-Core CPU (64 Bit)    |
+---------------------+-------------------------------------+
| RAM                 | >= 4 GB (OPSI), >= 1 GB (Docker)    |
+---------------------+-------------------------------------+
|Festplatte           | 100 GB+, nach Bedarf                |
+---------------------+-------------------------------------+

Für eine virtuelle Installation aller obigen Maschinen müssen die
Mindestwerte für die Hardware addiert werden.

Festplattenspeicher
-------------------

Der Festplattenplatz für den Server hängt stark von der Nutzerzahl und der intensiven Verwendung von LINBO-Abbildern ab. Ebenso muss für Backup weiterer Festplattenplatz z. B. auf einem NAS eingeplant werden.

Selbstverständlich können sowohl Daten als auch (bei Virtualisierung) die Server auf externem Speicher abgelegt werden (z. B. NFS-Speicher oder iSCSI-Speicher), um die Virtualisierungsumgebung ggf. bei Bedarf ausbauen zu können und auch ausfallsichere Szenarien leichter umsetzen zu können.

So *kann* bei minimaler Ausstattung (ohne Opsi und Docker) einer mittleren Schule (ca. 500 Benutzer) ein kleiner Server oder ein gut ausgestatteter PC ausreichend sein, selbst wenn alle Server virtualisiert laufen.

+---------------+-----------------+-----------------------+-----------------------+---------+----------+
| **Schule**    | **Features**    | **Festplatten**                               | **RAM**            |
|               |                 +-----------------------+-----------------------+---------+----------+
|               |                 | Standard              |Empfohlen              |Standard |Empfohlen |
+===============+=================+=======================+=======================+=========+==========+
| mittelgroß    | minimal         | ~650GB                | 1500GB+               | 8GB     | 16GB+    |
+---------------+-----------------+-----------------------+-----------------------+---------+----------+
| groß          | normal          | ~1000GB               | 2000GB+               | 10GB    | 16GB+    |
+---------------+-----------------+-----------------------+-----------------------+---------+----------+

..
  .. hint:: 
  Abbilder für drei verschiedene Hardwareklassen haben ca. 40G. Von
     jedem Image sollen drei Kopien vorgehalten werden, dann ist man
     schon bei 120G benötigtem Festplattenplatz alleine für die
     Arbeitsplätze.
  
     Auch im Verzeichnis ``/home`` oder im Cloud-Speicher sollte man
     Platz pro Benutzer einplanen. Bei 5GB für 100 Lehrer und 500MB für
     1000 Schüler kommt man auf weitere 1000GB.


.. _`net-infrastructure-label`:

Netzwerkstruktur
================

In Abhängigkeit vom Einsatzszenario muss die Netzwerkstruktur der linuxmuster.net
zu Beginn der Installation angepasst werden. Man sollte über den
Umfang der eingesetzten Geräte Bescheid wissen. Dementsprechend den IP-Bereich nicht zu klein wählen oder Subnetze einführen. Ebenso muss man den IP-Bereich auf die Umgebung (z. B. Verwaltungsnetz, extern vorgegebene Netze) abstimmen, damit keine Überschneidungen auftreten.

IP-Bereiche
-----------

Die linuxmuster.net-Lösung kann mit unterschiedlichen IP-Bereichen
arbeiten. Standardmäßig wird das interne Netz aus dem privaten
IPv4-Bereich 10.0.x.x mit der 16-bit Netzmaske 255.255.0.0 eingerichtet.
Die virtuellen Appliances sind mit diesem Netz voreingestellt.

Jedoch kann man sowohl die bisher in früheren Versionen von
linuxmuster.net verwendeten Netze (10.16.0.0/12 oder 10.32.0.0/12 
usw.) weiterverwenden. 

Sollten es zwingende Gründe erfordern, sind auch komplett andere private Adressbereiche
realisierbar.

Jede Zeile der folgenden Tabelle stellt eine Möglichkeit dar.

+-------------------+-----------------+------------+----------------------------------+
| Beginn IP-Bereich | Ende IP-Bereich | Server-IP  | Üblich in                        |
+===================+=================+============+==================================+
| 10.0.0.0          | 10.0.255.255    | 10.0.0.1   | voreingestellt in VMs von lmn-v7 |
+-------------------+-----------------+------------+----------------------------------+
| 10.16.0.0         | 10.31.255.255   | 10.16.1.1  | in linuxmuster.net < 7   üblich  |
+-------------------+-----------------+------------+----------------------------------+
| 10.32.0.0         | 10.47.255.255   | 10.32.1.1  | in linuxmuster.net < 7   möglich |
+-------------------+-----------------+------------+----------------------------------+
| ...               | ...             | ...        | ...                              |
+-------------------+-----------------+------------+----------------------------------+
| 192.168.0.0       | 192.168.255.255 | 192.168.0.1| nicht üblich                     |
+-------------------+-----------------+------------+----------------------------------+

Bei der Neuinstallation entscheidest du dich für einen der Bereiche.
Bei einer Migration wird empfohlen den früheren Bereich zu behalten,
alleine schon um eine erneute Konfiguration der Netzwerkswitche zu
vermeiden.

Standard IP-Adressen
--------------------

Einige IP-Adressen sind standardmäßig für spezielle Server/Dienste
vorgesehen.

+------------+---------------+--------------+
| **Server** |**IP-Bereich** |**IP-Bereich**|
|            |10.0.0.0/16    |10.16.0.0/12  |
+============+===============+==============+
| OPNsense   | 10.0.0.254    | 10.16.1.254  |
+------------+---------------+--------------+
| Server     | 10.0.0.1      | 10.16.1.1    |
+------------+---------------+--------------+
| Opsi       | 10.0.0.2      | 10.16.1.2    |
+------------+---------------+--------------+
| Dockerhost | 10.0.0.3      | 10.16.1.3    |
+------------+---------------+--------------+
| XOA (*)    | 10.0.0.4      | 10.16.1.4    |
+------------+---------------+--------------+
| Admin-PC   | 10.0.0.10     | 10.16.1.10   |
+------------+---------------+--------------+

.. hint::

   (*) Die XenOrchestra-Appliance (XOA) wird nur benötigt, wenn eine
   Virtualisierung mit XCP-ng erfolgen soll. Mithilfe von XenOrchestra
   kann die Virtualisierungsumgebung XCP-ng web-basiert verwaltet
   werden und es können hierüber auch sog. Enterprise-Funktionen wie
   z.B. Backup, Replikation etc. konfiguriert werden.



Netz-Grundstruktur
------------------

Aus historischen und anschaulichen Gründen verwendet die
linuxmuster.net in der Dokumentation weiterhin die Farbzuordnung, die
durch die Firewall-Lösung "IPFire" geprägt wurde:

.. figure:: media/simple-network.png
   :align: center
   :alt: Schematischer Aufbau eines Computernetzes mit linuxmuster.net.

   Schematischer Aufbau eines Computernetzes mit linuxmuster.net.


* Das interne Netzwerk wird GRÜNES Netzwerk genannt 
* Das externe Netzwerk wird ROTES Netzwerk genannt, es ist über einen Router mit dem Internet verbunden.
* Optional kann z.B. für WLAN-Accesspoints ein weiteres Netzwerk
  aufgebaut werden (BLAU/LILA), für welches andere
  Zugangsberechtigungen als im grünen Netzwerk gelten.
* Optional kann eine sog. demilitarisierte Zone (DMZ) als zusätzliches Netzwerk (ORANGE) aufgebaut werden.

Daraus ergeben sich folgende Mindestvoraussetzungen für einen Virtualisierungshost:

* mindestens zwei Netzwerk-Interfaces (rotes und grünes Netz)
* bei WLAN-Nutzung eine Netzwerkkarte zusätzlich (blaues Netz)
* sollen Serverdienste im Internet von außen zugänglich sein, empfehlen wir diese in die DMZ auszulagern. Dafür wird eine weiteres 
  Netzwerk-Interface benötigt (oranges Netz)

Durch die fortschreitende Digitalisierung in der Bildung ist der Auf- bzw. Ausbau einer funktionalen WLAN-Infrastruktur für jede Schule eine gute Entscheidung. Daraus ergibt sich aus unserer Sicht die Empfehlung zu mindestens drei Netzwerkkarten. Willst du für alle möglichen Einsatzszenarien gut gerüstet sein, empfiehlt sich allerdings gleich den Virtualisierungshost mit vier Netzwerk-Interfaces auszulegen.

Das obige Prinzip ist bereits ein Beispiel für Netzwerksegmentierung,
das im nächsten Abschnitt näher erläutert wird.


Getrennte Netze und VLAN
------------------------

Immer häufiger (z. B. durch Vorgaben vom Kultusministerium oder
Lastverteilung) besteht Bedarf an einer weiteren Trennung des internen
Netzes in mehrere logisch voneinander relativ getrennte
Netze. Neben den relativ stark abgetrennten Netzen für WLAN oder eine
demilitarisierte Zone (DMZ) wie oben abgebildet, erlaubt
linuxmuster.net Lösung sehr flexibel eine beliebige Einteilung des
Schulnetzes in Subnetze.

Wer vor der Entscheidung steht, Subnetze oder VLANs einzurichten,
sollte das Kapitel :ref:`Netzsegmentierung mit linuxmuster.net
<subnetting-basics-label>` lesen.


Virtualisierung
===============

Wenn man linuxmuster.net virtualisiert betreibt, gelten zu den obigen
Voraussetzungen noch folgende Hinweise:

* Das Netzwerk wird virtualisiert. Dadurch werden virtuelle Switche ("bridges") erstellt, denen die richtigen Schnittstellen zugeordnet werden müssen.
  Wird kein Layer3-Switch eingesetzt, sollte der Virtualisierungshost (Hypervisor) wenigstens mit der obengenannten Anzahl von Netzwerkkarten ausgestattet sein.
  Mit dem Einsatz eines Layer3-Switches wird die Konfiguration auf dem Hypervisor schnell komplex, die physikalische Verkabelung kann dadurch aber einfacher werden. So lassen sich auch etwaige neue Anforderungen durch zusätzliche VLANs realisieren.

* Der Speicherplatz wird virtualisiert. Darauf muss man bei der Verwendung externer (iSCSI/NFS) wie interner Speichersysteme (LVM) achten. Dies kann auch zur Vereinfachung eines Backupverfahrens beitragen.

* Da der VM-Host die einzelnen VMs zunächst kapselt, ist es aus Sicherheitsgründen empfehlenswert, den VM-Host nicht ins selbe Netz seiner VMs einzubinden. Außerdem wird der Zugriff auf die Daten eventuell erschwert.

Hypervisoren
------------

Die Voraussetzungen für einen virtualisierten Betrieb besteht
natürlich darin, vorab den Hypervisor/den VM-Host installiert zu haben
und Zugriff auf dessen Verwaltung zu haben. Wo es uns möglich ist,
haben wir eine Anleitung dazu geschrieben, um auf die Besonderheiten
der Schulnetzumgebung an geeigneter Stelle hinzuweisen.

VMs - Hinweise
--------------

linuxmuster.net bietet vorgefertigte virtuelle Maschinen zum direkten Import für die jeweilige Virtualisierungsumgebung an, um die Installations- und Konfigurationszeiten stark zu verringern.

- Für die Virtualisierer KVM, Proxmox, VirtualBox:
  werden die VMs im OVA-Format bereitgestellt. Dieses Format kann i.d.R. auch von
  anderen Virtualisierern erfolgreich importiert werden.
- Für XCP-ng als Opensource-Virtualisierungsumgebung werden die VMs im
  XVA-Format zum direkten Import angeboten. Die XVA-Dateien sind
  zusätzlich mit ZIP komprimiert worden.
- Für alle Virtualisierer, für die kein Import möglich ist, bietet es
  sich entweder an, die OVA-Dateien händisch zu entpacken und die
  Einstellungen händisch einzurichten, oder eine Installation von
  Grund auf zu starten.

Zu den jeweiligen Download-Dateien der VMs werden ebenfalls die
SHA1-Werte zur Überprüfung der Datenintegrität bereitgestellt.

Installations-Alternativen
==========================

Für eine Installation direkt auf der Hardware oder einer anderweitigen
Installation von Grund auf benötigt man:

- `Ubuntu 18.04 LTS 64-bit PC (AMD64) server install image
  <http://releases.ubuntu.com/bionic/>`_

- `OpnSense <https://opnsense.org/download>`_

Das dafür nötige Vorgehen kann auch der Entwicklerdokumentation entnommen werden.
Dort ist auch das Vorgehen um VirtualBox als Hypervisor einzusetzen beschrieben.

VirtualBox wird häufig im Testbetrieb und weniger im Produktivbetrieb verwendet. Die `Entwicklerdokumentation <https://github.com/linuxmuster/linuxmuster-base7/wiki/Die-Appliances>`_ beschreibt diese Konfiguration. Es muss mindestens die Virtualbox-Version 6.0 verwendet werden.

Das weitere Vorgehen
====================

Nachdem du entschieden hast, ob und wie du eine Virtualisierung
einsetzt, beginnst du mit Installation der Virtualisierung nach einer
der oben beschriebenen Anforderungen der einzelnen Hypervisoren in dieser Dokumentation.

=========================== ===================
Virtualisierung mit Proxmox |follow_me2proxmox|
Virtualisierung mit XCP-ng  |follow_me2xcp-ng|
Virtualisierung mit KVM     |follow_me2kvm|
=========================== ===================

Alternativ installierst du von Grund auf die Serverbetriebssysteme
*Ubuntu Server* und *OPNSense* direkt auf der Hardware oder innerhalb
deiner Virtualisierungslösung.

=========================== ===================
Alternative Installationen  |follow_me2scratch|
=========================== ===================

..
  Checkliste
  ==========

  Nutzen Sie die :download:`Checkliste
  <./media/preamble/checklist/checklist.pdf>`, um alle während der
  Installation gemachten Einstellungen festzuhalten. Es handelt sich um
  ein PDF-Formular, Sie können es also auch am PC ausfüllen. Halten Sie
  diese Checkliste bereit, wenn Sie den Telefon-Support in Anspruch
  nehmen wollen.
