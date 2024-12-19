---
lab:
  title: 'Lab: Verwalten von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Lab – Verwalten von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzerrolle oder Teilnehmerrolle in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den ausreichenden Berechtigungen, um Geräte dem mit diesem Azure-Abonnement verbundenen Entra-Mandanten beizutreten.
- Das Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals* wurde abgeschlossen

## Geschätzte Dauer

30 Minuten

## Labszenario

Sie verfügen über eine vorhandene Azure Virtual Desktop-Umgebung. Sie müssen Hostpools mit in Microsoft Entra eingebundenen Sitzungshosts konfigurieren, um eine Reihe von funktionalen und geschäftlichen Anforderungen zu unterstützen. Folgende Anforderungen müssen erfüllt sein:

- Bereitstellen zusätzlicher Sitzungshosts zur Aufnahme einer erhöhten Anzahl von Remotebenutzenden
- Minimieren der Kosten für die Azure Virtual Desktop-Umgebung durch Optimierung der Hostpool-Lastausgleichskonfiguration und Nutzung der Funktionalität *VM bei Verbindung starten*
- Maximieren der Verfügbarkeit von Sitzungshosts während der Geschäftszeiten durch Implementieren von Wartungsfenstern
- Aktivieren des einmaligen Anmeldens bei in Microsoft Entra eingebundenen Sitzungshosts
- Maximieren der Benutzerfreundlichkeit und des Benutzererlebnisses (wie etwa automatische Wiederverbindung getrennter Sitzungen)

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Konfigurieren von Microsoft Entra in Verbindung mit Azure Virtual Desktop Session Hosts zum Support einer Reihe von funktionalen und geschäftlichen Anforderungen

## Labdateien

- Keine

## Anweisungen

### Übung 1: Verwalten einer Azure Virtual Desktop-Umgebung mit in Microsoft Entra eingebundenen Sitzungshosts
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Bereitstellen von zusätzlichen Azure Virtual Desktop-Hostpool-Sitzungshosts
1. Überprüfen und Konfigurieren der Hostpooleigenschaften
1. Zuweisen der erforderlichen RBAC-Rolle zu einem Azure Virtual Desktop-Dienstprinzipal
1. Konfigurieren von geplanten Agentupdates
1. Konfigurieren der RDP-Eigenschaften des Hostpools

#### Aufgabe 1: Bereitstellen zusätzlicher Azure Virtual Desktop-Hostpool-Sitzungshosts

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich an, indem Sie die Anmeldedaten eines Benutzerkontos mit der Besitzerrolle in dem Abonnement angeben, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Suchen Sie im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** in der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Host-Pools** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools **az140-21-hp1** aus.
1. Wählen Sie auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Sitzungshosts** aus und überprüfen Sie, ob der Pool aus zwei Hosts besteht. 
1. Wälen Sie auf der Seite „**az140-21-hp1 \| Sitzungshosts** **+ Hinzufügen**.
1. Überprüfen Sie auf der Registerkarte **Allgemeine Informationen** der Seite **Virtuelle Maschinen zu einem Hostpool hinzufügen** die vorkonfigurierten Einstellungen und wählen Sie **Weiter: Virtuelle Maschinen** aus.
1. Geben Sie auf der Registerkarte **Virtuelle Maschinen** auf der Seite **Virtuelle Maschinen zu einem Hostpool hinzufügen** die folgenden Einstellungen an und wählen Sie **Überprüfen + erstellen** aus (belassen Sie die anderen Einstellungen bei den Standardeinstellungen):

    > **Hinweis**: Wenn Sie den Wert für **Präfixname** einstellen, wechseln Sie zur Registerkarte Ressourcen auf der rechten Seite des Lab-Sitzungsfensters. Suchen Sie dort die Zeichenfolge zwischen *User1-* und dem Zeichen *@*. Verwenden Sie diese Zeichenfolge, um den *zufälligen* Platzhalter zu ersetzen.

    |Einstellung|Wert|
    |---|---|
    |Resource group|**az140-21a-RG**|
    |Namenspräfix|**sh**-*random*|
    |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie die ersten zwei Sitzungshost-VMs bereitgestellt haben|
    |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
    |Sicherheitstyp|**Virtuelle Computer mit vertrauenswürdigem Start**|
    |Abbildung|**Windows 11 Enterprise für mehrere Sitzungen, Version 23H2 + Microsoft 365 Apps - Gen2**|
    |Größe des virtuellen Computers|**Standard DC2s_v3**|
    |Number of VMs (Anzahl von VMs)|**1**|
    |Typ des Betriebssystemdatenträgers|**SSD Standard**|
    |Größe des Betriebssystemdatenträgers|**Standardgröße (128 GB)**|
    |Startdiagnose|**Mit verwaltetem Speicherkonto aktivieren (empfohlen)**|
    |Virtuelles Netzwerk|**az140-vnet11e**|
    |Subnet|**hp1-Subnet**|
    |Netzwerksicherheitsgruppe|**Grundlegend**|
    |Öffentliche Eingangsports|**Nein**|
    |Wählen Sie das gewünschte Verzeichnis für die Einbindung aus.|**Microsoft Entra ID**|
    |VM bei Intune registrieren|**Nein**|
    |Benutzername|**Kursteilnehmer**|
    |Kennwort|dasselbe Kennwort, das Sie beim Bereitstellen der Sitzungshosts im Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe der Azure-Portal (Entra-ID)* verwendet haben 
    |Kennwort bestätigen|dasselbe Kennwort, das Sie zuvor angegeben haben|

    > **Hinweis**: Das Kennwort sollte mindestens 12 Zeichen lang sein und aus einer Kombination von Kleinbuchstaben, Großbuchstaben, Ziffern und Sonderzeichen bestehen. Ausführliche Informationen finden Sie unter den Informationen zu [den Kennwortanforderungen beim Erstellen einer Azure-VM](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

    > **Hinweis:** Wie Sie wahrscheinlich bereits festgestellt haben, ist es möglich, das Image und das Präfix der VMs zu ändern, wenn Sie Sitzungshosts zum vorhandenen Pool hinzufügen. Im Allgemeinen wird dies nur empfohlen, wenn Sie alle VMs im Pool ersetzen möchten. 

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** der Seite **Virtuelle Maschinen zu einem Host-Pool hinzufügen** **Erstellen** aus

    > **Hinweis:** Warten Sie nicht, bis der Bereitstellungsprozess abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Der Bereitstellungsprozess kann etwa 20 Minuten dauern. 

#### Aufgabe 2: Überprüfen und Konfigurieren der Hostpooleigenschaften

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop** und wählen Sie es aus. Wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Hostpools** und auf der Seite **Azure Virtual Desktop \| Hostpools** **az140-21-hp1** aus. 
1. Wählen sie auf der Seite **az140-21-hp1** im Abschnitt **Einstellungen** **Eigenschaften** aus.
1. Überprüfen Sie auf der Seite **az140-21-hp1\| Eigenschaften** die verfügbaren Konfigurationsoptionen, einschließlich:

    - **Bevorzugter App-Gruppentyp**: Mit dieser Option wird der bevorzugte App-Gruppentyp für den Hostpool auf **Desktop** oder **RemoteApp** festgelegt. Wenn Endbenutzende sowohl RemoteApp- als auch Desktop-Apps im Hostpool veröffentlicht haben, wird nur der ausgewählte App-Typ in ihrem Feed angezeigt.
    - **VM bei Verbindung starten**: Wenn diese Option aktiviert ist, können Benutzende einzelne virtuelle Maschinen im Host-Pool aus dem freigegebenen Zustand starten.
    - **Umgebung für die Prüfung**: Der Hostpool für die Prüfung dient dazu, Änderungen an Diensten zu testen, bevor sie in der Produktion eingesetzt werden.
    - **Algorithmus für den Lastenausgleich**: Diese Option bietet die Wahl zwischen dem breiten- und tiefenorientierten Lastenausgleich. Der breitenorientierte Lastenausgleich verteilt neue Benutzersitzungen auf alle verfügbaren Sitzungshosts im Hostpool. Der tiefenorientierte Lastenausgleich verteilt neue Benutzersitzungen an einen verfügbaren Sitzungshost mit der höchsten Anzahl an Verbindungen, der den maximalen Schwellenwert für die Sitzungsanzahl noch nicht erreicht hat.

1. Wälen Sie auf der Seite **az140-21-hp1\| Eigenschaften** in der Dropdown-Liste **Algorithmus für den Lastenausgleich** **Tiefenorientiert** aus.
1. Geben Sie im Textfeld **Maximale Anzahl von Sitzungen** die Zahl **8** ein.
1. Legen Sie auf der Seite **az140-21-hp1\|Eigenschaften** die Option **VM bei Verbindung starten** auf **Ja** fest.

    > **Hinweis**: Mit *VM bei Verbindung starten* können Sie Kosten senken, indem Sie es den Endbenutzenden ermöglichen, die als Sitzungshosts verwendeten virtuellen Maschinen (VMs) nur dann einzuschalten, wenn sie benötigt werden.. Bei persönlichen Hostpools schaltet *VM bei Verbindung starten* nur eine vorhandene Sitzungshost-VM ein, die bereits einer benutzenden Person zugewiesen ist oder zugewiesen werden kann. Bei gepoolten Hostpools schaltet *VM bei Verbindung starten* einen Sitzungshost nur dann ein, wenn noch keiner eingeschaltet ist, und weitere VMs werden nur dann eingeschaltet, wenn die erste VM die maximale Anzahl der Sitzungen erreicht.

1. Wälen Sie auf der Seite **az140-21-hp1\| Eigenschaften** **Speichern** aus.

    > **Hinweis**: Die Verwendung von *VM bei Verbindung starten* erfordert die Zuweisung der Rolle *Mitwirkende am Einschalten Desktop-Virtualisierung* für rollenbasierte Zugriffssteuerung (RBAC) zum *Azure Virtual Desktop*-Dienstprinzipal im Azure-Abonnementbereich. 

#### Aufgabe 3: Zuweisen eines Azure Virtual Desktop-Dienstprinzipals zu der erforderlichen RBAC-Rolle

1. Starten Sie vom Lab-Computer aus im Webbrowser, der das Azure-Portal anzeigt, eine PowerShell-Sitzung in der Azure Cloud Shell.

    > **Hinweis**: Wenn eine Eingabeaufforderung angezeigt wird, wählen Sie im Bereich **Erste Schritte** in der Dropdownliste **Abonnement** den Namen des Azure-Abonnements aus, das Sie in diesem Lab verwenden, und wählen Sie dann ‚**Anwenden** aus.

1. Führen Sie in der PowerShell-Sitzung im Bereich Azure Cloud Shell den folgenden Befehl aus, um den Wert der ID-Eigenschaft des Azure-Abonnements, das Sie in diesem Lab verwenden, abzurufen und in einer Variablen `$subId` zu speichern:

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Führen Sie den folgenden Befehl aus, um eine $parameters-Variable zu erstellen, die eine Hash-Tabelle speichert, die die Werte des Namens der RBAC-Rollendefinition, der Microsoft Entra-Anwendung, die den Dienstprinzipal von **Azure Virtual Desktop** darstellt, und des Abonnementbereichs enthält:

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Führen Sie den folgenden Befehl aus, um die RBAC-Rollenzuweisung zu erstellen:

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Schließen Sie den Cloud Shell-Bereich.

#### Aufgabe 4: Konfigurieren von geplanten Agent-Updates

> **Hinweis**: Mit der Funktion Geplante Agent-Updates können Sie bis zu zwei Wartungsfenster für die Updates des Azure Virtual Desktop-Agents, des parallelene Stapels und des Geneva Monitoring-Agents erstellen, sodass diese Updates außerhalb der Geschäftszeiten stattfinden. 

1. Navigieren Sie im Webbrowser, der das Azure-Portal anzeigt, zurück zur Seite des Hostpools **az140-21-hp1**.
1. Wählen Sie auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Einstellungen** den **Eintrag Geplante Agent Updates** aus und auf der Seite **az140-21-hp1\|Geplante Agent Updates** das Kontrollkästchen **Geplante Agent Updates** aus.
1. Aktivieren Sie im Abschnitt **Zeitplan** das Kontrollkästchen **Zeitzone des lokalen Sitzungshosts verwenden**.
1. Wählen Sie im Abschnitt **Wartungsfenster** in der Dropdownliste **Tag** die Option **Samstag** und in der Dropdownliste **Uhrzeit** die Option **23:00 Uhr** aus.
1. Wählen Sie **Übernehmen**.

#### Aufgabe 5: Konfigurieren von RDP-Eigenschaften des Hostpools

1. Wählen Sie im Webbrowser, der das Azure-Portal anzeigt, auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Einstellungen** den Eintrag **RDP-Eigenschaften** aus.
1. Überprüfen Sie auf der Registerkarte **Verbindungsinformationen** der Seite **az140-21-hp1\|RDP-Eigenschaften** die verfügbaren Konfigurationsoptionen, einschließlich:

    - **Microsoft Entra Single Sign-On**: Diese Option legt fest, ob Verbindungen versuchen, die Microsoft Entra-Authentifizierung zu nutzen, um sich bei Microsoft Entra-verbundenen Sitzungshosts anzumelden und effektiv eine Single Sign-On-Erfahrung zu bieten. Beachten Sie, dass es nicht erforderlich ist, dass der Clientcomputer mit Microsoft Entra verbunden ist. 
    - **Credential Security Support Provider**: Diese Option steuert die Verwendung von CredSSP für die Authentifizierung. CredSSP bietet die Möglichkeit, Benutzeranmeldeinformationen vom Clientgerät sicher an den Remotedesktop-Sitzungshost weiterzuleiten. Die Funktionen dieser Option enthalten jedoch keinen Support für die Entra-ID-Authentifizierung.
    - **Alternative Shell**: Mit dieser Option können Sie eine ausführbare Datei angeben, die immer dann gestartet wird, wenn eine neue Verbindung zu einem Sitzungshost hergestellt wird. Diese Einstellung gilt nur für Sitzungshosts, die Windows Server ausführen.
    - **KDC-Proxy-Name**: Diese Option bietet die Möglichkeit, den Kerberos-Authentifizierungsverkehr an Active Directory-Domänencontroller weiterzuleiten.

    > **Hinweis**: Da drei dieser Optionen in unserem Szenario nicht anwendbar sind (in dem es um Sitzungshosts geht, die Microsoft Entra beigetreten sind, ohne dass Active Directory Domain Services vorhanden sind), konfigurieren Sie nur die erste. Diese Option entspricht der RDP-Eigenschaft `enablerdsaadauth:i:value`.

1. Wäle Sie in der Dropdownliste **Microsoft Entra Single Sign-On** die Option **Verbindungen verwenden Microsoft Entra-Authentifizierung für Single Sign-On** und dann **Speichern** aus.

    > **Wichtig**: Es ist wichtig zu beachten, dass die Aktivierung dieser spezifischen RDP-Eigenschaft nur eine von mehreren Schritten ist, um die Single Sign-On-Funktionalität zu implementieren. Weitere Maßnahmen, die für dieses Szenario gelten, sind die Aktivierung der Microsoft Entra-Authentifizierung für RDP im Entra-Mandanten und die Konfiguration von Gerätegruppen, die in der aktuellen Version der Lab-Umgebung nicht unterstützt werden und daher nicht in den Anweisungen enthalten sind. Eine vollständige Liste der erforderlichen Maßnahmen zur Implementierung von Single Sign-On für Microsoft Entra ID finden Sie unter [Konfigurieren von Single Sign-On für Azure Virtual Desktop mit Microsoft Entra ID-Authentifizierung](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on).

1. Wählen Sie auf der Seite **az140-21-hp1\|RDP Eigenschaften** die Registerkarte **Sitzungsverhalten** aus und überprüfen die verfügbaren Konfigurationsoptionen, einschließlich:

    - **Wiederverbindung**: Diese Option legt fest, ob der Clientcomputer automatisch versucht, die Verbindung zum Remotecomputer wiederherzustellen, wenn die Verbindung unterbrochen wird.
    - **Automatische Bandbreitenerkennung**: Diese Option legt fest, ob die automatische Erkennung der Netzwerkbandbreite verwendet werden soll oder nicht.
    - **Automatische Netzwerkerkennung**: Mit dieser Option können Sie die automatische Erkennung des Netzwerktyps aktivieren. Sie wird in Verbindung mit der Option **automatischen Bandbreitenerkennung** verwendet. 
    - **Komprimierung**: Mit dieser Option wird bestimmt, ob die Verbindung die Massenkomprimierung verwenden soll.
    - **Videowiedergabe**: Diese Option ermöglicht die Verwendung eines effizienten RDP-Multimediastreamings für die Videowiedergabe.

1. Wählen Sie auf der Registerkarte **Sitzungsverhalten** in der Dropdownliste **Wiederverbindung** die Option **Client versucht automatisch, die Verbindung wiederherzustellen** aus und wählen Sie dann **Speichern** aus.
1. Wählen Sie auf der Seite **az140-21-hp1\|RDP Eigenschaften** die Registerkarte **Geräteumleitung** aus und überpfüfen die verfügbaren Konfigurationsoptionen, einschließlich zwei Hauptkategorien:

    - **Audio und Video**
    - **Lokale Geräte und Ressourcen**

    > **Hinweis**: Die Umleitung gilt standardmäßig für alle Datenträgerlaufwerke, einschließlich der Laufwerke, die nach dem Einrichten der ersten Verbindung bereitgestellt werden.

1. Wählen Sie auf der Seite **az140-21-hp1\|RDP Eigenschaften** die Registerkarte **Anzeigeeinstellungen** und prüfen die verfügbaren Konfigurationsoptionen, einschließlich der Unterstützung für mehrere Anzeigen, intelligente Größenanpassung und spezifische Desktop-Größen (in Pixeln). 
1. Wählen Sie auf der Seite **az140-21-hp1\|RDP Eigenschaften** die Registerkarte **Erweitert** und prüfen die vorhandenen Konfigurationseinstellungen. Beachten Sie, dass diese Einstellungen die Änderungen widerspiegeln, die Sie zuvor in dieser Aufgabe vorgenommen haben.