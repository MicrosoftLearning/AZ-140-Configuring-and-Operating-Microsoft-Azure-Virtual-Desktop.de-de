---
lab:
  title: "Lab: Bereitstellen von Hostpools und Hosts mithilfe von Azure\_Resource\_Manager-Vorlagen (AD DS)"
  module: 'Module 2: Implement a WVD Infrastructure'
---

# <a name="lab---deploy-host-pools-and-hosts-by-using-azure-resource-manager-templates"></a>Lab: Bereitstellen von Hostpools und Hosts mithilfe von Azure Resource Manager-Vorlagen
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lab-Abhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden.
- Ein Microsoft-Konto oder ein Azure AD-Konto mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle„Globaler Administrator“ im Azure AD-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)**

## <a name="estimated-time"></a>Geschätzte Dauer

45 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen die Bereitstellung von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von Azure-Resource Manager-Vorlagen automatisieren.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Bereitstellen von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von Azure Resource Manager-Vorlagen

## <a name="lab-files"></a>Lab-Dateien

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## <a name="instructions"></a>Anweisungen

### <a name="exercise-1-deploy-azure-virtual-desktop-host-pools-and-hosts-by-using-azure-resource-manager-templates"></a>Übung 1: Bereitstellen von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von Azure Resource Manager-Vorlagen
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten der Bereitstellung eines Azure Virtual Desktop-Hostpools mithilfe einer Azure Resource Manager-Vorlage
1. Bereitstellen von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von Azure Resource Manager-Vorlagen
1. Überprüfen der Bereitstellung von Azure Virtual Desktop-Hostpools und -Hosts
1. Vorbereiten auf das Hinzufügen von Hosts zum vorhandenen Azure Virtual Desktop-Hostpool mithilfe einer Azure Resource Manager-Vorlage
1. Hinzufügen von Hosts zum vorhandenen Azure Virtual Desktop-Hostpool mithilfe einer Azure Resource Manager-Vorlage
1. Überprüfen von Änderungen am Azure Virtual Desktop-Hostpool
1. Verwalten von Zuweisungen persönlicher Desktops im Azure Virtual Desktop-Hostpool

#### <a name="task-1-prepare-for-deployment-of-an-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>Aufgabe 1: Vorbereiten der Bereitstellung eines Azure Virtual Desktop-Hostpools mithilfe einer Azure Resource Manager-Vorlage

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie dann auf dem Blatt **Virtuelle Computer** auf **az140-dc-vm11**.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** die Option **Verbinden**, im Dropdownmenü **Bastion**, auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator.
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Distinguished Name der Organisationseinheit mit dem Namen **WVDInfra** zu ermitteln, in der die Computerobjekte der Azure Virtual Desktop-Hostpools gehostet werden:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das UPN-Attribut (Benutzerprinzipalname) des Kontos **ADATUM\\Student** zu ermitteln, das für den Beitritt der Azure Virtual Desktop-Hosts zur AD DS-Domäne ( **student@adatum.com** ) verwendet wird:

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Benutzerprinzipalnamen der Konten **ADATUM\\aduser7** und **ADATUM\\aduser8** zu ermitteln, die Sie später in diesem Lab verwenden, um Zuweisungen zu persönlichen Desktops zu testen.

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **Hinweis:** Notieren Sie alle ermittelten Werte für die Benutzerprinzipalnamen. Sie werden sie später in diesem Lab benötigen.

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um die Tokenablaufzeit zu berechnen, die zum Durchführen einer vorlagenbasierten Bereitstellung erforderlich ist.

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **Hinweis:** Der Wert muss das folgende Format aufweisen: `2022-03-27T00:51:28.3008055Z`. Notieren Sie ihn, da Sie ihn in der nächsten Aufgabe benötigen.

   > **Hinweis:** Ein Registrierungstoken wird für den Beitritt eines Hosts zum Pool benötigt. Der Wert für das Ablaufdatum des Tokens muss zwischen einer Stunde und einem Monat zwischen dem aktuellen Datum und der aktuellen Uhrzeit liegen.

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mithilfe der Azure AD-Anmeldeinformationen des Benutzerkontos mit der Rolle „Besitzer“ in dem Abonnement an, das Sie in diesem Lab verwenden.
1. Verwenden Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** oben auf der Seite des Azure-Portals, um nach **Virtuelle Netzwerke** zu suchen und dorthin zu navigieren, und wählen Sie auf dem Blatt **Virtuelle Netzwerke** die Option **az140-adds-vnet11** aus. 
1. Wählen Sie auf dem Blatt **az140-adds-vnet11** die Option **Subnetze** und auf dem Blatt **Subnetze** die Option **+ Subnetz** aus. Geben Sie auf dem Blatt **Subnetz hinzufügen** die folgenden Einstellungen an (übernehmen Sie bei allen anderen Einstellungen die Standardwerte), und klicken Sie auf **Speichern**:

   |Einstellung|Wert|
   |---|---|
   |Name|**hp2-Subnet**|
   |Subnetzadressbereich|**10.0.2.0/24**|

1. Verwenden Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** oben auf der Seite des Azure-Portals, um nach **Netzwerksicherheitsgruppen** zu suchen und dorthin zu navigieren, und wählen Sie auf dem Blatt **Netzwerksicherheitsgruppen** die Netzwerksicherheitsgruppe in der Ressourcengruppe **az140-11-RG** aus.
1. Klicken Sie auf dem Blatt „Netzwerksicherheitsgruppe“ im vertikalen Menü links im Abschnitt **Einstellungen** auf **Eigenschaften**.
1. Klicken Sie auf dem Blatt **Eigenschaften** auf das Symbol **In die Zwischenablage kopieren** rechts neben dem Textfeld **Ressourcen-ID**. 

   > **Hinweis:** Der Wert muss das folgende Format aufweisen: `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`. Die Abonnement-ID kann jedoch anders lauten. Notieren Sie ihn, da Sie ihn in der nächsten Aufgabe benötigen.

#### <a name="task-2-deploy-an-azure-virtual-desktop-host-pool-and-hosts-by-using-an-azure-resource-manager-template"></a>Aufgabe 2: Bereitstellen von Azure Virtual Desktop-Hostpools und -Hosts mithilfe von Azure Resource Manager-Vorlagen

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Öffnen Sie auf Ihrem Labcomputer im selben Webbrowserfenster eine weitere Webbrowserregisterkarte, und navigieren Sie zur GitHub Azure RDS-Vorlagenrepositoryseite [ARM-Vorlage zum Erstellen und Bereitstellen neuer Azure Virtual Desktop-Hostpools](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool). 
1. Wählen Sie auf der Seite **ARM-Vorlage zum Erstellen und Bereitstellen neuer Azure Virtual Desktop-Hostpools** die Option **In Azure bereitstellen** aus. Daraufhin wird der Browser automatisch an das Blatt **Benutzerdefinierte Bereitstellung** im Azure-Portal umgeleitet.
1. Wählen Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die Option **Parameter bearbeiten** aus.
1. Wählen Sie auf dem Blatt **Parameter bearbeiten** die Option **Datei laden**, im Dialogfeld **Öffnen** die Option **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json**, dann **Öffnen** und schließlich **Speichern** aus. 
1. Kehren Sie zum Blatt **Benutzerdefinierte Bereitstellung** zurück, und geben Sie die folgenden Einstellungen an (übernehmen Sie für andere Einstellungen die Standardwerte):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Ressourcengruppe|Der Name einer neuen Ressourcengruppe: **az140-23-RG**|
   |Region|Der Name der Azure-Region, in der Sie im Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)** Azure-VMs bereitgestellt haben, auf denen AD DS-Domänencontroller gehostet werden|
   |Standort|Der Name der Azure-Region, die als Wert für die **Region**-Parameter festgelegt wurde|
   |Standort des Arbeitsbereichs|Der Name der Azure-Region, die als Wert für die **Region**-Parameter festgelegt wurde|
   |Ressourcengruppe im Arbeitsbereich|Keine, da der Wert automatisch auf die Zielressourcengruppe der Bereitstellung festgelegt wird, wenn der Wert NULL ist|
   |Gesamte Anwendungsgruppenreferenz|Keine, da im Zielarbeitsbereich keine Anwendungsgruppen vorhanden sind (kein Arbeitsbereich vorhanden)|
   |VM-Standort|Der Name der Azure-Region, die als Wert für die **Standort**-Parameter festgelegt wurde|
   |Erstellen einer Netzwerksicherheitsgruppe|**false**|
   |ID der Netzwerksicherheitsgruppe|Der Wert des „resourceID“-Parameters der vorhandenen Netzwerksicherheitsgruppe, die Sie in der vorherigen Aufgabe ermittelt haben|
   |Tokenablaufzeit| Der Wert der Tokenablaufzeit, die Sie in der vorherigen Aufgabe ermittelt haben|

   > **Hinweis:** Im Rahmen der Bereitstellung wird ein Pool mit Zuweisungen zu persönlichen Desktops bereitgestellt.

1. Wählen Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die Option **Überprüfen + erstellen** und dann **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. Dies kann etwa 15 Minuten dauern. 

#### <a name="task-3-verify-deployment-of-the-azure-virtual-desktop-host-pool-and-hosts"></a>Aufgabe 3: Überprüfen der Bereitstellung von Azure Virtual Desktop-Hostpools und -Hosts

1. Wählen Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, **Azure Virtual Desktop**, auf dem Blatt **Azure Virtual Desktop** die Option **Hostpools** und auf dem Blatt **Azure Virtual Desktop \| Hostpools** den Eintrag **az140-23-hp2** aus, der den neu bereitgestellten Pool angibt.
1. Klicken Sie auf dem Blatt **az140-23-hp2** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf die Option **Sitzungshosts**. 
1. Vergewissern Sie sich auf dem Blatt **az140-23-hp2 \| Sitzungshosts**, dass die Bereitstellung aus zwei Hosts besteht.
1. Klicken Sie auf dem Blatt **az140-23-hp2 \| Sitzungshosts** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **Anwendungsgruppen**.
1. Vergewissern Sie sich auf dem Blatt **az140-23-hp2 \| Anwendungsgruppen**, dass die Bereitstellung die **Standarddesktop**-Anwendungsgruppe mit dem Namen **az140-23-hp2-DAG** enthält.

#### <a name="task-4-prepare-for-adding-of-hosts-to-the-existing-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>Aufgabe 4: Vorbereiten auf das Hinzufügen von Hosts zum vorhandenen Azure Virtual Desktop-Hostpool mithilfe einer Azure Resource Manager-Vorlage

1. Wechseln Sie von Ihrem Labcomputer zur Remotedesktopsitzung für **az140-dc-vm11**. 
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Token zu generieren, das für den Beitritt neuer Hosts zum Pool benötigt wird, den Sie weiter oben in dieser Übung bereitgestellt haben:

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Wert des Tokens abzurufen und in die Zwischenablage einzufügen:

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **Hinweis:** Notieren Sie den in die Zwischenablage kopierten Wert (indem Sie beispielsweise Editor starten und die Tastenkombination STRG+V drücken, um den Inhalt der Zwischenablage in Editor einzufügen), da Sie ihn in der nächsten Aufgabe benötigen. Achten Sie darauf, dass der verwendete Wert eine einzelne Textzeile ohne Zeilenumbrüche enthält. 

   > **Hinweis:** Ein Registrierungstoken wird für den Beitritt eines Hosts zum Pool benötigt. Der Wert für das Ablaufdatum des Tokens muss zwischen einer Stunde und einem Monat zwischen dem aktuellen Datum und der aktuellen Uhrzeit liegen.

#### <a name="task-5-add-hosts-to-the-existing-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>Aufgabe 5: Hinzufügen von Hosts zum vorhandenen Azure Virtual Desktop-Hostpool mithilfe einer Azure Resource Manager-Vorlage

1. Öffnen Sie auf Ihrem Labcomputer im selben Webbrowserfenster eine weitere Webbrowserregisterkarte, und navigieren Sie zur GitHub Azure RDS-Vorlagenrepositoryseite [ARM-Vorlage zum Hinzufügen von Sitzungshosts zu einem vorhandenen Azure Virtual Desktop-Hostpool](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool). 
1. Wählen Sie auf der Seite **ARM-Vorlage zum Hinzufügen von Sitzungshosts zu einem vorhandenen Azure Virtual Desktop-Hostpool** die Option **In Azure bereitstellen** aus. Daraufhin wird der Browser automatisch an das Blatt **Benutzerdefinierte Bereitstellung** im Azure-Portal umgeleitet.
1. Wählen Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die Option **Parameter bearbeiten** aus.
1. Wählen Sie auf dem Blatt **Parameter bearbeiten** die Option **Datei laden**, im Dialogfeld **Öffnen** die Option **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json**, dann **Öffnen** und schließlich **Speichern** aus. 
1. Kehren Sie zum Blatt **Benutzerdefinierte Bereitstellung** zurück, und geben Sie die folgenden Einstellungen an (übernehmen Sie für andere Einstellungen die Standardwerte):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Ressourcengruppe|**az140-23-RG**|
   |Hostpooltoken|Der Wert des Tokens, das Sie in der vorherigen Aufgabe generiert haben|
   |Standort des Hostpools|Der Name der Azure-Region, in der Sie weiter oben in diesem Lab den Hostpool bereitgestellt haben|
   |Benutzername des VM-Administratorkontos|**student**, @adatum.com darf nicht verwendet werden|
   |Kennwort für das VM-Administratorkonto|**Pa55w.rd1234**|
   |VM-Standort|Der Name der Azure-Region, die als Wert für die **Hostpoolstandort**-Parameter festgelegt wurde|
   |Erstellen einer Netzwerksicherheitsgruppe|**false**|
   |ID der Netzwerksicherheitsgruppe|Der Wert des „resourceID“-Parameters der vorhandenen Netzwerksicherheitsgruppe, die Sie in der vorherigen Aufgabe ermittelt haben|

1. Wählen Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die Option **Überprüfen + erstellen** und dann **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. Dies kann etwa fünf Minuten dauern.

#### <a name="task-6-verify-changes-to-the-azure-virtual-desktop-host-pool"></a>Aufgabe 6: Überprüfen von Änderungen am Azure Virtual Desktop-Hostpool

1. Wählen Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, **Virtuelle Computer** aus. Beachten Sie, dass die Liste auf dem Blatt **Virtuelle Computer** einen zusätzlichen virtuellen Computer mit dem Namen **az140-23-p2-2** enthält.
1. Wechseln Sie von Ihrem Labcomputer zur Remotedesktopsitzung für **az140-dc-vm11**. 
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um zu überprüfen, ob der dritte Host der AD DS-Domäne **adatum.com** erfolgreich beigetreten ist:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. Kehren Sie auf dem Labcomputer zu dem Webbrowser zurück, in dem das Azure-Portal angezeigt wird, suchen Sie nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Hostpools** aus und auf dem Blatt **Azure Virtual Desktop \| Hostpools** den Eintrag **az140-23-hp2**, der den neu geänderten Pool darstellt.
1. Vergewissern Sie sich, dass auf dem Blatt **az140-23-hp2** im Abschnitt **Essentials** der **Hostpooltyp** auf **Persönlich** und der **Zuweisungstyp** auf **Automatisch** festgelegt ist.
1. Klicken Sie auf dem Blatt **az140-23-hp2** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf die Option **Sitzungshosts**. 
1. Vergewissern Sie sich auf dem Blatt **az140-23-hp2 \| Sitzungshosts**, dass die Bereitstellung aus drei Hosts besteht. 

#### <a name="task-7-manage-personal-desktop-assignments-in-the-azure-virtual-desktop-host-pool"></a>Aufgabe 7: Verwalten von Zuweisungen persönlicher Desktops im Azure Virtual Desktop-Hostpool

1. Wählen Sie auf dem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, auf dem Blatt **az140-23-hp2 \| Sitzungshosts** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** die Option **Anwendungsgruppen** aus. 
1. Wählen Sie auf dem Blatt **az140-23-hp2 \| Anwendungsgruppen** in der Liste der Anwendungsgruppen den Eintrag **az140-23-hp2-DAG** aus.
1. Wählen Sie auf dem Blatt **az140-23-hp2-DAG** im vertikalen Menü auf der linken Seite die Option **Zuweisungen** aus. 
1. Wählen Sie auf dem Blatt **az140-23-hp2-DAG \| Zuweisungen** die Option **+ Hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Option **az140-wvd-personal** aus, und klicken Sie auf **Auswählen**.

   > **Hinweis:** Sehen wir uns nun die Erfahrung eines Benutzers an, der eine Verbindung mit dem Azure Virtual Desktop-Hostpool herstellt.

1. Suchen Sie auf Ihrem Labcomputer in dem Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-cl-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen an, und wählen **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Klicken Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11** auf **Starten**, und wählen Sie im Menü **Starten** die Client-App **Remotedesktop** aus.
2. Klicken Sie im Remotedesktopfenster auf das Symbol mit den Auslassungspunkte in der oberen rechten Ecke. Klicken Sie im Dropdownmenü auf **Unsubscribe** (Abbestellen) und dann auf **Weiter**, wenn Sie zur Bestätigung aufgefordert werden.
3. Klicken Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11** im Remotedesktopfenster auf der Seite **Los geht's!** auf **Abonnieren**.
4. Wählen Sie im Fenster des **Remotedesktopclients** die Option **Abonnieren** aus. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Anmeldeinformationen für **aduser7** an, indem Sie den entsprechenden userPrincipalName und **Pa55w.rd1234** als Kennwort angeben.

   > **Hinweis:** Wählen Sie alternativ im Fenster des **Remotedesktopclients** die Option **Mit URL abonnieren** aus. Geben Sie im Bereich **Arbeitsbereich abonnieren** unter **E-Mail oder Arbeitsbereichs-URL** die URL **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery** ein, wählen Sie **Weiter** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aduser7** an. (Verwenden Sie das userPrincipalName-Attribut als Benutzername und das beim Erstellen dieses Kontos festgelegte Kennwort.) 

1. Doppelklicken Sie auf der Seite **Remote Desktop** auf das Symbol **SessionDesktop**. Wenn Sie aufgefordert werden, Anmeldeinformationen einzugeben, geben Sie dasselbe Kennwort noch einmal ein. Aktivieren Sie das Kontrollkästchen **Speichern**, und klicken Sie auf **OK**.
1. Deaktivieren Sie im Fenster **Bei all Ihren Apps angemeldet bleiben** das Kontrollkästchen für **Verwaltung meines Geräts durch meine Organisation zulassen**, und aktivieren Sie **Nein, nur bei dieser App anmelden**. 
1. Vergewissern Sie sich, dass **aduser7** über Remotedesktop erfolgreich bei einem Host angemeldet wurde.
1. Klicken Sie innerhalb der Remotedesktopsitzung für einen der Hosts als **aduser7** mit der rechten Maustaste auf **Starten**. Wählen Sie im Kontextmenü die Option **Herunterfahren oder abmelden** aus, und klicken Sie im Untermenü auf **Abmelden**.

   > **Hinweis:** Wechseln wir nun bei der Zuweisung persönlicher Desktops vom direkten Modus in den automatischen Modus. 

1. Wechseln Sie auf Ihrem Labcomputer zu dem Webbrowser, in dem das Azure-Portal angezeigt wird. Klicken Sie auf dem Blatt **az140-23-hp2-DAG \| Zuweisungen** in der Informationsleiste direkt über der Liste mit Zuweisungen auf den Link **Assign VM** (VM zuweisen). Daraufhin werden Sie zum Blatt **az140-23-hp2 \| Sitzungshosts** umgeleitet. 
1. Vergewissern Sie sich auf dem Blatt **az140-23-hp2 \| Sitzungshosts**, dass bei einem der Hosts in der Spalte **Zugewiesener Benutzer** **aduser7** angezeigt wird.

   > **Hinweis:** Dies ist zu erwarten, da der Hostpool für die automatische Zuweisung konfiguriert wurde.

1. Öffnen Sie auf dem Labcomputer in dem Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um in den direkten Zuweisungsmodus zu wechseln:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. Navigieren Sie auf dem Labcomputer in dem Webbrowserfenster, in dem das Azure-Portal angezeigt wird, zum Blatt für den Hostpool **az140-23-hp2**. Vergewissern Sie sich, dass im Abschnitt **Essentials** der **Hostpooltyp** auf **Persönlich** und der **Zuweisungstyp** auf **Direkt** festgelegt ist.
1. Kehren Sie zur Remotedesktopsitzung für **az140-cl-vm11** zurück. Klicken Sie im Fenster **Remotedesktop** auf das Symbol mit den Auslassungspunkte in der oberen rechten Ecke. Klicken Sie im Dropdownmenü auf **Unsubscribe** (Abbestellen) und dann auf **Weiter**, wenn Sie zur Bestätigung aufgefordert werden.
1. Klicken Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11** im Fenster **Remotedesktop** auf der Seite **Los geht's!** auf **Abonnieren**.
1. Wenn Sie aufgefordert werden, sich anzumelden, klicken Sie im Bereich **Konto auswählen** auf **Anderes Konto verwenden**, und melden Sie sich bei entsprechender Aufforderung mit dem Benutzerprinzipalnamen des Benutzerkontos **aduser8** mit dem Kennwort an, das Sie beim Erstellen dieses Kontos festgelegt haben.
1. Deaktivieren Sie im Fenster **Bei all Ihren Apps angemeldet bleiben** das Kontrollkästchen für **Verwaltung meines Geräts durch meine Organisation zulassen**, und aktivieren Sie **Nein, nur bei dieser App anmelden**. 
1. Doppelklicken Sie auf der Seite **Remotedesktop** auf das Symbol **SessionDesktop**. Vergewissern Sie sich, dass eine Fehlermeldung mit folgendem Inhalt angezeigt wird: **We couldn't connect because there are currently no available resources. Try again later or contact tech support for help if this keeps happening** (Es konnte keine Verbindung hergestellt werden, da derzeit keine Ressourcen verfügbar sind. Versuchen Sie es später noch einmal, oder wenden Sie sich an den technischen Support, wenn dieses Problem weiterhin auftritt). Klicken Sie auf **OK**.

   > **Hinweis:** Dies ist zu erwarten, da der Hostpool für die direkte Zuweisung konfiguriert und **aduser8** kein Host zugewiesen wurde.

1. Wechseln Sie auf Ihrem Labcomputer zu dem Webbrowser, in dem das Azure-Portal angezeigt wird. Wählen Sie auf dem Blatt **az140-23-hp2 \| Sitzungshosts** in der Spalte **Zugewiesener Benutzer** neben einem der beiden restlichen nicht zugewiesenen Hosts den Link **(Zuweisen)** aus.
1. Wählen Sie unter **Benutzer zuweisen** den Benutzer **aduser8** aus, klicken Sie auf **Auswählen**, und wenn Sie zur Bestätigung aufgefordert werden, auf **OK**.
1. Kehren Sie zur Remotedesktopsitzung für **az140-cl-vm11** zurück. Doppelklicken Sie im Fenster **Remotedesktop** auf das Symbol **SessionDesktop**. Wenn Sie aufgefordert werden, das Kennwort einzugeben, geben Sie das Kennwort ein, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben. Klicken Sie auf **OK**, und vergewissern Sie sich, dass Sie sich beim zugewiesenen Host anmelden können.

### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>Übung 2: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der Azure-VMs, die im Lab bereitgestellt sind, und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung werden Sie die Zuordnungen der in diesem Labor bereitgestellten Azure-VMs aufheben, um die entsprechenden Computegebühren zu minimieren.

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>Aufgabe 1: Aufheben der Zuordnung von Azure-VMs, die im Lab bereitgestellt sind

1. Wechseln Sie zum Laborcomputer, und öffnen Sie im Webbrowser, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird, wie über Parameter „-NoWait“ festgelegt, asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet und ihre Zuordnungen aufgehoben werden.
