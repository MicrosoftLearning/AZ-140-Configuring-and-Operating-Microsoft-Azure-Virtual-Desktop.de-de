---
lab:
  title: 'Lab: Packen von Azure Virtual Desktop-Anwendungen (AD DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# Lab: Packen von Azure Virtual Desktop-Anwendungen (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder ein Microsoft Entra-Konto mit der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten, der dem Azure-Abonnement zugeordnet ist, und mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Konfigurieren von Richtlinien für bedingten Zugriff für AVD (AD DS)**
- Das abgeschlossene Lab **Implementieren und Verwalten von AVD-Profilen (AD DS)**

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen Azure Virtual Desktop-Anwendungen in einer Active Directory Domain Services-Umgebung (AD DS) packen und bereitstellen.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- MSIX-App-Pakete vorzubereiten und zu erstellen.
- Implementieren eines MSIX App Attach-Images für Azure Virtual Desktop in eine AD DS-Umgebung
- Implementieren der MSIX App Attach-Funktion in Azure Virtual Desktop in eine AD DS-Umgebung

## Labdateien

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## Anweisungen

>**Wichtig:** Microsoft hat **Azure Active Directory** (**Azure AD**) in **Microsoft Entra ID** umbenannt. Ausführliche Informationen zu dieser Änderung finden Sie unter [Neuer Name für Azure Active Directory](https://learn.microsoft.com/en-us/entra/fundamentals/new-name). Das ist ein laufendes Projekt, daher kann es immer noch vorkommen, dass die Labanweisungen und die Elemente auf der Benutzerfläche voneinander abweichen, während Sie einzelne Übungen absolvieren. Berücksichtigen Sie das (in diesem Lab ist **Microsoft Entra Connect** der neue Name von **Azure Active Directory Connect**).

### Übung 1: MSIX-App-Pakete vorzubereiten und zu erstellen.

Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Konfiguration von Azure Virtual Desktop-Sitzungshosts
1. Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage
1. Vorbereiten der Azure-VM, auf der Windows 10 ausgeführt wird, für das MSIX Packaging Tool
1. Generieren eines Signaturzertifikats
1. Herunterladen der zu packenden Software
1. Installieren des MSIX Packaging Tools
1. Erstellen eines MSIX-Pakets

#### Aufgabe 1: Vorbereiten der Konfiguration von Azure Virtual Desktop-Sitzungshosts

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ Folgendes aus, um die virtuellen Azure-Computer mit den Azure Virtual Desktop-Sitzungshosts zu starten, die Sie in diesem Lab verwenden:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   > **Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich gestartet werden. 

   > **Hinweis:** Wenn Sie PSRemoting auf den Sitzungshosts in der az140-21-RG-Ressourcengruppe im ersten Vorgang des vorherigen Labs aktiviert haben (Implementieren und Verwalten von AVD-Profilen), können Sie direkt mit der nächsten Aufgabe fortfahren, ohne auf den Start der Azure-VMs zu warten. Wenn Sie PSRemoting auf den Sitzungshosts in der Ressourcengruppe az140-21-RG noch nicht aktiviert haben, warten Sie, bis die VMs gestartet werden, und führen Sie dann den folgenden Befehl aus.

1. Führen Sie in der PowerShell-Sitzung der **Cloud Shell** Folgendes aus, um PowerShell-Remoting auf den Sitzungshosts zu aktivieren.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### Aufgabe 2: Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage

1. Klicken Sie auf Ihrem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, auf der Symbolleiste des Cloud Shell-Bereichs auf das Symbol **Dateien hochladen/herunterladen**. Wählen Sie im Dropdownmenü **Hochladen** aus, und laden Sie die Dateien **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** und **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** in das Cloud Shell-Basisverzeichnis hoch.
1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich den folgenden Befehl aus, um eine Azure-VM bereitzustellen, auf der Windows 10 ausgeführt wird und die Sie für das Erstellen von MSIX-Paketen und Verknüpfen mit der Microsoft Entra DS-Domäne verwenden:

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. Dies kann etwa zehn Minuten dauern. 

#### Aufgabe 3: Vorbereiten der Azure-VM, auf der Windows 10 ausgeführt wird, für das MSIX Packaging Tool

1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** in der Liste der virtuellen Computer den Eintrag **az140-cl-vm42** aus. Daraufhin wird das Blatt **az140-cl-vm42** geöffnet.
1. Wählen Sie auf dem Blatt **az140-cl-vm42** **Verbindung herstellen** aus und wählen Sie im Dropdownmenü **Verbindung über Bastion herstellen** aus.
1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Benutzernamen **wvdadmin1@adatum.com** und dem Kennwort an, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben. 
1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** **Windows PowerShell ISE** als Administrator*in, und führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Betriebssystem für MSIX-Pakete vorzubereiten:

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **Hinweis:** Mit den letzten dieser Registrierungsänderungen wird die Benutzerzugriffssteuerung deaktiviert. Dies ist technisch nicht erforderlich, vereinfacht aber den in diesem Lab dargestellten Prozess.

#### Aufgabe 4: Generieren eines Signaturzertifikats

> **Hinweis:** In diesem Lab verwenden Sie ein selbstsigniertes Zertifikat. In einer Produktionsumgebung sollten Sie je nach beabsichtigter Verwendung ein von einer öffentlichen Zertifizierungsstelle oder einer internen Zertifizierungsstelle ausgestelltes Zertifikat verwenden.

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um ein selbstsigniertes Zertifikat mit dem auf **Adatum** festgelegten Common Name-Attribut zu generieren. Speichern Sie das Zertifikat im Ordner **Persönlich** des Zertifikatspeichers **Lokaler Computer**:

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Konsole **Zertifikate** zu starten, die den Zertifikatspeicher „Lokaler Computer“ als Ziel hat:

   ```powershell
   certlm.msc
   ```

1. Erweitern Sie im Konsolenbereich **Zertifikate** den Ordner **Persönlich**, wählen Sie den Unterordner **Zertifikate** aus, klicken Sie mit der rechten Maustaste auf das Zertifikat **Adatum**, und wählen Sie im Rechtsklickmenü **Alle Aufgaben** gefolgt von **Exportieren** aus. Dadurch wird der **Zertifikatexport-Assistent** gestartet. 
1. Klicken Sie auf der Seite **Willkommen** des **Zertifikatexport-Assistenten** auf **Weiter**.
1. Aktivieren Sie auf der Seite **Privaten Schlüssel exportieren** des **Zertifikatexport-Assistenten** die Option **Ja, privaten Schlüssel exportieren**, und klicken Sie dann auf **Weiter**.
1. Aktivieren Sie auf der Seite **Dateiformat exportieren** des **Zertifikatexport-Assistenten** das Kontrollkästchen **Alle erweiterten Eigenschaften exportieren**, deaktivieren Sie das Kontrollkästchen **Zertifikatdatenschutz aktivieren**, und klicken Sie auf **Weiter**.
1. Aktivieren Sie auf der Seite **Sicherheit** des **Zertifikatexport-Assistenten** das Kontrollkästchen **Kennwort**, geben Sie im Textfeld unten **Pa55w.rd1234** ein, und klicken Sie auf **Weiter**.
1. Klicken Sie auf der Seite **Zu exportierende Datei** des **Zertifikatexport-Assistenten** im Textfeld **Dateiname** auf **Durchsuchen**. Navigieren Sie im Dialogfeld **Speichern unter** zum Ordner **C:\\Allfiles\\Labs\\04** (Ordner zunächst erstellen), geben Sie im Textfeld **Dateiname** **adatum.pfx** ein, und klicken Sie auf **Speichern**.
1. Stellen Sie zurück auf der Seite **Zu exportierende Datei** des **Zertifikatexport-Assistenten** sicher, dass das Textfeld den Eintrag **C:\\Allfiles\\Labs\\04\\adatum.pfx** enthält, und klicken Sie auf **Weiter**.
1. Klicken Sie auf der Seite **Completing Certificate Export Wizard** (Zertifikatexport-Assistenten abschließen) des **Zertifikatexport-Assistenten** zunächst auf **Fertig stellen** und dann auf **OK**, um den erfolgreichen Export zu bestätigen. 

   > **Hinweis:** Da Sie ein selbstsigniertes Zertifikat verwenden, müssen Sie es im Zertifikatspeicher **Vertrauenswürdige Personen** auf den Zielsitzungshosts installieren.

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das neu generierte Zertifikat im Zertifikatspeicher **Vertrauenswürdige Personen** auf den Zielsitzungshosts zu installieren:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### Aufgabe 5: Herunterladen der zu packenden Software

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** **Microsoft Edge**, und navigieren Sie zu **https://github.com/microsoft/XmlNotepad**.
1. Klicken Sie auf der Seite **microsoft/XmlNotepad** **readme.md** auf den Downloadlink für eigenständige herunterladbare Installationsprogramme, und laden Sie die komprimierten Installationsdateien herunter.
1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** den Datei-Explorer, und navigieren Sie zum Ordner **Downloads**. Öffnen Sie die komprimierte Datei, kopieren Sie den Inhalt aus dem Ordner in die komprimierte Datei, und fügen Sie sie in das Verzeichnis **C:\\AllFiles\\Labs\\04\\** ein. 

#### Aufgabe 6: Installieren des MSIX Packaging Tools

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** die **Microsoft Store**-App.
1. Suchen Sie in der **Microsoft Store**-App nach dem **MSIX Packaging Tool**, und wählen Sie es aus. Klicken Sie auf der Seite **MSIX Packaging Tool** auf **Herunterladen**.
1. Wenn Sie dazu aufgefordert werden, überspringen Sie den Anmeldeschritt. Warten Sie bis die Installation abgeschlossen ist, klicken Sie auf **Starten**, und wählen Sie im Dialogfeld **Diagnosedaten senden** die Option **Ablehnen** aus. 

#### Aufgabe 7: Erstellen eines MSIX-Pakets

1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-25-vm42** zum Fenster **Administrator: Windows PowerShell ISE**, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Windows-Suchdienst zu deaktivieren:

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Ordner zu erstellen, der das MSIX-Paket hostet:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den alternativen Datenstrom „Zone.Identifier“ aus den extrahierten Installationsprogrammdateien zu entfernen. Dieser weist den Wert „3“ auf, was bedeutet, dass er über das Internet heruntergeladen wurde:

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** zur Benutzeroberfläche des **MSIX Packaging Tools** und wählen Sie auf der Seite **Aufgabe auswählen** den Eintrag **Anwendungspaket – Erstellen Ihres App-Pakets** aus. Daraufhin wird der Assistent **Neues Paket erstellen** gestartet.
1. Stellen Sie sicher, dass auf der Seite **Umgebung auswählen** des Assistenten **Neues Paket erstellen** die Option **Create package on this computer** (Paket auf diesem Computer erstellen) aktiviert ist. Klicken Sie auf **Weiter**, und warten Sie auf den Abschluss der Installation des **MSIX Packaging Tool-Treibers**.

   > **Hinweis:** Die Installation des MSIX Packaging Tool Driver dauert 5–10 Minuten. In der Statusspalte wird zunächst **Überprüfung** und nach der Installation **Installiert** angezeigt.

1. Sehen Sie sich die Empfehlungen auf der Seite **Computer vorbereiten** des Assistenten **Neues Paket erstellen** an. Wenn ein Neustart aussteht, starten Sie das Betriebssystem neu, melden Sie sich wieder mit dem Konto **wvdadmin1@adatum.com** an, und starten Sie das **MSIX Packaging Tool**, bevor Sie fortfahren. 

   >**Hinweis:** Das MSIX Packaging Tool deaktiviert vorübergehend Windows Update und Windows Search. In diesem Fall ist der Windows-Suchdienst bereits deaktiviert. 

1. Klicken Sie auf der Seite **Computer vorbereiten** des Assistenten **Neues Paket erstellen** auf **Weiter**.
1. Klicken Sie auf der Seite **Installer auswählen** des Assistenten **Neues Paket erstellen** neben dem Textfeld **Choose the installer you want to package** (Zu packenden Installer auswählen) auf **Durchsuchen**. Navigieren Sie im Dialogfeld **Öffnen** zum Ordner **C:\\AllFiles\\Labs\\04**, wählen Sie **XmlNotepadSetup.msi** aus, und klicken Sie auf **Öffnen**. 
1. Wählen Sie auf der Seite **Installer auswählen** des Assistenten **Neues Paket erstellen** in der Dropdownliste **Signing preference** (Signierungseinstellung) den Eintrag **Sign with a certificate (.pfx)** (Mit Zertifikat signieren (PFX)) aus. Klicken Sie neben dem Textfeld **Browse for certificate** (Nach Zertifikat durchsuchen) auf **Durchsuchen**, und navigieren Sie im Dialogfeld **Öffnen** zum Ordner **C:\\AllFiles\\Labs\\04**. Wählen Sie die Datei **adatum.pfx** aus, klicken Sie auf **Öffnen**, geben Sie im Textfeld **Kennwort** **Pa55w.rd1234** ein, und klicken Sie auf **Weiter**.
1. Überprüfen Sie auf der Seite **Paketinformationen** des Assistenten **Neues Paket erstellen** die Paketinformationen. Stellen Sie anschließend sicher, dass der Name des Herausgebers auf **CN=Adatum** festgelegt ist, und klicken Sie auf **Weiter**.
1. Wählen Sie auf der Seite **Beschleuniger für das Anwenden auf das Paket auswählen** **Weiter** aus. Dadurch wird die Installation der heruntergeladenen Software initiiert.
1. Akzeptieren Sie im Fenster **XML Notepad Setup** die Bedingungen im Lizenzvertrag, und wählen Sie **Installieren** und nach Abschluss der Installation **Fertig stellen** aus.
1. Wählen Sie auf der Seite **Installation** des Assistenten zum **Erstellen eines neuen Pakets** **Weiter** aus.
1. Überprüfen Sie auf der Seite **Verwalten der ersten Startaufgaben** des Assistenten **Neues Paket erstellen** die bereitgestellten Informationen, und wählen Sie **Weiter** aus.
1. Wenn das Dialogfeld **Are you done?** (Sind Sie fertig?) angezeigt wird, klicken Sie auf **Yes, move on** (Ja, fortfahren).
1. Stellen Sie auf der Seite **Services report** (Dienstbericht) des Assistenten **Neues Paket erstellen** sicher, dass keine Dienste aufgeführt sind, und klicken Sie auf **Weiter**.
1. Geben Sie auf der Seite **Paket erstellen** des Assistenten **Neues Paket erstellen** im Textfeld **Speicherort** **C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix** ein, und klicken Sie auf **Erstellen**.
1. Notieren Sie sich den Speicherort des gespeicherten Pakets im Dialogfeld **Package successfully created** (Paket erfolgreich erstellt), und klicken Sie auf **Schließen**.
1. Wechseln Sie zum Fenster des Datei-Explorers, navigieren Sie zum Ordner **C:\\Allfiles\\Labs\\04\\XmlNotepad**, und überprüfen Sie, ob er die Dateien „*.msix“ und „*.xml“ enthält.
1. Kopieren Sie die Datei **XmlNotepad.msix** in den Ordner **C:\\Allfiles\\Labs\\04**.


### Übung 2: Implementieren eines MSIX App Attach-Images für Azure Virtual Desktop in eine AD DS-Umgebung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Aktivieren von Hyper-V auf den Azure-VMs, auf denen Windows 11 Enterprise Multi-Session ausgeführt wird
1. Erstellen eines MSIX App Attach-Images

#### Aufgabe 1: Aktivieren von Hyper-V auf den Azure-VMs, auf denen Windows 11 Enterprise Multi-Session ausgeführt wird

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Azure Virtual Desktop-Zielhosts für MSIX App Attach vorzubereiten: 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um auf den Azure Virtual Desktop-Hosts Hyper-V und die zugehörigen Verwaltungstools einschließlich des Hyper-V-PowerShell-Moduls zu installieren:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. Wenn Sie dazu aufgefordert werden, klicken Sie zum Neustarten des Zielbetriebssystems auf **Ja**.
1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um auf dem lokalen Computer Hyper-V und die zugehörigen Verwaltungstools einschließlich des Hyper-V-PowerShell-Moduls zu installieren:

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Klicken Sie nach Abschluss der Installation der Hyper-V-Komponenten auf **Ja**, um das Betriebssystem neu zu starten. Melden Sie sich nach dem Neustart wieder mit dem Benutzernamen **wvdadmin1@adatum.com** und dem Kennwort an, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben.

#### Aufgabe 2: Erstellen eines MSIX App Attach-Images

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** **Microsoft Edge**, und navigieren Sie zu **https://aka.ms/msixmgr**. Dadurch wird die Datei **msixmgr.zip** (MSIX Mgr-Toolarchiv) automatisch in den Ordner **Downloads** heruntergeladen.
1. Navigieren Sie im Datei-Explorer zum Ordner **Downloads**, öffnen Sie die komprimierte Datei, und kopieren Sie den Inhalt des Ordners **x64** (einschließlich des Ordners) in den Ordner **C:\\AllFiles\\Labs\\04**. 
1. Starten Sie in der Bastion-Sitzung für **az140-21-vm42-0** die **Windows PowerShell ISE** als Administrator*in, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Ordner zu erstellen, in dem das MSIX App Attach-Image gespeichert wird:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den VHD zu erstellen, der die MSIX-Dateien hostet. Entpacken Sie sie in das MSIX-Paket, das Sie in der vorherigen Aufgabe erstellt haben:

   ```powershell
   $appName = 'XmlNotepad'
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination ..\MSIXVhds\$appName.vhd -applyacls -create -filetype vhd -vhdSize 128 -rootDirectory Apps
   ```

1. Navigieren Sie in der Bastion-Sitzung für **az140-cl-vm42** im Datei-Explorer zum Ordner **C:\AllFiles\Labs\04\MSIXVhds** und stellen Sie sicher, dass Sie über einen virtuellen Datenträger mit dem Namen „XmlNotepad.vhd“ verfügen.

### Übung 3: Implementieren von MSIX App Attach auf Azure Virtual Desktop-Sitzungshosts

Die Hauptaufgaben für diese Übung sind Folgende:

1. Konfigurieren von Active Directory-Gruppen mit Azure Virtual Desktop-Hosts
1. Einrichten der Azure Files-Freigabe für MSIX App Attach
1. Bereitstellen und Registrieren des MSIX App Attach-Images auf Azure Virtual Desktop-Sitzungshosts
1. Veröffentlichen von MSIX-Apps in einer Anwendungsgruppe
1. Überprüfen der Funktionalität von MSIX App Attach

#### Aufgabe 1: Konfigurieren von Active Directory-Gruppen mit Azure Virtual Desktop-Hosts

1. Wechseln Sie zum Labcomputer, auf dem im Webbrowser das Azure-Portal angezeigt wird, suchen Sie nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-dc-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** **Verbindung herstellen** aus und wählen Sie im Dropdownmenü **Verbindung über Bastion herstellen** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Bastion-Sitzung auf **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator*in.
1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um ein AD DS-Gruppenobjekt zu erstellen, das mit dem in diesem Lab verwendeten Microsoft Entra-Mandanten synchronisiert wird:

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **Hinweis:** Sie verwenden diese Gruppe, um Azure Virtual Desktop-Hosts Berechtigungen für die Dateifreigabe **az140-42-msixvhds** zu gewähren.

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den im vorherigen Schritt erstellten Gruppen Mitglieder hinzuzufügen:

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Server neu zu starten, die Mitglieder der Gruppe „az140-hosts-42-p1“ sind:

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **Hinweis:** Mit diesem Schritt wird sichergestellt, dass die Änderung der Gruppenmitgliedschaft wirksam wird. 

1. Erweitern Sie in der Bastion-Sitzung für **az140-dc-vm11** im **Startmenü** den Ordner **Microsoft Azure AD Connect**, und wählen Sie **Microsoft Azure AD Connect** aus.
1. Klicken Sie auf der Seite **Willkommen bei Azure AD Connect** des Fensters **Microsoft Azure Active Directory Connect** auf **Konfigurieren**.
1. Klicken Sie auf der Seite **Weitere Aufgaben** im Fenster **Microsoft Entra Connect** zunächst auf **Synchronisierungsoptionen anpassen** und dann auf **Weiter**.
1. Authentifizieren Sie sich auf der Seite **Mit Microsoft Entra verbinden** im Fenster **Microsoft Azure Active Directory Connect** mit dem Benutzerprinzipalnamen des zuvor in dieser Aufgabe identifizierten Benutzerkontos **aadsyncuser** und dem Kennwort, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben.
1. Wählen Sie auf der Seite **Verzeichnisse verbinden** im Fenster **Microsoft Azure Active Directory Connect** die Option **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Filtern von Domänen und Organisationseinheiten** im Fenster **Microsoft Azure Active Directory Connect**, dass die Option **Ausgewählte Domänen und Organisationseinheiten synchronisieren** aktiviert ist. Erweitern Sie den Knoten **adatum.com**, aktivieren Sie das Kontrollkästchen neben der Organisationseinheit **WVDInfra** (alle anderen ausgewählten Kontrollkästchen unverändert lassen), und klicken Sie auf **Weiter**.
1. Übernehmen Sie auf der Seite **Optionale Features** im Fenster **Microsoft Azure Active Directory Connect** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Bereit zur Konfiguration** im Fenster **Microsoft Azure Active Directory Connect**, dass das Kontrollkästchen **Starten Sie den Synchronisierungsvorgang, nachdem die Konfiguration abgeschlossen wurde.** aktiviert ist, und klicken Sie auf **Konfigurieren**.
1. Überprüfen Sie die Informationen auf der Seite **Konfiguration abgeschlossen**, und wählen Sie **Beenden** aus, um das Fenster **Microsoft Azure Active Directory Connect** zu schließen.
1. Starten Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mithilfe der Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das im Microsoft Entra-Mandanten, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist, über die Rolle „Globale*r Administrator*in“ verfügt.
1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach **Microsoft Entra ID**, und wählen Sie diese Option aus, um zu dem Microsoft Edge-Mandanten zu navigieren, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist.
1. Wählen Sie auf dem Blatt „Microsoft Entra ID“ auf der vertikalen Menüleiste links im Abschnitt **Verwalten** **Gruppen** aus. 
1. Wählen Sie auf dem Blatt **Gruppen | Alle Gruppen** in der Liste der Gruppen den Eintrag **az140-hosts-42-p1** aus.

   > **Hinweis:** Möglicherweise müssen Sie die Seite aktualisieren, damit die Gruppe angezeigt wird.

1. Klicken Sie auf dem Blatt **az140-hosts-42-p1** auf der vertikalen Menüleiste links im Abschnitt **Verwalten** auf **Mitglieder**.
1. Überprüfen Sie auf dem Blatt **az140-hosts-42-p1 | Mitglieder**, ob die Liste **Direkte Mitglieder** die drei Hosts des Azure Virtual Desktop-Pools enthält, die Sie der Gruppe zuvor in dieser Aufgabe hinzugefügt haben.

#### Aufgabe 2: Einrichten der Azure Files-Freigabe für MSIX App Attach

1. Wechseln Sie auf dem Labcomputer zurück zur Bastion-Sitzung für **az140-cl-vm42**.
1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** Microsoft Edge im InPrivate-Modus. Navigieren Sie zum [Azure-Portal](https://portal.azure.com) und melden Sie sich mit den Anmeldeinformationen eines Benutzerkontos mit der Rolle „Besitzer*in“ im Abonnement an, das Sie in diesem Lab verwenden.

   > **Hinweis:** Stellen Sie sicher, dass Sie den Microsoft Edge-Modus „InPrivate“ verwenden.

1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** im Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach **Speicherkonten**, und wählen Sie die Option aus. Wählen Sie anschließend auf dem Blatt **Speicherkonten** das Speicherkonto aus, das Sie zum Hosten von Benutzerprofilen konfiguriert haben.

   > **Hinweis:** Dieser Teil des Labs setzt den Abschluss des Lab **Implementieren und Verwalten von Speicher für AVD (AD DS)** oder **Implementieren und Verwalten von Speicher für AVD (Microsoft Entra DS)** voraus.

   > **Hinweis:** In Produktionsszenarios sollten Sie die Verwendung eines separaten Speicherkontos in Betracht ziehen. Dies erfordert die Konfiguration dieses Speicherkontos für die Microsoft Entra DS-Authentifizierung, die Sie bereits für das Speicherkonto implementiert haben, das Benutzerprofile hosten soll. Sie verwenden dasselbe Speicherkonto, um doppelte Schritte in einzelnen Labs zu minimieren.

1. Klicken Sie auf dem Blatt für das Speicherkonto im vertikalen Menü links auf **Zugriffssteuerung (IAM)**.
1. Klicken Sie auf dem Blatt **Zugriffssteuerung (IAM)** für das Speicherkonto auf **+ Hinzufügen**, und wählen Sie im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus. 
1. Geben Sie auf dem Blatt **Rollenzuweisung hinzufügen** in der Registerkarte **Rolle** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

   |Einstellung|Wert|
   |---|---|
   |Auftragsfunktionsrolle|**Speicherdateidaten-SMB-Freigabemitwirkender**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** auf der Registerkarte **Mitglieder** **+ Mitglieder auswählen** aus, geben Sie die folgenden Einstellungen an, und wählen Sie **Auswählen** aus. 

   |Einstellung|Wert|
   |---|---|
   |Auswählen|**az140-wvd-users**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** die Option **Überprüfen + Zuweisen** und anschließend erneut die Option **Überprüfen + Zuweisen** aus.
1. Klicken Sie auf dem Blatt **Zugriffssteuerung (IAM)** für das Speicherkonto auf **+ Hinzufügen**, und wählen Sie im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus. 
1. Geben Sie auf dem Blatt **Rollenzuweisung hinzufügen** in der Registerkarte **Rolle** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

   |Einstellung|Wert|
   |---|---|
   |Auftragsfunktionsrolle|**Speicherdateidaten-SMB-Freigabemitwirkender mit erhöhten Rechten**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** auf der Registerkarte **Mitglieder** **+ Mitglieder auswählen** aus, geben Sie die folgenden Einstellungen an, und wählen Sie **Auswählen** aus. 

   |Einstellung|Wert|
   |---|---|
   |Auswählen|**az140-wvd-admins**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** die Option **Überprüfen + Zuweisen** und anschließend erneut die Option **Überprüfen + Zuweisen** aus.
1. Klicken Sie auf dem Blatt **Zugriffssteuerung (IAM)** für das Speicherkonto auf **+ Hinzufügen**, und wählen Sie im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus. 
1. Geben Sie auf dem Blatt **Rollenzuweisung hinzufügen** in der Registerkarte **Rolle** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

   |Einstellung|Wert|
   |---|---|
   |Auftragsfunktionsrolle|**Speicherdateidaten-SMB-Freigabemitwirkender mit erhöhten Rechten**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** auf der Registerkarte **Mitglieder** **+ Mitglieder auswählen** aus, geben Sie die folgenden Einstellungen an, und wählen Sie **Auswählen** aus. 

   |Einstellung|Wert|
   |---|---|
   |Auswählen|**az140-hosts-42-p1**|

1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** die Option **Überprüfen + Zuweisen** und anschließend erneut die Option **Überprüfen + Zuweisen** aus.

1. Klicken Sie auf dem Blatt für das Speicherkonto im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** auf **Dateifreigaben** und anschließend auf **+ Dateifreigabe**.
1. Geben Sie auf dem Blatt **Neue Dateifreigabe** die folgenden Einstellungen an und wählen Sie **Weiter: Sicherung >** aus (behalten Sie für die anderen Einstellungen die Standardwerte bei):

   |Einstellung|Wert|
   |---|---|
   |Name|**az140-42-msixvhds**|
   |Zugriffsebene|**Transaktion optimiert**|

1. Wählen Sie auf dem Blatt **Sicherung** das Kontrollkästchen **Sicherung aktivieren** ab, wählen Sie **Überprüfen + Erstellen** aus, warten Sie, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie dann **Erstellen** aus.
1. Wählen Sie im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, in der Liste der Dateifreigaben die neu erstellte Dateifreigabe aus. 
1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** die **Eingabeaufforderung**, und führen Sie über das **Eingabeaufforderungsfenster** Folgendes aus, um der Freigabe **az140-42-msixvhds** ein Laufwerk zuzuordnen (Platzhalter `<storage-account-name>` durch den Namen des Speicherkontos ersetzen). Überprüfen Sie anschließend, ob der Befehl erfolgreich abgeschlossen wurde:

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über das **Eingabeaufforderungsfenster** Folgendes aus, um den Computerkonten der Sitzungshosts die erforderlichen NTFS-Berechtigungen zu erteilen:

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**Hinweis:** Sie können diese Berechtigungen auch mithilfe des Datei-Explorers festlegen, während Sie als **wvdadmin1\@adatum.com** angemeldet sind. 

   >**Hinweis:** Im nächsten Schritt überprüfen Sie die Funktionalität von MSIX App Attach.

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die VHD-Datei, die Sie in der vorherigen Übung erstellt haben, in die Azure Files-Freigabe zu kopieren, die Sie zuvor in dieser Übung erstellt haben:

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages\' -Force
   ```

#### Aufgabe 3: Bereitstellen und Registrieren des MSIX App Attach-Images auf Azure Virtual Desktop-Sitzungshosts

1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** im Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach **Azure Virtual Desktop**, und wählen Sie die Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** **Hostpools** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools den Eintrag **az140-21-hp1** aus.
1. Klicken Sie auf dem Blatt **az140-21-hp1 \| Eigenschaften** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **MSIX-Pakete**.
1. Klicken Sie auf dem Blatt **az140-21-hp1 \| MSIX-Pakete** auf **+ Hinzufügen**.
1. Geben Sie auf dem Blatt **MSIX-Paket hinzufügen** im Textfeld **MSIX-Imagepfad** den Pfad zur Datei **XmlNotepad.vhd** im Format `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd` ein (Platzhalter `<storage-account-name>` durch den Namen des Speicherkontos ersetzen, das die Dateifreigabe **az140-42-msixvhds** hostet), und klicken Sie auf **Hinzufügen**.
1. Geben Sie auf dem Blatt **MSIX-Paket hinzufügen** die folgenden Einstellungen an, und klicken Sie auf **Hinzufügen**:

   |Einstellung|Wert|
   |---|---|
   |MSIX-Imagepfad|**\\\\\<storage-account-name\>.file.core.windows.net\\az140-42-msixvhds\\XmlNotepad.vhd**, wobei der Platzhalter `<storage-account-name>` den Namen des Speicherkontos angibt, das die Dateifreigabe **az140-42-msixvhds** hostet|
   |MSIX-Paket|Name, der bei der Paketerstellung generiert wurde|
   |Anzeigename|**XML Notepad**|
   |Registrierungstyp|**Bei Bedarf**|
   |Zustand|**Aktiv**|

#### Aufgabe 4: Veröffentlichen von MSIX-Apps in einer Anwendungsgruppe

> **Hinweis:** Sie veröffentlichen die MSIX-App sowohl in der Remote-App als auch in der Desktop-App-Gruppe. 

1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm42** im Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach **Azure Virtual Desktop**, und wählen Sie die Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** **Anwendungsgruppen** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** den Anwendungsgruppeneintrag **az140-21-hp1-Utilities-RAG** aus.
1. Klicken Sie auf dem Blatt **az140-21-hp1-Utilities-RAG** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **Anwendungen**. 
1. Wählen Sie auf dem Blatt **az140-21-hp1-Utilities-RAG \| Anwendungen** **+ Hinzufügen** aus.
1. Verwenden Sie auf dem Blatt **Anwendung hinzufügen** die Registerkarten **Basics** und **Symbol**, um die folgenden Einstellungen anzugeben, und wählen Sie **Überprüfen + Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**App anfügen**|
   |Paket|Name, der für das Paket steht, das im Image enthalten ist|
   |Application|**XMLNOTEPAD**|
   |Anwendungskennung|**XML Notepad**|
   |Anzeigename|**XML Notepad**|
   |Beschreibung|**XML Notepad**|

1. Überprüfen Sie die konfigurierten Einstellungen, und wählen Sie dann **Hinzufügen** aus.
1. Navigieren Sie zurück zum Blatt **Azure Virtual Desktop \| Anwendungsgruppen**, und wählen Sie den Anwendungsgruppeneintrag **az140-21-hp1-DAG** aus.
1. Klicken Sie auf dem Blatt **az140-21-hp1-DAG** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **Anwendungen**. 
1. Wählen Sie auf dem Blatt **az140-21-hp1-DAG \| Anwendungen** **+ Hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**MSIX-Paket**|
   |MSIX-Paket|Name, der für das Paket steht, das im Image enthalten ist|
   |Anwendungskennung|**XML Notepad**|
   |Anzeigename|**XML Notepad**|
   |Beschreibung|**XML Notepad**|

1. Überprüfen Sie die konfigurierten Einstellungen, und wählen Sie dann **Hinzufügen** aus.

#### Aufgabe 5: Überprüfen der Funktionalität von MSIX App Attach

1. Starten Sie innerhalb der Bastion.Sitzung für **az140-cl-vm42** Microsoft Edge, navigieren Sie zur [Downloadseite für den Windows-Desktopclient](https://go.microsoft.com/fwlink/?linkid=2068602) und wählen Sie nach Abschluss des Downloads **Datei öffnen** aus, um die Installation zu starten. Wählen Sie auf der Seite **Installationsbereich** des Assistenten **Remotedesktopsetup** die Option **Für alle Benutzer auf diesem Computer installieren** aus, und klicken Sie auf **Installieren**. 
1. Vergewissern Sie sich nach Abschluss der Installation, dass das Kontrollkästchen **Launch Remote Desktop when setup exits** (Remotedesktop starten, wenn das Setup abgeschlossen ist) aktiviert ist, und klicken Sie auf **Fertig stellen**, um den Remotedesktopclient zu starten.
1. Klicken Sie im Fenster für den **Remotedesktopclient** auf **Abonnieren**, und melden Sie sich bei Aufforderung mit dem Benutzerprinzipalnamen **aduser1** und dem Kennwort an, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben. 
1. Doppelklicken Sie im Fenster des **Remotedesktopclients** innerhalb des Abschnitts **az140-21-ws1** auf das **XML Notepad**-Symbol. Geben Sie bei Aufforderung das Kennwort ein, und überprüfen Sie, ob XML Notepad erfolgreich gestartet wurde.
1. Klicken Sie in der Bastion-Sitzung für **az140-cl-vm42** mit der rechten Maustaste auf **Start**. Wählen Sie im Kontextmenü die Option **Herunterfahren oder abmelden** aus, und wählen Sie im Untermenü **Abmelden** aus.
1. Wählen Sie im Dialogfeld **Getrennt** **Schließen** aus.

### Übung 4: Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab bereitgestellten und verwendeten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten und verwendeten Azure-VMs

1. Wechseln Sie zum Labcomputer, und öffnen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten und verwendeten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten und verwendeten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet werden und ihre Zuordnung aufgehoben wird.
