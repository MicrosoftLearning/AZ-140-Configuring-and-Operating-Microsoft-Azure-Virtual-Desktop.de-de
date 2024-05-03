---
lab:
  title: "Lab: Implementieren der automatischen Skalierung in Hostpools (AD\_DS)"
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# Lab: Implementieren der automatischen Skalierung in Hostpools (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Microsoft Entra-Konto mit der Rolle „Besitzer*in“ oder „Mitwirkende*r“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globale*r Administrator*in“ im Microsoft Entra-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)**

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen die automatische Skalierung von Azure Virtual Desktop-Sitzungshosts in einer AD DS-Umgebung (Active Directory Domain Services) konfigurieren.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Die Konfigurierung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts
- Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

## Labdateien

- Keine

## Anweisungen

### Übung 1: Die Konfigurierung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Skalierung von Azure Virtual Desktop-Sitzungshosts
2. Einrichten der Diagnose zum Nachverfolgen der automatischen Azure Virtual Desktop-Skalierung
3. Erstellen eines Skalierungsplans für Azure Virtual Desktop-Sitzungshosts

#### Aufgabe 1: Vorbereiten der Skalierung von Azure Virtual Desktop-Sitzungshosts

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Öffnen Sie auf dem Labcomputer in dem Webbrowserfenster, in dem das Azure-Portal geöffnet ist, die **PowerShell**-Sitzung im Bereich **Cloud Shell**.

   >**Hinweis:** Hostpools, die Sie mit automatischer Skalierung verwenden möchten, sollten mit einem nicht standardmäßigen Wert des Parameters **MaxSessionLimit** konfiguriert werden. Sie können diesen Wert in den Hostpooleinstellungen im Azure-Portal festlegen oder das Azure PowerShell-Cmdlet **Update-AzWvdHostPool** wie in diesem Beispiel ausführen. Sie können ihn auch explizit festlegen, wenn Sie einen Pool im Azure-Portal oder beim Ausführen des Azure PowerShell-Cmdlets **New-AzWvdHostPool** erstellen.

1. Führen Sie in der PowerShell-Sitzung im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert des Parameters **MaxSessionLimit** des Hostpools **az140-21-hp1** auf **2** festzulegen: 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**Hinweis:** In diesem Lab wird der Wert des Parameters **MaxSessionLimit** künstlich niedrig festgelegt, um das Auslösen des automatischen Skalierungsverhaltens zu erleichtern.

   >**Hinweis:** Bevor Sie Ihren ersten Skalierungsplan erstellen, müssen Sie Azure Virtual Desktop die RBAC-Rolle **Desktop Virtualization Power On Off-Mitwirkende*r** mit Ihrem Azure-Abonnement als Zielbereich zuweisen. 

1. Schließen Sie in dem Browserfenster, in dem das Azure-Portal geöffnet ist, den Cloud Shell-Bereich.
1. Suchen Sie im Azure-Portal nach **Abonnements** und wählen Sie es aus. Wählen Sie dann aus der Liste das Abonnement aus, das die Azure Virtual Desktop-Ressourcen enthält. 
1. Wählen Sie auf der Seite „Abonnements“ **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf der Seite **Zugriffssteuerung (IAM)** auf der Symbolleiste die Schaltfläche **+ Hinzufügen**aus, und wählen Sie dann **Rollenzuweisung hinzufügen** aus dem Dropdownmenü aus.
1. Geben Sie auf dem Blatt **Rollenzuweisung hinzufügen** in der Registerkarte **Rolle** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

   |Einstellung|Wert|
   |---|---|
   |Auftragsfunktionsrolle|**Power On/Off-Mitwirkende für Desktopvirtualisierung**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** auf der Registerkarte **Mitglieder** **+ Mitglieder auswählen** aus, geben Sie die folgenden Einstellungen an, und wählen Sie **Auswählen** aus. 

   |Einstellung|Wert|
   |---|---|
   |Auswählen|**Azure Virtual Desktop** oder **Windows Virtual Desktop**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** **Überprüfen + Zuweisen** aus.

   >**Hinweis:** Der Wert hängt davon ab, wann der **Microsoft.DesktopVirtualization**-Ressourcenanbieter das erste Mal in Ihrem Azure-Mandanten registriert wurde.

1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.

#### Aufgabe 2: Einrichten der Diagnose zum Nachverfolgen der automatischen Azure Virtual Desktop-Skalierung

1. Öffnen Sie auf dem Labcomputer in dem Webbrowserfenster, in dem das Azure-Portal geöffnet ist, die **PowerShell**-Sitzung im Bereich **Cloud Shell**.

   >**Hinweis:** Sie werden ein Azure Storage-Konto zum Speichern von automatischen Skalierungsereignissen verwenden. Sie können sie direkt aus dem Azure-Portal erstellen oder Azure PowerShell verwenden, wie in dieser Aufgabe dargestellt.

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ die folgenden Befehle aus, um ein Azure Storage-Konto zu erstellen:

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName 'az140-11-RG').Location
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**Hinweis:** Warten Sie, bis das Speicherkonto bereitgestellt wurde.

1. Schließen Sie in dem Browserfenster, in dem das Azure-Portal geöffnet ist, den Cloud Shell-Bereich.
1. Navigieren Sie auf Ihrem Labcomputer im Browser, in dem das Azure-Portal geöffnet ist, zur Seite des Hostpools **az140-21-hp1**.
1. Wählen Sie auf der Seite **az140-21-hp1** **Diagnoseeinstellungen** und dann **+ Diagnoseeinstellung hinzufügen** aus.
1. Geben Sie auf der Seite **Diagnoseeinstellung** im Textfeld **Name der Diagnoseeinstellung** **az140-51-scaling-plan-diagnostics** ein, und wählen Sie im Abschnitt **Kategoriegruppen** **Autoskalierung für Protokolle von gepoolten Hostpools** aus. 
1. Wählen Sie auf derselben Seite im Abschnitt **Zieldetails** die Option **In einem Speicherkonto archivieren** aus, und wählen Sie in der Dropdownliste **Speicherkonto** den Namen des Speicherkontos mit dem Präfix **az140st51** aus.
1. Wählen Sie **Speichern**.

#### Aufgabe 3: Erstellen eines Skalierungsplans für Azure Virtual Desktop-Sitzungshosts

1. Suchen Sie auf Ihrem Laborcomputer im Browser, in dem das Azure-Portal geöffnet ist, nach **Azure Virtual Desktop**, und wählen Sie es aus. 
1. Wählen Sie auf der Seite **Azure Virtual Desktop** die Option **Skalierungspläne** und dann **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Basics** des Assistenten zum **Erstellen eines Skalierungsplans** die folgenden Informationen an, und wählen Sie **Nächste  Zeitpläne >** aus (für die anderen Werte behalten Sie die Standardwerte):

   |Einstellung|Wert|
   |---|---|
   |Resource group|**az140-51-RG**|
   |Name|**az140-51-scaling-plan**|
   |Standort|die gleiche Azure-Region, in der Sie die Sitzungshosts in der vorherigen Aufgabe dieses Labs bereitgestellt haben|
   |Anzeigename|**az140-51 Skalierungsplan**|
   |Zeitzone|Ihre lokale Zeitzone|

   >**Hinweis:** Mit Ausschlusstags können Sie einen Tagnamen für Sitzungshosts festlegen, die Sie von Skalierungsvorgängen ausschließen möchten. Beispielsweise können Sie VMs mithilfe des Ausschlusstags „excludeFromScaling“ kennzeichnen, für die der Ausgleichsmodus festgelegt ist, damit die Autoskalierung den Ausgleichsmodus während der Wartung nicht außer Kraft setzt. 

1. Wählen Sie auf der Registerkarte **Zeitpläne** des Assistenten zum **Erstellen eines Skalierungsplans** **+ Zeitplan hinzufügen** aus.
1. Geben Sie auf der Registerkarte **Allgemein** des Assistenten **Zeitplan hinzufügen** die folgenden Informationen an, und wählen Sie **Weiter** aus.

   |Einstellung|Wert|
   |---|---|
   |Zeitplanname|**az140-51-schedule**|
   |Wiederholen am|**7 ausgewählt** (alle Wochentage auswählen)|

1. Geben Sie auf der Registerkarte **Anlaufzeiten** des Assistenten zum **Hinzufügen des Zeitplans** die folgenden Informationen an, und wählen Sie **Weiter** aus.

   |Einstellung|Wert|
   |---|---|
   |Startzeit (24-Stunden System)|Ihre aktuelle Zeit minus 9 Stunden|
   |Lastenausgleichsalgorithmus|**Breitenorientiert**|
   |Mindestprozentsatz der Hosts (%)|**20**|
   |Kapazitätsschwellenwert (%)|**60**|

   >**Hinweis:** Die hier ausgewählte Einstellung für den Lastenausgleich überschreibt die Einstellung, die Sie für Ihre ursprünglichen Hostpooleinstellungen ausgewählt haben.

   >**Hinweis:** Der minimale Prozentsatz der Hosts gibt den Prozentsatz der Sitzungshosts an, die immer eingeschaltet bleiben sollen. Wenn der eingegebene Prozentsatz keine ganze Zahl ist, wird er auf die nächste ganze Zahl aufgerundet. 

   >**Hinweis:** Der Kapazitätsschwellenwert stellt den Prozentsatz der verfügbaren Hostpool-Kapazität dar, der eine Skalierungsaktion auslösen wird. Wenn beispielsweise zwei Sitzungshosts im Hostpool mit einem maximalen Sitzungslimit von 20 aktiviert sind, beträgt die verfügbare Hostpoolkapazität 40. Wenn Sie den Kapazitätsschwellenwert auf 75 % festlegen und die Sitzungshosts über mehr als 30 Benutzersitzungen verfügen, aktiviert die Autoskalierung einen dritten Sitzungshost. Dadurch wird die verfügbare Hostpoolkapazität von 40 in 60 geändert.

1. Geben Sie auf der Registerkarte **Spitzenzeiten** des Assistenten zum **Hinzufügen des Zeitplans** die folgenden Informationen an, und wählen Sie **Weiter** aus.

   |Einstellung|Wert|
   |---|---|
   |Startzeit (24-Stunden System)|Ihre aktuelle Zeit minus 8 Stunden|
   |Lastenausgleichsalgorithmus|**Tiefensuche**|

   >**Hinweis:** Die Startzeit bestimmt die Endzeit für die Anlauf-Phase.

   >**Hinweis:** Der Kapazitätsschwellenwert in dieser Phase wird durch den Schwellenwert für den Kapazitätsaufbau bestimmt.

1. Geben Sie auf der Registerkarte **Auslaufzeiten** des Assistenten zum **Hinzufügen des Zeitplans** die folgenden Informationen an, und wählen Sie **Weiter** aus.

   |Einstellung|Wert|
   |---|---|
   |Startzeit (24-Stunden System)|Ihre aktuelle Zeit minus 2 Stunden|
   |Lastenausgleichsalgorithmus|**Tiefensuche**|
   |Mindestprozentsatz der Hosts (%)|**10**|
   |Kapazitätsschwellenwert (%)|**90**|
   |Abmeldung der Benutzer erzwingen|**Ja**|
   |Verzögerung bis zum Abmelden der Benutzer*innen und Herunterfahren von VMs (Minuten)|**30**|

   >**Hinweis:** Wenn die Option **Abmeldung der Benutzer*innen erzwingen** aktiviert sind, platziert die automatische Skalierung den Sitzungshost mit der geringsten Anzahl von Benutzersitzungen im Ausgleichsmodus, sendet alle aktiven Benutzersitzungen eine Benachrichtigung über das anstehende Herunterfahren und erzwingt nach Ablauf der angegebenen Verzögerungszeit die Abmeldung. Nachdem die Autoskalierung alle Benutzersitzungen abgemeldet hat, wird die Zuordnung der VM aufgehoben. 

   >**Hinweis:** Wenn Sie die erzwungene Abmeldung während der Auslaufzeit nicht aktiviert haben, werden Sitzungshosts ohne aktive oder getrennte Sitzungen freigegeben.

1. Geben Sie auf der Registerkarte **Nebenzeiten** des Assistenten zum **Hinzufügen des Zeitplans** die folgenden Informationen an, und wählen Sie **Hinzufügen** aus.

   |Einstellung|Wert|
   |---|---|
   |Startzeit (24-Stunden System)|Ihre aktuelle Uhrzeit minus 1 Stunde|
   |Lastenausgleichsalgorithmus|**Tiefensuche**|

   >**Hinweis:** Der Kapazitätsschwellenwert in dieser Phase wird durch den Schwellenwert für den Kapazitätsabbau bestimmt.

1. Wählen Sie auf der Registerkarte **Zeitpläne** des Assistenten zum **Erstellen eines Skalierungsplans** **Weiter: Hostpoolzuweisungen >** aus:
1. Wählen Sie auf der Seite **Hostpoolzuweisungen** in der Dropdownliste **Hostpool auswählen** **az140-21-hp1** aus, stellen Sie sicher, dass das Kontrollkästchen **Automatische Skalierung aktivieren** aktiviert ist, wählen Sie **Überprüfen + Erstellen** und dann **Erstellen** aus.


### Übung 2: Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

Die Hauptaufgaben für diese Übung sind Folgende:

1. Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts


#### Aufgabe 1: Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

1. Öffnen Sie auf dem Labcomputer in dem Webbrowserfenster, in dem das Azure-Portal geöffnet ist, die **PowerShell**-Sitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um die Azure-VMs mit den Azure Virtual Desktop-Sitzungshosts zu starten, die Sie in diesem Lab verwenden:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Hinweis:** Warten Sie, bis die Azure VMs des Sitzungshosts ausgeführt werden.

1. Navigieren Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal geöffnet ist, zur Seite des Hostpools **az140-21-hp1**.
1. Wählen Sie auf der Seite **az140-21-hp1** **Sitzungshosts** aus.
1. Warten Sie, bis mindestens ein Sitzungshost mit dem Status **Herunterfahren** aufgeführt ist.

   > **Hinweis:** Möglicherweise müssen Sie die Seite aktualisieren, um den Status der Sitzungshosts zu aktualisieren.

   > **Hinweis:** Wenn alle Sitzungshosts nach 15 Minuten verfügbar bleiben, navigieren Sie zurück zur Seite **az140-51-scaling-plan** und verringern Sie den Wert der Einstellung **Minimalprozentsatzes von Hosts (%)** **Auslaufzeiten**.

   > **Hinweis:** Sobald sich der Status eines oder mehrerer Sitzungshosts ändert, sollten die Protokolle der automatischen Skalierung im Azure Storage-Konto verfügbar sein. 

1. Suchen Sie im Azure-Portal nach **Speicherkonten** und wählen Sie diese Option aus. Wählen Sie auf der Seite **Speicherkonten** den Eintrag aus, der das zuvor in dieser Übung erstellte Speicherkonto darstellt (der Name beginnt mit dem Präfix **az140st51**).
1. Klicken Sie auf der Seite „Speicherkonto“ auf **Container**.
1. Wählen Sie in der Liste der Container **insights-logs-autoscaleevaluationpooled** aus.
1. Navigieren Sie auf der Seite **insights-logs-autoscaleevaluationpooled** durch die Ordnerhierarchie, bis Sie den Eintrag erreicht haben, der ein im Container gespeichertes JSON-formatiertes Blob darstellt.
1. Wählen Sie den Blob-Eintrag aus, wählen Sie das Auslassungszeichen rechts auf der Seite aus, und wählen Sie dann im Dropdownmenü **Herunterladen** aus.
1. Öffnen Sie auf Ihrem Laborcomputer das heruntergeladene Blob in einem Text-Editor Ihrer Wahl, und sehen Sie sich dessen Inhalt an. Sie sollten in der Lage sein, Verweise auf Ereignisse der Automatischen Skalierung zu finden, und in diesem Fall können wir nach „deallocated“ suchen, um dies leichter zu identifizieren.

   >**Hinweis:** Hier ist ein Beispielinhalt eines Blob, der Verweise auf Ereignisse der automatischen Skalierung enthält:

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "AutoscaleEvaluationPooled"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### Übung 3: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab verwendeten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten Azure-VMs

1. Wechseln Sie zum Labcomputer und öffnen Sie im Webbrowser, in dem das Azure-Portal geöffnet ist, eine **PowerShell**-Sitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab verwendeten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab verwendeten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet werden und ihre Zuordnung aufgehoben wird.
