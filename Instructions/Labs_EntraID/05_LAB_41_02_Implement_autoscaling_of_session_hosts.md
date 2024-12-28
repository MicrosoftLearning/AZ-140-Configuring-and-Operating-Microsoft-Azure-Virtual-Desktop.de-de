---
lab:
  title: 'Lab: Implementieren der automatischen Skalierung von Sitzungshosts'
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Lab – Implementieren und Überwachen der automatischen Skalierung von Sitzungshosts
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzerrolle oder Teilnehmerrolle in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den ausreichenden Berechtigungen, um Geräte dem mit diesem Azure-Abonnement verbundenen Entra-Mandanten beizutreten.
- Das Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals* wurde abgeschlossen
- Das Lab *Verwalten von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)* wurde abgeschlossen
- Das Lab *Herstellen einer Verbindung zu Sitzungshosts (Entra ID)* wurde abgeschlossen.

## Geschätzte Dauer

45 Minuten

## Labszenario

Sie verfügen über eine Azure Virtual Desktop-Umgebung, in der sich die Nutzung regelmäßig ändert. Sie möchten die Kosten minimieren, indem Sie die Funktionen der Autoskalierungspläne nutzen.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Implementieren und Bewerten der automatischen Azure Virtual Desktop-Skalierung

## Labdateien

- Keine

## Anweisungen

### Übung 1: Implementieren von Plänen zur automatischen Skalierung von Azure Virtual Desktop
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Zuweisen der erforderlichen RBAC-Rolle zu einem Azure Virtual Desktop-Dienstprinzipal
1. Beenden und Aufheben der Zuordnung aller Sitzungshosts
1. Anpassen der Hostpooleinstellungen
1. Erstellen eines Skalierungsplans
1. Auswerten der automatischen Skalierungsfunktion
1. Automatische Skalierung des Hostpools deaktivieren

#### Aufgabe 1: Zuweisen von RBAC-Rollen zum Azure Virtual Desktop-Dienstprinzipal

> **Hinweis**: Damit die automatische Skalierung funktioniert, müssen Sie dem Azure Virtual Desktop-Dienstprinzipal die Berechtigungen zum Verwalten des Betriebszustands der VMs des Sitzungshosts zuweisen. Diese Berechtigungen können mithilfe der integrierten RBAC-Rolle **Mitwirkende für das Ein-/Ausschalten der Desktop Virtualisierung** erteilt werden. Es ist wichtig zu beachten, dass die Rollenzuweisung im Abonnementbereich ausgeführt werden muss. Wenn Sie diese Rolle auf einer niedrigeren Ebene als Ihr Abonnement zuweisen, z. B. auf Ressourcengruppen-, Hostpool- oder VM-Ebene, wird verhindert, dass die Autoskalierung ordnungsgemäß funktioniert. 

> **Hinweis**: Diese Rolle unterscheidet sich von der Rolle (**Mitwirkende beim Einschalten der Desktop Virtualisierung**), die im Lab *Verwalten von Hostpools und Sitzungshosts mithilfe des Azure-Portals (Entra ID)* verwendet wurde und für den Support der Funktionalität *VM bei Verbindung starten* erforderlich war.

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich an, indem Sie die Anmeldedaten eines Benutzerkontos mit der Besitzerrolle in dem Abonnement angeben, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Starten Sie vom Lab-Computer aus im Webbrowser, der das Azure-Portal anzeigt, eine PowerShell-Sitzung in der Azure Cloud Shell.

    > **Hinweis**: Wenn eine Eingabeaufforderung angezeigt wird, wählen Sie im Bereich **Erste Schritte** in der Dropdownliste **Abonnement** den Namen des Azure-Abonnements aus, das Sie in diesem Lab verwenden, und wählen Sie dann ‚**Anwenden** aus.

1. Führen Sie in der PowerShell-Sitzung im Bereich Azure Cloud Shell den folgenden Befehl aus, um den Wert der ID-Eigenschaft des Azure-Abonnements, das Sie in diesem Lab verwenden, abzurufen und in einer Variablen `$subId` zu speichern:

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Führen Sie den folgenden Befehl aus, um eine $parameters-Variable zu erstellen, die eine Hash-Tabelle speichert, die die Werte des Namens der RBAC-Rollendefinition, der Microsoft Entra-Anwendung, die den Dienstprinzipal von **Azure Virtual Desktop** darstellt, und des Abonnementbereichs enthält:

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Off Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Führen Sie den folgenden Befehl aus, um die RBAC-Rollenzuweisung zu erstellen:

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Schließen Sie den Cloud Shell-Bereich.

#### Aufgabe 2: Beenden und Aufheben der Zuordnung aller Sitzungshosts

> **Hinweis**: Um die Funktionalität Automatische Skalierung zu bewerten, stoppen Sie alle Sitzungshosts in der Azure Virtual Desktop-Umgebung und heben die Zuweisung auf. 

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** in der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Hostpools** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools **az140-21-hp1** aus.
1. Wählen Sie auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Verwalten** **Sitzungshosts** aus.
1. Aktivieren Sie auf der Seite Sitzungshosts von **az140-21-hp1 \|** das Kontrollkästchen neben dem Namen jedes Sitzungshosts und wählen Sie dann **Beenden** aus.
1. Wenn Sie im Popupfenster **Sitzungshosts beenden** zur Bestätigung aufgefordert werden, wählen Sie **Beenden** aus.

    > **Hinweis**: Möglicherweise müssen Sie das Symbol mit den Auslassungspunkten (`...`) in der Symbolleiste auswählen, um die Schaltfläche **Beenden** anzuzeigen.

    > **Hinweis:** Warten Sie nicht, bis die Sitzungshosts beendet und freigegeben wurden, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Das Beenden und Freigeben von Sitzungshosts kann etwa 2 Minuten dauern.

#### Aufgabe 3: Anpassen der Hostpooleinstellungen

> **Hinweis**: Wenn Sie die automatische Skalierung für gepoolte Hostpools verwenden, muss für diesen Hostpool ein MaxSessionLimit-Parameter konfiguriert sein. In diesem Lab wird er künstlich niedrig eingestellt, um die automatische Skalierung zu veranschaulichen.

1. Wählen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, auf der Seite **az140-21-hp1** im Abschnitt **Einstellungen** die Option **Eigenschaften** aus.
1. Geben Sie auf der Seite **az140-21-hp1\| Eigenschaften** im Textfeld **Maximale Anzahl von Sitzungen** die Zahl **1** ein.
1. Wälen Sie auf der Seite **az140-21-hp1\| Eigenschaften** **Speichern** aus.

#### Aufgabe 4: Erstellen eines Skalierungsplans

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop** und wählen Sie es aus. Wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Skalierungspläne** aus.
1. Wählen Sie auf der Seite „**Azure Virtual Desktop \| Skalierungspläne** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Erstellen eines Skalierungsplans** die folgenden Einstellungen an und wählen Sie **Weiter: Zeitpläne** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe **az140-412e-RG**|
    |Name des Skalierungsplans|**az140-scalingplan412e**|
    |Region|Der Namen der Azure-Region, in der Sie die Azure Virtual Desktop-Umgebung bereitgestellt haben|
    |Anzeigename|**az140-scalingplan412e**|
    |Zeitzone|Die lokale Zeitzone der Azure-Region, in der Sie die Azure Virtual Desktop-Umgebung bereitgestellt haben|
    |Hostpooltyp|**In einem Pool zusammengefasst**|
    |Skalierungsmethode|**Automatische Skalierung bei der Energieverwaltung**|

    > **Hinweis: Lassen Sie die Eigenschaft** **Ausschlusstag** nicht festgelegt. Im Allgemeinen können Sie dieses Feature verwenden, um Azure-VMs mit willkürlich festgelegten Tags von der automatischen Skalierung auszuschließen.

1. Wählen Sie auf der Registerkarte **Zeitplan** **+ Zeitplan hinzufügen** aus.

    > **Hinweis**: Mit Zeitplänen können Sie Anlauf-, Spitzen-, Auslauf- und Nebenzeiten für Wochentage definieren und Auslöser für die automatische Skalierung festlegen. Der Skalierungsplan muss einen zugeordneten Zeitplan für mindestens einen Tag der Woche enthalten. 

1. Passen Sie auf der Registerkarte **Allgemein** des Bereichs **Zeitplan hinzufügen** die Standardkonfiguration an die folgenden Einstellungen an und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Zeitzone|Die lokale Zeitzone Ihrer Azure Virtual Desktop-Umgebung (basierend auf der Region, die Sie zuvor in dieser Aufgabe ausgewählt haben)|
    |Zeitplanname|**week_schedule**|
    |Wiederholen am|**7 ausgewählt** (alle Wochentage auswählen)|

    > **Hinweis**: Der Zeitplan deckt jeden Wochentag effektiv ab, was die Bewertung des Ergebnisses der automatischen Skalierung erleichtert.

1. Passen Sie auf der Registerkarte **Ramp-up** des Bereichs **Zeitplan hinzufügen** die Standardkonfiguration an die folgenden Einstellungen an und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Startzeit (12-Stunden System)|Ihre aktuelle Uhrzeit minus 1 Stunde|
    |Lastenausgleichsalgorithmus|**Breitensuche**|
    |Mindestprozentsatz der Hosts (%)|**30**|
    |Kapazitätsschwellenwert (%)|**60**|

    > **Hinweis**: Bei gepoolten Hostpools ignoriert die automatische Skalierung vorhandene Lastenausgleichsalgorithmen in den Einstellungen Ihres Hostpools und wendet stattdessen einen Lastenausgleich basierend auf Ihrer Zeitplankonfiguration an.

    > **Hinweis**: Die Einstellung **Mindestprozentsatz an Hosts** gibt den Mindestprozentsatz an virtuellen Maschinen für Sitzungshosts an, die für die Anlauf- und Spitzenzeiten gestartet werden sollen. Wenn beispielsweise **Mindestprozentsatz der Hosts** auf 30 % festgelegt ist und die Gesamtzahl der Sitzungshosts in Ihrem Hostpool 3 beträgt, stellt die automatische Skalierung sicher, dass mindestens 1 Sitzungshost verfügbar ist, um Verbindungen von Benutzenden anzunehmen.

    > **Hinweis**: Autoskalierung rundet auf die nächste ganze Zahl auf.

    > **Hinweis**: Die Einstellung „**Kapazitätsschwelle**“ ist der Prozentsatz der verwendeten Kapazität des Hostpools, der bei der Bewertung berücksichtigt wird, ob virtuelle Maschinen während der Anlauf- und Spitzenzeiten ein- oder ausgeschaltet werden sollen. Wenn der Kapazitätsschwellenwert beispielsweise mit 60% angegeben ist und die Kapazität Ihres Hostpools 1 Sitzung beträgt (mit einem laufenden Host), schaltet Autoscale zusätzliche Sitzungshosts ein, sobald die Last des Hostpools 60% übersteigt (in diesem Fall sind es 100%).

1. Passen Sie auf der Registerkarte **Spitzenzeiten** im Bereich **Zeitplan hinzufügen** die Standardkonfiguration an die folgenden Einstellungen an und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Startzeit (12-Stunden System)|Ihre aktuelle Zeit plus 1 Stunde|
    |Lastenausgleichsalgorithmus|**Tiefensuche**|
    |Kapazitätsschwellenwert (%)|**60**|

    > **Hinweis**: Die Einstellung **Kapazitätsschwellenwert (%)** gilt sowohl für die Einstellungen **Anlaufzeiten** als auch **Spitzenzeiten**.

1. Passen Sie auf der Registerkarte **Anlaufzeit** des Bereichs **Zeitplan hinzufügen** die Standardkonfiguration an die folgenden Einstellungen an und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Startzeit (12-Stunden System)|Ihre aktuelle Zeit plus 2 Stunden|
    |Lastenausgleichsalgorithmus|**Tiefensuche**|
    |Mindestprozentsatz aktiver Hosts (%)|**10**|
    |Kapazitätsschwellenwert (%)|**80**|
    |Abmeldung der Benutzer erzwingen|**Nein**|
    |Beenden Sie die VMs, wenn|**VMs keine aktiven oder getrennten Sitzungen haben**|

    > **Hinweis**: Die Einstellung „**Mindestprozentsatz aktiver Hosts (%)**“ gibt den Mindestprozentsatz der virtuellen Maschinen mit Sitzungshost an, den Sie für die Auslauf- und Nebenzeiten erreichen möchten. Wenn beispielsweise der **Mindestprozentsatz aktiver Hosts (%)** auf 10 % festgelegt ist und die Gesamtzahl der Sitzungshosts in Ihrem Hostpool 3 beträgt, stellt die automatische Skalierung sicher, dass mindestens 1 Sitzungshost verfügbar ist, um Verbindungen von Benutzenden anzunehmen.

    > **Hinweis**: Die Einstellung **Kapazitätsschwellenwert (%)** gibt den Prozentsatz der verwendeten Kapazität des Hostpools an, der bei der Bewertung berücksichtigt wird, ob virtuelle Maschinen während der Auslauf- und Nebenzeiten ausgeschaltet werden sollen. Wenn beispielsweise bei einer Benutzerverbindung und 3 laufenden Hosts der Kapazitätsschwellenwert auf 80 % festgelegt ist, würde Autoscale 1 Host abschalten (was zu einer zu 50 % genutzten Hostpool-Kapazität führt).

    > **Hinweis**: Im Allgemeinen beendet die automatische Skalierung Sitzungshosts und hebt die Zuweisung auf, und zwar gemäß den folgenden Regeln:

    - Die Kapazität der verwendeten Hostpools liegt unter dem Kapazitätsschwellenwert.
    - Das Ausschalten von Sitzungshosts führt nicht zu einer Überschreitung des Kapazitätsschwellenwertes.
    - Die Autoskalierung schaltet nur Sitzungshosts aus, auf denen sich keine Benutzersitzungen befinden, es sei denn, der Skalierungsplan befindet sich in der Auslaufhase und Sie haben die Einstellung aktiviert, die Benutzerabmeldung zu erzwingen. 
    - Pooled Autoscale schaltet Sitzungshosts in der Anlaufphase nicht ab, um sicherzustellen, dass das Benutzererlebnis nicht beeinträchtigt wird.

1. Passen Sie auf der Registerkarte **Nebenzeiten** des Bereichs **Zeitplan hinzufügen** die Standardkonfiguration an die folgenden Einstellungen an und wählen Sie dann **Hinzufügen** aus:

    |Einstellung|Wert|
    |---|---|
    |Startzeit (12-Stunden System)|Ihre aktuelle Zeit plus 3 Stunden|
    |Lastenausgleichsalgorithmus|**Tiefensuche**|
    |Kapazitätsschwellenwert (%)|**80**|

    > **Hinweis**: Die Einstellung für den **Kapazitätsschwellenwert** wird von den Einstellungen für **Auslaufzeiten** und **Nebenzeiten** gemeinsam genutzt.

1. Auf der Registerkarte **Zeitplan** der Seite **Skalierungsplan erstellen** wählen Sie **Weiter: Hostpool-Zuweisungen**.
1. Wählen Sie auf der Registerkarte **Hostpool-Zuweisungen** in der Dropdownliste „**Hostpool auswählen** die Option **az140-21-hp1** aus, stellen Sie sicher, dass das Kontrollkästchen„**Autoskalierung aktivieren** aktiviert ist, und wählen Sie dann **Überprüfen + erstellen** aus.
1. Wählen Sie auf der Seite **Bewerten + erstellen** die Option **Erstellen**.

    > **Hinweis**: Warten Sie, bis die automatische Skalierung abgeschlossen ist. Dies dauert in der Regel nur ein paar Sekunden.

#### Aufgabe 5: Bewerten der Funktionen für die automatische Skalierung

> **Hinweis**: Sie beginnen mit der Bewertung der Einstellungen für **Anlaufzeiten**.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** in der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Hostpools** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools **az140-21-hp1** aus.
1. Wählen Sie auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Verwalten** **Sitzungshosts** aus.
1. Überprüfen Sie auf der Seite „**az140-21-hp1 \| Sitzungshosts**“ die Werte der Einstellung „**Betriebszustand**“ der Sitzungshosts und vergewissern Sie sich, dass einer von ihnen als **Wird ausgeführt** aufgeführt ist.

    > **Hinweis:** Es kann einige Minuten dauern, bis der erste Sitzungshost den Status **wird ausgeführt** erreicht.

    > **Hinweis**: Dies ist zu erwarten, da gemäß den Einstellungen für den **Anlauf** des neu erstellten Skalierungsplans immer mindestens ein Sitzungshost online sein sollte. Zu diesem Zeitpunkt beträgt die Hostpool-Kapazität 1 (da nur ein Host läuft), aber die genutzte Hostpool-Kapazität beträgt 0 %, da es keine Benutzerverbindungen gibt.

    > **Hinweis**: Als Nächstes bewerten Sie die Einstellung des Kapazitätsschwellenwertes für die **Anlauf-** und **Spitzenzeiten**, indem Sie eine einzelne Benutzersitzung starten. Wir können dies auch außerhalb des Fensters **Spitzenzeiten** bewerten, da die beiden Stufen die gleiche Kapazitätsschwelle haben.

1. Starten Sie auf dem Lab-Computer den Microsoft-Remotedesktop Client.
1. Wählen Sie auf dem Lab-Computer im Client-Fenster **Remotedesktop** die Option **Abonnieren** aus und melden Sie sich bei der Eingabeaufforderung mit den Anmeldeinformationen des Benutzerkontos „`User2` Entra ID“ an, das Sie auf der Registerkarte „**Ressourcen**“ im rechten Bereich des Fensters der Benutzeroberfläche des Labs finden.
1. Stellen Sie sicher, dass auf der Seite **Remotedesktop** vier Symbole angezeigt werden, darunter Microsoft Word, Microsoft Excel, Microsoft PowerPoint und Eingabeaufforderung. 
1. Doppelklicken Sie auf das Eingabeaufforderungssymbol. 
1. Geben Sie bei der Eingabeaufforderung zur Anmeldung im Dialogfeld **Windows-Sicherheit** das Kennwort des Microsoft Entra-Benutzerkontos ein, das Sie für die Verbindung mit der Zielumgebung von Azure Virtual Desktop verwendet haben.
1. Überprüfen Sie, ob kurz darauf ein **Eingabeaufforderungsfenster** angezeigt wird. 

    > **Hinweis**: Zu diesem Zeitpunkt beträgt die Auslastung des Hostpools 100 %, was über dem Kapazitätsschwellenwert (% 60) liegt. Dies sollte dazu führen, dass Autoskalierung einen weiteren Host einschaltet, was die genutzte Hostpool-Kapazität auf 50 % bringt. Da dies unter dem Kapazitätsschwellenwert liegt, bleibt der dritte Host gestoppt/freigegeben. Überprüfen Sie dies als Nächstes.

1. Wechseln Sie vom Lab-Computer zu einem Webbrowser, der das Azure-Portal anzeigt. 
1. Auf der Seite Sitzungshosts der **az140-21-hp1 \|-Sitzung** **Aktualisieren** auswählen, die Werte der Einstellung **Betriebszustand** der Sitzungshosts überprüfen und sicherstellen, dass jetzt zwei von ihnen als **wird ausgeführt** aufgeführt sind.

    > **Hinweis**: Als Nächstes bewerten Sie die Einstellung des **Auslauf**-Kapazitätsschwellenwerts, indem Sie das Zeitfenster anpassen. 

1. Wählen Sie auf der Seite **az140-21-hp1 \| Sitzungshosts** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Skalierungspläne** und dann auf der Seite **Skalierungspläne** **az140-scalingplan412e** auswählen.
1. Wählen Sie auf der Seite **az140-scalingplan412e** im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Zeitpläne** und dann **week_schedule** aus.
1. Navigieren Sie im Bereich **week_schedule** zur Registerkarte **Auslaufzeiten** und stellen den Wert der Einstellung **Startzeit (12-Stunden-System)** auf eine Zeit zwischen der **Startzeit (12-Stunden-System)** der Phase **Spitzenzeiten** und Ihrer aktuellen Zeit.

    > **Hinweis**: Möglicherweise müssen Sie den Wert der **Startzeit (12-Stunden-System)** der Phase **Spitzenzeiten** anpassen.

1. Navigieren Sie im Bereich **week_schedule** zur Registerkarte **Nebenzeiten**, und wählen Sie ** Speichern**.
1. Wechseln Sie zum Fenster **Eingabeaufforderung**, das die einzige RDP-Sitzung zum Hostpool darstellt, und geben Sie in der Eingabeaufforderung Folgendes ein und drücken Sie die **Eingabetaste**:

    ```cmd
    logoff
    ```

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** in der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Hostpools** aus.
1. Wählen Sie auf der Seite **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools **az140-21-hp1** aus.
1. Wählen Sie auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Verwalten** **Sitzungshosts** aus.
1. Überprüfen Sie auf der Seite **az140-21-hp1 \| Sitzungshosts** die Werte der Einstellung **Betriebszustand** der Sitzungshosts und vergewissern Sie sich, dass jetzt nur einer von ihnen als **Wird ausgeführt** aufgeführt ist.

    > **Hinweis**: Es kann 1 bis 2 Minuten dauern, bis der Sitzungshost heruntergefahren wird.

#### Aufgabe 6: Deaktivieren der automatischen Skalierung des Hostpools

> **Hinweis**: Um sicherzustellen, dass die Konfiguration der automatischen Skalierung keine Auswirkungen auf andere Labs hat, entfernen Sie die Hostpool-Zuweisung des Skalierungsplans, den Sie in diesem Lab implementiert haben.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus, und wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Skalierungspläne** und dann auf der Seite **Skalierungspläne** **az140-scalingplan412e** auswählen.
1. Wählen Sie auf der Seite **az140-scalingplan412e** im Abschnitt **Verwalten** **Hostpool-Zuweisungen** au.
1. Wählen Sie auf der Seite **az140-scalingplan412e \| Hostpool-Zuweisungen** die Option **az140-21-hp1**, dann wählen Sie **Zuweisung aufheben** und bei der Eingabeaufforderung zur Bestätigung im Dialogfeld **Hostpool-Zuweisung aufheben** wählen Sie **Zuweisung aufheben**.