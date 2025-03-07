---
lab:
  title: "Lab: Implementieren und Verwalten von Azure\_Virtual\_Desktop-Profilen (Microsoft Entra\_DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# Lab: Implementieren und Verwalten von Azure Virtual Desktop-Profilen (Microsoft Entra DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder ein Microsoft Entra-Konto mit der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten, der dem Azure-Abonnement zugeordnet ist, und mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement
- Eine Azure Virtual Desktop-Umgebung, die im Lab **Einführung in Azure Virtual Desktop (Microsoft Entra DS)** bereitgestellt wurde

## Geschätzte Dauer

30 Minuten

## Labszenario

Sie müssen die Azure Virtual Desktop-Profilverwaltung in einer Microsoft Entra DS-Umgebung implementieren.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Konfigurieren von Azure Files, um Profilcontainer für Azure Virtual Desktop in einer Microsoft Entra DS-Umgebung zu speichern
- Implementieren von FSLogix-basierten Profilen für Azure Virtual Desktop in einer Microsoft Entra DS-Umgebung

## Labdateien

- Keine

## Anweisungen

### Übung 1: FSLogix-basierte Profile für Azure Virtual Desktop zu implementieren.

Die Hauptaufgaben für diese Übung sind Folgende:

1. Konfigurieren der Gruppe der lokalen Administratoren auf Azure Virtual Desktop-Sitzungshost-VMs
1. Konfigurieren von FSLogix-basierten Profilen auf Azure Virtual Desktop-Sitzungshost-VMs
1. Testen von FSLogix-basierten Profilen mit Azure Virtual Desktop
1. Löschen von Azure-Labressourcen

#### Aufgabe 1: Konfigurieren der Gruppe der lokalen Administratoren auf Azure Virtual Desktop-Sitzungshost-VMs

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und wählen Sie dann **Speicher erstellen** aus. 

1. Führen Sie in der PowerShell-Sitzung im Bereich **Cloud Shell** Folgendes aus, um die Azure-VMs mit den Azure Virtual Desktop-Sitzungshosts zu starten, die Sie in diesem Lab verwenden:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**Hinweis:** Warten Sie, bis die Azure-VMs ausgeführt werden, bevor Sie mit dem nächsten Schritt fortfahren.
   
      
1. Führen Sie in der PowerShell-Sitzung auf der **Cloud Shell**-Seite Folgendes aus, um PowerShell-Remoting auf den Sitzungshosts zu aktivieren.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. Schließen der Cloud Shell
1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11a** aus. Daraufhin wird das Blatt **az140-cl-vm11a** geöffnet.
1. Wählen Sie auf dem Blatt **az140-cl-vm11a** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11a \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**aadadmin1@adatum.com**|
   |Kennwort|Zuvor konfiguriertes Kennwort|

1. Navigieren Sie in der Bastion-Sitzung für **az140-cl-vm11a** über das Startmenü zum Ordner **Windows-Verwaltungsprogramme**, erweitern Sie ihn, und wählen Sie **Active Directory-Benutzer*innen und -Computer** aus.
1. Klicken Sie in der Konsole **Active Directory-Benutzer und -Computer** mit der rechten Maustaste auf den Domänenknoten, und wählen Sie **Neu** und dann **Organisationseinheit** aus. Geben Sie im Dialogfeld **Neues Objekt – Organisationseinheit** in das Feld **Name** den Namen **ADDC-Benutzer** ein, und wählen Sie **OK** aus.
1. Klicken Sie in der Konsole **Active Directory-Benutzer und -Computer** mit der rechten Maustaste auf **ADDC-Benutzer**, und wählen Sie **Neu** und dann **Gruppe** aus. Geben Sie im Dialogfeld **Neues Objekt – Gruppe** die folgenden Einstellungen ein, und wählen Sie **OK** aus:

   |Einstellung|Wert|
   |---|---|
   |Gruppenname|**Lokale Administratoren**|
   |Gruppenname (vor Windows 2000)|**Lokale Administratoren**|
   |Group scope|**Global**|
   |Gruppentyp|**Security**|

1. Zeigen Sie in der Konsole **Active Directory-Benutzer und -Computer** die Eigenschaften der Gruppe **Lokale Administratoren** an, wechseln Sie zur Registerkarte **Mitglieder**, und wählen Sie **Hinzufügen** aus. Geben Sie im Dialogfeld **Benutzer, Kontakte, Computer, Dienstkonten oder Gruppen** unter **Geben Sie die auszuwählenden Objektnamen ein** die Namen **aadadmin1;wvdaadmin1** ein, und wählen Sie **OK** aus.
1. Navigieren Sie in der Bastion-Sitzung für **az140-cl-vm11a** über das Startmenü zum Ordner **Windows-Verwaltungsprogramme**, erweitern Sie ihn, und wählen Sie **Gruppenrichtlinienverwaltung** aus.
1. Navigieren Sie in der Konsole **Gruppenrichtlinienverwaltung** zur Organisationseinheit **AADDC-Computer**, klicken Sie mit der rechten Maustaste auf das Symbol **AADDC-Computer-GPO**, und wählen Sie **Bearbeiten** aus.
1. Erweitern Sie in der Konsole **Gruppenrichtlinienverwaltungs-Editor** die Einträge **Computerkonfiguration**, **Richtlinien**, **Windows-Einstellungen** und **Sicherheitseinstellungen**, klicken Sie mit der rechten Maustaste auf **Eingeschränkte Gruppen**, und wählen Sie **Gruppe hinzufügen** aus.
1. Wählen Sie im Dialogfeld **Gruppe hinzufügen** im Textfeld **Gruppe** die Option **Durchsuchen** aus. Geben Sie **Lokale Administratoren** im Dialogfeld **Gruppen auswählen** unter **Geben Sie die auszuwählenden Objektnamen ein** ein, und wählen Sie **OK** aus.
1. Navigieren Sie zurück zum Dialogfeld **Gruppe hinzufügen**, und wählen Sie **OK** aus.
1. Wählen Sie im Dialogfeld **ADATUM\Eigenschaften lokaler Administratoren** im Abschnitt **Diese Gruppe ist Mitglied von** die Option **Hinzufügen** aus. Geben Sie im Dialogfeld **Gruppenmitgliedschaft** den Begriff **Administratoren** ein, und wählen Sie **OK** und dann erneut **OK** aus, um die Änderung abzuschließen.

   >**Hinweis:** Stellen Sie sicher, dass Sie den Abschnitt **Diese Gruppe ist Mitglied von** verwenden.

1. Starten Sie in der Bastion-Sitzung für die Azure-VM „az140-cl-vm11a“ die PowerShell ISE als Administrator*in, und führen Sie den folgenden Befehl aus, um die zwei Azure Virtual Desktop-Hosts neu zu starten und dadurch die Verarbeitung der Gruppenrichtlinie auszulösen:

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. Warten Sie, bis das Skript abgeschlossen ist. Dieser Vorgang dauert etwa drei Minuten.

#### Aufgabe 2: Konfigurieren von FSLogix-basierten Profilen auf Azure Virtual Desktop-Sitzungshost-VMs

1. Starten Sie in der Bastion-Sitzung für **az140-cl-vm11a** eine Remotedesktopsitzung für **az140-21-p1-0**, und melden Sie sich bei entsprechender Aufforderung mit dem Benutzernamen **ADATUM\wvdaadmin1** und dem Kennwort an, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben. 

   > **Hinweis:** Wenn keine RDP-Verbindung hergestellt werden kann, verwenden Sie das Azure-Portal, um mithilfe von Bastion eine Verbindung mit dem virtuellen Computer herzustellen.

1. Starten Sie in der Remotedesktopsitzung für **az140-21-p1-0** Microsoft Edge, navigieren Sie zur [Downloadseite für FSLogix](https://aka.ms/fslogix_download), laden Sie komprimierte FSLogix-Installationsbinärdateien herunter, und extrahieren Sie sie im Ordner **C:\\Quelle**. Navigieren Sie zum Unterordner **x64\\Release**, und installieren Sie mithilfe von **FSLogixAppsSetup.exe** die Microsoft FSLogix-Apps mit den Standardeinstellungen.

   > **Hinweis:** Abhängig davon, ob das Image FSLogix bereits enthält, ist die Installation von FSLogix unter Umständen nicht erforderlich. Nach der Installation von FSLogix ist ein Neustart erforderlich.

1. Starten Sie in der Remotedesktop-Sitzung für **az140-21-p1-0** die **Windows PowerShell ISE** als Administrator*in, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des PowerShellGet-Moduls zu installieren, und wählen Sie **Ja** aus, wenn Sie zur Bestätigung aufgefordert werden:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des Az PowerShell-Moduls zu installieren, und wählen Sie **Ja**, alle aus, wenn Sie zur Bestätigung aufgefordert werden:

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Ausführungsrichtlinie zu ändern:

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den Namen des Azure Storage-Kontos abzurufen, das Sie weiter oben in diesem Lab konfiguriert haben:

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. Führen Sie innerhalb der Remotedesktop-Sitzung für **az140-21-p1-0** über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um Profilregistrierungseinstellungen zu konfigurieren:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey -Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >**Hinweis:** Wenn der Befehl einen Fehler generiert, fahren Sie mit dem nächsten Schritt fort.
   
1. Klicken Sie innerhalb der Remotedesktopsitzung für **az140-21-p1-0** mit der rechten Maustaste auf **Start**. Wählen Sie im angezeigten Kontextmenü **Ausführen** aus. Geben Sie im Dialogfeld **Ausführen** in das Textfeld **Öffnen** Folgendes ein, und wählen Sie **OK** aus, um das Fenster **Lokale Benutzer und Gruppen** zu starten:

   ```cmd
   lusrmgr.msc
   ```

1. In der Konsole **Lokale Benutzer und Gruppen** sehen Sie die vier Gruppen, deren Name jeweils mit der Zeichenfolge **FSLogix** beginnt:

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. Doppelklicken Sie in der Konsole **Lokale Benutzer und Gruppen** auf den Eintrag für die Gruppe **FSLogix Profile Include List**. Beachten Sie, dass sie die Gruppe **\\Jeder** enthält. Wählen Sie **OK** aus, um das Fenster mit den **Eigenschaften** der Gruppe zu schließen. 
1. Doppelklicken Sie in der Konsole **Lokale Benutzer und Gruppen** auf den Eintrag für die Gruppe **FSLogix Profile Exclude List**. Beachten Sie, dass sie standardmäßig keine Gruppenmitglieder enthält. Wählen Sie **OK** aus, um das Fenster mit den **Eigenschaften** der Gruppe zu schließen. 

   > **Hinweis:** Um eine einheitliche Benutzeroberfläche bereitzustellen, müssen Sie FSLogix-Komponenten auf allen Azure Virtual Desktop-Sitzungshosts installieren und konfigurieren. Sie führen diese Aufgabe unbeaufsichtigt für die anderen Sitzungshosts in unserer Labumgebung aus. 

1. Führen Sie innerhalb der Remotedesktop-Sitzung für **az140-21-p1-0** über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um FSLogix-Komponenten auf dem Sitzungshost **az140-21-p1-1** zu installieren:

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. Starten Sie in der Remotedesktopsitzung für **az140-21-p1-0** die **Windows PowerShell ISE** als Administrator, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um Profilregistrierungseinstellungen auf dem Sitzungshost **az140-21-p1-1** zu installieren:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey -Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **Hinweis:** Vor dem Testen der FSLogix-basierten Profilfunktionen müssen Sie das lokal zwischengespeicherte Profil des Kontos „ADATUM\wvdaadmin1“ entfernen, das Sie zum Testen über die im vorherigen Lab verwendeten Azure Virtual Desktop-Sitzungshosts nutzen.

1. Wechseln Sie zur Bastion-Sitzung für **az140-25-vm11a**. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** zum Fenster **Administrator: Windows PowerShell ISE**, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das lokal zwischengespeicherte Profil des Kontos „ADATUM\aaduser1“ zu entfernen:

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Aufgabe 3: Testen von FSLogix-basierten Profilen mit Azure Virtual Desktop

1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-25-vm11a** zum Remotedesktopclient.
1. Doppelklicken Sie in der Bastion-Sitzung für **az140-cl-vm11a** im Fenster des **Remotedesktopclients** in der Liste der Anwendungen auf **Eingabeaufforderung**. Geben Sie bei entsprechender Aufforderung das Kennwort ein, und vergewissern Sie sich, dass ein **Eingabeaufforderungsfenster** geöffnet wird. 

   > **Hinweis:** Es kann anfangs einige Minuten dauern, bis die Anwendung gestartet wird. Danach sollte der Anwendungsstart aber viel schneller erfolgen.

1. Klicken Sie oben links im **Eingabeaufforderungsfenster** mit der rechten Maustaste auf das Symbol **Eingabeaufforderung**, und wählen Sie im Dropdownmenü die Option **Eigenschaften** aus.
1. Wählen Sie im Dialogfeld für die **Eigenschaften der Eingabeaufforderung** die Registerkarte **Schriftart** aus, ändern Sie die Einstellungen für Größe und Schriftart, und wählen Sie **OK** aus.
1. Geben Sie **logoff** im **Eingabeaufforderungsfenster** ein, und drücken Sie die **EINGABETASTE**, um sich bei der Remotedesktopsitzung abzumelden.
1. Doppelklicken Sie in der Bastion-Sitzung für **az140-cl-vm11a** im Clientfenster des **Remotedesktop** in der Liste der Anwendungen auf **SessionDesktop**, und vergewissern Sie sich, dass eine Remotedesktopsitzung gestartet wird. 
1. Klicken Sie innerhalb der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Starten**. Wählen Sie im angezeigten Kontextmenü **Ausführen** aus. Geben Sie **cmd** in das Textfeld **Öffnen** des Dialogfelds **Ausführen** ein, und wählen Sie **OK** aus, um ein **Eingabeaufforderungsfenster** zu starten:
1. Überprüfen Sie, ob die Eigenschaften des **Eingabeaufforderungsfensters** mit den weiter oben in dieser Aufgabe festgelegten Eigenschaften übereinstimmen.
1. Minimieren Sie in der Sitzung **SessionDesktop** alle Fenster, klicken Sie mit der rechten Maustaste auf den Desktop, und wählen Sie im Kontextmenü **Neu** und im Untermenü **Verknüpfung** aus. 
1. Geben Sie auf der Seite **Für welches Element soll eine Verknüpfung erstellt werden?** des Assistenten **Verknüpfung erstellen** in das Textfeld **Position des Elements eingeben** den Begriff **Editor** ein, und wählen Sie **Weiter** aus.
1. Geben Sie auf der Seite **Welcher Name soll für die Verknüpfung verwendet werden?** des Assistenten **Verknüpfung erstellen** in das Textfeld **Namen für diese Verknüpfung eingeben** den Namen **Editor** ein, und wählen Sie **Fertig stellen** aus.
1. Klicken Sie in der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Starten**, wählen Sie im Kontextmenü die Option **Herunterfahren oder abmelden** aus, und wählen Sie dann im Untermenü die Option **Abmelden** aus.
1. Navigieren Sie zurück zur Bastion-Sitzung für **az140-cl-vm11a**, und klicken Sie im Fenster des **Remotedesktop**-Clients in der Liste der Anwendungen doppelt auf **SessionDesktop**, um eine neue Remotedesktopsitzung zu starten. 
1. Überprüfen Sie in der Sitzung **SessionDesktop**, ob die Verknüpfung **Editor** auf dem Desktop angezeigt wird.
1. Klicken Sie in der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Starten**, wählen Sie im Kontextmenü die Option **Herunterfahren oder abmelden** aus, und wählen Sie dann im Untermenü die Option **Abmelden** aus.
1. Navigieren Sie zur Bastion-Sitzung für **az140-dc-vm11a** und dann zum Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist.
1. Navigieren Sie im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Speicherkonten**, und wählen Sie den Eintrag aus, der das in der vorherigen Übung erstellte Speicherkonto darstellt.
1. Wählen Sie auf dem Blatt für das Speicherkonto im Abschnitt **Dateidienste** die Option **Dateifreigaben** und anschließend in der Liste der Dateifreigaben den Eintrag **az140-22a-profiles** aus. 
1. Wählen Sie auf dem Blatt **az140-22a-profiles** **Durchsuchen** aus und vergewissern Sie sich, dass darin ein Ordner enthalten ist, dessen Name aus der Sicherheits-ID (SID) des Kontos **ADATUM\\aaduser1** gefolgt vom Suffix **_aaduser1** besteht.
1. Wählen Sie den im vorherigen Schritt ermittelten Ordner aus. Er enthält eine einzelne Datei mit dem Namen **Profile_aaduser1.vhd**.

### Übung 2: Löschen von Azure-Labressourcen (Optional)

1. Entfernen Sie die Microsoft Entra DS-Bereitstellung gemäß den Anweisungen unter [Löschen einer von Azure Active Directory Domain Services verwalteten Domäne mithilfe des Azure-Portals]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds).
1. Entfernen Sie alle Azure-Ressourcengruppen, die Sie in den Microsoft Entra DS-Labs dieses Kurses bereitgestellt haben, indem Sie die Anweisungen unter [Löschen von Ressourcengruppen und Ressourcen mit Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal) befolgen.
