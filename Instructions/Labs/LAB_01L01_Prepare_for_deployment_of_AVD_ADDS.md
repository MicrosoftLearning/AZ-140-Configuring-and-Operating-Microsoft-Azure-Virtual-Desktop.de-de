---
lab:
  title: "Lab: Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD\_DS)"
  module: 'Module 1: Plan a AVD Architecture'
---

# <a name="lab---prepare-for-deployment-of-azure-virtual-desktop-ad-ds"></a>Lab: Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lab-Abhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden.
- Ein Microsoft-Konto oder Azure AD-Konto mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globaler Administrator“ im Azure AD-Mandanten, der diesem Azure-Abonnement zugeordnet ist

## <a name="estimated-time"></a>Geschätzte Dauer

60 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen die Bereitstellung einer AD DS-Umgebung (Active Directory Domain Services) vorbereiten.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Bereitstellen einer AD DS-Gesamtstruktur (Active Directory Domain Services) mit einer einzelnen Domäne unter Verwendung virtueller Azure-Computer
- Integrieren einer AD DS-Gesamtstruktur in einen Azure AD-Mandanten (Azure Active Directory)

## <a name="lab-files"></a>Labdateien

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## <a name="instructions"></a>Anweisungen

### <a name="exercise-0-increase-the-number-of-vcpu-quotas"></a>Übung 0: Erhöhen der Anzahl von vCPU-Kontingenten

Die Hauptaufgaben für diese Übung sind Folgende:

1. Identifizieren der aktuellen vCPU-Nutzung
1. Anfordern einer Erhöhung des vCPU-Kontingents

#### <a name="task-1-identify-current-vcpu-usage"></a>Aufgabe 1: Identifizieren der aktuellen vCPU-Nutzung

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis:** Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt.** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement und anschließend **Speicher erstellen** aus. 

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um den Ressourcenanbieter **Microsoft.Compute** zu registrieren, falls er nicht registriert sein sollte:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um den Registrierungsstatus des Ressourcenanbieters **Microsoft.Compute** zu überprüfen:

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Hinweis:** Vergewissern Sie sich, dass der Status **Registriert** lautet. Warten Sie andernfalls einige Minuten, und wiederholen Sie diesen Schritt.

1. Führen Sie im Azure-Portal in der PowerShell-Sitzung von **Cloud Shell** Folgendes aus, um die aktuelle Nutzung von vCPUs und die entsprechenden Grenzwerte für die virtuellen Computer vom Typ **StandardDSv3Family** und **StandardBSFamily** zu ermitteln. Ersetzen Sie dabei den Platzhalter `<Azure_region>` durch den Namen der Azure-Region, die Sie für dieses Lab verwenden möchten (beispielsweise `eastus`):

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardBSFamily'}
   ```

   > **Hinweis:** Führen Sie im Bereich **Cloud Shell** in der PowerShell-Eingabeaufforderung `(Get-AzLocation).Location` aus, um die Namen der Azure-Regionen zu ermitteln.
   
1. Überprüfen Sie die Ausgabe des ausgeführten Befehls aus dem vorherigen Schritt, und vergewissern Sie sich, dass sowohl bei den **vCPUs der Standard DSv3-Familie** als auch bei den **vCPUs der Standard BS-Familie** der virtuellen Azure-Computer in der Azure-Zielregion mindestens **30** verfügbare vCPUs vorhanden sind. Wenn das bereits der Fall ist, können Sie direkt mit der nächsten Übung fortfahren. Machen Sie andernfalls mit der nächsten Aufgabe dieser Übung weiter. 

#### <a name="task-2-request-vcpu-quota-increase"></a>Aufgabe 2: Anfordern einer Erhöhung des vCPU-Kontingents

1. Suchen Sie im Azure-Portal nach **Abonnements**, und wählen Sie auf dem Blatt **Abonnements** den Eintrag für das Azure-Abonnement aus, das Sie für dieses Lab verwenden möchten.
1. Wählen Sie im Azure-Portal auf dem Abonnementblatt im vertikalen Menü auf der linken Seite im Abschnitt **Einstellungen** die Option **Nutzung + Kontingente** aus. 

   **Hinweis:** Für die Kontingenterhöhung muss möglicherweise kein Supportticket erstellt werden.
   
1. Wählen Sie auf dem Blatt **Azure Pass-Förderung | Nutzung + Kontingente** die Option **Region** aus. Aktivieren Sie in der Dropdownliste das Kontrollkästchen neben dem Namen der Azure-Region, die Sie für dieses Lab verwenden möchten. Vergewissern Sie sich, dass in der Dropdownliste links neben dem Eintrag **Region** der Eintrag **Compute** angezeigt wird, und geben Sie **Standard BS** in das Suchfeld ein. 
1. Aktivieren Sie in der Ergebnisliste das Kontrollkästchen neben dem Element **vCPUs der Standard BS-Familie**, wählen Sie auf der Symbolleiste den Eintrag **Kontingenterhöhung anfordern** aus, und wählen Sie anschließend in der Dropdownliste die Option **Neuen Grenzwert eingeben** aus.
1. Geben Sie im Bereich **Kontingenterhöhung anfordern** im Spaltentextfeld **Neuer Grenzwert** den Wert **30** ein, und wählen Sie anschließend **Absenden** aus.
1. Warten Sie, bis die Kontingentanforderung abgeschlossen wurde.  Kurz darauf sehen Sie auf dem Blatt **Kontingentdetails**, dass die Anforderung genehmigt und das Kontingent erhöht wurde. Schließen Sie das Blatt **Kontingentdetails**.
1. Wiederholen Sie die Schritte 3 bis 6, um die Kontingentgrenze für die VM-Größe **Standard DSv3** auf **30** festzulegen.

   >**Hinweis:** Je nach gewählter Azure-Region und aktuellem Bedarf muss ggf. eine Supportanfrage erstellt werden. Eine Anleitung zum Erstellen einer Supportanfrage finden Sie unter [Erstellen einer Azure-Supportanfrage])https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### <a name="exercise-1-deploy-an-active-directory-domain-services-ad-ds-domain"></a>Übung 1: Bereitstellen einer AD DS-Domäne (Active Directory Domain Services)

Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Bereitstellung eines virtuellen Azure-Computers
1. Verwenden einer Azure Resource Manager-Schnellstartvorlage zum Bereitstellen eines virtuellen Azure-Computers, auf dem ein AD DS-Domänencontroller ausgeführt wird
1. Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage
1. Bereitstellen von Azure Bastion

#### <a name="task-1-prepare-for-an-azure-vm-deployment"></a>Aufgabe 1: Vorbereiten der Bereitstellung eines virtuellen Azure-Computers

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Navigieren Sie in dem Webbrowser, in dem das Azure-Portal angezeigt wird, zum Blatt **Übersicht** des Azure AD-Mandanten, und klicken Sie im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **Eigenschaften**.
1. Wählen Sie auf dem Blatt **Eigenschaften** Ihres Azure AD-Mandanten am unteren Rand des Blatts den Link **Sicherheitsstandards verwalten** aus.
1. Wählen Sie auf dem Blatt **Sicherheitsstandards aktivieren** bei Bedarf **Nein** aus, aktivieren Sie das Kontrollkästchen **Meine Organisation verwendet den bedingten Zugriff.** , und wählen Sie anschließend **Speichern** aus.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Wenn Sie aufgefordert werden, entweder **Bash** oder **PowerShell** auszuwählen, wählen Sie **PowerShell** aus. 

   >**Hinweis:** Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt.** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement und anschließend **Speicher erstellen** aus. 


#### <a name="task-2-deploy-an-azure-vm-running-an-ad-ds-domain-controller-by-using-an-azure-resource-manager-quickstart-template"></a>Aufgabe 2: Verwenden einer Azure Resource Manager-Schnellstartvorlage zum Bereitstellen eines virtuellen Azure-Computers, auf dem ein AD DS-Domänencontroller ausgeführt wird

1. Führen Sie auf dem Labcomputer im Webbrowser mit dem Azure-Portal über die PowerShell-Sitzung im Cloud Shell-Bereich Folgendes aus, um eine Ressourcengruppe zu erstellen. Ersetzen Sie dabei den Platzhalter `<Azure_region>` durch den Namen der Azure-Region, die Sie für dieses Lab verwenden möchten (beispielsweise `eastus`):

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Schließen Sie im Azure-Portal den Bereich **Cloud Shell**.
1. Öffnen Sie auf Ihrem Labcomputer im gleichen Webbrowserfenster einen weiteren Webbrowser-Tab, und navigieren Sie zu einer angepassten Version der Schnellstartvorlage [Erstellen einer neuen Windows-VM und Erstellen einer neuen AD-Gesamtstruktur, einer Domäne und eines Domänencontrollers](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain). 
1. Wählen Sie auf der Seite **Erstellen einer neuen Windows-VM und Erstellen einer neuen AD-Gesamtstruktur, einer Domäne und eines Domänencontrollers** die Option **In Azure bereitstellen** aus. Dadurch wird der Browser automatisch zum Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** im Azure-Portal umgeleitet.
1. Wählen Sie auf dem Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** die Option **Parameter bearbeiten** aus.
1. Wählen Sie auf dem Blatt **Parameter bearbeiten** die Option **Datei laden**, im Dialogfeld **Öffnen** die Datei **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**, dann **Öffnen** und schließlich **Speichern** aus. 
1. Geben Sie auf dem Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** die folgenden Einstellungen an (übernehmen Sie für andere Einstellungen die vorhandenen Werte):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-11-RG**|
   |Domänenname|**adatum.com**|

1. Wählen Sie auf dem Blatt **Azure-VM mit einer neuen AD-Gesamtstruktur erstellen** die Option **Überprüfen + erstellen** und anschließend **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Das dauert etwa 15 Minuten. 

#### <a name="task-3-deploy-an-azure-vm-running-windows-10-by-using-an-azure-resource-manager-quickstart-template"></a>Aufgabe 3: Bereitstellen eines virtuellen Azure-Computers mit Windows 10 mithilfe einer Azure Resource Manager-Schnellstartvorlage

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

   > **Hinweis**: Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung dauert ungefähr zehn Minuten.

#### <a name="task-4-deploy-azure-bastion"></a>Aufgabe 4: Bereitstellen von Azure Bastion 

> **Hinweis:** Azure Bastion ermöglicht die Verbindungsherstellung mit den virtuellen Azure-Computern ohne öffentliche Endpunkte, die Sie in der vorherigen Aufgabe dieser Übung bereitgestellt haben, und bietet gleichzeitig Schutz vor Brute-Force-Angriffen, die auf Anmeldeinformationen auf Betriebssystemebene abzielen.

> **Hinweis:** Stellen Sie sicher, dass bei Ihrem Browser die Popupfunktion aktiviert ist.

1. Öffnen Sie im Browserfenster mit dem Azure-Portal eine weitere Registerkarte, und navigieren Sie auf der Registerkarte zum Azure-Portal.
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
1. Suchen Sie im Azure-Portal nach **Bastions**, wählen Sie diese Option aus, und wählen Sie auf dem Blatt **Bastions** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Basic** des Blatts **Bastion erstellen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-11-RG**|
   |Name|**az140-11-bastion**|
   |Region|Dieselbe Azure-Region, in der Sie die Ressourcen in den vorherigen Aufgaben dieser Übung bereitgestellt haben|
   |Tarif|**Grundlegend**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnet|**AzureBastionSubnet (10.0.254.0/24)**|
   |Öffentliche IP-Adresse|**Neu erstellen**|
   |Name der öffentlichen IP-Adresse|**az140-adds-vnet11-ip**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Bastion erstellen** die Option **Erstellen** aus:

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Übung fortfahren. Die Bereitstellung dauert etwa fünf Minuten.

### <a name="exercise-2-integrate-an-ad-ds-forest-with-an-azure-ad-tenant"></a>Übung 2: Integrieren einer AD DS-Gesamtstruktur in einen Azure AD-Mandanten
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen von AD DS-Benutzern und -Gruppen, die mit Azure AD synchronisiert werden
1. Konfigurieren des AD DS-UPN-Suffix
1. Erstellen eines Azure AD-Benutzers zum Konfigurieren der Synchronisierung mit Azure AD
1. Installieren von Azure AD Connect
1. Konfigurieren der Hybrid-Azure AD-Einbindung

#### <a name="task-1-create-ad-ds-users-and-groups-that-will-be-synchronized-to-azure-ad"></a>Aufgabe 1: Erstellen von AD DS-Benutzern und -Gruppen, die mit Azure AD synchronisiert werden

1. Suchen Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-dc-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und wählen Sie **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator.
1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die erweiterte Sicherheit von Internet Explorer für Administratoren zu deaktivieren:

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um eine AD DS-Organisationseinheit zu erstellen, die Objekte enthält, die in die Synchronisierung mit dem in dieser Übung verwendeten Azure AD-Mandanten einbezogen werden:

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um eine AD DS-Organisationseinheit zu erstellen, die Computerobjekte von in die Domäne eingebundenen Windows 10-Clientcomputern enthält:

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um AD DS-Benutzerkonten zu erstellen, die mit dem in diesem Lab verwendeten Azure AD-Mandanten synchronisiert werden. Ersetzen Sie dabei den Platzhalter `<password>` durch ein beliebiges komplexes Kennwort:

   > **Hinweis:** Merken Sie sich das verwendete Kennwort. Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt.

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString "<password>" -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString "<password>" -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **Hinweis:** Das Skript erstellt neun nicht privilegierte Benutzerkonten namens **aduser1** - **aduser9** und ein privilegiertes Konto namens **wvdadmin1**, das der Gruppe **ADATUM\\Domänenadministratoren** angehört.

1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um AD DS-Gruppenobjekte zu erstellen, die mit dem in diesem Lab verwendeten Azure AD-Mandanten synchronisiert werden:

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um den im vorherigen Schritt erstellten Gruppen Mitglieder hinzuzufügen:

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### <a name="task-2-configure-ad-ds-upn-suffix"></a>Aufgabe 2: Konfigurieren des AD DS-UPN-Suffix

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des PowerShellGet-Moduls zu installieren, und wählen Sie **Ja** aus, wenn Sie zur Bestätigung aufgefordert werden:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des Az PowerShell-Moduls zu installieren, und wählen Sie **Ja, alle** aus, wenn Sie zur Bestätigung aufgefordert werden:

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die ID-Eigenschaft des Azure AD-Mandanten abzurufen, der Ihrem Azure-Abonnement zugeordnet ist:

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die neueste Version des Azure AD PowerShell-Moduls zu installieren und zu importieren:

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um sich bei Ihrem Azure AD-Mandanten zu authentifizieren:

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. Melden Sie sich bei entsprechender Aufforderung mit den gleichen Anmeldeinformationen an, die Sie zuvor in dieser Aufgabe verwendet haben. 
1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um den primären DNS-Domänennamen des Azure AD-Mandanten abzurufen, der Ihrem Azure-Abonnement zugeordnet ist:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um den primären DNS-Domänennamen des Azure AD-Mandanten, der Ihrem Azure-Abonnement zugeordnet ist, der Liste mit den UPN-Suffixen Ihrer AD DS-Gesamtstruktur hinzuzufügen:

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den primären DNS-Domänennamen des Azure AD-Mandanten, der Ihrem Azure-Abonnement zugeordnet ist, als UPN-Suffix aller Benutzer in der AD DS-Domäne zuzuweisen:

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. Führen Sie in der Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um das UPN-Suffix **adatum.com** dem Domänenbenutzer **Student** zuzuweisen:

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### <a name="task-3-create-an-azure-ad-user-that-will-be-used-to-configure-directory-synchronization"></a>Aufgabe 3: Erstellen eines Azure AD-Benutzers zum Konfigurieren der Verzeichnissynchronisierung

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um einen neuen Azure AD-Benutzer zu erstellen, und ersetzen Sie dabei den Platzhalter `<password>` durch ein beliebiges komplexes Kennwort:

   > **Hinweis:** Merken Sie sich das verwendete Kennwort. Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um dem neu erstellten Azure AD-Benutzer die Rolle „Globaler Administrator“ zuzuweisen: 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **Hinweis:** Im PowerShell-Modul für Azure AD wird die Rolle „Globaler Administrator“ als „Unternehmensadministrator“ bezeichnet.

1. Führen Sie im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um den Benutzerprinzipalnamen des neu erstellten Azure AD-Benutzers zu ermitteln:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **Hinweis:** Erfassen Sie den Benutzerprinzipalnamen. Er wird später in dieser Übung benötigt. 


#### <a name="task-4-install-azure-ad-connect"></a>Aufgabe 4: Installieren von Azure AD Connect

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Skriptbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um TLS 1.2 zu aktivieren:

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
   
1. Starten Sie in der Remotedesktopsitzung für **az140-dc-vm11** Internet Explorer, und navigieren Sie zur [Downloadseite für Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download).
1. Laden Sie auf der [Downloadseite für Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download) die neueste stabile Version von Microsoft Edge herunter, installieren Sie sie, starten Sie sie, und konfigurieren Sie sie mit den Standardeinstellungen.
1. Navigieren Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** in Microsoft Edge zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Azure AD-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Verwenden Sie im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** (im oberen Bereich der Azure-Portalseite), um nach dem Blatt **Azure Active Directory** zu suchen und dorthin zu navigieren. Wählen Sie dann auf dem Blatt Ihres Azure AD-Mandanten im Abschnitt **Verwalten** des Hubmenüs die Option **Azure AD Connect** aus.
1. Wählen Sie auf dem Blatt **Azure AD Connect** den Link **Azure AD Connect herunterladen** aus. Daraufhin wird automatisch ein neuer Browser-Tab mit der Downloadseite **Microsoft Azure Active Directory Connect** geöffnet.
1. Wählen Sie auf der Downloadseite **Microsoft Azure Active Directory Connect** die Option **Herunterladen** aus.
1. Wenn Sie gefragt werden, ob das Installationsprogramm **AzureADConnect.msi** ausgeführt oder gespeichert werden soll, wählen Sie **Ausführen** aus. Öffnen Sie andernfalls die Datei, nachdem sie heruntergeladen wurde, um den Assistenten für **Microsoft Azure Active Directory Connect** zu starten.
1. Aktivieren Sie auf der Seite **Willkommen bei Azure AD Connect** des Assistenten für **Microsoft Azure Active Directory Connect** das Kontrollkästchen **Ich akzeptiere die Lizenzbedingungen und den Datenschutzhinweis.** , und wählen Sie anschließend **Weiter** aus.
1. Wählen Sie auf der Seite **Expresseinstellungen** des Assistenten für **Microsoft Azure Active Directory Connect** die Option **Anpassen** aus.
1. Lassen Sie auf der Seite **Erforderliche Komponenten installieren** alle optionalen Konfigurationsoptionen deaktiviert, und wählen Sie **Installieren** aus.
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
1. Beachten Sie auf der Seite **Azure AD-Anmeldungskonfiguration** die Warnung **Die Benutzer können sich nicht mit lokalen Anmeldeinformationen bei Azure AD anmelden, wenn das UPN-Suffix keiner überprüften Domäne zugeordnet werden kann.** , aktivieren Sie das Kontrollkästchen **Ohne Abgleich aller UPN-Suffixe mit überprüften Domänen fortfahren**, und wählen Sie anschließend **Weiter** aus.

   > **Hinweis:** Dieses Verhalten war zu erwarten, da der Azure AD-Mandant über keine überprüfte benutzerdefinierte DNS-Domäne verfügt, die einem der UPN-Suffixe der AD DS-Instanz von **adatum.com** entspricht.

1. Wählen Sie auf der Seite **Filtern von Domänen und Organisationseinheiten** die Option **Ausgewählte Domänen und Organisationseinheiten synchronisieren** aus, erweitern Sie den Knoten „adatum.com“, deaktivieren Sie alle Kontrollkästchen, aktivieren Sie nur das Kontrollkästchen neben der Organisationseinheit **ToSync**, und wählen Sie anschließend **Weiter** aus.
1. Übernehmen Sie auf der Seite **Eindeutige Identifizierung Ihrer Benutzer** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Übernehmen Sie auf der Seite **Benutzer und Geräte filtern** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Übernehmen Sie auf der Seite **Optionale Features** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Bereit zur Konfiguration**, dass das Kontrollkästchen **Starten Sie den Synchronisierungsvorgang, nachdem die Konfiguration abgeschlossen wurde.** aktiviert ist, und wählen Sie anschließend **Installieren** aus.

   > **Hinweis**: Die Installation sollte ungefähr 2 Minuten dauern.

1. Überprüfen Sie die Informationen auf der Seite **Konfiguration abgeschlossen**, und wählen Sie **Beenden** aus, um das Fenster **Microsoft Azure Active Directory Connect** zu schließen.
1. Navigieren Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** in dem Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, zum Blatt **Benutzer | Alle Benutzer** des Azure AD-Mandanten „Adatum Lab“.
1. Beachten Sie auf dem Blatt **Benutzer \| Alle Benutzer**, dass die Liste der Benutzerobjekte die Auflistung der AD DS-Benutzerkonten enthält, die Sie zuvor in dieser Übung erstellt haben, und dass in der Spalte **Verzeichnissynchronisierung** der Eintrag **Ja** angezeigt wird.

   > **Hinweis:** Möglicherweise müssen Sie einige Minuten warten und die Browserseite aktualisieren, damit die AD DS-Benutzerkonten angezeigt werden.
