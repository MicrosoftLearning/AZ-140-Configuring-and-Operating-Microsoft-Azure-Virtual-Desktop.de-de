---
lab:
  title: "Lab: Implementieren und Verwalten von Azure\_Virtual\_Desktop-Profilen (AD\_DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# Lab: Implementieren und Verwalten von Azure Virtual Desktop-Profilen (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Microsoft Entra-Konto mit der Rolle „Besitzer*in“ oder „Mitwirkende*r“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globale*r Administrator*in“ im Microsoft Entra-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Implementieren und Verwalten von Speicher für WVD (AD DS)**

## Geschätzte Dauer

30 Minuten

## Labszenario

Sie müssen die Azure Virtual Desktop-Profilverwaltung mit einer AD DS-Umgebung (Active Directory Domain Services) implementieren.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- FSLogix-basierte Profile für Azure Virtual Desktop zu implementieren.

## Labdateien

- Keine

## Anweisungen

### Übung 1: FSLogix-basierte Profile für Azure Virtual Desktop zu implementieren.

Die Hauptaufgaben für diese Übung sind Folgende:

1. Konfigurieren von FSLogix-basierten Profilen auf Azure Virtual Desktop-Sitzungshost-VMs
1. Testen von FSLogix-basierten Profilen mit Azure Virtual Desktop
1. Entfernen von im Lab bereitgestellten Azure-Ressourcen

#### Aufgabe 1: Konfigurieren von FSLogix-basierten Profilen auf Azure Virtual Desktop-Sitzungshost-VMs

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie dann auf dem Blatt **Virtuelle Computer** die Option **az140-21-p1-0** aus.
1. Wählen Sie auf dem Blatt **az140-21-p1-0** die Option **Starten** aus, und warten Sie, bis sich der Status des virtuellen Computers auf **Wird ausgeführt** ändert.
1. Wählen Sie auf dem Blatt **az140-21-p1-0** die Option **Verbinden** aus. Wählen Sie in der Dropdownliste die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-21-p1-0 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den folgenden Anmeldeinformationen an:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Öffnen Sie in der Bastion-Sitzung für **az140-21-p1-0** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, eine PowerShell-Sitzung mit dem Cloud Shell-Bereich. 
1. Führen Sie in der PowerShell-Sitzung im Bereich **Cloud Shell** Folgendes aus, um die Azure-VMs mit den Azure Virtual Desktop-Sitzungshosts zu starten, die Sie in diesem Lab verwenden:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Hinweis:** Warten Sie, bis die Azure-VMs ausgeführt werden, bevor Sie mit dem nächsten Schritt fortfahren.

1. Führen Sie in der PowerShell-Sitzung auf der **Cloud Shell**-Seite Folgendes aus, um PowerShell-Remoting auf den Sitzungshosts zu aktivieren.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
1. Schließen der Cloud Shell
1. Starten Sie in der Bastion-Sitzung für **az140-21-p1-0** Microsoft Edge, navigieren Sie zur [Downloadseite für FSLogix](https://aka.ms/fslogix_download), und laden Sie die komprimierten FSLogix-Installationsbinärdateien herunter. Extrahieren Sie sie in den Ordner **C:\\Allfiles\\Labs\\04** (erstellen Sie den Ordner bei Bedarf), navigieren Sie zum Unterordner **x64\\Release**, und klicken Sie doppelt auf die Datei **FSLogixAppsSetup.exe**, um den Setup-Assistenten für **Microsoft FSLogix-Apps** zu starten. Führen Sie die Installation von Microsoft FSLogix-Apps mit den Standardeinstellungen durch.

   > **Hinweis:** Die Installation von FSLogix ist nicht erforderlich, wenn der Dienst bereits im Image enthalten ist.

1. Starten Sie in der Bastion-Sitzung für **az140-21-p1-0** die **Windows PowerShell ISE** als Administrator*in, und führen Sie über den Skriptbereich **Administrator*in: Windows PowerShell ISE** Folgendes aus, um die neueste Version des PowerShellGet-Moduls zu installieren, und wählen Sie **Ja** aus, wenn Sie zur Bestätigung aufgefordert werden:

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
1. Führen Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** über die Konsole **Administrator*in: Windows PowerShell ISE** Folgendes aus, um den Namen des Azure Storage-Kontos abzurufen, das Sie zuvor in diesem Lab konfiguriert haben:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** über die Konsole **Administrator*in: Windows PowerShell ISE** den folgenden Befehl aus, um Profilregistrierungseinstellungen zu konfigurieren:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. Klicken Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** mit der rechten Maustaste auf **Starten**. Wählen Sie im angezeigten Kontextmenü die Option **Ausführen** aus. Geben Sie in das Textfeld **Öffnen** des Dialogfelds **Ausführen** Folgendes ein, und wählen Sie **OK** aus, um die Konsole **Lokale Benutzer*innen und Gruppen** zu starten:

   ```cmd
   lusrmgr.msc
   ```

1. In der Konsole **Lokale Benutzer und Gruppen** sehen Sie die vier Gruppen, deren Name jeweils mit der Zeichenfolge **FSLogix** beginnt:

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. Klicken Sie in der Konsole **Lokale Benutzer und Gruppen** in der Liste der Gruppen doppelt auf die Gruppe **FSLogix Profile Include List**. Beachten Sie, dass sie die Gruppe **\\Jeder** enthält. Wählen Sie **OK** aus, um das Fenster mit den **Eigenschaften** der Gruppe zu schließen. 
1. Klicken Sie in der Konsole **Lokale Benutzer und Gruppen** in der Liste der Gruppen doppelt auf die Gruppe **FSLogix Profile Exclude List**. Beachten Sie, dass sie standardmäßig keine Gruppenmitglieder enthält. Wählen Sie **OK** aus, um das Fenster mit den **Eigenschaften** der Gruppe zu schließen. 

   > **Hinweis:** Um eine einheitliche Benutzeroberfläche bereitzustellen, müssen Sie FSLogix-Komponenten auf allen Azure Virtual Desktop-Sitzungshosts installieren und konfigurieren. Sie führen diese Aufgabe unbeaufsichtigt für die anderen Sitzungshosts in unserer Labumgebung aus. 

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** über die Konsole **Administrator*in: Windows PowerShell ISE** den folgenden Befehl aus, um FSLogix-Komponenten auf den Sitzungshosts **az140-21-p1-1** und **az140-21-p1-2** zu installieren:

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **Hinweis:** Warten Sie, bis die Skriptausführung abgeschlossen ist. Dies kann etwa zwei Minuten dauern.

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** über die Konsole **Administrator*in: Windows PowerShell ISE** den folgenden Befehl aus, um Profilregistrierungseinstellungen auf den Sitzungshosts **az140-21-p1-1** und **az140-21-p1-1** zu konfigurieren:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **Hinweis:** Vor dem Testen der FSLogix-basierten Profilfunktionen müssen Sie das lokal zwischengespeicherte Profil des Kontos **ADATUM\\aduser1** entfernen, das Sie zum Testen über die im vorherigen Lab verwendeten Azure Virtual Desktop-Sitzungshosts nutzen.

1. Führen Sie innerhalb der Bastion-Sitzung für **az140-21-p1-0** über die Konsole **Administrator*in: Windows PowerShell ISE**, und führen Sie Folgendes aus, um das lokal zwischengespeicherte Profil des Kontos **ADATUM\\aduser1** von allen virtuellen Azure-Computern zu entfernen, die als Sitzungshosts fungieren:

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Aufgabe 2: Testen von FSLogix-basierten Profilen mit Azure Virtual Desktop

1. Wechseln Sie zu Ihrem Labcomputer, suchen Sie im Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-cl-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Wählen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11** **Starten** und im Menü **Starten** **Remotedesktop** aus, um den Remotedesktop-Client zu starten.
1. Wählen Sie in der Bastion-Sitzung für **az140-cl-vm11** im Fenster des **Remotedesktop**-Clients die Option **Abonnieren** aus. Melden Sie sich mit den Anmeldeinformationen von **aduser1** an, wenn Sie dazu aufgefordert werden.

   >**Hinweis**: Wenn Sie nicht dazu aufgefordert werden, ein Abonnement abzuschließen, müssen Sie möglicherweise ein bereits bestehendes Abonnement kündigen.
3. klicken Sie in der Liste der Anwendungen doppelt auf **Eingabeaufforderung**. Wenn Sie dazu aufgefordert werden, geben Sie das Kennwort des Kontos **aduser1** ein, und überprüfen Sie, ob ein **Eingabeaufforderungsfenster** geöffnet wird.
4. Klicken Sie oben links im **Eingabeaufforderungsfenster** mit der rechten Maustaste auf das Symbol **Eingabeaufforderung**, und wählen Sie im Dropdownmenü die Option **Eigenschaften** aus.
5. Wählen Sie im Dialogfeld für die **Eigenschaften der Eingabeaufforderung** die Registerkarte **Schriftart** aus, ändern Sie die Einstellungen für Größe und Schriftart, und wählen Sie **OK** aus.
6. Geben Sie **logoff** im **Eingabeaufforderungsfenster** ein, und drücken Sie die **EINGABETASTE**, um sich bei der Remotedesktopsitzung abzumelden.
7. Klicken Sie in der Bastion-Sitzung für **az140-cl-vm11** im Fenster des **Remotedesktop**-Clients in der Liste der Anwendungen unter „az140-21-ws1“ doppelt auf **SessionDesktop**, und vergewissern Sie sich, dass eine Remotedesktopsitzung gestartet wird. 
8. Klicken Sie innerhalb der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Starten**. Wählen Sie im angezeigten Kontextmenü **Ausführen** aus. Geben Sie **cmd** in das Textfeld **Öffnen** des Dialogfelds **Ausführen** ein, und wählen Sie **OK** aus, um ein **Eingabeaufforderungsfenster** zu starten:
9. Vergewissern Sie sich, dass die Einstellungen für ein **Eingabeaufforderungsfenster** mit denen übereinstimmen, die Sie zuvor in dieser Aufgabe konfiguriert haben.
10. Minimieren Sie in der Sitzung **SessionDesktop** alle Fenster, klicken Sie mit der rechten Maustaste auf den Desktop, und wählen Sie im Kontextmenü **Neu** und im Untermenü **Verknüpfung** aus. 
11. Geben Sie auf der Seite **Für welches Element soll eine Verknüpfung erstellt werden?** des Assistenten **Verknüpfung erstellen** in das Textfeld **Position des Elements eingeben** den Begriff **Editor** ein, und wählen Sie **Weiter** aus.
12. Geben Sie auf der Seite **Welcher Name soll für die Verknüpfung verwendet werden?** des Assistenten **Verknüpfung erstellen** in das Textfeld **Namen für diese Verknüpfung eingeben** den Namen **Editor** ein, und wählen Sie **Fertig stellen** aus.
13. Klicken Sie in der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Starten**, wählen Sie im Kontextmenü die Option **Herunterfahren oder abmelden** aus, und wählen Sie dann im Untermenü die Option **Abmelden** aus.
14. Navigieren Sie zurück zur Bastion-Sitzung für **az140-cl-vm11**, und klicken Sie im Fenster des **Remotedesktop**-Clients in der Liste der Anwendungen doppelt auf **SessionDesktop**, um eine neue Remotedesktopsitzung zu starten. 
15. Überprüfen Sie in der Sitzung **SessionDesktop**, ob die Verknüpfung **Editor** auf dem Desktop angezeigt wird.
16. Klicken Sie in der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Starten**, wählen Sie im Kontextmenü die Option **Herunterfahren oder abmelden** aus, und wählen Sie dann im Untermenü die Option **Abmelden** aus.
17. Wechseln Sie zu Ihrem Labcomputer. Navigieren Sie im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Speicherkonten**, und wählen Sie den Eintrag aus, der das in der vorherigen Übung erstellte Speicherkonto darstellt.
18. Wählen Sie auf dem Blatt für das Speicherkonto im Abschnitt **Dateidienste** die Option **Dateifreigaben** und anschließend in der Liste der Dateifreigaben den Eintrag **az140-22-profiles** aus. 
19. Wählen Sie auf dem Blatt **az140-22-profiles** **Durchsuchen** aus und vergewissern Sie sich, dass darin ein Ordner enthalten ist, dessen Name aus der Sicherheits-ID (SID) des Kontos **ADATUM\\aduser1** gefolgt vom Suffix **_aduser1** besteht.
20. Wählen Sie den im vorherigen Schritt ermittelten Ordner aus. Er enthält eine einzelne Datei mit dem Namen **Profile_aduser1.vhd**.

### Übung 2: Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab bereitgestellten und verwendeten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten und verwendeten Azure-VMs

1. Wechseln Sie zum Labcomputer, und öffnen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten und verwendeten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten und verwendeten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet werden und ihre Zuordnung aufgehoben wird.
