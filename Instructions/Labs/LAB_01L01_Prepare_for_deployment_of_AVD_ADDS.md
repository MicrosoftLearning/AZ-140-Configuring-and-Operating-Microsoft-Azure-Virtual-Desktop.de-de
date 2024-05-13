---
lab:
  title: "Lab: Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD\_DS)"
  module: 'Module 1: Plan an AVD Architecture'
---

# Lab: Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Microsoft Entra-Konto mit der Rolle „Besitzer*in“ oder „Mitwirkende*r“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globale*r Administrator*in“ im Microsoft Entra-Mandanten, der diesem Azure-Abonnement zugeordnet ist.

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen die Bereitstellung einer AD DS-Umgebung (Active Directory Domain Services) vorbereiten.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Bereitstellen einer AD DS-Gesamtstruktur (Active Directory Domain Services) mit einer einzelnen Domäne unter Verwendung virtueller Azure-Computer
- Integrieren einer AD DS-Gesamtstruktur in einen Microsoft Entra-Mandanten

## Labdateien

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

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

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um die Ressourcenanbieter **Microsoft.Compute** und **Microsoft.Network** zu registrieren, falls sie nicht bereits registriert sind:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um den Registrierungsstatus des Ressourcenanbieters **Microsoft.Compute** zu überprüfen:

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Hinweis:** Vergewissern Sie sich, dass der Status **Registriert** lautet. Warten Sie andernfalls einige Minuten, und wiederholen Sie diesen Schritt.

1. Führen Sie im Azure-Portal über die PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um den Ort für die nächsten Befehle festzulegen. Ersetzen Sie dabei den Platzhalter `<Azure_region>` durch den Namen der Azure-Region, die Sie für dieses Lab verwenden möchten (beispielsweise `eastus`):

   ```powershell
   $location = '<Azure_region>'
   ```

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um die aktuelle Nutzung von vCPUs und die entsprechenden Grenzwerte für die Azure-VMs vom Typ **StandardDSv3Family** und **StandardBSFamily** zu ermitteln: 

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **Hinweis:** Führen Sie im Bereich **Cloud Shell** in der PowerShell-Eingabeaufforderung `(Get-AzLocation).Location` aus, um die Namen der Azure-Regionen zu ermitteln.
   
1. Überprüfen Sie die Ausgabe des im letzten Schritt ausgeführten Befehls und vergewissern Sie sich, dass mindestens **30** vCPUs bei den **vCPUs der Standard DSv3-Familie** der Azure-VMs in der Azure-Zielregion verfügbar sind. Wenn das bereits der Fall ist, können Sie direkt mit der nächsten Übung fortfahren. Machen Sie andernfalls mit der nächsten Aufgabe dieser Übung weiter. 

#### Aufgabe 2: Anfordern einer Erhöhung des vCPU-Kontingents

1. Suchen Sie im Azure-Portal nach **Abonnements**, und wählen Sie auf dem Blatt **Abonnements** den Eintrag für das Azure-Abonnement aus, das Sie für dieses Lab verwenden möchten.
1. Wählen Sie im Azure-Portal auf dem Abonnementblatt im vertikalen Menü auf der linken Seite im Abschnitt **Einstellungen** die Option **Nutzung + Kontingente** aus. 

   **Hinweis:** Für die Kontingenterhöhung muss möglicherweise kein Supportticket erstellt werden.

   **Hinweis:** Das Anfordern einer Kontingenterhöhung erfordert die Multi-Faktor-Authentifizierung (MFA). Wenn Sie Ihr Konto mit MFA konfigurieren müssen, lesen Sie den Abschnitt [Planen einer Bereitstellung von Azure Active Directory Multi-Faktor-Authentifizierung](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted). 
   
1. Wählen Sie auf dem Blatt **Azure Pass-Förderung | Nutzung + Kontingente** die Option **Region** aus. Aktivieren Sie in der Dropdownliste das Kontrollkästchen neben dem Namen der Azure-Region, die Sie für dieses Lab verwenden möchten und wählen Sie **Anwenden**. Vergewissern Sie sich, dass in der Dropdownliste links neben dem Eintrag **Region** der Eintrag **Compute** angezeigt wird, und geben Sie **Standard DSv3** in das Suchfeld ein. 
1. Aktivieren Sie in der Ergebnisliste das Kontrollkästchen neben dem Element **vCPUs der Standard DSv3-Familie**, wählen Sie auf der Symbolleiste den Eintrag **Kontingenterhöhung anfordern** aus, und wählen Sie anschließend in der Dropdownliste die Option **Neuen Grenzwert eingeben** aus.
1. Geben Sie im Bereich **Kontingenterhöhung anfordern** im Spaltentextfeld **Neuer Grenzwert** den Wert **30** ein, und wählen Sie anschließend **Absenden** aus.
1. Wenn Sie dazu aufgefordert werden, wählen Sie im Bereich zum **Erhöhen des Anforderungskontingents** die Option **Authentifizieren mit Multi-Faktor-Authentifizierung** aus, und folgen Sie den Anweisungen zur Authentifizierung.
1. Warten Sie, bis die Kontingentanforderung abgeschlossen wurde.  Kurz darauf sehen Sie auf dem Blatt **Kontingentdetails**, dass die Anforderung genehmigt und das Kontingent erhöht wurde. Schließen Sie das Blatt **Kontingentdetails**.

   >**Hinweis:** Je nach gewählter Azure-Region und aktuellem Bedarf muss ggf. eine Supportanfrage erstellt werden. Eine Anleitung zum Erstellen einer Supportanfrage finden Sie unter [Erstellen einer Azure-Supportanfrage](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Übung 1: Bereitstellen einer AD DS-Domäne (Active Directory Domain Services)

Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Bereitstellung eines virtuellen Azure-Computers
1. Verwenden einer Azure Resource Manager-Schnellstartvorlage zum Bereitstellen eines virtuellen Azure-Computers, auf dem ein AD DS-Domänencontroller ausgeführt wird
1. Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage
1. Bereitstellen von Azure Bastion

#### Aufgabe 1: Vorbereiten der Bereitstellung eines virtuellen Azure-Computers

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Verwenden Sie im Azure-Portal oben auf der Azure-Portalseite das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** und navigieren zum Blatt **Microsoft Entra ID**.
1. Wählen Sie auf dem Blatt **Übersicht** des Microsoft Entra-Mandanten im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** **Eigenschaften** aus.
1. Wählen Sie auf dem Blatt **Eigenschaften** Ihres Microsoft Entra-Mandanten am unteren Rand des Blatts den Link **Sicherheitsstandards verwalten** aus.
1. Wählen Sie auf dem Blatt **Sicherheitsstandards aktivieren** bei Bedarf **Deaktiviert (nicht empfohlen)** aus, wählen Sie die Optionsschaltfläche **Meine Organisation plant die Verwendung von bedingtem Zugriff** aus, und wählen Sie **Speichern** und dann **Deaktivieren** aus.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und wählen Sie dann **Speicher erstellen** aus. 


#### Aufgabe 2: Verwenden einer Azure Resource Manager-Schnellstartvorlage zum Bereitstellen eines virtuellen Azure-Computers, auf dem ein AD DS-Domänencontroller ausgeführt wird

1. Führen Sie auf dem Labcomputer im Webbrowser mit dem Azure-Portal über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um eine Ressourcengruppe zu erstellen. Ersetzen Sie dabei den Platzhalter `<Azure_region>` durch den Namen der Azure-Region, die Sie für dieses Lab verwenden möchten (beispielsweise `eastus`):

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Schließen Sie im Azure-Portal den Bereich **Cloud Shell**.
1. Öffnen Sie auf Ihrem Labcomputer im gleichen Webbrowserfenster einen weiteren Webbrowser-Tab, und navigieren Sie zu einer angepassten Version der Schnellstartvorlage [Erstellen einer neuen Windows-VM und Erstellen einer neuen AD-Gesamtstruktur, einer Domäne und eines Domänencontrollers](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain). 
1. Scrollen Sie auf der Seite **Erstellen einer neuen Windows-VM und Erstellen einer neuen AD-Gesamtstruktur, einer Domäne und eines Domänencontrollers** bis zur Option **In Azure bereitstellen** und wählen Sie sie aus. Dadurch wird der Browser automatisch zum Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** im Azure-Portal umgeleitet.
1. Wählen Sie auf dem Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** die Option **Parameter bearbeiten** aus.
1. Wählen Sie auf dem Blatt **Parameter bearbeiten** die Option **Datei laden**, im Dialogfeld **Öffnen** die Datei **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**, dann **Öffnen** und schließlich **Speichern** aus. 
1. Geben Sie auf dem Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** die folgenden Einstellungen an (übernehmen Sie für andere Einstellungen die vorhandenen Werte):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-11-RG**|
   |Domänenname|**adatum.com**|

1. Wählen Sie auf dem Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** die Option **Überprüfen + erstellen** und anschließend **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Die Bereitstellung dauert etwa 20–25 Minuten. 

#### Aufgabe 3: Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage

1. Öffnen Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, eine PowerShell-Sitzung im Cloud Shell-Bereich, und führen Sie Folgendes aus, um dem virtuellen Netzwerk **az140-adds-vnet11**, das Sie in der vorherigen Aufgabe erstellt haben, ein Subnetz namens **cl-Subnet** hinzuzufügen:

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Wählen Sie im Azure-Portal auf der Symbolleiste des Cloud Shell-Bereichs das Symbol **Dateien hochladen/herunterladen** aus, wählen Sie im Dropdownmenü die Option **Hochladen** aus, und laden Sie die Dateien **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json** und **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** in das Cloud Shell-Basisverzeichnis hoch.
1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um einen virtuellen Azure-Computer mit Windows 10, der als Client fungieren soll, in dem neu erstellten Subnetz bereitzustellen:

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **Hinweis**: Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung dauert ungefähr 10 Minuten.

#### Aufgabe 4: Bereitstellen von Azure Bastion 

> **Hinweis:** Azure Bastion ermöglicht die Verbindungsherstellung mit den virtuellen Azure-Computern ohne öffentliche Endpunkte, die Sie in der vorherigen Aufgabe dieser Übung bereitgestellt haben, und bietet gleichzeitig Schutz vor Brute-Force-Angriffen, die auf Anmeldeinformationen auf Betriebssystemebene abzielen.

> **Hinweis:** Stellen Sie sicher, dass bei Ihrem Browser die Popupfunktion aktiviert ist.

1. Öffnen Sie im Browserfenster mit dem Azure-Portal einen weiteren Tab, und navigieren Sie zum [Azure-Portal](https://portal.azure.com).
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Führen Sie über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um dem virtuellen Netzwerk **az140-adds-vnet11**, das Sie weiter oben in dieser Übung erstellt haben, ein Subnetz namens **AzureBastionSubnet** hinzuzufügen:

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Azure-Portal nach **Bastions**, wählen Sie diese Option aus, und wählen Sie auf dem Blatt **Bastions** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Bastion erstellen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-11-RG**|
   |Name|**az140-11-bastion**|
   |Region|Die gleiche Azure-Region, in der Sie die Ressourcen in den vorherigen Aufgaben dieser Übung bereitgestellt haben|
   |Tarif|**Grundlegend**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnetz|**AzureBastionSubnet (10.0.254.0/24)**|
   |Öffentliche IP-Adresse|**Neu erstellen**|
   |Name der öffentlichen IP-Adresse|**az140-adds-vnet11-ip**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Bastion erstellen** die Option **Erstellen** aus:

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Die Bereitstellung dauert ungefähr 10 Minuten.

### Übung 2: Integrieren einer AD DS-Gesamtstruktur in einen Microsoft Entra-Mandanten
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen von AD DS-Benutzer*innen und -Gruppen, die mit Microsoft Entra synchronisiert werden
1. Konfigurieren des AD DS-UPN-Suffix
1. Erstellen von Microsoft Entra-Benutzer*innen, die zum Konfigurieren der Synchronisierung mit Microsoft Entra verwendet werden
1. Installieren von Microsoft Entra Connect

#### Aufgabe 1: Erstellen von AD DS-Benutzer*innen und -Gruppen, die mit Microsoft Entra synchronisiert werden

1. Suchen Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-dc-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** **Verbindung herstellen** aus und wählen Sie im Dropdownmenü **Verbindung über Bastion herstellen** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und wählen Sie **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Bastion-Sitzung auf **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator*in.
1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die erweiterte Sicherheit von Internet Explorer für Administratoren zu deaktivieren:

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um eine AD DS-Organisationseinheit zu erstellen, die Objekte enthält, die in die Synchronisierung mit dem in diesem Lab verwendeten Microsoft Entra-Mandanten einbezogen werden:

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um eine AD DS-Organisationseinheit zu erstellen, die Computerobjekte von in die Domäne eingebundenen Windows 10-Clientcomputern enthält:

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um AD DS-Benutzerkonten zu erstellen, die mit dem in diesem Lab verwendeten Microsoft Entra-Mandanten synchronisiert werden (ersetzen Sie dabei die beiden `<password>`-Platzhalter durch ein beliebiges komplexes Kennwort):

   > **Hinweis:** Stellen Sie sicher, dass Sie sich die verwendeten Kennwörter notieren. Sie werden sie später in diesem Lab sowie in nachfolgenden Labs benötigt.

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **Hinweis:** Das Skript erstellt neun nicht privilegierte Benutzerkonten namens **aduser1** - **aduser9** und ein privilegiertes Konto namens **wvdadmin1**, das der Gruppe **ADATUM\\Domänenadministratoren** angehört.

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um AD DS-Gruppenobjekte zu erstellen, die mit dem in diesem Lab verwendeten Microsoft Entra-Mandanten synchronisiert werden:

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den im vorherigen Schritt erstellten Gruppen Mitglieder hinzuzufügen:

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### Aufgabe 2: Konfigurieren des AD DS-UPN-Suffix

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des PowerShellGet-Moduls zu installieren, und wählen Sie **Ja** aus, wenn Sie zur Bestätigung aufgefordert werden:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des Az PowerShell-Moduls zu installieren, und wählen Sie **Ja**, alle aus, wenn Sie zur Bestätigung aufgefordert werden:

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

   > **Hinweis:** Es kann sein, dass Sie 3–5 Minuten warten müssen, bevor eine Ausgabe der Installation des Az-Moduls erscheint. Möglicherweise müssen Sie auch weitere 5 Minuten warten, **nachdem** die Ausgabe beendet wurde. Dieses Verhalten wird erwartet.

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die ID-Eigenschaft des Microsoft Entra-Mandanten abzurufen, der Ihrem Azure-Abonnement zugeordnet ist:

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des Azure AD PowerShell-Moduls zu installieren und zu importieren:

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um sich bei Ihrem Microsoft Entra-Mandanten zu authentifizieren:

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den gleichen Anmeldeinformationen an, die Sie zuvor in dieser Aufgabe verwendet haben (mit dem Benutzerkonto, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt). 
1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den primären DNS-Domänennamen des Microsoft Entra-Mandanten abzurufen, der Ihrem Azure-Abonnement zugeordnet ist:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den primären DNS-Domänennamen des Microsoft Entra-Mandanten, der Ihrem Azure-Abonnement zugeordnet ist, der Liste mit den UPN-Suffixen Ihrer AD DS-Gesamtstruktur hinzuzufügen:

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den primären DNS-Domänennamen des Microsoft Entra-Mandanten, der Ihrem Azure-Abonnement zugeordnet ist, als UPN-Suffix aller Benutzer*innen in der AD DS-Domäne zuzuweisen:

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um das UPN-Suffix **adatum.com** wieder dem Domänenbenutzer **Studierende** zuzuweisen:

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### Aufgabe 3: Erstellen von Microsoft Entra-Benutzer*innen zum Konfigurieren der Verzeichnissynchronisierung

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um eine*n neue*n Microsoft Entra-Benutzer*in zu erstellen, und ersetzen Sie dabei den Platzhalter `<password>` durch ein beliebiges komplexes Kennwort:

   > **Hinweis:** Stellen Sie sicher, dass Sie sich das verwendete Kennwort notieren. **Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt**.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um dem bzw. der neu erstellten Microsoft Entra-Benutzer*in die Rolle „Globale*r Administrator*in“ zuzuweisen: 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den Benutzerprinzipalnamen des bzw. der neu erstellten Microsoft Entra-Benutzer*in zu ermitteln:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **Hinweis:** Notieren Sie den Benutzerprinzipalnamen **und** das Kennwort. Er wird später in dieser Übung benötigt. 


#### Aufgabe 4: Installieren von Microsoft Entra Connect

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um TLS 1.2 zu aktivieren:

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. Starten Sie in der Bastion-Sitzung für **az140-dc-vm11** den Internet Explorer, und navigieren Sie zur [Downloadseite für Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download).
1. Laden Sie auf der [Downloadseite für Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download) die neueste stabile Version von Microsoft Edge herunter, installieren Sie sie, starten Sie sie, und konfigurieren Sie sie mit den Standardeinstellungen.
1. Navigieren Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** in Microsoft Edge zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Verwenden Sie im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** (im oberen Bereich der Azure-Portalseite), um nach dem Blatt **Microsoft Entra ID** zu suchen und dorthin zu navigieren. Wählen Sie dann auf dem Blatt Ihres Microsoft Entra-Mandanten im Abschnitt **Verwalten** des Hubmenüs die Option **Microsoft Entra Connect** aus.
1. Wählen Sie auf dem Blatt **Microsoft Entra Connect** im Dienstmenü den Link **Synchronisierung verbinden** aus, und wählen Sie dann den Link **Microsoft Entra Connect herunterladen** aus. Daraufhin wird automatisch ein neuer Browser-Tab mit der Downloadseite **Microsoft Entra Connect** geöffnet.
1. Wählen Sie auf der Downloadseite **Microsoft Entra Connect** die Option **Herunterladen** aus.
1. Wenn Sie gefragt werden, ob das Installationsprogramm **AzureADConnect.msi** ausgeführt oder gespeichert werden soll, wählen Sie **Ausführen** aus. Öffnen Sie andernfalls die Datei, nachdem sie heruntergeladen wurde, um den Assistenten für **Microsoft Azure Active Directory Connect** zu starten.
1. Aktivieren Sie auf der Seite **Willkommen bei Azure AD Connect** des Assistenten für **Microsoft Azure Active Directory Connect** das Kontrollkästchen **Ich akzeptiere die Lizenzbedingungen und den Datenschutzhinweis.**, und wählen Sie anschließend **Weiter** aus.
1. Wählen Sie auf der Seite **Expresseinstellungen** des Assistenten für **Microsoft Azure Active Directory Connect** die Option **Anpassen** aus.
1. Belassen Sie auf der Seite **Erforderliche Komponenten installieren** alle optionalen Konfigurationsoptionen deaktiviert, und wählen Sie **Installieren** aus.
1. Stellen Sie auf der Seite **Benutzeranmeldung** sicher, dass nur die **Kennwort-Hashsynchronisierung** aktiviert ist, und wählen Sie **Weiter** aus.
1. Authentifizieren Sie sich auf der Seite **Mit Azure AD verbinden** mit den Anmeldeinformationen des Benutzerkontos **aadsyncuser**, das Sie in der vorherigen Übung erstellt haben, und wählen Sie anschließend **Weiter** aus. 

   > **Hinweis:** Geben Sie das weiter oben in dieser Übung erfasste Attribut „userPrincipalName“ des Kontos **aadsyncuser** sowie das zugehörige Kennwort an, das Sie zuvor in dieser Übung als Kennwort festgelegt haben.

1. Wählen Sie auf der Seite **Verzeichnisse verbinden** rechts neben dem Gesamtstruktureintrag **adatum.com** die Schaltfläche **Verzeichnis hinzufügen** aus.
1. Stellen Sie im Fenster **AD-Gesamtstrukturkonto** sicher, dass die Option **Neues AD-Konto erstellen** ausgewählt ist, geben Sie die folgenden Anmeldeinformationen an, und wählen Sie anschließend **OK** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**ADATUM\Student**|
   |Kennwort|**Pa55w.rd1234**|

1. Stellen Sie auf der Seite **Verzeichnisse verbinden** sicher, dass der Eintrag **adatum.com** als konfiguriertes Verzeichnis angezeigt wird, und wählen Sie anschließend **Weiter** aus.
1. Beachten Sie auf der Seite **Azure AD-Anmeldungskonfiguration** die Warnung **Die Benutzer können sich nicht mit lokalen Anmeldeinformationen bei Azure AD anmelden, wenn das UPN-Suffix keiner überprüften Domäne zugeordnet werden kann.**, aktivieren Sie das Kontrollkästchen **Ohne Abgleich aller UPN-Suffixe mit überprüften Domänen fortfahren**, und wählen Sie anschließend **Weiter** aus.

   > **Hinweis:** Dieses Verhalten war zu erwarten, da der Microsoft Entra-Mandant über keine überprüfte benutzerdefinierte DNS-Domäne verfügt, die einem der UPN-Suffixe der AD DS-Instanz von **adatum.com** entspricht.

1. Wählen Sie auf der Seite **Filtern von Domänen und Organisationseinheiten** die Option **Ausgewählte Domänen und Organisationseinheiten synchronisieren** aus, erweitern Sie den Knoten „adatum.com“, deaktivieren Sie alle Kontrollkästchen, aktivieren Sie nur das Kontrollkästchen neben der Organisationseinheit **ToSync**, und wählen Sie anschließend **Weiter** aus.
1. Übernehmen Sie auf der Seite **Eindeutige Identifizierung Ihrer Benutzer** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Übernehmen Sie auf der Seite **Benutzer und Geräte filtern** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Übernehmen Sie auf der Seite **Optionale Features** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Bereit zur Konfiguration**, dass das Kontrollkästchen **Starten Sie den Synchronisierungsvorgang, nachdem die Konfiguration abgeschlossen wurde.** aktiviert ist, und wählen Sie anschließend **Installieren** aus.

   > **Hinweis:** Die Installation sollte ungefähr 5 Minuten dauern.

1. Überprüfen Sie die Informationen auf der Seite **Konfiguration abgeschlossen**, und wählen Sie **Beenden** aus, um das Fenster **Microsoft Azure Active Directory Connect** zu schließen.
1. Navigieren Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** in dem Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, zum Blatt **Benutzer*innen – Alle Benutzer*innen** des Microsoft Entra-Mandanten „Adatum Lab“.
1. Beachten Sie auf dem Blatt **Benutzer*innen \| Alle Benutzer*innen**, dass die Liste der Benutzerobjekte die Auflistung der AD DS-Benutzerkonten enthält, die Sie zuvor in diesem Lab erstellt haben, und dass der Eintrag **Ja** in der Spalte **Lokale Synchronisierung aktiviert** angezeigt wird.

   > **Hinweis:** Möglicherweise müssen Sie einige Minuten warten und die Browserseite aktualisieren, damit die AD DS-Benutzerkonten angezeigt werden.
