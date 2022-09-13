---
lab:
  title: "Lab: Implementieren der automatischen Skalierung in Hostpools (AD\_DS)"
  module: 'Module 5: Monitor and Maintain a WVD Infrastructure'
---

# <a name="lab---implement-autoscaling-in-host-pools-ad-ds"></a>Lab: Implementieren der automatischen Skalierung in Hostpools (AD DS)
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lab-Abhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden.
- Ein Microsoft-Konto oder ein Azure AD-Konto mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle„Globaler Administrator“ im Azure AD-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)**

## <a name="estimated-time"></a>Geschätzte Dauer

60 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen die automatische Skalierung von Azure Virtual Desktop-Sitzungshosts in einer AD DS-Umgebung (Active Directory Domain Services) konfigurieren.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Die Konfigurierung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts
- Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

## <a name="lab-files"></a>Labdateien

- Keine

## <a name="instructions"></a>Anweisungen

### <a name="exercise-1-configure-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Übung 1: Die Konfigurierung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts
1. Erstellen und Konfigurieren eines Azure Automation-Kontos
1. Erstellen einer Azure-Logik-App

#### <a name="task-1-prepare-for-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Aufgabe 1: Vorbereiten der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um die virtuellen Azure-Computer mit den Azure Virtual Desktop-Sitzungshosts zu starten, die Sie in diesem Lab verwenden:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich gestartet werden. 

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>Aufgabe 2: Erstellen und Konfigurieren eines Azure Automation-Kontos

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie dann auf dem Blatt **Virtuelle Computer** auf **az140-dc-vm11**.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** die Option **Verbinden**, im Dropdownmenü **Bastion**, auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung auf **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator.
1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Azure AD-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE**-Skriptbereich Folgendes aus, um das PowerShell-Skript herunterzuladen, das Sie zum Erstellen des Azure Automation-Kontos verwenden, das Teil der Lösung für die automatische Skalierung ist:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   New-Item -ItemType Directory -Path $labFilesfolder -Force
   Set-Location -Path $labFilesfolder
   $uri = 'https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1'
   Invoke-WebRequest -Uri $Uri -OutFile '.\CreateOrUpdateAzAutoAccount.ps1'
   ```

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE**-Skriptbereich Folgendes aus, um die Werte von Variablen festzulegen, die Sie Skriptparametern zuweisen:

   ```powershell
   $aadTenantId = (Get-AzContext).Tenant.Id
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   $suffix = Get-Random
   $automationAccountName = "az140-automation-51$suffix"
   $workspaceName = "az140-workspace-51$suffix"
   ```

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** Skriptbereich Folgendes aus, um die Ressourcengruppe zu erstellen, die Sie in diesem Lab verwenden:

   ```powershell
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   ```

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows Power Shell ISE**-Skriptbereich Folgendes aus, um einen Azure Log Analytics-Arbeitsbereich zu erstellen, den Sie in diesem Lab verwenden:

   ```powershell
   New-AzOperationalInsightsWorkspace -Location $location -Name $workspaceName -ResourceGroupName $resourceGroupName
   ```

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** die Option „Datei“ aus dem oberen Menü aus, und öffnen Sie das Skript „**C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1**“. Schließen Sie den Code zwischen den Zeilen **82** und **86** in den mehrzeiligen Kommentar ein, und speichern Sie ihn, sodass Sie folgendes Ergebnis erhalten:

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
    throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. Öffnen Sie in der Remotedesktopsitzung für **az140-dc-vm11** im Skriptbereich **Administrator: Windows PowerShell ISE** eine neue Registerkarte, fügen Sie das folgendes Skript ein, und führen Sie es aus, um das Azure Automation-Konto zu erstellen, das Teil der Lösung automatischen Skalierung ist:

   ```powershell
   $Params = @{
     "AADTenantId" = $aadTenantId
     "SubscriptionId" = $subscriptionId 
     "UseARMAPI" = $true
     "ResourceGroupName" = $resourceGroupName
     "AutomationAccountName" = $automationAccountName
     "Location" = $location
     "WorkspaceName" = $workspaceName
   }

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   .\CreateOrUpdateAzAutoAccount.ps1 @Params
   ```

   >**Hinweis:** Warten Sie, bis das Skript abgeschlossen ist. Dies kann etwa zehn Minuten dauern.

1. Überprüfen Sie innerhalb der Remotedesktopsitzung auf **az140-dc-vm11** im Skriptbereich **Administrator: Windows PowerShell ISE** die Ausgabe des Skripts. 

   >**Hinweis:** Die Ausgabe enthält einen Webhook-URI, die ID des Log Analytics-Arbeitsbereichs und die entsprechenden Primärschlüsselwerte, die Sie beim Bereitstellen der Azure-Logik-App, die Teil der Lösung der automatischen Skalierung ist, angeben müssen. 
   
   >**Hinweis:** Notieren Sie sich den Wert des Webhook-URI. Sie benötigen ihn später in diesem Lab.

1. Um die Konfiguration des Azure Automation-Kontos zu überprüfen, starten Sie in der Remotedesktopsitzung auf **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Azure AD-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie innerhalb der Remotedesktopsitzung auf **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach dem Blatt **Automatisierungskonten**. Wählen Sie dann auf dem Blatt **Automatisierungskonten** den Eintrag aus, der das kürzlich bereitgestellte Azure Automation-Konto darstellt (der neue Name beginnt mit dem Präfix **az140-automation-51**).
1. Wählen Sie auf dem Blatt „Automatisierungskonto“ im vertikalen Menü auf der linken Seite im Abschnitt **Prozessautomatisierung** die Option **Runbooks** aus, und überprüfen Sie in der Liste der Runbooks, ob das Runbook **WVDAutoScaleRunbookARMBased** vorhanden ist.
1. Wählen Sie auf dem Blatt „Automatisierungskonto“ im vertikalen Menü auf der linken Seite im Abschnitt **Kontoeinstellungen** die Option **Ausführende Konten** aus, und klicken Sie in der Liste der Konten auf der rechten Seite neben **+Ausführendes Azure-Konto** auf **Erstellen**.
1. Klicken Sie auf dem Blatt **Ausführendes Azure-Konto hinzufügen** auf **Erstellen**, und überprüfen Sie, ob das neue Konto erfolgreich erstellt wurde.

#### <a name="task-3-create-an-azure-logic-app"></a>Aufgabe 3: Erstellen einer Azure-Logik-App

1. Wechseln Sie innerhalb der Remotedesktopsitzung auf **az140-25-vm11** zum Fenster **Administrator: Windows PowerShell ISE**, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um das PowerShell-Skript herunterzuladen, das Sie zum Erstellen der Azure Logik-App verwenden, die Teil der Lösung für die automatische Skalierung ist.

   ```powershell
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   Set-Location -Path $labFilesfolder
   $uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
   Invoke-WebRequest -Uri $uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
   ```

1. Wählen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** die Option **Datei** aus dem oberen Menü aus, und öffnen Sie das Skript „**C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzLogicApp.ps1**“. Schließen Sie den Code zwischen den Zeilen **134** und **138** in den mehrzeiligen Kommentar ein, und speichern Sie sie, sodass sie wie folgt aussehen:

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
    throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** Folgendes aus, um die Werte der Variablen festzulegen, die Sie Skriptparametern zuweisen. Ersetzen Sie dazu den Platzhalter `<webhook_URI>` durch den Wert des Webhook-URI, das Sie zuvor in diesem Lab notiert haben.

   ```powershell
   $AADTenantId = (Get-AzContext).Tenant.Id
   $AzSubscription = (Get-AzContext).Subscription.Id
   $ResourceGroup = Get-AzResourceGroup -Name 'az140-51-RG'
   $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" -Name 'az140-21-hp1'
   $LogAnalyticsWorkspace = (Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $LogAnalyticsWorkspaceId = $LogAnalyticsWorkspace.CustomerId
   $LogAnalyticsWorkspaceKeys = (Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName $ResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspace.Name)
   $LogAnalyticsPrimaryKey = $LogAnalyticsWorkspaceKeys.PrimarySharedKey
   $RecurrenceInterval = 2
   $BeginPeakTime = '1:00'
   $EndPeakTime = '1:01'
   $TimeDifference = '0:00'
   $SessionThresholdPerCPU = 1
   $MinimumNumberOfRDSH = 1
   $MaintenanceTagName = 'CustomMaintenance'
   $LimitSecondsToForceLogOffUser = 5
   $LogOffMessageTitle = 'Autoscaling'
   $LogOffMessageBody = 'Forcing logoff due to autoscaling'

   $AutoAccount = (Get-AzAutomationAccount -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

   $WebhookURIAutoVar = '<webhook_URI>'
   ```

   >**Hinweis:** Durch die Werte der Parameter soll das Verhalten der automatischen Skalierung beschleunigt werden. In Ihrer Produktionsumgebung sollten Sie sie anpassen, damit sie Ihren eigenen spezifischen Anforderungen entsprechen.

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** Folgendes aus, um die Azure-Logik-App zu erstellen, die Teil der Lösung der automatischen Skalierung ist.

   ```powershell
   $Params = @{
     "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
     "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
     "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
     "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
     "UseARMAPI"                     = $true
     "HostPoolName"                  = $WVDHostPool.Name
     "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
     "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
     "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
     "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
     "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
     "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
     "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
     "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
     "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
     "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
     "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
     "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
     "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shut down."
     "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
     "WebhookURI"                    = $WebhookURIAutoVar
   }

   .\CreateOrUpdateAzLogicApp.ps1 @Params
   ```

   >**Hinweis:** Warten Sie, bis das Skript abgeschlossen ist. Dies kann etwa zwei Minuten dauern.

1. Um die Konfiguration der Azure-Logik-App zu überprüfen, wechseln Sie in der Remotedesktopsitzung auf **az140-dc-vm11** zum Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird. Suchen und wählen Sie **Logik-Apps** aus, und wählen Sie im Blatt **Logik-Apps** den Eintrag aus, der die neu bereitgestellte Azure-Logik-App mit dem Namen **az140-21-hp1_Autoscale_Scheduler** darstellt.
1. Wählen Sie auf dem Blatt **az140-21-hp1_Autoscale_Scheduler** im vertikalen Menü auf der linken Seite im Abschnitt **Entwicklungstools** die Option **Logik-App-Designer** aus. 
1. Klicken Sie im Designer-Bereich auf das Rechteck mit der Bezeichnung **Serie**. Beachten Sie dabei, dass Sie darüber die Häufigkeit steuern können, mit der die Notwendigkeit für die automatische Skalierung ausgewertet wird. 

### <a name="exercise-2-verify-and-review-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Übung 2: Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

Die Hauptaufgaben für diese Übung sind Folgende:

1. Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts
1. Verwenden von Azure Log Analytics zum Nachverfolgen von Azure Virtual Desktop-Ereignissen

#### <a name="task-1-verify-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Aufgabe 1: Die Überprüfung der automatischen Skalierung von Azure Virtual Desktop-Sitzungshosts

1. Um die automatische Skalierung der Azure Virtual Desktop-Sitzungshosts zu überprüfen, suchen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie die Option aus. Überprüfen Sie auf dem Blatt **Virtuelle Computer** dann den Status der drei Azure-VMs in der Ressourcengruppe **az140-21-RG**.
1. Stellen Sie sicher, dass für zwei der drei Azure-VMs entweder gerade die Zuordnung aufgehoben wird oder sie bereits **Beendet (freigegeben)** sind.

   >**Hinweis:** Sobald Sie feststellen können, dass die automatische Skalierung funktioniert, deaktivieren Sie die Azure-Logik-App, um mögliche Gebühren zu vermeiden.

1. Um die Azure-Logik-App zu deaktivieren, suchen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Logik-Apps** aus, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Logik-Apps** den Eintrag aus, der die neu bereitgestellte Azure-Logik-App mit dem Namen **az140-21-hp1_Autoscale_Scheduler** darstellt.
1. Klicken Sie auf dem Blatt **az140-21-hp1_Autoscale_Scheduler** in der Symbolleiste auf **Deaktivieren**. 
1. Sehen Sie sich auf dem Blatt **az140-21-hp1_Autoscale_Scheduler** im Bereich **Essentials** die Informationen an, einschließlich der Anzahl erfolgreicher Ausführungen in den letzten 24 Stunden sowie den Bereich **Zusammenfassung**, in dem die Häufigkeit von Wiederholungen angegeben ist. 
1. Suchen Sie innerhalb der Remotedesktopsitzung auf **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach dem Blatt **Automatisierungskonten**. Wählen Sie dann auf dem Blatt **Automatisierungskonten** den Eintrag aus, der das kürzlich bereitgestellte Azure Automation-Konto darstellt (der neue Name beginnt mit dem Präfix **az140-automation-51**).
1. Wählen Sie auf dem Blatt **Automatisierungskonto** im vertikalen Menü auf der linken Seite im Abschnitt **Prozessautomatisierung** die Option **Aufträge** aus, und überprüfen Sie die Liste der Aufträge, die einzelnen Aufrufen des Runbooks **WVDAutoScaleRunbookARMBased** entsprechen.
1. Wählen Sie den neuesten Auftrag aus, und klicken Sie auf dem dazugehörigen Blatt auf die Registerkartenüberschrift **Alle Protokolle**. Dadurch wird eine detaillierte Auflistung der Schritte zur Ausführung des Auftrags angezeigt.

#### <a name="task-2-use-azure-log-analytics-to-track-azure-virtual-desktop-events"></a>Aufgabe 2: Verwenden von Azure Log Analytics zum Nachverfolgen von Azure Virtual Desktop-Ereignissen

>**Hinweis:** Um die automatische Skalierung und alle anderen Azure Virtual Desktop-Ereignisse zu analysieren, können Sie Log Analytics verwenden.

1. Wählen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, **Log Analytics-Arbeitsbereiche** aus. Wählen Sie dann auf dem Blatt **Log Analytics-Arbeitsbereiche** den Eintrag aus, der den in dieser Übung verwendeten Azure Log Analytics-Arbeitsbereich darstellt (der Name beginnt mit dem Präfix **az140-workspace-51**.
1. Klicken Sie auf dem Blatt „Log Analytics-Arbeitsbereich“ im vertikalen Menü auf der linken Seite im Bereich **Allgemein** auf **Protokolle**. Schließen Sie dann ggf. das Fenster **Willkommen bei Log Analytics**, und fahren Sie im Bereich **Abfrage** fort.
1. Wählen Sie im Bereich **Abfragen** im vertikalen Menü **Alle Abfragen** auf der linken Seite **Azure Virtual Desktop** aus, und überprüfen Sie die vordefinierten Abfragen.
1. Schließen Sie den Bereich **Abfragen**. Dadurch wird automatisch die Registerkarte **Neue Abfrage 1** angezeigt.
1. Fügen Sie im Abfragefenster die folgende Abfrage ein, und klicken Sie auf **Ausführen**, um alle Ereignisse für den Hostpool anzuzeigen, der in diesem Lab verwendet wird:

   ```kql
   WVDTenantScale_CL
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

   >**Hinweis:** Wenn in der zweiten Zeile bei Verwendung eines „Ausschneiden und Einfügen“-Konstrukts ein senkrechter Strich (|) vorkommt, entfernen Sie diesen, um Fehler zu vermeiden. Dies kann für jede Abfrage gelten.
   >**Hinweis:** Wenn keine Ergebnisse angezeigt werden, warten Sie ein paar Minuten, und versuchen Sie es erneut.

1. Fügen Sie im Abfragefenster die folgende Abfrage ein, und klicken Sie auf **Ausführen**, um die Gesamtanzahl der aktuell ausgeführten Sitzungshost und aktiven Benutzersitzungen im Zielhostpool anzuzeigen:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Number of running session hosts:"
     or logmessage_s contains "Number of user sessions:"
     or logmessage_s contains "Number of user sessions per Core:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. Fügen Sie im Abfragefenster die folgende Abfrage ein, und klicken Sie auf **Ausführen**, um den Status aller Sitzungshost-VMs in einem Hostpool anzuzeigen:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Session host:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. Fügen Sie im Abfragefenster die folgende Abfrage ein, und klicken Sie auf **Ausführen**, um skalierungsbezogene Fehler und Warnungen anzuzeigen:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

>**Hinweis:** Ignorieren Sie die Fehlermeldung bezüglich `TenantId`.

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>Übung 3: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der Azure-VMs, die im Lab bereitgestellt sind, und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung werden Sie die Zuordnungen der in diesem Labor bereitgestellten Azure-VMs aufheben, um die entsprechenden Computegebühren zu minimieren.

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>Aufgabe 1: Aufheben der Zuordnung von Azure-VMs, die im Lab bereitgestellt sind

1. Wechseln Sie zum Laborcomputer, und öffnen Sie im Webbrowser, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird, wie über Parameter „-NoWait“ festgelegt, asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet und ihre Zuordnungen aufgehoben werden.
