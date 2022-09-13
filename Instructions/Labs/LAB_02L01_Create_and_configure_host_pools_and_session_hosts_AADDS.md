---
lab:
  title: "Lab: Erstellen und Konfigurieren von Hostpools und Sitzungshosts (Azure AD\_DS)"
  module: 'Module 2: Implement a AVD Infrastructure'
---

# <a name="lab---create-and-configure-host-pools-and-session-hosts-azure-ad-ds"></a>Lab: Erstellen und Konfigurieren von Hostpools und Sitzungshosts (Azure AD DS)
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder Azure AD-Konto mit der Rolle „Globaler Administrator“ im Azure AD-Mandanten, das dem Azure-Abonnement zugeordnet ist und im Azure-Abonnement die Rolle „Besitzer“ oder „Mitwirkender“ innehat.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (Azure AD DS)**

## <a name="estimated-time"></a>Geschätzte Dauer

60 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen Hostpools und Sitzungshosts in einer Azure AD DS-Umgebung (Azure Active Directory Domain Services) erstellen und konfigurieren.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Konfigurieren einer Azure Virtual Desktop-Umgebung in einer Azure AD DS-Domäne 
- Überprüfen einer Azure Virtual Desktop-Umgebung in einer Azure AD DS-Domäne 

## <a name="lab-files"></a>Labdateien

- Keine 

## <a name="instructions"></a>Anweisungen

### <a name="exercise-1-configure-an-azure-virtual-desktop-environment"></a>Übung 1: Konfigurieren einer Azure Virtual Desktop-Umgebung
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten einer AD DS-Domäne und des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools
1. Bereitstellen eines Azure Virtual Desktop-Hostpools
1. Konfigurieren von Azure Virtual Desktop-Anwendungsgruppen
1. Konfigurieren von Azure Virtual Desktop-Arbeitsbereichen

#### <a name="task-1-prepare-ad-ds-domain-and-the-azure-subscription-for-deployment-of-an-azure-virtual-desktop-host-pool"></a>Aufgabe 1: Vorbereiten einer AD DS-Domäne und des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11a** aus. Daraufhin wird das Blatt **az140-cl-vm11a** geöffnet.
1. Wählen Sie auf dem Blatt **az140-cl-vm11a** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11a \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen an, und wählen **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**aadadmin1@adatum.com**|
   |Kennwort|Im vorherigen Lab definiertes Kennwort|

1. Starten Sie im Remotedesktop für den virtuellen Azure-Computer **az140-cl-vm11a** Microsoft Edge, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Geben Sie dabei den Benutzernamen des Benutzerkontos **aadadmin1** mit dem Kennwort an, das Sie bei der Kontoerstellung festgelegt haben.

   >**Hinweis:** Sie können das UPN-Attribut (User Principal Name, Benutzerprinzipalname) des Kontos **aadadmin1** ermitteln, indem Sie sich in der Konsole „Active Directory-Benutzer und -Computer“ das zugehörige Eigenschaftendialogfeld ansehen oder indem Sie zu Ihrem Labcomputer zurückwechseln und sich dessen Eigenschaften im Azure-Portal auf dem Azure AD-Mandantenblatt ansehen.

1. Öffnen Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** in der Microsoft Edge-Instanz, in der das Azure-Portal angezeigt wird, im Bereich **Cloud Shell** eine PowerShell-Sitzung, und führen Sie Folgendes aus, um den Ressourcenanbieter **Microsoft.DesktopVirtualization** zu registrieren:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Suchen Sie in der Remotedesktopsitzung für **az140-cl-vm11a** in der Microsoft Edge-Instanz, in der das Azure-Portal angezeigt wird, nach **Virtuelle Netzwerke**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Netzwerke** den Eintrag **az140-aadds-vnet11a** aus. 
1. Wählen Sie auf dem Blatt **az140-aadds-vnet11a** die Option **Subnetze** und auf dem Blatt **Subnetze** die Option **+ Subnetz** aus. Geben Sie auf dem Blatt **Subnetz hinzufügen** im Textfeld **Name** den Namen **hp1-Subnet** ein, übernehmen Sie bei allen anderen Einstellungen die Standardwerte, und wählen Sie anschließend **Speichern** aus. 

#### <a name="task-2-deploy-an-azure-virtual-desktop-host-pool"></a>Aufgabe 2: Bereitstellen eines Azure Virtual Desktop-Hostpools

1. Suchen Sie in der Remotedesktopsitzung für **az140-cl-vm11a** in dem Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** die Option **Hostpools** und auf dem Blatt **Azure Virtual Desktop \| Hostpools** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Virtuelle Computer >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|Der Name einer neuen Ressourcengruppe: **az140-21a-RG**|
   |Hostpoolname|**az140-21a-hp1**|
   |Standort|Der Name der Azure-Region, in der Sie weiter oben in diesem Lab die Azure AD DS-Instanz bereitgestellt haben|
   |Überprüfungsumgebung|**Nein**|
   |Hostpooltyp|**In einem Pool zusammengefasst**|
   |Maximale Anzahl von Sitzungen|**10**|
   |Lastenausgleichsalgorithmus|**Breitensuche**|

1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Virtuelle Computer** die folgenden Einstellungen an, übernehmen Sie für die anderen Einstellungen die Standardwerte, und wählen Sie anschließend **Weiter: Arbeitsbereich >** aus. Ersetzen Sie den Platzhalter *<Azure_AD_domain_name>* durch den Namen des Azure AD-Mandanten, der dem Abonnement zugeordnet ist, in dem Sie die Azure AD DS-Instanz bereitgestellt haben, und ersetzen Sie den Platzhalter `<password>` durch das Kennwort, das Sie bei der Erstellung des Kontos „aadadmin1“ festgelegt haben:

   > **Hinweis:** Merken Sie sich das verwendete Kennwort. Es wird später in diesem Lab sowie in nachfolgenden Labs benötigt.

   |Einstellung|Wert|
   |---|---|
   |Virtuelle Computer hinzufügen|**Ja**|
   |Resource group|**Standardmäßig identisch mit der Ressourcengruppe des Hostpools**|
   |Namenspräfix|**az140-21-p1**|
   |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie in der ersten Übung dieses Labs Ressourcen bereitgestellt haben|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Sicherheitstyp|**Standard**
   |Imagetyp|**Galerie**|
   |Bild|**Windows 10 Enterprise (mehrere Sitzungen), Version 2004 + Microsoft 365 Apps**|
   |Größe des virtuellen Computers|**Standard D2s v3**|
   |Number of VMs (Anzahl von VMs)|**2**|
   |Typ des Betriebssystemdatenträgers|**SSD Standard**|
   |Virtuelles Netzwerk|**az140-aadds-vnet11a**|
   |Subnet|**hp1-Subnet (10.10.1.0/24)**|
   |Netzwerksicherheitsgruppe|**Grundlegend**|
   |Öffentliche Eingangsports|**Nein**|
   |Wählen Sie das gewünschte Verzeichnis für die Einbindung aus.|**Active Directory**|
   |UPN "AD-Domänenmitgliedschaft"|**aadadmin1@adatum.com**|
   |Kennwort|Verwenden Sie das Kennwort für „aadadmin1“.|
   |Domäne oder Einheit angeben|**Ja**|
   |Domäne für den Beitritt|**adatum.com**|
   |Pfad der Organisationseinheit|**OU=AADDC-Computer,DC=adatum,DC=com**|
   |Benutzername des Administratorkontos für den virtuellen Computer|**student**|
   |Kennwort des Administratorkontos für den virtuellen Computer|**Pa55w.rd1234**|

1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Arbeitsbereich** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Desktop-App-Gruppe registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Hostpool erstellen** die Option **Erstellen** aus.

   > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dieser Vorgang dauert etwa 15 Minuten.

#### <a name="task-3-configure-azure-virtual-desktop-application-groups"></a>Aufgabe 3: Konfigurieren von Azure Virtual Desktop-Anwendungsgruppen

1. Suchen Sie in der Remotedesktopsitzung für **az140-cl-vm11a** nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Anwendungsgruppen** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** die automatisch generierte Desktopanwendungsgruppe **az140-21a-hp1-DAG** aus.
1. Wählen Sie auf dem Blatt **az140-21a-hp1-DAG** im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** die Option **Zuweisungen** aus.
1. Wählen Sie auf dem Blatt **az140-21a-hp1-DAG \| Zuweisungen** die Option **+ Hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Option **az140-wvd-apooled** aus, und klicken Sie auf **Auswählen**.
1. Kehren Sie zum Blatt **Azure Virtual Desktop \| Anwendungssicherheitsgruppen** zurück, und wählen Sie **+ Hinzufügen** aus.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Anwendungsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-21a-RG**|
   |Hostpool|**az140-21a-hp1**|
   |Anwendungsgruppentyp|**RemoteApp**|
   |Name der Anwendungsgruppe|**az140-21a-hp1-Office365-RAG**|

1. Wählen Sie auf der Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Anwendungen hinzufügen** aus.
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
1. Wählen Sie auf der Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Azure AD-Benutzer oder -Benutzergruppen hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Option **az140-wvd-aremote-app** aus, und klicken Sie auf **Auswählen**.
1. Kehren Sie zur Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie **Weiter: Arbeitsbereich >** .
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die folgende Einstellung an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendung registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Anwendungsgruppe erstellen** die Option **Erstellen** aus.

   > **Hinweis:** Nun wird eine Anwendungsgruppe erstellt, die auf dem Dateipfad als Anwendungsquelle basiert.

1. Suchen Sie in der Remotedesktopsitzung für **az140-cl-vm11a** in dem Webbrowserfenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Anwendungsgruppen** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** die Option **+ Erstellen** aus. 
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Anwendungsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-21a-RG**|
   |Hostpool|**az140-21a-hp1**|
   |Anwendungsgruppentyp|**RemoteApp**|
   |Name der Anwendungsgruppe|**az140-21a-hp1-Utilities-RAG**|

1. Wählen Sie auf der Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Anwendungen hinzufügen** aus.
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
1. Wählen Sie auf der Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Azure AD-Benutzer oder -Benutzergruppen hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Azure AD-Benutzer oder -Benutzergruppen auswählen** die Optionen **az140-wvd-aremote-app** und **az140-wvd-aadmins** aus, und klicken Sie auf **Auswählen**.
1. Kehren Sie zur Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie **Weiter: Arbeitsbereich >** .
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die folgende Einstellung an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendung registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Anwendungsgruppe erstellen** die Option **Erstellen** aus.

#### <a name="task-4-configure-azure-virtual-desktop-workspaces"></a>Aufgabe 4: Konfigurieren von Azure Virtual Desktop-Arbeitsbereichen

1. Suchen Sie in der Remotedesktopsitzung für **az140-cl-vm11a** in dem Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Arbeitsbereiche** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Arbeitsbereiche** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungsgruppen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**az140-21a-RG**|
   |Arbeitsbereichname|**az140-21a-ws1**|
   |Angezeigter Name|**az140-21a-ws1**|
   |Standort|Der Name der Azure-Region, in der Sie Ressourcen in diesem Lab bereitgestellt haben.||

1. Geben Sie auf der Registerkarte **Anwendungsgruppen** des Blatts **Arbeitsbereich erstellen** die folgenden Einstellungen an:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsgruppen registrieren|**Ja**|

1. Wählen Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die Option **+ Anwendungsgruppen registrieren** aus.
1. Wählen Sie auf dem Blatt **Anwendungsgruppen hinzufügen** das Plussymbol neben den Einträgen **az140-21a-hp1-DAG**, **az140-21a-hp1-Office365-RAG** und **az140-21a-hp1-Utilities-RAG** aus, und klicken Sie anschließend auf **Auswählen**. 
1. Wählen Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Anwendungsgruppen** die Option **Überprüfen + erstellen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Arbeitsbereich erstellen** die Option **Erstellen** aus.

### <a name="exercise-2-validate-azure-virtual-desktop-environment"></a>Übung 2: Überprüfen der Azure Virtual Desktop-Umgebung
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Installieren des Microsoft-Remotedesktopclients (MSRDC) auf einem Windows 10-Computer
1. Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs
1. Testen von Azure Virtual Desktop-Apps

#### <a name="task-1-install-microsoft-remote-desktop-client-msrdc-on-a-windows-10-computer"></a>Aufgabe 1: Installieren des Microsoft-Remotedesktopclients (MSRDC) auf einem Windows 10-Computer

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** Microsoft Edge, und navigieren Sie zur [Downloadseite für den Windows-Desktopclient](https://go.microsoft.com/fwlink/?linkid=2068602). Führen Sie bei entsprechender Aufforderung die Installation gemäß den Anweisungen aus. Wählen Sie die Option **Für alle Benutzer auf diesem Computer installieren** aus. 
1. Starten Sie nach Abschluss der Installation den Remotedesktopclient.

#### <a name="task-2-subscribe-to-a-azure-virtual-desktop-workspace"></a>Aufgabe 2: Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs

1. Wählen Sie im Clientfenster **Remotedesktop** die Option **Abonnieren** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aaduser1** an. Verwenden Sie dabei das userPrincipalName-Attribut als Benutzername und das beim Erstellen dieses Kontos festgelegte Kennwort. 

   > **Hinweis:** Wählen Sie alternativ im Clientfenster **Remotedesktop** die Option **Mit URL abonnieren** aus. Geben Sie im Bereich **Arbeitsbereich abonnieren** unter **E-Mail oder Arbeitsbereichs-URL** die URL **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery** ein, wählen Sie **Weiter** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aaduser1** an. Verwenden Sie dabei das userPrincipalName-Attribut als Benutzername und **Pa55w.rd1234** als Kennwort. 

   > **Hinweis:** Der Benutzerprinzipalname von **aaduser1** muss folgendes Format aufweisen: **aaduser1@***<Name der Azure AD-Domäne>* . Der Platzhalter *<Name der Azure AD-Domäne>* entspricht dem Namen des Azure AD-Mandanten, der dem Abonnement zugeordnet ist, in dem Sie die Azure AD DS-Instanz bereitgestellt haben.

1. Deaktivieren Sie im Fenster **Bei all Ihren Apps angemeldet bleiben** das Kontrollkästchen für **Verwaltung meines Geräts durch meine Organisation zulassen**, und aktivieren Sie **Nein, nur bei dieser App anmelden**. 
1. Vergewissern Sie sich, dass auf der Seite **Remotedesktop** die Liste der Anwendungen angezeigt wird, die in den Anwendungsgruppen enthalten sind, die dem Benutzerkonto **aaduser1** über ihre Gruppenmitgliedschaft zugeordnet sind. 

#### <a name="task-3-test-azure-virtual-desktop-apps"></a>Aufgabe 3: Testen von Azure Virtual Desktop-Apps

1. Doppelklicken Sie in der Remotedesktopsitzung für **az140-cl-vm11a** im Clientfenster **Remotedesktop** in der Liste der Anwendungen auf **Eingabeaufforderung**, und vergewissern Sie sich, dass ein Fenster vom Typ **Eingabeaufforderung** geöffnet wird. Geben Sei bei der Aufforderung zur Authentifizierung das Kennwort ein, das Sie für das Benutzerkonto **aaduser1** festgelegt haben, aktivieren Sie das Kontrollkästchen **Anmeldedaten speichern**, und wählen Sie anschließend **OK** aus.

   > **Hinweis:** Es kann anfangs einige Minuten dauern, bis die Anwendung gestartet wird. Danach sollte der Anwendungsstart aber viel schneller erfolgen.

1. Geben Sie an der Eingabeaufforderung **hostname** ein, und drücken Sie die **EINGABETASTE**, um den Namen des Computers anzuzeigen, auf dem die Eingabeaufforderung ausgeführt wird.

   > **Hinweis:** Vergewissern Sie sich, dass der angezeigte Name nicht **az140-cl-vm11a**, sondern **az140-21-p1-0** oder **az140-21-p1-1** lautet.

1. Geben Sie an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.
1. Doppelklicken Sie in der Remotedesktopsitzung für **az140-cl-vm11a** im Clientfenster **Remotedesktop** in der Liste der Anwendungen auf **SessionDesktop**, und vergewissern Sie sich, dass eine Remotedesktopsitzung gestartet wird. 
1. Klicken Sie in der Sitzung **Standarddesktop** mit der rechten Maustaste auf **Start**, und wählen Sie **Ausführen** aus. Geben Sie **cmd** in das Textfeld **Öffnen** des Dialogfelds **Ausführen** ein, und wählen Sie **OK** aus. 
1. Geben Sie in der **Standarddesktopsitzung** an der Eingabeaufforderung **hostname** ein, und drücken Sie die **EINGABETASTE**, um den Namen des Computers anzuzeigen, auf dem die Remotedesktopsitzung ausgeführt wird.
1. Vergewissern Sie sich, dass der angezeigte Name **az140-21-p1-0**, **az140-21-p1-1** oder **az140-21-p1-2** lautet.
