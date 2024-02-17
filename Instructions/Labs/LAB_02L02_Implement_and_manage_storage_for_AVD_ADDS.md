---
lab:
  title: 'Lab: Implementieren und Verwalten von Speicher für virtuelle Android-Geräte (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Lab: Implementieren und Verwalten von Speicher für virtuelle Android-Geräte (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Microsoft Entra-Konto mit der Rolle „Besitzer*in“ oder „Mitwirkende*r“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globale*r Administrator*in“ im Microsoft Entra-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**

## Geschätzte Dauer

30 Minuten

## Labszenario

Sie müssen Speicher für eine Azure Virtual Desktop-Bereitstellung in einer Microsoft Entra DS-Umgebung implementieren und verwalten.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Konfigurieren von Azure Files für das Speichern von Profilcontainern für Azure Virtual Desktop

## Labdateien

- Keine

## Anweisungen

### Übung 1: Konfigurieren von Azure Files für das Speichern von Profilcontainern für Azure Virtual Desktop

Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen eines Azure-Speicherkontos
1. Erstellen einer Azure Files-Freigabe
1. Aktivieren der AD DS-Authentifizierung für das Azure Storage-Speicherkonto 
1. Konfigurieren der RBAC-basierten Azure Files-Berechtigungen
1. Konfigurieren der Azure Files-Dateisystemberechtigungen

#### Aufgabe 1: Erstellen eines Azure-Speicherkontos

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie auf dem Blatt **Virtuelle Computer** auf **az140-dc-vm11**.
1. Klicken Sie auf dem Blatt **az140-dc-vm11** auf **Verbinden**. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Suchen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Speicherkonten**, und wählen Sie die Option aus. Klicken Sie dann auf dem Blatt **Speicherkonten** auf **+ Erstellen**.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Speicherkonto erstellen** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|Der Name einer neuen Ressourcengruppe: **az140-22-RG**|
   |Speicherkontoname|Global gültiger Name zwischen 3 und 15 Zeichen, der aus Kleinbuchstaben und Ziffern besteht und mit einem Buchstaben beginnt|
   |Region|Name einer Azure-Region, in der die Azure Virtual Desktop-Labumgebung gehostet wird|
   |Leistung|**Standard**|
   |Redundanz|**Georedundanter Speicher (GRS)**|
   |Bei regionaler Nichtverfügbarkeit Lesezugriff auf die Daten bereitstellen|enabled|

   >**Hinweis:** Stellen Sie sicher, dass die Länge des Speicherkontonamens 15 Zeichen nicht überschreitet. Der Name wird zum Erstellen eines Computerkontos in der Active Directory Domain Services-Domäne (AD DS) verwendet, die mit dem Microsoft Entra-Mandanten integriert ist, der wiederum dem Azure-Abonnement mit dem Speicherkonto zugeordnet ist. Dadurch wird die AD DS-basierte Authentifizierung beim Zugriff auf Dateifreigaben möglich, die in diesem Speicherkonto gehostet werden.

1. Klicken Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Speicherkonto erstellen** auf **Überprüfen + erstellen**, warten Sie, bis der Überprüfungsprozess abgeschlossen ist, und klicken Sie dann auf **Erstellen**.

   >**Hinweis**: Warten Sie, bis das Speicherkonto erstellt wurde. Dieser Vorgang dauert etwa zwei Minuten.

#### Aufgabe 2: Erstellen einer Azure Files-Freigabe

1. Navigieren Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Speicherkonten**, und wählen Sie den Eintrag aus, der das neu erstellte Speicherkonto darstellt.
1. Klicken Sie auf dem Blatt für das Speicherkonto im Abschnitt **Datenspeicher** auf **Dateifreigaben** und anschließend auf **+ Dateifreigabe**.
1. Legen Sie auf dem Blatt **Neue Dateifreigabe** die folgenden Einstellungen fest, und klicken Sie auf **Erstellen** (Standardwerte für die anderen Einstellungen übernehmen):

   |Einstellung|Wert|
   |---|---|
   |Name|**az140-22-profiles**|
   |Ebenen|**Transaktion optimiert**|

#### Aufgabe 3: Aktivieren der AD DS-Authentifizierung für das Azure Storage-Speicherkonto 

1. Öffnen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** eine weitere Registerkarte im Microsoft Edge-Fenster, navigieren Sie zum [GitHub-Repository mit Azure Files-Beispielen](https://github.com/Azure-Samples/azure-files-samples/releases), laden Sie die aktuellste Version des komprimierten PowerShell-Moduls **AzFilesHybrid.zip** herunter, und extrahieren Sie den Inhalt in den Ordner **C:\\Allfiles\\Labs\\02** (bei Bedarf Ordner erstellen).
1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator*in, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den alternativen Datenstrom **Zone.Identifier** zu entfernen, der den Wert **3** hat, was bedeutet, dass er über das Internet heruntergeladen wurde:

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Variablen festzulegen, die zum Ausführen des nachfolgenden Skripts erforderlich sind:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um ein AD DS-Computerobjekt zu erstellen, das das Azure Storage-Konto darstellt, das Sie zuvor in dieser Aufgabe erstellt haben und das zum Implementieren der AD DS-Authentifizierung verwendet wird:

   >**Hinweis:** Wenn bei der Ausführung dieses Skriptblocks eine Fehlermeldung angezeigt wird, stellen Sie sicher, dass Sie sich in dem Verzeichnis befinden, in dem auch „CopyToPSPath.ps1“ enthalten ist. Je nachdem, wie die Dateien zuvor in diesem Lab extrahiert wurden, befinden sie sich möglicherweise in einem Unterordner namens „AzFilesHybrid“. Im PowerShell-Kontext können Sie mithilfe von **cd AzFilesHybrid** Verzeichnisse in den Ordner ändern.

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. Führen Sie in der Remotedesktopsitzung auf **az140-dc-vm11** aus dem **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um zu überprüfen, ob die AD DS-Authentifizierung für das Azure Storage-Konto aktiviert ist:

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. Überprüfen Sie, ob die Ausgabe des Befehls `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties` `AD` zurückgibt, womit der Verzeichnisdienst des Speicherkontos dargestellt wird, und ob die Ausgabe des Befehls `$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions`, die die Verzeichnisdomäneninformationen darstellt, das folgende Format aufweist (die Werte von `DomainGuid`, `DomainSid` und `AzureStorageSid` weichen ab):

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. Wechseln Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** zum Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird. Klicken Sie auf dem Blatt mit dem Speicherkonto auf **Dateifreigaben** und überprüfen Sie, ob die Einstellung **Identitätsbasierter Zugriff** den Status **Konfiguriert** aufweist.

   >**Hinweis:** Möglicherweise müssen Sie die Browserseite aktualisieren, damit die Änderungen im Azure-Portal angezeigt werden.

#### Aufgabe 4: Konfigurieren der RBAC-basierten Azure Files-Berechtigungen

1. Klicken Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, auf dem Blatt mit den Eigenschaften des zuvor in dieser Übung erstellten Speicherkontos im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** auf **Dateifreigaben**.
1. Wählen Sie auf dem Blatt **Dateifreigaben** in der Liste der Freigaben den Eintrag **az140-22-profiles** aus.
1. Klicken Sie auf dem Blatt **az140-22-profiles** im vertikalen Menü auf der linken Seite auf **Zugriffssteuerung (IAM)**.
1. Klicken Sie auf dem Blatt **Zugriffssteuerung (IAM)** für das Speicherkonto auf **+ Hinzufügen**, und wählen Sie im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus. 
1. Geben Sie auf dem Blatt **Rollenzuweisung hinzufügen** die folgenden Einstellungen an, und klicken Sie auf **Überprüfen + zuweisen**:

   |Einstellung|Wert|
   |---|---|
   |Rolle|**Speicherdateidaten-SMB-Freigabemitwirkender**|
   |Zugriff zuweisen zu|**Benutzer, Gruppe oder Dienstprinzipal**|
   |Auswählen|**az140-wvd-users**|

1. Klicken Sie auf dem Blatt **Zugriffssteuerung (IAM)** für das Speicherkonto auf **+ Hinzufügen**, und wählen Sie im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus. 
1. Geben Sie auf dem Blatt **Rollenzuweisung hinzufügen** die folgenden Einstellungen an, und klicken Sie auf **Überprüfen + zuweisen**:

   |Einstellung|Wert|
   |---|---|
   |Rolle|**Speicherdateidaten-SMB-Freigabemitwirkender mit erhöhten Rechten**|
   |Zugriff zuweisen zu|**Benutzer, Gruppe oder Dienstprinzipal**|
   |Auswählen|**az140-wvd-admins**|

#### Aufgabe 5: Konfigurieren der Azure Files-Dateisystemberechtigungen

1. Wechseln Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** zum Fenster **Administrator: Windows PowerShell ISE**, und führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Variable zu erstellen, die auf den Namen und Schlüssel des Speicherkontos verweist, das Sie zuvor in dieser Übung erstellt haben:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Laufwerkszuordnung für die Dateifreigabe zu erstellen, die Sie zuvor in dieser Übung erstellt haben:

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die aktuellen Dateisystemberechtigungen anzuzeigen:

   ```powershell
   icacls Z:
   ```

   >**Hinweis:** Standardmäßig verfügen sowohl **NT-Autorität\\Authentifizierte Benutzer** als auch **BUILTIN-Benutzer\\** über Berechtigungen, mit denen Benutzer*innen die Profilcontainer anderer Benutzer*innen lesen können. Sie entfernen sie und fügen stattdessen die mindestens erforderlichen Berechtigungen hinzu.

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Dateisystemberechtigungen anzupassen, um das Prinzip der geringsten Berechtigungen einzuhalten:

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**Hinweis:** Alternativ können Sie Berechtigungen mithilfe des Datei-Explorers festlegen.
