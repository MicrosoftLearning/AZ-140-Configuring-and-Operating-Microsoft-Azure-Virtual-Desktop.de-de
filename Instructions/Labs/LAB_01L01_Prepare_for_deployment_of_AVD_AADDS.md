---
lab:
  title: "Lab: Vorbereiten der Bereitstellung von Azure Virtual Desktop (Microsoft Entra\_DS)"
  module: 'Module 1: Plan an AVD Architecture'
---

# Lab: Vorbereiten der Bereitstellung von Azure Virtual Desktop (Microsoft Entra DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder ein Microsoft Entra-Konto mit der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten, der dem Azure-Abonnement zugeordnet ist, und mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement

## Geschätzte Dauer

150 Minuten

>**Hinweis:** Die Bereitstellung einer Microsoft Entra DS-Instanz beinhaltet etwa 90 Minuten Wartezeit.

## Labszenario

Sie müssen die Bereitstellung von Azure Virtual Desktop in einer Microsoft Entra DS-Umgebung (Microsoft Entra Domain Services) vorbereiten.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Implementieren einer Microsoft Entra DS-Domäne
- Konfigurieren der Microsoft Entra DS-Domänenumgebung

## Labdateien

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## Anweisungen

### Übung 0: Erhöhen der Anzahl von vCPU-Kontingenten

Die Hauptaufgaben für diese Übung sind Folgende:

1. Identifizieren der aktuellen vCPU-Nutzung
1. Anfordern einer Erhöhung des vCPU-Kontingents

#### Aufgabe 1: Identifizieren der aktuellen vCPU-Nutzung

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und wählen Sie dann **Speicher erstellen** aus. 

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um den Ressourcenanbieter **Microsoft.Compute** zu registrieren, falls er nicht registriert sein sollte:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um den Registrierungsstatus des Ressourcenanbieters **Microsoft.Compute** zu überprüfen:

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Hinweis:** Vergewissern Sie sich, dass der Status **Registriert** lautet. Warten Sie andernfalls einige Minuten, und wiederholen Sie diesen Schritt.

1. Führen Sie im Azure-Portal über die PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um eine PowerShell-Variable mit dem Namen einer Azure-Region zu erstellen. Ersetzen Sie dabei den Platzhalter `<Azure_region>` durch den Namen der Azure-Region, die Sie für dieses Lab verwenden möchten (beispielsweise `eastus`):

   ```powershell
   $location = '<Azure_region>'
   ```

   > **Hinweis:** Führen Sie im Bereich **Cloud Shell** in der PowerShell-Eingabeaufforderung `(Get-AzLocation).Location` aus, um die Namen der Azure-Regionen zu ermitteln.
   
1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um die aktuelle Nutzung von vCPUs und die entsprechenden Grenzwerte für die Azure-VMs vom Typ **StandardDSv3Family** zu ermitteln:

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. Überprüfen Sie die Ausgabe des im letzten Schritt ausgeführten Befehls und vergewissern Sie sich, dass mindestens **20** vCPUs bei den Azure-VMs der **Standard DSv3-Familie** in der Azure-Zielregion verfügbar sind. Wenn das bereits der Fall ist, können Sie direkt mit der nächsten Übung fortfahren. Machen Sie andernfalls mit der nächsten Aufgabe dieser Übung weiter. 

#### Aufgabe 2: Anfordern einer Erhöhung des vCPU-Kontingents

1. Suchen Sie im Azure-Portal nach **Abonnements**, und wählen Sie auf dem Blatt **Abonnements** den Eintrag für das Azure-Abonnement aus, das Sie für dieses Lab verwenden möchten.
1. Wählen Sie im Azure-Portal auf dem Abonnementblatt im vertikalen Menü auf der linken Seite im Abschnitt **Einstellungen** die Option **Nutzung + Kontingente** aus. 
1. Wählen Sie auf dem Blatt  **Azure Pass – Förderung | Nutzung + Kontingente** in der oberen Suchleiste die folgenden Dropdownpfeile aus:

   |**Einstellung**|**Wert**|
   |---|---|
   |**Suche**|**Standard DSv3**|
   |**Alle Standorte**|**Auswahl aufheben** und dann *Ihren Standort* aktivieren|
   |**Ressourcenanbieter** | **Microsoft.Compute** |
   
1. Wählen Sie im zurückgegebenen Element **vCPUs der Standard DSv3-Familie** das Bleistiftsymbol (**Bearbeiten**) aus.
1. Geben Sie auf dem Blatt **Kontingentdetails** im Textfeld **Neuer Grenzwert** den Wert **30** ein, und wählen Sie anschließend **Speichern und weiter** aus.
1. Warten Sie, bis die Kontingentanforderung abgeschlossen wurde.  Kurz darauf sehen Sie auf dem Blatt **Kontingentdetails**, dass die Anforderung genehmigt und das Kontingent erhöht wurde. Schließen Sie das Blatt **Kontingentdetails**.

    >**Hinweis:** Je nach gewählter Azure-Region und aktuellem Bedarf muss ggf. eine Supportanfrage erstellt werden. Eine Anleitung zum Erstellen einer Supportanfrage finden Sie unter [Erstellen einer Azure-Supportanfrage](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Übung 1: Implementieren einer Azure AD DS-Domäne (Active Directory Domain Services)

Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen und Konfigurieren eines Microsoft Entra-Benutzerkontos für die Verwaltung der Microsoft Entra DS-Domain
1. Bereitstellen einer Microsoft Entra DS-Instanz über das Azure-Portal
1. Konfigurieren der Netzwerk- und Identitätseinstellungen der Microsoft Entra DS-Bereitstellung

#### Aufgabe 1: Erstellen und Konfigurieren eines Microsoft Entra-Benutzerkontos für die Verwaltung der Microsoft Entra DS-Domain

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ und in dem Microsoft Entra-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globale*r Administrator*in“ verfügt.
1. Navigieren Sie in dem Webbrowser, in dem das Azure-Portal geöffnet ist, zum Blatt **Übersicht** des Microsoft Entra-Mandanten, und wählen Sie im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** **Eigenschaften** aus.
1. Wählen Sie auf dem Blatt **Eigenschaften** Ihres Microsoft Entra-Mandanten am unteren Rand des Blatts den Link **Sicherheitsstandards verwalten** aus.
1. Wählen Sie auf dem Blatt **Sicherheitsstandards aktivieren** bei Bedarf **Nein** aus, aktivieren Sie das Kontrollkästchen **Meine Organisation verwendet den bedingten Zugriff.**, und wählen Sie anschließend **Speichern** aus.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und wählen Sie dann **Speicher erstellen** aus. 

1. Führen Sie im Cloud Shell-Bereich Folgendes aus, um sich bei Ihrem Microsoft Entra-Mandanten anzumelden:

   ```powershell
   Connect-AzureAD
   ```

1. Führen Sie im Cloud Shell-Bereich Folgendes aus, um den primären DNS-Domänennamen des Microsoft Entra-Mandanten abzurufen, der Ihrem Azure-Abonnement zugeordnet ist:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. Führen Sie im Cloud Shell-Bereich Folgendes aus, um Microsoft Entra-Benutzer*innen zu erstellen, denen erhöhte Berechtigungen gewährt werden, und ersetzen Sie dabei den Platzhalter `<password>` durch ein beliebiges komplexes Kennwort:

   > **Hinweis:** Merken Sie sich das verwendete Kennwort. Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Führen Sie im Cloud Shell-Bereich Folgendes aus, um dem bzw. der ersten der neu erstellten Microsoft Entra-Benutzer*innen die Rolle „Globale*r Administrator*in“ zuzuweisen:

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **Hinweis:** Im PowerShell-Modul für Azure AD wird die Rolle „Globaler Administrator“ als „Unternehmensadministrator“ bezeichnet.

1. Führen Sie im Cloud Shell-Bereich Folgendes aus, um den Benutzerprinzipalnamen des bzw. der neu erstellten Microsoft Entra-Benutzer*in zu ermitteln:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **Hinweis:** Erfassen Sie den Benutzerprinzipalnamen. Er wird später in dieser Übung benötigt. 

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Azure-Portal nach **Abonnements**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Abonnements** das für dieses Lab verwendete Azure-Abonnement aus. 
1. Wählen Sie auf dem Blatt mit den Eigenschaften Ihres Azure-Abonnements die Option **Zugriffssteuerung (IAM)** aus, wählen Sie **Hinzufügen** aus, und wählen Sie dann **Rollenzuweisung hinzufügen** aus. 
1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** die Option **Besitzer** aus, und klicken Sie anschließend auf **Weiter**.
1. Klicken Sie auf den Link **+ Mitglieder auswählen**.
1. Wählen Sie auf dem Blatt **Mitglieder auswählen** das Element **aadadmin1**, dann die Schaltfläche **Auswählen** und anschließend **Weiter** aus.
1. Wählen Sie auf dem Blatt **Überprüfen + zuweisen** die Schaltfläche **Überprüfen + zuweisen** aus.

   > **Hinweis:** Das Konto **aadadmin1** wird später im Lab verwendet, um Ihr Azure-Abonnement und den entsprechenden Microsoft Entra-Mandanten über eine virtuelle, in Microsoft Entra DS eingebundene Azure-VM mit Windows 10 zu verwalten. 


#### Aufgabe 2: Bereitstellen einer Microsoft Entra DS-Instanz über das Azure-Portal

1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Microsoft Entra Domain Services**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Microsoft Entra Domain Services** die Option **+ Erstellen** aus. Daraufhin wird das Blatt **Microsoft Entra Domain Services-Instanz erstellen** geöffnet.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** die folgenden Einstellungen an, und wählen Sie anschließend **Weiter** aus. Behalten Sie für alle anderen Einstellungen die bereits vorhandenen Werte bei:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|Wählen Sie „Neu erstellen“ aus. (**az140-11a-RG**)|
   |DNS-Domänenname|**adatum.com**|
   |Region|Der Name der Region, in der Ihre AVD-Bereitstellung gehostet werden soll.|
   |SKU|**Standard**|

   > **Hinweis:** Es empfiehlt sich im Allgemeinen, einen Microsoft Entra DS-Domänennamen zuzuweisen, der sich von allen ggf. vorhandenen Azure-DNS-Namespaces bzw. lokalen DNS-Namespaces unterscheidet, auch wenn dies aus technischer Sicht nicht unbedingt erforderlich ist.

1. Wählen Sie auf der Registerkarte **Netzwerk** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** neben der Dropdownliste **Virtuelles Netzwerk** die Option **Neu erstellen** aus.
1. Weisen Sie auf dem Blatt **Virtuelles Netzwerk erstellen** die folgenden Einstellungen zu, und wählen Sie anschließend **OK** aus:

   |Einstellung|Wert|
   |---|---|
   |Name|**az140-aadds-vnet11a**|
   |Adressbereich|**10.10.0.0/16**|
   |Subnetzname|**aadds-Subnet**|
   |Adressbereich|**10.10.0.0/24**|

1. Wählen Sie **Weiter** aus, wenn Sie sich wieder auf der Registerkarte **Netzwerk erstellen** des Blatts **Virtuelles Netzwerk erstellen** befinden. (Behalten Sie für alle anderen Einstellungen die bereits vorhandenen Werte bei.)
1. Akzeptieren Sie auf der Registerkarte **Verwaltung** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** die Standardeinstellungen und wählen Sie **Weiter** aus.
1. Stellen Sie auf der Registerkarte **Synchronisierung** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** sicher, dass **Alle** ausgewählt ist, und wählen Sie **Weiter** aus.
1. Akzeptieren Sie auf der Registerkarte **Sicherheitseinstellungen** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Akzeptieren Sie auf der Registerkarte **Tags** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** die Standardeinstellungen und wählen Sie „Weiter“ aus.
2. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Microsoft Entra Domain Services-Instanz erstellen** die Option **Erstellen** aus. 
3. Lesen Sie die Benachrichtigung zu Einstellungen, die nach der Erstellung der Microsoft Entra DS-Domäne nicht mehr geändert werden können, und wählen Sie **OK** aus.

   >**Hinweis:** Zu den Einstellungen, die nach der Bereitstellung einer Microsoft Entra DS-Domäne nicht mehr geändert werden können, zählen der DNS-Name, das Azure-Abonnement, die Ressourcengruppe, das virtuelle Netzwerk und das Subnetz, in dem die Domänencontroller gehostet werden, sowie die Art der Gesamtstruktur.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Das kann etwa 90 Minuten dauern. 

#### Aufgabe 3: Konfigurieren der Netzwerk- und Identitätseinstellungen der Microsoft Entra DS-Bereitstellung

1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Microsoft Entra Domain Services**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Microsoft Entra Domain Services** den Eintrag **adatum.com** aus, um zu der neu bereitgestellten Microsoft Entra DS-Instanz zu navigieren. 
1. Klicken Sie auf dem Blatt **adatum.com** der Microsoft Entra DS-Instanz auf die Warnung, die mit **Es wurden Konfigurationsprobleme für Ihre verwaltete Domäne erkannt.** beginnt. 
1. Wählen Sie auf dem Blatt**adatum.com | Konfigurationsdiagnose** **Ausführen** aus.
1. Erweitern Sie im Abschnitt **Überprüfung** den Bereich **DNS-Datensätze**, und klicken Sie auf **Reparieren**.
1. Klicken Sie auf dem Blatt **DNS-Datensätze** erneut auf **Reparieren**.
1. Kehren Sie zum Blatt **adatum.com** der Microsoft Entra DS-Instanz zurück, und lesen Sie im Abschnitt **Erforderliche Konfigurationsschritte** die Informationen zur Microsoft Entra DS-Kennworthashsynchronisierung. 

   > **Hinweis:** Alle vorhandenen reinen Cloudbenutzer*innen, die Zugriff auf Microsoft Entra DS-Domänencomputer und deren Ressourcen benötigen, müssen ihre Kennwörter entweder ändern oder zurücksetzen lassen. Das gilt auch für das Konto **aadadmin1**, das Sie zuvor in diesem Lab erstellt haben.

1. Öffnen Sie auf Ihrem Labcomputer im Azure-Portal eine Sitzung vom Typ **PowerShell** im Bereich **Cloud Shell**.
1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um das Attribut „objectID“ des Microsoft Entra-Benutzerkontos **aadadmin1** zu ermitteln:

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um das Kennwort des Benutzerkontos **aadadmin1** zurückzusetzen, dessen Objekt-ID Sie im vorherigen Schritt ermittelt haben. Ersetzen Sie dabei den Platzhalter `<password>` durch ein beliebiges komplexes Kennwort:

   > **Hinweis:** Merken Sie sich das verwendete Kennwort. Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt.

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **Hinweis:** In der Praxis wird der Wert von **-ForceChangePasswordNextLogin** in der Regel auf „$true“ festgelegt. Hier wird **$false** verwendet, um die Labschritte zu vereinfachen. 

1. Wiederholen Sie die beiden vorherigen Schritte, um das Kennwort für das Benutzerkonto **wvdaadmin1** zurückzusetzen.


### Übung 2: Konfigurieren der Microsoft Entra DS-Domänenumgebung
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage
1. Bereitstellen von Azure Bastion
1. Überprüfen der Standardkonfiguration der Microsoft Entra DS-Domäne
1. Erstellen von AD DS-Benutzer*innen und -Gruppen, die mit Microsoft Entra DS synchronisiert werden

#### Aufgabe 1: Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage

1. Führen Sie auf Ihrem Labcomputer im Azure-Portal über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um dem virtuellen Netzwerk **az140-aadds-vnet11a**, das Sie in der vorherigen Aufgabe erstellt haben, ein Subnetz namens **cl-Subnet** hinzuzufügen:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Suchen Sie auf Ihrem Labcomputer nach der Parameterdatei **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json**, und öffnen Sie sie mit Visual Studio Code.

1.  Suchen Sie in Zeile 21 den Wert des Parameters „domainPassword“. Aktualisieren Sie das vorhandene Kennwort in der Parameterdatei, um das Kennwort zu verwenden, das Sie zuvor in diesem Lab für das Benutzerkonto **aadadmin1** festgelegt haben, und wählen Sie anschließend **Speichern** aus, um die Datei zu speichern.

1. Wählen Sie im Azure-Portal auf der Symbolleiste des Cloud Shell-Bereichs das Symbol **Dateien hochladen/herunterladen** aus, wählen Sie im Dropdownmenü die Option **Hochladen** aus, und laden Sie die Dateien **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** und **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** in das Cloud Shell-Basisverzeichnis hoch.
1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um eine Azure-VM mit Windows 10 bereitzustellen, der als Azure Virtual Desktop-Client fungieren soll, und ihn in die Microsoft Entra DS-Domäne einzubinden:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **Hinweis:** Die Bereitstellung dauert ungefähr zehn Minuten. Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. 


#### Aufgabe 2: Bereitstellen von Azure Bastion 

> **Hinweis:** Azure Bastion ermöglicht die Verbindungsherstellung mit den virtuellen Azure-Computern ohne öffentliche Endpunkte, die Sie in der vorherigen Aufgabe dieser Übung bereitgestellt haben, und bietet gleichzeitig Schutz vor Brute-Force-Angriffen, die auf Anmeldeinformationen auf Betriebssystemebene abzielen.

> **Hinweis:** Stellen Sie sicher, dass bei Ihrem Browser die Popupfunktion aktiviert ist.

1. Öffnen Sie im Browserfenster mit dem Azure-Portal einen weiteren Tab, und navigieren Sie zum [**Azure-Portal**](https://portal.azure.com).
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um dem virtuellen Netzwerk **az140-adds-vnet11**, das Sie weiter oben in dieser Übung erstellt haben, ein Subnetz namens **AzureBastionSubnet** hinzuzufügen:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Azure-Portal nach **Bastions**, wählen Sie diese Option aus, und wählen Sie auf dem Blatt **Bastions** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Bastion erstellen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-11a-RG**|
   |Name|**az140-11a-bastion**|
   |Region|Die gleiche Azure-Region, in der Sie die Ressourcen in der vorherigen Aufgabe dieser Übung bereitgestellt haben|
   |Tarif|**Grundlegend**|
   |Virtuelles Netzwerk|**az140-aadds-vnet11a**|
   |Subnetz|**AzureBastionSubnet (10.10.254.0/24)**|
   |Öffentliche IP-Adresse|**Neu erstellen**|
   |Name der öffentlichen IP-Adresse|**az140-aadds-vnet11a-ip**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Bastion erstellen** die Option **Erstellen** aus:

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe dieser Übung fortfahren. Die Bereitstellung kann ungefähr 5 Minuten dauern.


#### Aufgabe 3: Überprüfen der Standardkonfiguration der Microsoft Entra DS-Domäne

> **Hinweis:** Damit Sie sich bei dem neu in Microsoft Entra DS eingebundenen Computer anmelden können, müssen Sie zuerst das Benutzerkonto, mit dem Sie sich anmelden möchten, der Microsoft Entra-Gruppe **AAD DC Administrator*innen** hinzufügen. Diese Microsoft Entra-Gruppe wird automatisch in dem Microsoft Entra-Mandanten erstellt, der dem Azure-Abonnement zugeordnet ist, in dem Sie die Microsoft Entra DS-Instanz bereitgestellt haben.

> **Hinweis:** Diese Gruppe kann mit bereits vorhandenen Microsoft Entra-Benutzerkonten aufgefüllt werden, wenn Sie eine Microsoft Entra DS-Instanz bereitstellen.

1. Führen Sie auf Ihrem Labcomputer im Azure-Portal über den Cloud Shell-Bereich Folgendes aus, um das Microsoft Entra-Benutzerkonto **aadadmin1** der Microsoft Entra-Administratorgruppe **AAD DC Administrator*innen** hinzuzufügen:

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11a** aus. Daraufhin wird das Blatt **az140-cl-vm11a** geöffnet.
1. Wählen Sie auf dem Blatt **az140-cl-vm11a** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** aus, geben Sie auf der Registerkarte **Bastion** von **az140-cl-vm11a** die folgenden Anmeldeinformationen an, und wählen Sie anschließend **Verbinden** aus:
1. Melden Sie sich als Benutzer **aadadmin1** an, wenn Sie dazu aufgefordert werden. Verwenden Sie dabei den Prinzipalnamen, den Sie zuvor in dieser Übung ermittelt haben, und das Kennwort, das Sie weiter oben in diesem Lab beim Erstellen des Benutzerkontos festgelegt haben.
1. Starten Sie im Bastion für die Azure-VM **az140-cl-vm11a** die **Windows PowerShell ISE** als Administrator*in, und führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die Active Directory- und DNS-bezogenen Remoteserver-Verwaltungstools zu installieren:

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **Hinweis:** Warten Sie, bis die Installation abgeschlossen ist, bevor Sie mit dem nächsten Schritt fortfahren. Dies kann etwa zwei Minuten dauern.

1. Navigieren Sie im Bastion für die Azure-VM **az140-cl-vm11a** über das Menü **Start** zum Ordner **Windows-Verwaltungsprogramme**, erweitern Sie ihn, und starten Sie aus der Toolliste das Tool **Active Directory-Benutzer*innen und -Computer**. 
1. Überprüfen Sie in der Konsole **Active Directory-Benutzer und -Computer** die Standardhierarchie, einschließlich der Organisationseinheiten **AADDC-Computer** und **AADDC-Benutzer**. Wie Sie sehen, enthält die erstgenannte Einheit das Computerkonto **az140-cl-vm11a**. Die andere Einheit enthält die Benutzerkonten, die aus dem Microsoft Entra-Mandanten synchronisiert werden, der dem Azure-Abonnement zugeordnet ist, unter dem die Bereitstellung der Microsoft Entra DS-Instanz gehostet wird. Die Organisationseinheit **AADDC-Benutzer*innen** enthält auch die Gruppe **AAD DC Administrator*innen**, die aus dem gleichen Microsoft Entra-Mandanten synchronisiert wird (zusammen mit ihrer Gruppenmitgliedschaft). Diese Mitgliedschaft kann nicht direkt in der Microsoft Entra DS-Domäne geändert werden. Sie muss stattdessen im Microsoft Entra DS-Mandanten verwaltet werden. Alle Änderungen werden automatisch mit dem Replikat der Gruppe synchronisiert, das in der Microsoft Entra DS-Domäne gehostet wird. 

   **Hinweis:** Wenn unter **Active Directory-Benutzer und -Computer** keine domänenbezogenen Inhalte aufgeführt sind, klicken Sie mit der rechten Maustaste auf **Active Directory-Benutzer und -Computer**, wählen Sie **Domäne ändern** aus, und wählen Sie anschließend die Domäne **Adatum** aus.

   > **Hinweis:** Derzeit enthält die Gruppe nur das Benutzerkonto **aadadmin1**.

1. Wählen Sie in der Konsole **Active Directory-Benutzer*innen und -Computer** in der Organisationseinheit **AADDC-Benutzer*innen** das Benutzerkonto **aadadmin1** aus, zeigen Sie das zugehörige Dialogfeld **Eigenschaften** an, wechseln Sie zur Registerkarte **Konten**, und beachten Sie, dass das Suffix des Benutzerprinzipalnamens dem primären Microsoft Entra-DNS-Domänennamen entspricht und nicht geändert werden kann. 
1. Überprüfen Sie in der Konsole **Active Directory-Benutzer und -Computer** den Inhalt der Organisationseinheit **Domänencontroller**, und beachten Sie, dass sie Computerkonten von zwei Domänencontrollern mit zufällig generierten Namen enthält. 

#### Aufgabe 4: Erstellen von AD DS-Benutzer*innen und -Gruppen, die mit Microsoft Entra DS synchronisiert werden

1. Starten Sie im Bastion für die Azure-VM **az140-cl-vm11a** Microsoft Edge, navigieren Sie zum [Azure-Portal](https://portal.azure.com) und melden Sie sich an. Geben Sie dabei den Benutzernamen des Benutzerkontos **aadadmin1** mit dem Kennwort an, das Sie zuvor in diesem Lab als Kennwort festgelegt haben.
1. Öffnen Sie **Cloud Shell** im Azure-Portal.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis:** Da Sie **Cloud Shell** zum ersten Mal mit dem Benutzerkonto **aadadmin1** starten, müssen Sie das Cloud Shell-Basisverzeichnis konfigurieren. Wenn die Meldung **Für Sie wurde kein Speicher bereitgestellt.** angezeigt wird, wählen Sie das Abonnement aus, das Sie in diesem Lab verwenden, und wählen Sie dann **Speicher erstellen** aus. 

1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um sich anzumelden und bei Ihrem Microsoft Entra-Mandanten zu authentifizieren:

   ```powershell
   Connect-AzureAD
   ```

1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um den primären DNS-Domänennamen des Microsoft Entra-Mandanten abzurufen, der Ihrem Azure-Abonnement zugeordnet ist:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um die Microsoft Entra-Benutzerkonten für die nächsten Labs zu erstellen. Ersetzen Sie dabei den Platzhalter `<password>` durch ein beliebiges komplexes Kennwort:

   > **Hinweis:** Merken Sie sich das verwendete Kennwort. Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um eine Microsoft Entra-Gruppe mit dem Namen **az140-wvd-aadmins** zu erstellen und sie den Benutzerkonten **aadadmin1** und **wvdaadmin1** hinzuzufügen:

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. Wiederholen Sie im Cloud Shell-Bereich den vorherigen Schritt, um Microsoft Entra-Gruppen für Benutzer*innen zu erstellen, die in den nächsten Labs verwendet werden, und ihnen zuvor erstellte Microsoft Entra-Benutzerkonten hinzuzufügen:

   >**Hinweis:** Hinweis: Aufgrund der begrenzten Größe der Zwischenablage auf dem virtuellen Computer werden nicht alle aufgelisteten Cmdlets ordnungsgemäß kopiert. Öffnen Sie auf dem virtuellen Computer den Editor, und kopieren Sie alle Cmdlets mithilfe des Konstrukts „Type Text“ > „Type Clipboard Text“ des Lightning Bolt-Steuerelements. Wenn sich alle Cmdlets im Editor befinden, schneiden Sie sie aus, fügen Sie sie blockweise in Cloud Shell ein, und führen Sie sie aus.

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Bastion für die Azure-VM **az140-cl-vm11a** in dem Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach dem Blatt **Azure Active Directory**, und wählen Sie es aus. Wählen Sie auf dem Blatt Ihres Microsoft Entra-Mandanten auf der vertikalen Menüleiste auf der linken Seite im Abschnitt **Verwalten** die Option **Benutzer*innen** aus. Vergewissern Sie sich auf dem Blatt **Benutzer*innen \| Alle Benutzer*innen**, dass die neuen Benutzerkonten erstellt wurden.
1. Kehren Sie zum Blatt des Microsoft Entra-Mandanten zurück. Wählen Sie auf der vertikalen Menüleiste auf der linken Seite im Abschnitt **Verwalten** die Option **Gruppen** aus. Vergewissern Sie sich auf dem Blatt **Gruppen \| Alle Gruppen**, dass die neuen Gruppenkonten erstellt wurden.
1. Wechseln Sie im Bastion für die Azure-VM **az140-cl-vm11a** zur Konsole **Active Directory-Benutzer*innen und -Computer**. Navigieren Sie in der Konsole **Active Directory-Benutzer*innen und -Computer** zur Organisationseinheit **AADDC-Benutzer*innen**, und vergewissern Sie sich, dass sie die gleichen Benutzer- und Gruppenkonten enthält.

   >**Hinweis:** Unter Umständen muss die Ansicht der Konsole aktualisiert werden.
