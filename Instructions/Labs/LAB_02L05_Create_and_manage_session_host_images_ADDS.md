---
lab:
  title: 'Lab: Erstellen und Verwalten von Sitzungshostimages (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Lab: Erstellen und Verwalten von Sitzungshostimages (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Microsoft Entra-Konto mit der Rolle „Besitzer*in“ oder „Mitwirkende*r“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globale*r Administrator*in“ im Microsoft Entra-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen Azure Virtual Desktop-Hostimages in einer Microsoft Entra DS-Umgebung erstellen und verwalten.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Erstellen und Verwalten von Azure Virtual Desktop-Sitzungshostimages

## Labdateien

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## Anweisungen

### Übung 1: Erstellung und Verwaltung von Sitzungshostimages
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Konfiguration eines Azure Virtual Desktop-Hostimages
1. Bereitstellen von Azure Bastion
1. Konfigurieren eines Azure Virtual Desktop-Hostimages
1. Erstellen eines Azure Virtual Desktop-Hostimages
1. Bereitstellen eines Azure Virtual Desktop-Hostpools mit dem benutzerdefinierten Image

#### Aufgabe 1: Vorbereiten der Konfiguration eines Azure Virtual Desktop-Hostimages

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 
1. Führen Sie auf dem Labcomputer im Webbrowser mit dem Azure-Portal aus der PowerShell-Sitzung im Cloud Shell-Bereich den folgenden Code aus, um eine Ressourcengruppe zu erstellen, die das Azure Virtual Desktop-Hostimage enthält:

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. Wählen Sie im Azure-Portal in der Symbolleiste des Cloud Shell-Bereichs das Symbol für **Dateien hochladen/herunterladen** aus, klicken Sie in der Dropdownliste auf **Hochladen**, und laden Sie die Dateien **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json** und **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json** in das Cloud Shell-Basisverzeichnis hoch.
1. Führen Sie aus der PowerShell-Sitzung im Cloud Shell-Bereich den folgenden Code aus, um eine Azure-VM unter Windows 10 bereitzustellen, die als Azure Virtual Desktop-Client für das neu erstellte Subnetz dient:

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Die Bereitstellung dauert ungefähr 10 Minuten.

#### Aufgabe 2: Bereitstellen von Azure Bastion 

> **Hinweis:** Azure Bastion ermöglicht die Verbindungsherstellung mit den virtuellen Azure-Computern ohne öffentliche Endpunkte, die Sie in der vorherigen Aufgabe dieser Übung bereitgestellt haben, und bietet gleichzeitig Schutz vor Brute-Force-Angriffen, die auf Anmeldeinformationen auf Betriebssystemebene abzielen.

> **Hinweis:** Stellen Sie sicher, dass bei Ihrem Browser die Popupfunktion aktiviert ist.

1. Öffnen Sie im Browserfenster mit dem Azure-Portal einen weiteren Tab, und navigieren Sie zum Azure-Portal.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Führen Sie aus der PowerShell-Sitzung im Cloud Shell-Bereich den folgenden Code aus, um ein Subnetz namens **AzureBastionSubnet** zum virtuellen Netzwerk namens **az140-25-vnet** hinzuzufügen, das Sie in einer vorherigen Übung erstellt haben:

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Azure-Portal nach **Bastions**, wählen Sie diese Option aus, und wählen Sie auf dem Blatt **Bastions** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Bastion erstellen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-25-RG**|
   |Name|**az140-25-bastion**|
   |Region|Die gleiche Azure-Region, in der Sie die Ressourcen in den vorherigen Aufgaben dieser Übung bereitgestellt haben|
   |Tarif|**Grundlegend**|
   |Virtuelles Netzwerk|**az140-25-vnet**|
   |Subnetz|**AzureBastionSubnet (10.25.254.0/24)**|
   |Öffentliche IP-Adresse|**Neu erstellen**|
   |Name der öffentlichen IP-Adresse|**az140-25-vnet-ip**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Bastion erstellen** die Option **Erstellen** aus:

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Die Bereitstellung kann ungefähr 5 Minuten dauern.

#### Aufgabe 3: Konfigurieren eines Azure Virtual Desktop-Hostimages

1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie dann auf dem Blatt **Virtuelle Computer** auf **az140-25-vm0**.
1. Wählen Sie auf dem Blatt **az140-25-vm0** die Option **Verbinden** aus. Wählen Sie in der Dropdownliste die Option **Bastion** aus und auf der Registerkarte **Bastion** des Blatts **az140-25-vm0 \| Verbinden** die Option **Bastion verwenden**.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

   > **Hinweis:** Beginnen Sie mit dem Installieren von FSLogix-Binärdateien.

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** die **Windows PowerShell ISE** als Administrator*in.
1. Führen Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** über die Konsole **Administrator*in: Windows PowerShell ISE** den folgenden Code aus, um einen Ordner zu erstellen, den Sie als temporären Speicherort für die Imagekonfiguration verwenden werden:

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. Starten Sie in der Bastion-Sitzung für **az140-25-vm0** Microsoft Edge und navigieren Sie zur [Downloadseite für FSLogix](https://aka.ms/fslogix_download). Laden Sie komprimierte FSLogix-Installationsbinärdateien in den Ordner **C:\\Allfiles\\Labs\\02** herunter, und extrahieren Sie den Unterordner **x64** im Datei-Explorer in denselben Ordner.
1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** zum Fenster **Administrator*in: Windows PowerShell ISE**, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Code aus, um eine Pro-Computer-Installation von OneDrive durchzuführen:

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **Hinweis**: Warten Sie, bis die Installation abgeschlossen ist. Dies kann etwa eine Minute dauern. Wenn die Installation einen Neustart auslöst, stellen Sie wieder eine Verbindung mit **az140-25-vm0** her.

   > **Hinweis:** Als Nächstes erfahren Sie, wie Sie zu Lernzwecken Microsoft Teams installieren und konfigurieren, da Teams in dem für dieses Lab verwendete Image bereits verfügbar ist.

1. Klicken Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** mit der rechten Maustaste auf **Start**. Wählen Sie in dem erscheinenden Menü **Ausführen** aus. Geben Sie im Dialogfeld **Ausführen** das Textfeld **Öffnen** **cmd** ein und drücken Sie die **EINGABETASTE**, um die **Eingabeaufforderung** zu starten.
1. Klicken Sie im Fenster **Administrator: C:\windows\system32\cmd.exe** in der Eingabeaufforderung, und führen Sie den folgenden Code aus, um die Pro-Computer-Installation von Microsoft Teams vorzubereiten:

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. Navigieren Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** in Microsoft Edge zur [Downloadseite von Microsoft Visual C++ Redistributable](https://aka.ms/vs/16/release/vc_redist.x64.exe) und speichern Sie **VC_redist.x64** im Ordner **C:\\Allfiles\\Labs\\02**.
1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** zum Fenster **Administrator*in: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um Microsoft Visual C++ Redistributable zu installieren:

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. Navigieren Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** in Microsoft Edge zur Dokumentationsseite mit dem Titel [Bereitstellen der Teams-Desktop-App auf der VM](https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm), wählen Sie den Link für die **64-Bit-Version**, und speichern Sie die Datei **Teams_windows_x64.msi** in den Ordner **C:\\Allfiles\\Labs\\02**, wenn Sie dazu aufgefordert werden.
1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** zum Fenster **Administrator*in: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um eine Pro-Computer-Installation von Microsoft Teams durchzuführen:

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **Hinweis:** Das Installationsprogramm unterstützt die Parameter „ALLUSER=1“ und „ALLUSERS=1“. Der Parameter „ALLUSER=1“ ist nur für Pro-Computer-Installationen in VDI-Umgebungen gedacht. Der Parameter ALLUSERS=1 kann in Nicht-VDI- und in VDI-Umgebungen verwendet werden. 
   > **Befolgen Sie** die folgenden Schritte, wenn die Fehlermeldung **Eine andere Version des Produkts ist bereits installiert** angezeigt wird: Wechseln Sie zu **Einstellungen > Programme > Programme und Features**. Klicken Sie mit der rechten Maustaste auf das Programm **Computerweiter Teams-Installer**, und wählen Sie **Deinstallieren** aus. Entfernen Sie das Programm, und führen Sie Schritt 13 noch einmal aus. 

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** die **Windows PowerShell ISE** als Administrator*in, und führen Sie über die Konsole **Administrator*in: Windows PowerShell ISE** den folgenden Code aus, um zu Lernzwecken Microsoft Edge Chromium zu installieren, da Edge für das in diesem Lab verwendete Image bereits verfügbar ist:

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **Hinweis**: Warten Sie, bis die Installation abgeschlossen ist. Dies kann etwa zwei Minuten dauern.

   > **Hinweis:** Wenn Sie in einer mehrsprachigen Umgebung arbeiten, sollten Sie Sprachpakete installieren. Weitere Informationen zu diesem Vorgang finden Sie im Artikel [Hinzufügen von Sprachpaketen zu einem Windows 10-Image mit mehreren Sitzungen](https://docs.microsoft.com/en-us/azure/virtual-desktop/language-packs) in der Microsoft-Dokumentation.

   > **Hinweis:** Als Nächstes deaktivieren Sie automatische Windows-Updates und Speicheroptimierung und konfigurieren die Zeitzonenumleitung sowie die Telemetrieerfassung. Allgemein sollten Sie zuerst alle aktuellen Updates anwenden. In diesem Lab wird dieser Schritt übersprungen, um die Dauer zu verkürzen.

1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** zum Fenster **Administrator*in: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um automatische Updates zu deaktivieren:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. Klicken Sie im Fenster **Administrator: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um Speicheroptimierung zu deaktivieren:

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. Klicken Sie im Fenster **Administrator: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um die Zeitzonenumleitung zu konfigurieren:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. Klicken Sie im Fenster **Administrator: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um das Erfassen von Telemetriedaten über den Feedback-Hub zu deaktivieren:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. Klicken Sie im Fenster **Administrator: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus den folgenden Code aus, um den temporären Ordner zu löschen, den Sie zuvor in dieser Aufgabe erstellt haben:

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. Klicken Sie im Fenster **Administrator: C:\windows\system32\cmd.exe**, und führen Sie von der Eingabeaufforderung aus die Laufwerkbereinigungsfunktion aus. Klicken Sie auf **OK**, wenn sie abgeschlossen ist:

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### Aufgabe 4: Erstellen eines Azure Virtual Desktop-Hostimages

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-25-vm0** über das Fenster **Administrator*in: C:\windows\system32\cmd.exe** aus der Eingabeaufforderung die Systemvorbereitungsfunktion aus, um das Betriebssystem auf das Generieren eines Images vorzubereiten und es automatisch herunterzufahren:

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm
   ```

   > **Hinweis:** Warten Sie, bis der Systemvorbereitungsvorgang abgeschlossen ist. Dies kann etwa zwei Minuten dauern. Dadurch wird das Betriebssystem automatisch heruntergefahren. 

1. Suchen Sie auf Ihrem Labcomputer im Webbrowser mit dem Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie dann auf dem Blatt **Virtuelle Computer** die Option **az140-25-vm0** aus.
1. Klicken Sie auf dem Blatt **az140-25-vm0** in der Symbolleiste über dem Abschnitt **Essentials** auf **Aktualisieren**, überprüfen Sie, dass der **Status** der Azure-VM auf **Beendet** geändert wurde, klicken Sie auf **Beenden**, und dann auf **OK**, wenn Sie zum Bestätigen aufgefordert werden, um die Azure-VM in den Status **Beendet (Zuordnung aufgehoben)** zu wechseln.
1. Überprüfen Sie auf dem Blatt **az140-25-vm0**, dass der **Status** der Azure-VM zum Status **Beendet (Zuordnung aufgehoben)** geändert wurde, und klicken Sie in der Symbolleiste auf **Aufzeichnen**. So wird automatisch das Blatt **Image erstellen** angezeigt.
1. Geben Sie die folgenden Einstellungen auf der Registerkarte **Grundlagen** des Blatts **Image erstellen** an:

   |Einstellung|Wert|
   |---|---|
   |Image in der Azure Compute Gallery freigeben|**Ja, als Imageversion für einen Katalog freigeben**|
   |Diesen virtuellen Computer nach dem Erstellen des Images automatisch löschen|Kontrollkästchen deaktiviert|
   |Azure Compute Gallery-Ziel|Der Name des neuen Katalogs **az14025imagegallery**|
   |Betriebssystemstatus|**Generalisiert**|

1. Klicken Sie auf der Registerkarte **Grundlagen** des Blatts **Image erstellen** unter dem Textfeld **Imagedefinition der Ziel-VM** auf **Neu erstellen**.
1. Geben Sie auf dem Blatt **VM-Imagedefinition erstellen** die folgenden Einstellungen an, und klicken Sie auf **OK**:

   |Einstellung|Wert|
   |---|---|
   |Name der VM-Imagedefinition|**az140-25-host-image**|
   |Herausgeber|**MicrosoftWindowsDesktop**|
   |Angebot|**office-365**|
   |SKU|**win11-22h2-avd-m365**|

1. Geben Sie die folgenden Einstellungen auf der Registerkarte **Grundlagen** des Blatts **Image erstellen** an, und klicken Sie auf **Überprüfen + erstellen**:

   |Einstellung|Wert|
   |---|---|
   |Versionsnummer|**1.0.0**|
   |Aus Neueste ausschließen|Kontrollkästchen deaktiviert|
   |Datum für Ende des Lebenszyklus|Ein Jahr ab dem aktuellen Datum|
   |Anzahl von Standardreplikaten|**1**|
   |Anzahl von Replikaten für die Zielregion|**1**|
   |Speicherkontotyp|**SSD Premium, LRS**|

1. Klicken Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Image erstellen** auf **Erstellen**.

   > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa 20 Minuten dauern.

1. Suchen Sie auf Ihrem Labcomputer im Webbrowser mit dem Azure-Portal nach **Azure Compute Galleries**, und wählen Sie diese aus. Klicken Sie auf dem Blatt **Azure Compute Galleries** auf den Eintrag **az14025imagegallery**, und überprüfen Sie auf dem Blatt ****az14025imagegallery****, dass der Eintrag **az140-25-host-image** vorhanden ist, der das neu erstellte Image darstellt.

#### Aufgabe 5: Bereitstellen eines Azure Virtual Desktop-Hostpools mit einem benutzerdefinierten Image

1. Verwenden Sie auf dem Labcomputer im Azure-Portal oben auf der Seite das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen**, um nach **Virtuellen Netzwerke** zu suchen und dorthin zu navigieren. Wählen Sie dann auf dem Blatt **Virtuelle Netzwerke** die Option **az140-adds-vnet11** aus. 
1. Wählen Sie auf dem Blatt **az140-adds-vnet11** die Option **Subnetze** und auf dem Blatt **Subnetze** die Option **+ Subnetz** aus. Geben Sie auf dem Blatt **Subnetz hinzufügen** die folgenden Einstellungen an (übernehmen Sie bei allen anderen Einstellungen die Standardwerte), und klicken Sie auf **Speichern**:

   |Einstellung|Wert|
   |---|---|
   |Name|**hp4-Subnet**|
   |Subnetzadressbereich|**10.0.4.0/24**|

1. Suchen Sie auf dem Labcomputer im Azure-Portal im Webbrowser, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Hostpools** und auf dem Blatt **Azure Virtual Desktop \| Hostpools** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Virtuelle Computer >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-25-RG**|
   |Hostpoolname|**az140-25-hp4**|
   |Standort|Der Name der Azure-Region, in der Sie in der ersten Übung dieses Labs Ressourcen bereitgestellt haben|
   |Überprüfungsumgebung|**Nein**|
   |Hostpooltyp|**In einem Pool zusammengefasst**|
   |Maximale Anzahl von Sitzungen|**50**|
   |Lastenausgleichsalgorithmus|**Breitensuche**|

1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Virtuelle Computer** die folgenden Einstellungen:

   |Einstellung|Wert|
   |---|---|
   |Virtuelle Azure-Computer hinzufügen|**Ja**|
   |Resource group|**Standardmäßig mit der Ressourcengruppe des Hostpools identisch.**|
   |Namenspräfix|**az140-25-p4**|
   |Typ des virtuellen Computers|**Virtueller Azure-Computer**|
   |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie in der ersten Übung dieses Labs Ressourcen bereitgestellt haben|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Sicherheitstyp|**Standard**|
   
1. Klicken Sie auf der Registerkarte **Virtuelle Computer** des Blatts **Hostpool erstellen** direkt unter der Dropdownliste **Image** auf den Link **Alle Images anzeigen**.
1. Wählen Sie auf dem Blatt **Image auswählen** unter **Weitere Elemente** **Freigegebene Images** aus und wählen sie in der Liste der freigegebenen Images **az140-25-host-image** aus. 
1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Virtuelle Computer** die folgenden Einstellungen an, und wählen Sie **Weiter: Arbeitsbereich>**

   |Einstellung|Wert|
   |---|---|
   |Größe des virtuellen Computers|**Standard D2s v3**|
   |Number of VMs (Anzahl von VMs)|**1**|
   |Typ des Betriebssystemdatenträgers|**SSD Standard**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnetz|**hp4-Subnet (10.0.4.0/24)**|
   |Netzwerksicherheitsgruppe|**Grundlegend**|
   |Öffentliche Eingangsports|**Ja**|
   |Zugelassene Eingangsports|**RDP**|
   |UPN für AD-Domänenbeitritt|**student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|
   |Domäne oder Einheit angeben|**Ja**|
   |Domäne für den Beitritt|**adatum.com**|
   |Pfad der Organisationseinheit|**OU=WVDInfra,DC=adatum,DC=com**|
   |Benutzername|Kursteilnehmer|
   |Kennwort|Pa55w.rd1234|
   |Kennwort bestätigen|Pa55w.rd1234|

1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Arbeitsbereich** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Desktop-App-Gruppe registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Hostpool erstellen** die Option **Erstellen** aus.

   > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa zehn Minuten dauern.
   > 
   > **Hinweis**: Wenn die Bereitstellung fehlschlägt, weil die Kontingentgrenze erreicht wurde, führen Sie die Schritte aus dem ersten Lab durch, um eine automatische Kontingenterhöhung der Standard D2sv3-Grenze auf 30 anzufordern.

   > **Hinweis:** Nach dem Bereitstellen von auf benutzerdefinierten Images basierenden Hosts sollten Sie das Optimierungstool von Virtual Desktop nutzen, das im [entsprechenden GitHub-Repository](https://github.com/The-Virtual-Desktop-Team/) verfügbar ist.


### Übung 2: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab bereitgestellten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten Azure-VMs

1. Wechseln Sie zum Labcomputer, und öffnen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet werden und ihre Zuordnung aufgehoben wird.
