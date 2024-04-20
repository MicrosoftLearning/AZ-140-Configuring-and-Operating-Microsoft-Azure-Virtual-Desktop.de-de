---
lab:
  title: "Lab: Bereitstellen und Verwalten von Hostpools und Hosts mithilfe von PowerShell (AD\_DS)"
  module: 'Module 2: Implement a WVD Infrastructure'
---

# Lab: Bereitstellen und Verwalten von Hostpools und Hosts mithilfe von PowerShell
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Azure AD-Konto mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globaler Administrator“ im Azure AD-Mandanten, der diesem Azure-Abonnement zugeordnet ist
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen die Bereitstellung von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von PowerShell in einer AD DS-Umgebung (Active Directory Domain Services) automatisieren.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Bereitstellen von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von PowerShell
- Hinzufügen von Hosts zum Azure Virtual Desktop-Hostpool mithilfe von PowerShell

## Labdateien

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## Anweisungen

### Übung 1: Implementieren von Azure Virtual Desktop-Hostpools und -Sitzungshosts mithilfe von PowerShell
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Bereitstellung von Azure Virtual Desktop-Hostpools mithilfe von PowerShell
1. Erstellen eines Azure Virtual Desktop-Hostpools mit PowerShell
1. Ausführen einer vorlagenbasierten Bereitstellung einer Azure-VM mit Windows 10 Enterprise mithilfe von PowerShell
1. Hinzufügen einer Azure-VM mit Windows 10 Enterprise als Sitzungshost zum Azure Virtual Desktop-Hostpool mithilfe von PowerShell
1. Überprüfen der Bereitstellung des Azure Virtual Desktop-Sitzungshosts

#### Aufgabe 1: Vorbereiten der Bereitstellung von Azure Virtual Desktop-Hostpools mithilfe von PowerShell

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie auf dem Blatt **Virtuelle Computer** auf **az140-dc-vm11**.
1. Klicken Sie auf dem Blatt **az140-dc-vm11** auf **Verbinden**. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und wählen Sie **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Bastion-Sitzung auf **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator*in.
1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Distinguished Name der Organisationseinheit mit dem Namen **WVDInfra** zu identifizieren, in der die Computerobjekte der Azure Virtual Desktop-Sitzungshostpools gehostet werden:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das UPN-Suffix des Kontos **ADATUM\\Student** zu identifizieren, über das die Azure Virtual Desktop-Hosts der AD DS-Domäne (**student@adatum.com**) beitreten:

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das PowerShell-Modul „DesktopVirtualization“ zu installieren (wenn Sie dazu aufgefordert werden, klicken Sie auf **Ja, alle**):

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **Hinweis:** Ignorieren Sie alle Warnungen zu den verwendeten PowerShell-Modulen.

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Azure AD-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Azure-Portal oben auf der Seite über das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** nach **Virtuelle Netzwerke**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Netzwerke** die Option **az140-adds-vnet11** aus. 
1. Wählen Sie auf dem Blatt **az140-adds-vnet11** die Option **Subnetze** und auf dem Blatt **Subnetze** die Option **+ Subnetz** aus. Geben Sie auf dem Blatt **Subnetz hinzufügen** die folgenden Einstellungen an (übernehmen Sie bei allen anderen Einstellungen die Standardwerte), und klicken Sie auf **Speichern**:

   |Einstellung|Wert|
   |---|---|
   |Name|**hp3-Subnet**|
   |Subnetzadressbereich|**10.0.3.0/24**|

1. Verwenden Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Azure-Portal oben auf der Seite das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen**, um nach **Netzwerksicherheitsgruppen** zu suchen und dorthin zu navigieren. Wählen Sie auf dem Blatt **Netzwerksicherheitsgruppen** die Sicherheitsgruppe in der Ressourcengruppe **az140-11-RG** aus.
1. Klicken Sie auf dem Blatt „Netzwerksicherheitsgruppe“ im vertikalen Menü auf der linken Seite im Abschnitt **Einstellungen** auf **Eigenschaften**.
1. Klicken Sie auf dem Blatt **Eigenschaften** rechts neben dem Textfeld **Ressourcen-ID** auf das Symbol **In Zwischenablage kopieren**. 

   > **Hinweis:** Der Wert muss das folgende Format aufweisen: `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`. Die Abonnement-ID kann jedoch anders aussehen. Notieren Sie den Wert, da Sie ihn in der nächsten Aufgabe benötigen werden.

#### Aufgabe 2: Erstellen eines Azure Virtual Desktop-Hostpools mit PowerShell

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Azure-Region zu identifizieren, in der das virtuelle Azure-Netzwerk **az140-adds-vnet11** gehostet wird:

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Ressourcengruppe zu erstellen, in der der Hostpool und dessen Ressourcen gehostet werden:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um einen leeren Hostpool zu erstellen:

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **Hinweis:** Mit dem Cmdlet **New-AzWvdHostPool** können Sie einen Hostpool, einen Arbeitsbereich und die Desktop-App-Gruppe erstellen sowie die Desktop-App-Gruppe beim Arbeitsbereich registrieren. Dabei können Sie einen neuen Arbeitsbereich erstellen oder einen vorhandenen Arbeitsbereich verwenden.

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Attribut „objectID“ der Azure AD-Gruppe mit dem Namen **az140-wvd-pooled** abzurufen:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Azure AD-Gruppe mit dem Namen **az140-wvd-pooled** der Desktop-App-Standardgruppe des neu erstellten Hostpools zuzuweisen:

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### Aufgabe 3: Ausführen einer vorlagenbasierten Bereitstellung einer Azure-VM mit Windows 10 Enterprise mithilfe von PowerShell

1. Navigieren Sie auf Ihrem Lab-Computer zu dem bereitgestellten Speicherkonto. Wählen Sie auf dem Blatt „Dateifreigabe“ die Dateifreigabe **az140-22-profiles** aus.

1. Klicken Sie auf **Hochladen**, und laden Sie die beiden Lab-Dateien **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json** und **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json** zur Dateifreigabe hoch.

1. Öffnen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** den Datei-Explorer, und navigieren Sie zum zuvor konfigurierten Laufwerk Z oder zu dem Laufwerkbuchstaben, der der Verbindung mit der Dateifreigabe zugewiesen ist. Kopieren Sie die hochgeladenen Bereitstellungsdateien in **C:\AllFiles\Labs\02**.

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Azure-VM mit Windows 10 Enterprise (mit mehreren Sitzungen) bereitzustellen, die als Azure Virtual Desktop-Sitzungshost im Hostpool fungiert, den Sie in der vorherigen Aufgabe erstellt haben:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. Dies kann etwa fünf Minuten dauern. 

   > **Hinweis:** Die Bereitstellung stellt die Azure-VM mithilfe einer Azure Resource Manager-Vorlage bereit und wendet eine VM-Erweiterung an, durch die das Betriebssystem automatisch der AD DS-Domäne **adatum.com** beitritt.

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um zu überprüfen, ob der dritte Sitzungshost der AD DS-Domäne **adatum.com** erfolgreich beigetreten ist:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### Aufgabe 4: Hinzufügen einer Azure-VM mit Windows 10 Enterprise als Host zum Azure Virtual Desktop-Hostpool mithilfe von PowerShell

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** im Browserfenster, in dem das Azure-Portal geöffnet ist, nach **VMs** und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **VMs** in der Liste der VMs den Eintrag **az140-24-p3-0** aus.
1. Wählen Sie auf dem Blatt **az140-24-p3-0** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **RDP** aus und auf der Registerkarte **RDP** des Blatts **az140-24-p3-0 \| Verbinden** in der Dropdownliste **IP-Adresse** den Eintrag **Private IP-Adresse (10.0.3.4)**. Klicken Sie anschließend auf **RDP-Datei herunterladen**.
1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den folgenden Anmeldeinformationen an:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**ADATUM\\Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-24-p3-0** die **Windows PowerShell ISE** als Administrator.
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-24-p3-0** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um einen Ordner zum Hosten der Dateien zu erstellen, die Sie benötigen, um dem Hostpool, den Sie weiter oben in diesem Lab bereitgestellt haben, die neu bereitgestellte Azure-VM als Sitzungshost hinzuzufügen:

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >**Hinweis**: Verwenden Sie das [T]-Konstrukt zum Kopieren der PowerShell-Cmdlets mit Sorgfalt. Es kann vorkommen, dass der kopierte Text fehlerhaft ist und z. B. anstelle des $-Zeichens die Zahl 4 angezeigt wird. Korrigieren Sie die fehlerhaften Zeichen, bevor Sie das Cmdlet ausgeben. Kopieren Sie die Cmdlets in den PowerShell ISE-**Skriptbereich**, und nehmen Sie dort die Korrekturen vor. Markieren Sie anschließend den korrigierten Text, und drücken Sie **F8** (**Auswahl ausführen**).

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-24-p3-0** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Installer für den Azure Virtual Desktop-Agent und das Startladeprogramm herunterzuladen, die Sie benötigen, um den Sitzungshost dem Hostpool hinzuzufügen:

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-24-p3-0** über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die neueste Version des PowerShellGet-Moduls zu installieren (wenn Sie zur Bestätigung aufgefordert werden, klicken Sie auf **Ja**):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die neueste Version des PowerShell-Moduls „Az.DesktopVirtualization“ zu installieren:

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   Install-Module -Name Az -AllowClobber -Force
   ```

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die PowerShell-Ausführungsrichtlinie zu ändern und sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Führen Sie innerhalb der Remotedesktopliveid-Sitzung für **az140-24-p3-0** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Token zu generieren, das für den Beitritt neuer Sitzungshosts zum Pool benötigt wird, den Sie weiter oben in dieser Übung bereitgestellt haben:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **Hinweis:** Sie benötigen ein Registrierungstoken, um einen Sitzungshost für den Beitritt zum Hostpool zu autorisieren. Der Wert für das Ablaufdatum des Tokens muss zwischen einer Stunde und einem Monat zwischen dem aktuellen Datum und der aktuellen Uhrzeit liegen.

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-24-p3-0** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Azure Virtual Desktop-Agent zu installieren:

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-24-p3-0** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Azure Virtual Desktop-Startladeprogramm zu installieren:

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

#### Aufgabe 5: Überprüfen der Bereitstellung des Azure Virtual Desktop-Hosts

1. Suchen Sie auf dem Lab-Computer im Webbrowser, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Hostpools** aus und auf dem Blatt **Azure Virtual Desktop \| Hostpools** den Eintrag **az140-24-hp3**, der den neu geänderten Pool darstellt.
1. Klicken Sie auf dem Blatt **az140-24-hp3** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf die Option **Sitzungshosts**. 
1. Überprüfen Sie auf dem Blatt **az140-24-hp3 \| Sitzungshosts**, ob die Bereitstellung einen einzelnen Host enthält.

#### Aufgabe 6: Verwalten von App-Gruppen mit PowerShell

1. Wechseln Sie vom Labcomputer zur Bastion-Sitzung für **az140-dc-vm11** und führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Remote-App-Gruppe zu erstellen:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die **Startmenü**-Apps für die Hosts aufzulisten und die Ausgabe zu überprüfen:

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **Hinweis:** Zeichnen Sie für alle Anwendungen, die Sie veröffentlichen möchten, die Informationen in der Ausgabe auf, wie etwa die Parameter **FilePath**, **IconPath** und **IconIndex**.

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um Microsoft Word zu veröffentlichen:

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um Microsoft Word zu veröffentlichen:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. Wählen Sie auf dem Lab-Computer im Webbrowser, in dem das Azure-Portal angezeigt wird, auf dem Blatt **az140-24-hp3 \| Sitzungshosts** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** die Option **Anwendungsgruppen** aus.
1. Wählen Sie auf dem Blatt **az140-24-hp3 \| Anwendungsgruppen** in der Liste der Anwendungsgruppen den Eintrag **az140-24-hp3-Office365-RAG** aus.
1. Überprüfen Sie auf dem Blatt **az140-24-hp3-Office365-RAG** die Konfiguration der Anwendungsgruppe, einschließlich der Anwendungen und Zuweisungen.

### Übung 2: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab bereitgestellten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten Azure-VMs

1. Wechseln Sie zum Labcomputer, und öffnen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet werden und ihre Zuordnung aufgehoben wird.
