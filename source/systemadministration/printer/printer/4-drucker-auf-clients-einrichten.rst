Wie richte ich Drucker am Client ein?
=====================================

Windows Clients
---------------

Nachdem die Drucker auf dem Server eingerichtet wurden, sind diese auf Windows-Clients nun als Freigaben sichtbar.

.. image:: media/14-printer-share-windows.png
   :alt: Windows: Printer-Share
   :align: center

Die Treiber sind nun über die Microsoft Management Console (MMC) hinzuzufügen.
Dies erfolgt in der MMC im Menü `Druckerverwaltung` -> `hinzufügen`. 

.. hint::
   Zuerst sind Treiber hochzuladen. Es können ausschließlich v3 Druckertreiber verwendet werden. V4 Druckertreiber 
   werden Stand Samba 4.7 (September 2019) noch nicht unterstützt.

Falls o.g. Weg nicht funktionieren sollte, ist der Treiber manuell auf Windows Client zu installieren. 

Anschließend ist der Druckertreiber dem Drucker auf dem Server zuzuweisen.

Hierzu sind die eingerichteten Drucker auf dem Server zunächst auszugeben:

.. code::

   rppclient 10.0.0.1 -U "LINUXMUSTER\global-admin" -c "enumdrivers 3"

LINUXMUSTER stellt den Namen der eigenen Samba-Domäne dar, global-admin ist der Administrator auf dem Server, 
10.0.0.1 ist die IP des Server.

Danach ist der lokale Druckertreiber dem Drucker zuzuordnen - in nachstehendem Beispiel ist dies
der Druckertreiber `HP Universal Printing PS`:


.. code::

   rppclient 10.0.0.1 -U "LINUXMUSTER\global-admin" -c 'setdriver "DemoPrinter" "HP Universal Printing PS"'

Das Ergebnis kann mit dem zuvor genannten Befehl kontrolliert werden:

.. code::

   rppclient 10.0.0.1 -U "LINUXMUSTER\global-admin" -c "enumdrivers 3"

Danach ist der Druck zu testen. Funktioniert der Drucker wie gewünscht ist ein neues Image für den Windows-Client zu erstellen.


Linux-Clients
-------------

.. hint::

   still to be documented

Auszug:
Printer Driver Info 3:
Version: [3]

