---
lab:
  title: "Lab: Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD\_DS)"
  module: 'Module 2: Implement a AVD Infrastructure'
---

# <a name="lab---deploy-host-pools-and-session-hosts-by-using-the-azure-portal-ad-ds"></a>Lab: Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lab-Abhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden.
- Ein Microsoft-Konto oder ein Azure AD-Konto mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle„Globaler Administrator“ im Azure AD-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)** .

## <a name="estimated-time"></a>Geschätzte Dauer

60 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen Hostpools und Sitzungshosts in einer Umgebung von Active Directory Domain Services (AD DS) erstellen und konfigurieren.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Implementieren einer Azure Virtual Desktop-Umgebung in einer AD DS-Domäne
- Überprüfen einer Azure Virtual Desktop-Umgebung in einer AD DS-Domäne

## <a name="lab-files"></a>Labdateien

- Keine

## <a name="instructions"></a>Anweisungen

### <a name="exercise-1-implement-an-azure-virtual-desktop-environment-in-an-ad-ds-domain"></a>Übung 1: Implementieren einer Azure Virtual Desktop-Umgebung in einer AD DS-Domäne
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten einer AD DS-Domäne und des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools
1. Bereitstellen eines Azure Virtual Desktop-Hostpools
1. Verwalten der Sitzungshosts der Azure Virtual Desktop-Hostpools
1. Konfigurieren von Azure Virtual Desktop-Anwendungsgruppen
1. Konfigurieren von Azure Virtual Desktop-Arbeitsbereichen

#### <a name="task-1-prepare-ad-ds-domain-and-the-azure-subscription-for-deployment-of-an-azure-virtual-desktop-host-pool"></a>Aufgabe 1: Vorbereiten einer AD DS-Domäne und des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal]( ), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie dann auf dem Blatt **Virtuelle Computer** auf **az140-dc-vm11**.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** die Option **Verbinden**, im Dropdownmenü **Bastion**, auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator.
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Organisationseinheit zu erstellen, in der die Computerobjekte der Azure Virtual Desktop-Hosts gehostet werden:

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um sich bei Ihrem Azure-Abonnement anzumelden:

   ```powershell
   Connect-AzAccount
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Benutzerprinzipalnamen des Kontos **aduser1** zu ermitteln:

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **Hinweis:** Notieren Sie sich den in diesem Schritt ermittelten Benutzerprinzipalnamen. Sie benötigen ihn später in diesem Lab.

1. Führen Sie über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Ressourcenanbieter **Microsoft.DesktopVirtualization** zu registrieren:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mithilfe der Azure AD-Anmeldeinformationen des Benutzerkontos mit der Rolle „Besitzer“ in dem Abonnement an, das Sie in diesem Lab verwenden.
1. Verwenden Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** oben auf der Seite des Azure-Portals, um nach **Virtuelle Netzwerke** zu suchen und dorthin zu navigieren, und wählen Sie auf dem Blatt **Virtuelle Netzwerke** die Option **az140-adds-vnet11** aus. 
1. Wählen Sie auf dem Blatt **az140-adds-vnet11** die Option **Subnetze** und auf dem Blatt **Subnetze** die Option **+ Subnetz** aus. Geben Sie auf dem Blatt **Subnetz hinzufügen** die folgenden Einstellungen an (übernehmen Sie bei allen anderen Einstellungen die Standardwerte), und klicken Sie auf **Speichern**:

   |Einstellung|Wert|
   |---|---|
   |Name|**hp1-Subnet**|
   |Subnetzadressbereich|**10.0.1.0/24**|

#### <a name="task-2-deploy-an-azure-virtual-desktop-host-pool"></a>Aufgabe 2: Bereitstellen eines Azure Virtual Desktop-Hostpools

1. Suchen Sie in der Remotedesktopsitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Hostpools** und auf dem Blatt **Azure Virtual Desktop \| Hostpools** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Virtuelle Computer >** aus (behalten Sie für die anderen Einstellungen die Standardwerte bei):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|Der Name einer neuen Ressourcengruppe: **az140-21-RG**|
   |Hostpoolname|**az140-21-hp1**|
   |Lastenausgleichsalgorithmus|**Breitensuche**|
   |Maximale Anzahl von Sitzungen|**50**|
   |Standort|Der Name der Azure-Region, in der Sie Ressourcen in der ersten Übung dieses Labs bereitgestellt haben, oder einer Region in deren Nähe |
   |Überprüfungsumgebung|**Nein**|
   |Hostpooltyp|**In einem Pool zusammengefasst**|


1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Virtuelle Computer** die folgenden Einstellungen an, und wählen Sie **Weiter: Arbeitsbereiche >** aus (behalten Sie für die anderen Einstellungen die Standardwerte bei):

   |Einstellung|Wert|
   |---|---|
   |Virtuelle Azure-Computer hinzufügen|**Ja**|
   |Resource group|**Standardmäßig identisch mit der Ressourcengruppe des Hostpools**|
   |Namenspräfix|**az140-21-p1**|
   |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie in der ersten Übung dieses Labs Ressourcen bereitgestellt haben|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Imagetyp|**Galerie**|
   |Bild|**Windows 10 Enterprise (mehrere Sitzungen), Version 20H2 + Microsoft 365-Apps**|
   |Größe des virtuellen Computers|**Standard D2s v3**|
   |Number of VMs (Anzahl von VMs)|**2**|
   |Typ des Betriebssystemdatenträgers|**SSD Standard**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnet|**hp1-Subnet (10.0.1.0/24)**|
   |Netzwerksicherheitsgruppe|**Grundlegend**|
   |Öffentliche Eingangsports|**Nein**|
   |Wählen Sie das gewünschte Verzeichnis für die Einbindung aus.|**Active Directory**|
   |UPN "AD-Domänenmitgliedschaft"|**student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|
   |Domäne oder Einheit angeben|**Ja**|
   |Domäne für den Beitritt|**adatum.com**|
   |Pfad der Organisationseinheit|**OU=WVDInfra,DC=adatum,DC=com**|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|
   |Kennwort bestätigen|**Pa55w.rd1234**|

1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Arbeitsbereich** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Desktop-App-Gruppe registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Hostpool erstellen** die Option **Erstellen** aus.

   > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa zehn Minuten dauern.

#### <a name="task-3-manage-the-azure-virtual-desktop-host-pool-session-hosts"></a>Aufgabe 3: Verwalten der Sitzungshosts der Azure Virtual Desktop-Hostpools

1. Suchen Sie in der Remotedesktopsitzung für **az140-dc-vm11** im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** auf der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Hostpools** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools den Eintrag **az140-21-hp1** aus.
1. Wählen Sie auf dem Blatt **az140-21-hp1** auf der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Sitzungshosts** aus, und überprüfen Sie, ob der Pool aus zwei Hosts besteht. 
1. Wählen Sie auf dem Blatt **az140-21-hp1 \| Sitzungshosts** die Option **+ Hinzufügen** aus.
1. Überprüfen Sie auf dem Blatt **VMs zu Hostpool hinzufügen** auf der Registerkarte **Grundeinstellungen** die vorkonfigurierten Einstellungen, und wählen Sie **Weiter: Virtuelle Computer** aus.
1. Geben Sie auf der Registerkarte **Virtuelle Computer** des Blatts **VMs zu Hostpool hinzufügen** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Resource group|**az140-21-RG**|
   |Namenspräfix|**az140-21-p1**|
   |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie die ersten zwei Sitzungshost-VMs bereitgestellt haben|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Imagetyp|**Galerie**|
   |Bild|**Windows 10 Enterprise (mehrere Sitzungen), Version 2004 + Microsoft 365 Apps**|
   |Number of VMs (Anzahl von VMs)|**1**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnet|**hp1-Subnet (10.0.1.0/24)**|
   |Netzwerksicherheitsgruppe|**Grundlegend**|
   |Öffentliche Eingangsports|**Nein**|
   |UPN "AD-Domänenmitgliedschaft"|**student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|
   |Domäne oder Einheit angeben|**Ja**|
   |Domäne für den Beitritt|**adatum.com**|
   |Pfad der Organisationseinheit|**OU=WVDInfra,DC=adatum,DC=com**|   
   |Benutzername des Administratorkontos für den virtuellen Computer|**student**|
   |Kennwort des Administratorkontos für den virtuellen Computer|**Pa55w.rd1234**|

   > **Hinweis:** Wie Sie wahrscheinlich bereits festgestellt haben, ist es möglich, das Image und das Präfix der VMs zu ändern, wenn Sie Sitzungshosts zum vorhandenen Pool hinzufügen. Im Allgemeinen wird dies nur empfohlen, wenn Sie alle VMs im Pool ersetzen möchten. 

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **VMs zu Hostpool hinzufügen** die Option **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. Die Bereitstellung kann ungefähr fünf Minuten dauern. 

#### <a name="task-4-configure-azure-virtual-desktop-application-groups"></a>Aufgabe 4: Konfigurieren von Azure Virtual Desktop-Anwendungsgruppen

1. Suchen Sie in der Remotedesktopsitzung für **az140-dc-vm11** im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Anwendungsgruppen** aus.
1. Auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** wird die vorhandene, automatisch generierte Desktopanwendungsgruppe **az140-21-hp1-DAG** angezeigt. Wählen Sie diese Gruppe aus. 
1. Wählen Sie auf dem Blatt **az140-21-hp1-DAG** die Option **Zuweisungen** aus.
1. Wählen Sie auf dem Blatt **az140-21-hp1-DAG \| Zuweisungen** die Option **+ Hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Option **az140-wvd-pooled** aus, und klicken Sie auf **Auswählen**.
1. Navigieren Sie zurück zum Blatt **Azure Virtual Desktop \| Anwendungssicherheitsgruppen**, und wählen Sie **+ Erstellen** aus. 
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Anwendungsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-21-RG**|
   |Hostpool|**az140-21-hp1**|
   |Anwendungsgruppentyp|**RemoteApp**|
   |Name der Anwendungsgruppe|**az140-21-hp1-Office365-RAG**|

1. Wählen Sie auf der Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Startmenü**|
   |Application|**Word**|
   |BESCHREIBUNG|**Microsoft Word**|
   |Befehlszeile erforderlich|**Nein**|

1. Navigieren Sie zurück zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen**, und wählen Sie die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Startmenü**|
   |Application|**Excel**|
   |BESCHREIBUNG|**Microsoft Excel**|
   |Befehlszeile erforderlich|**Nein**|

1. Navigieren Sie zurück zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen**, und wählen Sie die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Startmenü**|
   |Application|**PowerPoint**|
   |BESCHREIBUNG|**Microsoft PowerPoint**|
   |Befehlszeile erforderlich|**Nein**|

1. Wechseln Sie zurück zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen**, und wählen Sie **Weiter: Zuweisungen >** aus.
1. Wählen Sie auf der Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Azure AD-Benutzer oder -Benutzergruppen hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Option **az140-wvd-remote-app** aus, und klicken Sie auf **Auswählen**.
1. Wechseln Sie zurück zur Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen**, und wählen Sie **Weiter: Arbeitsbereich >** .
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die folgende Einstellung an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendung registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Anwendungsgruppe erstellen** die Option **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Anwendungsgruppe erstellt wurde. Das sollte weniger als eine Minute dauern. 

   > **Hinweis:** Als Nächstes erstellen Sie eine Anwendungsgruppe, die auf dem Dateipfad als Anwendungsquelle basiert.

1. Suchen Sie in der Remotedesktopsitzung für **az140-dc-vm11** nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Anwendungsgruppen** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** die Option **+ Erstellen** aus. 
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Anwendungsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-21-RG**|
   |Hostpool|**az140-21-hp1**|
   |Anwendungsgruppentyp|**RemoteApp**|
   |Name der Anwendungsgruppe|**az140-21-hp1-Utilities-RAG**|

1. Wählen Sie auf der Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Dateipfad**|
   |Anwendungspfad|**C:\Windows\system32\cmd.exe**|
   |Anwendungsname|**Eingabeaufforderung**|
   |Anzeigename|**Eingabeaufforderung**|
   |Symbolpfad|**C:\Windows\system32\cmd.exe**|
   |Symbolindex|**0**|
   |BESCHREIBUNG|**Windows-Eingabeaufforderung**|
   |Befehlszeile erforderlich|**Nein**|

1. Wechseln Sie zurück zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen**, und wählen Sie **Weiter: Zuweisungen >** aus.
1. Wählen Sie auf der Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Azure AD-Benutzer oder -Benutzergruppen hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Option **az140-wvd-remote-app** und **az140-wvd-admins** aus, und klicken Sie auf **Auswählen**.
1. Wechseln Sie zurück zur Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen**, und wählen Sie **Weiter: Arbeitsbereich >** .
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die folgende Einstellung an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendung registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Anwendungsgruppe erstellen** die Option **Erstellen** aus.

#### <a name="task-5-configure-azure-virtual-desktop-workspaces"></a>Aufgabe 5: Konfigurieren von Azure Virtual Desktop-Arbeitsbereichen

1. Suchen Sie in der Remotedesktopsitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Arbeitsbereiche** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Arbeitsbereiche** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungsgruppen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-21-RG**|
   |Arbeitsbereichname|**az140-21-ws1**|
   |Angezeigter Name|**az140-21-ws1**|
   |Standort|Der Name der Azure-Region, in der Sie Ressourcen in der ersten Übung dieses Labs bereitgestellt haben, oder einer Region in deren Nähe|

1. Geben Sie auf der Registerkarte **Anwendungsgruppen** des Blatts **Arbeitsbereich erstellen** die folgenden Einstellungen an:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsgruppen registrieren|**Ja**|

1. Wählen Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die Option **Anwendungsgruppen registrieren** aus:
1. Wählen Sie auf dem Blatt **Anwendungsgruppen hinzufügen** das Plussymbol neben den Einträgen **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** und **az140-21-hp1-Utilities-RAG** aus, und klicken Sie auf **Auswählen**. 
1. Wählen Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Anwendungsgruppen** die Option **Überprüfen + erstellen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Arbeitsbereich erstellen** die Option **Erstellen** aus.

### <a name="exercise-2-validate-azure-virtual-desktop-environment"></a>Übung 2: Überprüfen der Azure Virtual Desktop-Umgebung
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Installieren des Microsoft-Remotedesktopclients (MSRDC) auf einem Windows 10-Computer
1. Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs
1. Testen von Azure Virtual Desktop-Apps

#### <a name="task-1-install-microsoft-remote-desktop-client-msrdc-on-a-windows-10-computer"></a>Aufgabe 1: Installieren des Microsoft-Remotedesktopclients (MSRDC) auf einem Windows 10-Computer

1. Suchen Sie in der Remotedesktopsitzung für **az140-dc-vm11** im Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Scrollen Sie auf dem Blatt **az140-cl-vm11** nach unten zum Abschnitt **Vorgänge**, und wählen Sie **Befehl ausführen** aus. 
1. Wählen Sie auf dem Blatt **az140-cl-vm11 \| Befehl ausführen** die Option **EnableRemotePS** und dann **Ausführen** aus. 

   > **Hinweis:** Warten Sie, bis der Befehl ausgeführt wurde, bevor Sie mit dem nächsten Schritt fortfahren. Dies kann etwa eine Minute dauern. Sie erhalten möglicherweise Fehlermeldungen mit rotem Text, die sich auf das verwendete öffentliche Profil und nicht auf das Domänenprofil beziehen. Sie können diese Meldungen ignorieren und mit dem nächsten Schritt fortfahren.

1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um alle Mitglieder von **ADATUM\\az140-wvd-users** der lokalen Gruppe **Remotedesktopbenutzer** auf dem virtuellen Azure-Computer **az140-cl-vm11** unter Windows 10 hinzuzufügen, den Sie im Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)** bereitgestellt haben.

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. Wechseln Sie zu Ihrem Labcomputer, suchen Sie im Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-cl-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen an, und wählen **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11** Microsoft Edge, und navigieren Sie zur [Downloadseite für den Windows-Desktopclient](https://go.microsoft.com/fwlink/?linkid=2068602). Wählen Sie bei entsprechender Aufforderung **Ausführen** aus, um die Installation zu starten. Wählen Sie auf der Seite **Installationsbereich** des Assistenten **Remotedesktopsetup** die Option **Für alle Benutzer auf diesem Computer installieren** aus, und klicken Sie auf **Installieren**. Wenn Sie von der Benutzerkontensteuerung zur Eingabe von Administratoranmeldeinformationen aufgefordert werden, authentifizieren Sie sich mit dem Benutzernamen **ADATUM\\Student** und dem Kennwort **Pa55w.rd1234**.
1. Vergewissern Sie sich nach Abschluss der Installation, dass das Kontrollkästchen **Remotedesktop starten, wenn das Setup abgeschlossen ist** aktiviert ist, und klicken Sie auf **Fertig stellen**, um den Remotedesktopclient zu starten.

#### <a name="task-2-subscribe-to-a-azure-virtual-desktop-workspace"></a>Aufgabe 2: Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs

1. Wählen Sie im Fenster des **Remotedesktopclients** die Option **Abonnieren** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aduser1** an. Geben Sie dabei den weiter oben in diesem Lab ermittelten Benutzerprinzipalnamen (userPrincipalName) und das beim Erstellen dieses Kontos festgelegte Kennwort an.

   > **Hinweis:** Wählen Sie alternativ im Fenster des **Remotedesktopclients** die Option **Mit URL abonnieren** aus. Geben Sie im Bereich **Arbeitsbereich abonnieren** unter **E-Mail oder Arbeitsbereichs-URL** die URL **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery** ein, wählen Sie **Weiter** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aduser1** an. (Verwenden Sie das userPrincipalName-Attribut als Benutzername und das beim Erstellen dieses Kontos festgelegte Kennwort.) 

1. Deaktivieren Sie im Fenster **Bei all Ihren Apps angemeldet bleiben** das Kontrollkästchen für **Verwaltung meines Geräts durch meine Organisation zulassen**, und aktivieren Sie **Nein, nur bei dieser App anmelden**. 
1. Vergewissern Sie sich, dass auf der Seite **Remotedesktop** die Liste der Anwendungen angezeigt wird, die in den Anwendungsgruppen enthalten sind, die im Arbeitsbereich veröffentlicht wurden und dem Benutzerkonto **aduser1** über ihre Gruppenmitgliedschaft zugeordnet sind. 

#### <a name="task-3-test-azure-virtual-desktop-apps"></a>Aufgabe 3: Testen von Azure Virtual Desktop-Apps

1. Doppelklicken Sie in der Remotedesktopsitzung für **az140-cl-vm11** im Fenster des **Remotedesktopclients** in der Liste der Anwendungen auf **Eingabeaufforderung**, und vergewissern Sie sich, dass ein **Eingabeaufforderungsfenster** geöffnet wird. Geben Sei bei der Aufforderung zur Authentifizierung das Kennwort ein, das Sie beim Erstellen des Benutzerkontos **aduser1** festgelegt haben, aktivieren Sie das Kontrollkästchen **Anmeldedaten speichern**, und wählen Sie **OK** aus.

   > **Hinweis:** Es kann anfangs einige Minuten dauern, bis die Anwendung gestartet wird, danach sollte der Anwendungsstart aber viel schneller erfolgen.

1. Geben Sie an der Eingabeaufforderung **hostname** ein, und drücken Sie die **EINGABETASTE**, um den Namen des Computers anzuzeigen, auf dem die Eingabeaufforderung ausgeführt wird.

   > **Hinweis:** Vergewissern Sie sich, dass der angezeigte Name **az140-21-p1-0**, **az140-21-p1-1** oder **az140-21-p1-2** und nicht **az140-cl-vm11** lautet.

1. Geben Sie an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.
1. Doppelklicken Sie in der Remotedesktopsitzung für **az140-cl-vm11** im Fenster des **Remotedesktopclients** in der Liste der Anwendungen auf **SessionDesktop**, und vergewissern Sie sich, dass eine Remotedesktopsitzung gestartet wird. 
1. Klicken Sie innerhalb der **Standarddesktopsitzung** mit der rechten Maustaste auf **Start**, und wählen Sie **Ausführen** aus. Geben Sie **cmd** in das Textfeld **Öffnen** des Dialogfelds **Ausführen** ein, und wählen Sie **OK** aus. 
1. Geben Sie in der **Standarddesktopsitzung** an der Eingabeaufforderung **hostname** ein, und drücken Sie die **EINGABETASTE**, um den Namen des Computers anzuzeigen, auf dem die Remotedesktopsitzung ausgeführt wird.
1. Vergewissern Sie sich, dass der angezeigte Name **az140-21-p1-0**, **az140-21-p1-1** oder **az140-21-p1-2** lautet.

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>Übung 3: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der Azure-VMs, die im Lab bereitgestellt sind, und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung werden Sie die Zuordnungen der in diesem Labor bereitgestellten Azure-VMs aufheben, um die entsprechenden Computegebühren zu minimieren.

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>Aufgabe 1: Aufheben der Zuordnung von Azure-VMs, die im Lab bereitgestellt sind

1. Wechseln Sie zum Laborcomputer, und öffnen Sie im Webbrowser, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird, wie über Parameter „-NoWait“ festgelegt, asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet und ihre Zuordnungen aufgehoben werden.
