---
lab:
  title: 'Lab: Implementieren von Azure Private Link für Azure Virtual Desktop'
  module: 'Module 1.1: Plan, implement, and manage networking for Azure Virtual Desktop'
---

# Lab – Implementieren von Azure Private Link für Azure Virtual Desktop
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzende in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den erforderlichen Berechtigungen, um Geräte mit dem Entra-Mandanten zu verbinden, der diesem Azure-Abonnement zugeordnet ist.
- Das Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals* wurde abgeschlossen

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie verfügen über eine vorhandene Azure Virtual Desktop-Umgebung. Sie müssen die Verbindung mit der Umgebung mithilfe von Azure Private Link implementieren. 

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Implementieren von Azure Private Link für Azure Virtual Desktop

## Labdateien

- Keine

## Anweisungen

### Übung 1: Implementieren von Azure Private Link für Azure Virtual Desktop
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Erneutes Registrieren des Azure Virtual Desktop-Ressourcenanbieters
1. Erstellen eines Subnetzes für ein Azure Virtual Network
1. Implementieren eines privaten Endpunkts für Verbindungen mit einem Hostpool
1. Implementieren eines privaten Endpunkts für den Feeddownload
1. Implementieren eines privaten Endpunkts für die anfängliche Feedsuche
1. Überprüfen der Funktionalität des privaten Endpunkts
1. Zugriff auf öffentliche Netzwerke auf einen Hostpool und Arbeitsbereich zulassen

> **Hinweis**: Azure Virtual Desktop verfügt über drei Workflows mit drei entsprechenden Ressourcentypen, die mit privaten Endpunkten verwendet werden können. Dies sind die Workflows:

- **Anfängliche Feedsuche**: Ermöglicht RDP-Clients, alle Arbeitsbereiche zu ermitteln, die einer benutzenden Person zugewiesen sind. Um diesen Workflow über einen privaten Link zu implementieren, müssen Sie einen einzelnen privaten Endpunkt zur globalen Unterressource in einem beliebigen Arbeitsbereich erstellen, der Teil Ihrer Azure Virtual Desktop-Bereitstellung ist. Unabhängig vom gewählten Arbeitsbereich kann es jedoch nur einen einzigen privaten Endpunkt geben, der diese Funktionalität pro Bereitstellung bereitstellt.
- **Feeddownload**: ermöglicht RDP-Clients das Herunterladen von Verbindungsdetails für alle Arbeitsbereiche, die die Anwendungsgruppen der aktuellen benutzenden Person hosten. Um diesen Workflow über einen privaten Link zu implementieren, müssen Sie einen privaten Endpunkt für die Feedunterressource für jeden Arbeitsbereich erstellen, den Sie über den privaten Endpunkt zur Verfügung stellen möchten.
- **Verbindungen zu Hostpools**: ermöglicht RDP-Clients und Sitzungshosts die Verbindung zu einem Hostpool. Um diesen Workflow über einen privaten Link zu implementieren, müssen Sie für jeden Hostpool, den Sie über den privaten Endpunkt verfügbar machen möchten, einen privaten Endpunkt für die Verbindungs-Unterressource erstellen.

> **Hinweis**: Sie können diese Workflows in den folgenden Anordnungen implementieren:

- Für alle Teile der Verbindung werden private Routen verwendet (anfängliche Feedermittlung, Feeddownload und Remotesitzungsverbindungen für Clients und Sitzungshosts).
- Feeddownload- und Remotesitzungsverbindungen für Clients und Sitzungshosts verwenden private Routen, aber für die anfängliche Feedermittlung werden öffentliche Routen verwendet. 
- Nur Remotesitzungsverbindungen für Clients und Sitzungshosts verwenden private Routen, aber für die anfängliche Feedermittlung und den Feeddownload werden öffentliche Routen verwendet.
- Sowohl Clients als auch Sitzungshost-VMs verwenden öffentliche Routen. Private Link wird in diesem Szenario nicht verwendet.

> **Hinweis**: In diesem Lab implementieren Sie die erste Anordnung.

#### Aufgabe 1: Erneutes Registrieren des Azure Virtual Desktop-Ressourcenanbieters

> **Hinweis**: Bevor Sie Private Link mit Azure Virtual Desktop verwenden können, sollten Sie den Ressourcenanbieter **Microsoft.DesktopVirtualization** neu registrieren. 

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich an, indem Sie die Anmeldedaten eines Benutzerkontos mit der Besitzerrolle in dem Abonnement angeben, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Abonnements**, wählen Sie diese aus, wählen Sie auf der Seite **Abonnements** das Azure-Abonnement aus, das Sie in diesem Lab verwenden, und wählen Sie im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Ressourcenanbieter** aus.
1. Geben Sie auf der Registerkarte **Ressourcenanbieter** im Suchtextfeld **Microsoft.DesktopVirtualization** ein, wählen Sie in der Ergebnisliste den kleinen Kreis links neben dem Eintrag **Microsoft.DesktopVirtualization**“ aus und wählen Sie dann **Erneut registrieren** aus.

    > **Hinweis**: Warten Sie, bis der Prozess der erneuten Registrierung abgeschlossen ist. Dieser Schritt dauert normalerweise weniger als eine Minute.

#### Aufgabe 2: Erstellen eines Subnetzes für Azure Virtual Network

> **Hinweis**: Sie könnten ein vorhandenes Subnetz eines virtuellen Azure-Netzwerks verwenden, um private Endpunkte im Lab-Szenario zu implementieren, aber es ist gängige Praxis, ein dediziertes Subnetz für diesen Zweck zu verwenden.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **virtuellen Netzwerken**, wählen Sie diese aus und wählen Sie auf der Seite **Virtuelle Netzwerke** **az140-vnet11e** aus.
1. Wählen Sie auf der Seite **az140-vnet11e** im Abschnitt **Einstellungen** des vertikalen Navigationsmenüs **Subnetze** aus.
1. Wählen Sie auf der Seite **az140-vnet11e \| Subnetze** **+ Subnetz** aus.
1. Im Bereich **Subnetz hinzufügen** die folgenden Einstellungen angeben und **Hinzufügen** auswählen (andere Einstellungen mit ihren Standardwerten belassen):

    |Einstellung|Wert|
    |---|---|
    |Name|**pe-Subnet**|
    |Startadresse|**10.20.255.0**|
    |Privates Subnetz aktivieren (kein standardmäßiger ausgehender Zugriff)|Disabled|

#### Aufgabe 3: Implementieren eines privaten Endpunkts für Verbindungen mit einem Hostpool

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop** und wählen Sie es aus. Wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Hostpools** und auf der Seite **Azure Virtual Desktop \| Hostpools** **az140-21-hp1** aus. 
1. Wählen Sie auf der Seite **az140-21-hp1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-hp1 \| Networking** die Registerkarte **Private Endpunktverbindungen** aus und wählen Sie dann **+ Neuer privater Endpunkt**.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Ressource >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-11e-RG**|
    |Name|**az140-11-pehp1**|
    |Name der Netzwerkschnittstelle|**az140-11-pehp1-nic**|
    |Region|Name der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitgestellt haben|

1. Geben Sie auf der Registerkarte **Ressource** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Virtuelles Netzwerk >** aus:

    |Einstellung|Wert|
    |---|---|
    |Zielunterressource|**Verbindung**|

1. Geben Sie auf der Registerkarte **Virtuelles Netzwerk** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: DNS >** aus (belassen Sie die anderen Einstellungen auf ihren Standardwerten):

    |Einstellung|Wert|
    |---|---|
    |Virtuelles Netzwerk|**az140-vnet11e (az140-11e-RG)**|
    |Subnet|**pe-Subnet**|
    |Netzwerkrichtlinie für private Endpunkte|**Disabled**|
    |Konfiguration der privaten IP-Adresse|**Dynamisches Zuweisen von IP-Adressen**|

1. Geben Sie auf der Registerkarte **DNS** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Tags >** aus:

    |Einstellung|Wert|
    |---|---|
    |Integration in eine private DNS-Zone|**Ja**|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-11e-RG**|

    > **Hinweis**: Dieser Schritt führt zur Erstellung einer privaten DNS-Zone mit dem Namen **privatelink.wvd.microsoft.com**.

1. Wählen Sie auf der Registerkarte **Tags** der Seite **Erstellen eines privaten Endpunkts****Weiter: Überprüfen + erstellen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** der Seite **Erstellen eines privaten Endpunkts** **Erstellen** aus.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Die Bereitstellung dauert ungefähr drei Minuten.

    > **Hinweis**: Sie müssen für jeden Hostpool, den Sie mit Private Link verwenden möchten, einen privaten Endpunkt für die Verbindungsunterressource erstellen.

#### Aufgabe 4: Implementieren eines privaten Endpunkts für den Feeddownload

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Arbeitsbereiche** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Arbeitsbereiche** **az140-21-ws1** aus.
1. Wählen Sie auf der Seite **az140-21-ws1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-ws1 \| Networking** die Registerkarte **Private Endpunktverbindungen** aus und wählen Sie dann **+ Neuer privater Endpunkt**.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Ressource >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-11e-RG**|
    |Name|**az140-11-pefeeddwnld**|
    |Name der Netzwerkschnittstelle|**az140-11-pefeeddwnld-nic**|
    |Region|Name der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitgestellt haben|

1. Geben Sie auf der Registerkarte **Ressource** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Virtuelles Netzwerk >** aus:

    |Einstellung|Wert|
    |---|---|
    |Zielunterressource|**feed**|

1. Geben Sie auf der Registerkarte **Virtuelles Netzwerk** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: DNS >** aus (belassen Sie die anderen Einstellungen auf ihren Standardwerten):

    |Einstellung|Wert|
    |---|---|
    |Virtuelles Netzwerk|**az140-vnet11e (az140-11e-RG)**|
    |Subnet|**pe-Subnet**|
    |Netzwerkrichtlinie für private Endpunkte|**Disabled**|
    |Konfiguration der privaten IP-Adresse|**Dynamisches Zuweisen von IP-Adressen**|

1. Geben Sie auf der Registerkarte **DNS** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Tags >** aus:

    |Einstellung|Wert|
    |---|---|
    |Integration in eine private DNS-Zone|**Ja**|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-11e-RG**|

    > **Hinweis**: In diesem Schritt wird die private DNS-Zone mit dem Namen **privatelink.wvd.microsoft.com** genutzt, die Sie in der vorherigen Aufgabe erstellt haben.

1. Wählen Sie auf der Registerkarte **Tags** der Seite **Erstellen eines privaten Endpunkts****Weiter: Überprüfen + erstellen** aus.
1. Wälen Sie auf der Registerkarte **Überprüfen + erstellen** auf der Seite **Erstellen eines privaten Endpunkts ** **Erstellen** aus.

    > **Hinweis**: Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefährt eine Minute dauern.

    > **Hinweis**: Sie müssen für jeden Arbeitsbereich, den Sie mit Private Link verwenden möchten, einen privaten Endpunkt für die Feed-Unterressource erstellen.

#### Aufgabe 5: Implementieren eines privaten Endpunkts für die anfängliche Feedsuche

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Arbeitsbereiche** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Arbeitsbereiche** **az140-21-ws1** aus.
1. Wählen Sie auf der Seite **az140-21-ws1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-ws1 \| Networking** die Registerkarte **Private Endpunktverbindungen** aus und wählen Sie dann **+ Neuer privater Endpunkt**.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Ressource >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-11e-RG**|
    |Name|**az140-11-pefeeddisc**|
    |Name der Netzwerkschnittstelle|**az140-11-pefeeddisc-nic**|
    |Region|Name der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitgestellt haben|

1. Geben Sie auf der Registerkarte **Ressource** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Virtuelles Netzwerk >** aus:

    |Einstellung|Wert|
    |---|---|
    |Zielunterressource|**global**|

1. Geben Sie auf der Registerkarte **Virtuelles Netzwerk** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: DNS >** aus (belassen Sie die anderen Einstellungen auf ihren Standardwerten):

    |Einstellung|Wert|
    |---|---|
    |Virtuelles Netzwerk|**az140-vnet11e (az140-11e-RG)**|
    |Subnet|**pe-Subnet**|
    |Netzwerkrichtlinie für private Endpunkte|**Disabled**|
    |Konfiguration der privaten IP-Adresse|**Dynamisches Zuweisen von IP-Adressen**|

1. Geben Sie auf der Registerkarte **DNS** der Seite **Erstellen eines privaten Endpunkts** die folgenden Einstellungen an und wählen Sie **Weiter: Tags >** aus:

    |Einstellung|Wert|
    |---|---|
    |Integration in eine private DNS-Zone|**Ja**|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-11e-RG**|

    > **Hinweis**: Dieser Schritt führt zur Erstellung einer privaten DNS-Zone mit dem Namen **privatelink.wvd.microsoft.com**.

1. Wählen Sie auf der Registerkarte **Tags** der Seite **Erstellen eines privaten Endpunkts****Weiter: Überprüfen + erstellen** aus.
1. Wälen Sie auf der Registerkarte **Überprüfen + erstellen** auf der Seite **Erstellen eines privaten Endpunkts ** **Erstellen** aus.

    > **Hinweis**: Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefährt eine Minute dauern.

    > **Hinweis**: Sie müssen für jeden Arbeitsbereich, den Sie mit Private Link verwenden möchten, einen privaten Endpunkt für die globale Unterressource erstellen.

    > **Hinweis**: Damit die Netzwerkänderungen wirksam werden, müssen Sie die Sitzungshosts im Zielhostpool neu starten.

1. Navigieren Sie vom Lab-Computer aus im Webbrowser, der das Azure-Portal anzeigt, zur Seite **Azure Virtual Desktop**, wählen Sie im Abschnitt **Verwalten** des vertikalen Navigationsmenüs ** Hostpools** und auf der Seite **Azure Virtual Desktop \| Hostpools** **az140-21-hp1** auswählen.
1. Wählen Sie auf der Seite **az140-21-hp1** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Sitzungshosts** aus. 
1. Aktivieren Sie in der Liste der Sitzungshosts alle Kontrollkästchen links neben den einzelnen Sitzungshosts und wählen Sie dann in der Symbolleiste **Neu starten** aus.

    > **Hinweis:** Warten Sie, bis sich alle Sitzungshosts im Status **wird ausgeführt** befinden. 

#### Aufgabe 6: Überprüfen der Funktionalität des privaten Endpunkts

> **Hinweis**: Standardmäßig ist die Konnektivität mit Azure Virtual Desktop-Arbeitsbereichen und Hostpools aus öffentlichen Netzwerken zulässig. Zunächst ändern Sie die Standardeinstellungen und erzwingen den privaten Zugriff.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Arbeitsbereiche** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Arbeitsbereiche** **az140-21-ws1** aus.
1. Wählen Sie auf der Seite **az140-21-ws1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-ws1 \| Networking** auf der Registerkarte **Öffentlicher Zugriff** die Option **Öffentlicher Zugriff deaktivieren und privaten Zugriff verwenden** aus und wählen Sie dann **Speichern** aus.
1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop** und wählen Sie es aus. Wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Hostpools** und auf der Seite **Azure Virtual Desktop \| Hostpools** **az140-21-hp1** aus. 
1. Wählen Sie auf der Seite **az140-21-hp1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-hp1 \| Networking** auf der Registerkarte **Öffentlicher Zugriff** die Option **Öffentlicher Zugriff deaktivieren und privaten Zugriff verwenden** aus und wählen Sie dann **Speichern** aus.

    > **Hinweis**: Zur Prüfung der Funktionalität des privaten Endpunkts muss ein RDP-Client mit einem Netzwerk verbunden sein, das über eine private Verbindung zum virtuellen Azure-Netzwerk verfügt, das das Subnetz mit den zuvor in diesem Lab erstellten privaten Endpunkten enthält. Um dieses Szenario zu simulieren, erstellen Sie ein weiteres Subnetz im selben virtuellen Netzwerk, das zum Erstellen privater Endpunkte und zum Bereitstellen einer Azure-VM mit Windows 11 in diesem Subnetz verwendet wird.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **virtuellen Netzwerken**, wählen Sie diese aus und wählen Sie auf der Seite **Virtuelle Netzwerke** **az140-vnet11e** aus.
1. Wählen Sie auf der Seite **az140-vnet11e** im Abschnitt **Einstellungen** des vertikalen Navigationsmenüs **Subnetze** aus.
1. Wählen Sie auf der Seite **az140-vnet11e \| Subnetze** **+ Subnetz** aus.
1. Im Bereich **Subnetz hinzufügen** die folgenden Einstellungen angeben und **Hinzufügen** auswählen (andere Einstellungen mit ihren Standardwerten belassen):

    |Einstellung|Wert|
    |---|---|
    |Name|**Clientsubnetz**|
    |Startadresse|**10.20.2.0**|
    |Privates Subnetz aktivieren (kein standardmäßiger ausgehender Zugriff)|Disabled|

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **virtuellen Maschinen**, wählen Sie diese aus, wählen Sie auf der Seite **Virtuelle Maschinen** die Option **+ Erstellen** aus und wählen Sie in der Dropdownliste die Option **Azure Virtual Machine** aus.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Erstellen eines virtuellen Computers** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten) und wählen Sie **Weiter: Datenträger >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe **az140-111e-RG**|
    |Name des virtuellen Computers|**az140-111e-vm0**|
    |Region|Name der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitgestellt haben|
    |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
    |Sicherheitstyp|**Standard**|
    |Abbildung|**Windows 11 Pro, Version 24H2 – x64 Gen2**|
    |Größe|**Standard DC2s_v3**|
    |Benutzername|Beliebiger gültiger Benutzername Ihrer Wahl|
    |Kennwort|Ein beliebiges gültiges Kennwort Ihrer Wahl|
    |Öffentliche Eingangsports|**Keine**|
    |Lizenzierung|Aktivieren Sie das Kontrollkästchen|

    > **Hinweis**: Das Kennwort sollte mindestens 12 Zeichen lang sein und aus einer Kombination von Kleinbuchstaben, Großbuchstaben, Ziffern und Sonderzeichen bestehen. Ausführliche Informationen finden Sie unter den Informationen zu [den Kennwortanforderungen beim Erstellen einer Azure-VM](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Stellen Sie auf der Registerkarte **Datenträger** auf der Seite **Erstellen eines virtuellen Computers** den **Betriebssystem-Datenträgertyp** auf **HDD Standard (lokal redundanter Speicher)** ein und wählen Sie **Weiter: Networking >** aus.
1. Geben Sie auf der Registerkarte **Networking** der Seite **Erstellen einer virtuellen Maschine** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten):

    |Einstellung|Wert|
    |---|---|
    |Virtuelles Netzwerk|**az140-vnet11e**|
    |Subnet|**Clientsubnetz**|
    |Öffentliche IP-Adresse|**(new) az140-111e-vm0-ip**|
    |NIC-Netzwerksicherheitsgruppe|**Advanced**|

1. Wählen Sie auf der Registerkarte **Netzwerk** der Seite **Erstellen einer virtuellen Maschine** neben der Dropdownliste **Netzwerksicherheitsgruppe konfigurieren** die Option **Neu erstellen** aus.
1. Löschen Sie auf der Seite **Erstellen einer Netzwerksicherheitsgruppe** die vorab erstellte eingehende Regel **1000: default-allow-rdp** und wählen Sie dann **+ Hinzufügen einer eingehenden Sicherheitsregel** aus.
1. Wählen Sie im Bereich **Hinzufügen einer eingehenden Sicherheitsregel** in der Dropdownliste „**Quelle** die Option **Meine IP-Adresse** aus, um die öffentliche IP-Adresse zu identifizieren, die Ihre Verbindung zum Internet darstellt.
1. Geben Sie im Bereich **Hinzufügen einer eingehenden Sicherheitsregel** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten) und wählen Sie dann **Hinzufügen** aus:

    |Einstellung|Wert|
    |---|---|
    |`Source`|**IP-Adressen**|
    |IP-Quelladressen/CIDR-Bereiche|Unverändert lassen (dies sollte weiterhin Ihre öffentliche IP-Adresse enthalten)|
    |Quellportbereiche|*|
    |Destination|**Alle**|
    |Dienst|**RDP**|
    |Aktion|**Zulassen**|
    |Priorität|**300**|
    |Name|**AllowCidrBlockRDPInbound**|

1. Zurück auf der Seite **Erstellen einer Netzwerk-Sicherheitsgruppe ** wählen Sie **OK** aus.
1. Zurück auf der Registerkarte **Networking** auf der Seite **Erstellen eines virtuellen Computers** wählen Sie **Weiter: Management >** aus:
1. Geben Sie auf der Registerkarte **Management** der Seite **Erstellen eines virtuellen Computers** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten) und wählen Sie dann **Weiter: Überwachung >** aus:

    |Einstellung|Wert|
    |---|---|
    |Kostenlosen Basic-Plan aktivieren|deaktiviert|
    |Optionen zur Patchorchestrierung|**Manuelle Updates**|

1. Geben Sie auf der Registerkarte **Überwachung** auf der Seite **Erstellen einer virtuellen Maschine** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten) und wählen Sie dann **Überprüfen + erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Startdiagnose|**Deaktivieren**|

1. Wälhen Sie auf der Registerkarte **Überprüfen + erstellen** der Seite **Erstellen einer virtuellen Maschine** **Erstellen** aus.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Die Bereitstellung kann ungefähr 5 Minuten dauern.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Virtuelle Maschinen**, und wählen Sie diese aus. Wählen Sie auf der Seite **Virtuelle Maschinen** **az140-111e-vm0** aus.
1. Wählen Sie auf der Seite **az140-111e-vm0** die Option **Verbinden** und wählen Sie im Dropdownmenü die Option **Verbinden** aus.
1. Wälhen Sie auf der Seite **az140-111e-vm0 \| Verbinden** im Abschnitt **Häufigste** **RDP-Datei herunterladen** aus.
1. Wälhen Sie im Popupfenster **Download** **Speichern** und dann **Datei öffnen** aus.
1. Bei Eingabeaufforderung **Verbinden** auswählen und dann im Dialogfeld **Windows-Sicherheit** den Benutzernamen und das Kennwort eingeben, die Sie bei der Bereitstellung der Azure-VM angegeben haben.
1. Bei der Eingabeaufforderung zur Bestätigung erneut **Verbinden** auswählen.
1. Wählen Sie innerhalb der Remote-Desktop-Sitzung zu **az140-111e-vm0** Ihre bevorzugten Datenschutzeinstellungen aus und akzeptieren Sie diese.
1. Starten Sie innerhalb der Remote-Desktop-Sitzung zu **az140-111e-vm0** Microsoft Edge, navigieren Sie zu der Seite [Verbindung mit Azure Virtual Desktop über den Remote-Desktop-Client für Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), und scrollen Sie nach unten zum Abschnitt **Herunterladen und Installieren des Remotedesktop-Clients (MSI)**, und wählen Sie den Link[Windows 64-bit](https://go.microsoft.com/fwlink/?linkid=2139369) aus. 
1. Öffnen Sie den Datei-Explorer, navigieren Sie zum Ordner **Downloads** und starten Sie die Installation der neu heruntergeladenen MSI-Datei. 
1. Akzeptieren Sie bei der Eingabeaufforderung die Bedingungen der Lizenzvereinbarung und wählen Sie die Option**Für alle Benutzenden dieses Computers installieren** aus. Wenn Sie dazu aufgefordert werden, akzeptieren Sie die Eingabeaufforderung zur Benutzerkontensteuerung, um mit der Installation fortzufahren. 
1. Stellen Sie nach Abschluss der Installation sicher, dass das Kontrollkästchen **Remotedesktop starten, wenn das Setup beendet ist** aktiviert ist und wählen Sie **Beenden** um den Microsoft Remotedesktop-Client zu starten.
1. Wählen Sie in der Remote-Desktop-Sitzung zu **az140-111e-vm0**, im Clientfenster **Remotedesktop**die Option**Abonnieren** und melden Sie sich bei der Eingabeaufforderung mit den Anmeldeinformationen des `User2` Entra-ID-Benutzerkontos an, das Sie auf der Registerkarte **Ressourcen** im rechten Bereich des Fensters der Lab-Benutzeroberfläche finden.

   > **Hinweis**: Wählen Sie das Benutzerkonto aus, das Mitglied der Entra-Gruppe mit dem Präfix **AVD-RemoteApp** ist.

1. Stellen Sie sicher, dass auf der Seite **Remotedesktop** vier Symbole angezeigt werden, darunter Eingabeaufforderung, Microsoft Word, Microsoft Excel und Microsoft PowerPoint. 
1. Doppelklicken Sie auf das Eingabeaufforderungssymbol. 
1. Geben Sie bei der Eingabeaufforderung zur Anmeldung im Dialogfeld **Windows-Sicherheit** das Kennwort des gleichen Microsoft Entra-Benutzerkontos ein, das Sie für die Verbindung mit der Zielumgebung von Azure Virtual Desktop verwendet haben.
1. Überprüfen Sie, ob kurz darauf ein **Eingabeaufforderungsfenster** angezeigt wird. 
1. Geben Sie an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.

   > **Hinweis**: Optional können Sie versuchen, den Feed zu abonnieren und eine Verbindung mit dem Azure Virtual Desktop-Arbeitsbereich vom Lab-Computer herzustellen, um zu überprüfen, ob diese Verbindung fehlschlägt. 

#### Aufgabe 7: Zulassen des Zugriffs auf ein öffentliches Netzwerk auf einen Hostpool und einen Arbeitsbereich

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Arbeitsbereiche** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Arbeitsbereiche** **az140-21-ws1** aus.
1. Wählen Sie auf der Seite **az140-21-ws1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-ws1 \| Networking** auf der Registerkarte **Öffentlicher Zugriff** die Option **Öffentlichen Zugriff von allen Netzwerken aus aktivieren** aus und wählen Sie dann **Speichern** aus.
1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop** und wählen Sie es aus. Wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Hostpools** und auf der Seite **Azure Virtual Desktop \| Hostpools** **az140-21-hp1** aus. 
1. Wählen Sie auf der Seite **az140-21-hp1** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Netzwerk** aus.
1. Wählen Sie auf der Seite **az140-21-hp1 \| Networking** auf der Registerkarte **Öffentlicher Zugriff** die Option **Öffentlichen Zugriff von allen Netzwerken aus aktivieren** aus und wählen Sie dann **Speichern** aus.
