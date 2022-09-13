---
lab:
  title: 'Lab: Konfigurieren von Richtlinien für bedingten Zugriff für WVD (AD DS)'
  module: 'Module 3: Manage Access and Security'
---

# <a name="lab---configure-conditional-access-policies-for-wvd-ad-ds"></a>Lab: Konfigurieren von Richtlinien für bedingten Zugriff für WVD (AD DS)
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder Azure AD-Konto mit der Rolle „Globaler Administrator“ im Azure AD-Mandanten, das dem Azure-Abonnement zugeordnet ist und im Azure-Abonnement die Rolle „Besitzer“ oder „Mitwirkender“ innehat.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)**

## <a name="estimated-time"></a>Geschätzte Dauer

60 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen den Zugriff auf eine Bereitstellung von Azure Virtual Desktop in einer Azure AD DS-Umgebung (Azure Active Directory Domain Services) mithilfe des bedingten Zugriffs von Azure AD (Azure Active Directory) steuern.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Azure AD-basierten bedingten Zugriff für Azure Virtual Desktop vorzubereiten.
- Azure AD-basierten bedingten Zugriff für Azure Virtual Desktop zu implementieren.

## <a name="lab-files"></a>Labdateien

- Keine 

## <a name="instructions"></a>Anweisungen

### <a name="exercise-1-prepare-for-azure-ad-based-conditional-access-for-azure-virtual-desktop"></a>Übung 1: Vorbereiten des Azure AD-basierten bedingten Zugriffs für Azure Virtual Desktop

Die Hauptaufgaben für diese Übung sind Folgende:

1. Konfigurieren einer Azure AD Premium P2-Lizenz
1. Konfigurieren der Azure AD-Multi-Faktor-Authentifizierung (MFA)
1. Registrieren eines Benutzers für die Azure AD-MFA
1. Konfigurieren der Hybrid-Azure AD-Einbindung
1. Auslösen der Azure AD Connect-Deltasynchronisierung

#### <a name="task-1-configure-azure-ad-premium-p2-licensing"></a>Aufgabe 1: Konfigurieren einer Azure AD Premium P2-Lizenz

>**Hinweis:** Für die Implementierung des bedingten Azure AD-Zugriffs werden Premium P1- oder P2-Lizenzen für Azure AD benötigt. Für dieses Lab verwenden Sie eine 30-Tage-Testversion.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Suchen Sie im Azure-Portal nach **Azure Active Directory**, und wählen Sie diese Option aus, um zu dem Azure AD-Mandanten zu navigieren, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist.
1. Klicken Sie auf dem Blatt „Azure Active Directory“ auf der vertikalen Menüleiste links im Abschnitt **Verwalten** auf **Benutzer**. 
1. Wählen Sie auf dem Blatt **Benutzer | Alle Benutzer (Vorschau)** die Option **aduser5** aus.
1. Klicken Sie auf dem Blatt **aduser5 | Profil** auf der Symbolleiste auf **Bearbeiten**. Wählen Sie im Abschnitt **Einstellungen** in der Dropdownliste **Nutzungsstandort** das Land aus, in dem sich die Labumgebung befindet, und klicken Sie auf der Symbolleiste auf **Speichern**.
1. Ermitteln Sie auf dem Blatt **aduser5 | Profil** im Abschnitt **Identität** den Benutzerprinzipalnamen des Kontos **aduser5**.

    >**Hinweis**: Notieren Sie sich diesen Wert. Sie benötigen ihn später in diesem Lab.

1. Wählen Sie auf dem Blatt **Benutzer | Alle Benutzer (Vorschau)** das Benutzerkonto aus, mit dem Sie sich am Anfang dieser Aufgabe angemeldet haben, und wiederholen Sie den vorherigen Schritt, falls Ihrem Konto kein **Nutzungsstandort** zugewiesen ist. 

    >**Hinweis:** Die Eigenschaft **Nutzungsstandort** muss festgelegt werden, damit Benutzerkonten eine Azure AD Premium P2-Lizenz zugewiesen werden kann.

1. Wählen Sie auf dem Blatt **Benutzer | Alle Benutzer (Vorschau)** das Benutzerkonto **aadsyncuser** aus, und ermitteln Sie seinen Benutzerprinzipalnamen.

    >**Hinweis**: Notieren Sie sich diesen Wert. Sie benötigen ihn später in diesem Lab.

1. Navigieren Sie im Azure-Portal zurück zum Blatt **Übersicht** des Azure AD-Mandanten, und klicken Sie auf der vertikalen Menüleiste auf der linken Seite im Abschnitt **Verwalten** auf **Lizenzen**.
1. Klicken Sie auf dem Blatt **Lizenzen \| Übersicht** auf der vertikalen Menüleiste links im Abschnitt **Verwalten** auf **Alle Produkte**.
1. Klicken Sie auf dem Blatt **Lizenzen \| Alle Produkte** auf der Symbolleiste auf **+ Ausprobieren/Kaufen**.
1. Klicken Sie auf dem Blatt **Aktivieren** im Abschnitt **ENTERPRISE MOBILITY + SECURITY E5** auf **Kostenlose Testversion** und dann auf **Aktivieren**. 
1. Aktualisieren Sie auf dem Blatt **Lizenzen \| Übersicht** das Browserfenster, um zu überprüfen, ob die Aktivierung erfolgreich war. 
1. Wählen Sie auf dem Blatt **Lizenzen – Alle Produkte** den Eintrag **Enterprise Mobility + Security E5** aus. 
1. Klicken Sie auf dem Blatt **Enterprise Mobility + Security E5** auf der Symbolleiste auf **+ Zuweisen**.
1. Klicken Sie auf dem Blatt **Lizenz zuweisen** auf **Benutzer und Gruppen hinzufügen**, wählen Sie auf dem Blatt **Benutzer und Gruppen hinzufügen** den Eintrag **aduser5** und Ihr Benutzerkonto aus, und klicken Sie auf **Auswählen**.
1. Navigieren Sie zurück zum Blatt **Lizenzen zuweisen**, und klicken Sie auf **Zuweisungsoptionen**. Stellen Sie auf dem Blatt **Zuweisungsoptionen** sicher, dass alle Optionen aktiviert sind, und klicken Sie auf **Überprüfen + zuweisen** und auf **Zuweisung**.

#### <a name="task-2-configure-azure-ad-multi-factor-authentication-mfa"></a>Aufgabe 2: Konfigurieren der Azure AD-Multi-Faktor-Authentifizierung (MFA)

1. Navigieren Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Übersicht** des Azure AD-Mandanten, und klicken Sie im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **Sicherheit**.
1. Klicken Sie auf dem Blatt **Sicherheit | Erste Schritte** im vertikalen Menü auf der linken Seite im Abschnitt **Schützen** auf **Identity Protection**.
1. Klicken Sie auf dem Blatt **Identity Protection | Übersicht** im vertikalen Menü auf der linken Seite im Abschnitt **Schützen** auf **MFA-Registrierungsrichtlinie**. (Aktualisieren Sie ggf. die Webbrowserseite.)
1. Klicken Sie auf dem Blatt **Identity Protection | MFA-Registrierungsrichtlinie** im Abschnitt **Zuweisungen** der **MFA-Registrierungsrichtlinie** auf **Alle Benutzer** und auf der Registerkarte **Einschließen** auf die Option **Einzelne Benutzer und Gruppen auswählen**. Klicken Sie unter **Benutzer auswählen** auf **aduser5** > **Auswählen**, legen Sie unten auf dem Blatt den Schalter für **Richtlinie erzwingen** auf **Ein** fest, und klicken Sie auf **Speichern**.

#### <a name="task-3-register-a-user-for-azure-ad-mfa"></a>Aufgabe 3: Registrieren eines Benutzers für die Azure AD-MFA

1. Öffnen Sie auf Ihrem Labcomputer eine Webbrowsersitzung vom Typ **InPrivate**, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an, indem Sie den Benutzerprinzipalnamen für **aduser5**, den Sie weiter oben in dieser Übung ermittelt haben, und das beim Erstellen dieses Kontos festgelegte Kennwort eingeben.
1. Wenn Sie die Meldung **Weitere Informationen erforderlich** angezeigt wird, klicken Sie auf **Weiter**. Dadurch werden Sie im Browser automatisch zur Seite **Microsoft Authenticator** umgeleitet.
1. Wählen Sie auf der Seite **Zusätzliche Sicherheitsüberprüfung** im Abschnitt **Schritt 1: Auf welchem Weg sollen wir Sie kontaktieren?** Ihre bevorzugte Authentifizierungsmethode aus, und befolgen Sie die Anweisungen zum Abschließen des Registrierungsprozesses. 
1. Klicken Sie im Azure-Portal in der oberen rechten Ecke auf das Symbol für den Benutzeravatar und dann auf **Abmelden**, und schließen Sie das **InPrivate**-Browserfenster. 

#### <a name="task-4-configure-hybrid-azure-ad-join"></a>Aufgabe 4: Konfigurieren der Hybrid-Azure AD-Einbindung

> **Hinweis:** Mit dieser Funktion kann zusätzliche Sicherheit implementiert werden, wenn Sie bedingten Zugriff für Geräte basierend auf Ihrem Azure AD-Einbindungsstatus einrichten.

1. Suchen Sie auf dem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-dc-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Erweitern Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Menü **Start** den Ordner **Azure AD Connect**, und wählen Sie **Azure AD Connect** aus.
> **Hinweis:** Wenn ein Fenster mit dem Fehler angezeigt wird, dass der Synchronisierungsdienst nicht ausgeführt wird, navigieren Sie zum PowerShell-Befehlsfenster, und geben Sie **Start-Service "ADSync"** ein. Wiederholen Sie anschließend Schritt 4.
3. Wählen Sie auf der Seite **Willkommen bei Azure AD Connect** des Fensters **Microsoft Azure Active Directory Connect** die Option **Konfigurieren** aus.
4. Wählen Sie auf der Seite **Weitere Aufgaben** im Fenster **Microsoft Azure Active Directory Connect** die Option **Geräteoptionen konfigurieren** und anschließend **Weiter** aus.
5. Überprüfen Sie auf der Seite **Übersicht** im Fenster **Microsoft Azure Active Directory Connect** die Information zu **Hybrid-Azure AD-Einbindung** und **Geräterückschreiben**, und wählen Sie **Weiter** aus.
6. Authentifizieren Sie sich auf der Seite **Mit Azure AD verbinden** im Fenster **Microsoft Azure Active Directory Connect** mit den Anmeldeinformationen des Benutzerkontos **aadsyncuser**, das Sie in der vorherigen Übung erstellt haben, und wählen Sie anschließend **Weiter** aus.  

   > **Hinweis:** Geben Sie das weiter oben in diesem Lab erfasste Attribut „userPrincipalName“ des Kontos **aadsyncuser** sowie das Kennwort an, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben. 

1. Vergewissern Sie sich auf der Seite **Geräteoptionen** im Fenster **Microsoft Azure Active Directory Connect**, dass die Option **Hybrid-Azure AD-Einbindung konfigurieren** aktiviert ist, und wählen Sie **Weiter** aus. 
1. Aktivieren Sie auf der Seite **Gerätebetriebssysteme** im Fenster **Microsoft Azure Active Directory Connect** das Kontrollkästchen **In die Domäne eingebundene Geräte mit Windows 10 oder höher**, und wählen Sie **Weiter** aus. 
1. Aktivieren Sie auf der Seite **SCP-Konfiguration** im Fenster **Microsoft Azure Active Directory Connect** das Kontrollkästchen neben dem Eintrag **adatum.com**, und wählen Sie in der Dropdownliste **Authentifizierungsdienst** die Option **Azure Active Directory** und anschließend **Hinzufügen** aus. 
1. Wenn Sie dazu aufgefordert werden, geben Sie im Dialogfeld **Unternehmensadministrator-Anmeldeinformationen** die folgenden Anmeldeinformationen ein, und wählen Sie **OK** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**ADATUM\Student**|
   |Kennwort|**Pa55w.rd1234**|

1. Navigieren Sie zurück zur Seite **SCP-Konfiguration** im Fenster **Microsoft Azure Active Directory Connect**, und wählen Sie **Weiter** aus.
1. Wählen Sie auf der Seite **Bereit zur Konfiguration** im Fenster **Microsoft Azure Active Directory Connect** die Option **Konfigurieren** und nach Abschluss der Konfiguration die Option **Beenden** aus.
1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator.
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Computerkonto **az140-cl-vm11** in die Organisationseinheit **WVDClients** zu verschieben:

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. Erweitern Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Menü **Start** den Ordner **Azure AD Connect**, und wählen Sie **Azure AD Connect** aus.
1. Klicken Sie auf der Seite **Willkommen bei Azure AD Connect** des Fensters **Microsoft Azure Active Directory Connect** auf **Konfigurieren**.
1. Wählen Sie auf der Seite **Weitere Aufgaben** im Fenster **Microsoft Azure Active Directory Connect** die Option **Synchronisierungsoptionen anpassen** und anschließend **Weiter** aus.
1. Authentifizieren Sie sich auf der Seite **Mit Azure AD verbinden** im Fenster **Microsoft Azure Active Directory Connect** mit den Anmeldeinformationen des Benutzerkontos **aadsyncuser**, das Sie in der vorherigen Übung erstellt haben, und wählen Sie anschließend **Weiter** aus. 

   > **Hinweis:** Geben Sie das weiter oben in diesem Lab erfasste Attribut „userPrincipalName“ des Kontos **aadsyncuser** sowie das Kennwort an, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben. 

1. Wählen Sie auf der Seite **Verzeichnisse verbinden** im Fenster **Microsoft Azure Active Directory Connect** die Option **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Filtern von Domänen und Organisationseinheiten** im Fenster **Microsoft Azure Active Directory Connect**, dass die Option **Ausgewählte Domänen und Organisationseinheiten synchronisieren** aktiviert ist, und erweitern Sie den Knoten **adatum.com**. Stellen Sie sicher, dass das Kontrollkästchen neben der Organisationseinheit **ToSync** aktiviert ist, aktivieren Sie das Kontrollkästchen neben der Organisationseinheit **WVDClients**, und wählen Sie **Weiter** aus.
1. Übernehmen Sie auf der Seite **Optionale Features** im Fenster **Microsoft Azure Active Directory Connect** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Bereit zur Konfiguration** im Fenster **Microsoft Azure Active Directory Connect**, dass das Kontrollkästchen **Starten Sie den Synchronisierungsvorgang, nachdem die Konfiguration abgeschlossen wurde.** aktiviert ist, und klicken Sie auf **Konfigurieren**.
1. Überprüfen Sie die Informationen auf der Seite **Konfiguration abgeschlossen**, und wählen Sie **Beenden** aus, um das Fenster **Microsoft Azure Active Directory Connect** zu schließen.

#### <a name="task-5-trigger-azure-ad-connect-delta-synchronization"></a>Aufgabe 5: Auslösen der Azure AD Connect-Deltasynchronisierung

1. Wechseln Sie innerhalb der Remotedesktopsitzung für **az140-25-vm11** zum Fenster **Administrator: Windows PowerShell ISE**.
1. Führen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Konsolenbereich **Administrator: Windows PowerShell ISE** Folgendes aus, um die Azure AD Connect-Deltasynchronisierung auszulösen:

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mithilfe der Azure AD-Anmeldeinformationen des Benutzerkontos an, das im Azure AD-Mandanten, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Suchen Sie innerhalb der Remotedesktopsitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Azure Active Directory**, und wählen Sie diese Option aus, um zu dem Azure AD-Mandanten zu navigieren, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist.
1. Klicken Sie auf dem Blatt „Azure Active Directory“ auf der vertikalen Menüleiste links im Abschnitt **Verwalten** auf **Geräte**. 
1. Überprüfen Sie auf dem Blatt **Geräte | Alle Geräte** die Liste der Geräte, und vergewissern Sie sich, dass das Gerät **az140-cl-vm11** in der Spalte **Jointyp** mit dem Eintrag **Hybrid, in Azure AD eingebunden** aufgeführt ist.

   > **Hinweis:** Möglicherweise müssen Sie einige Minuten warten, bis die Synchronisierung wirksam wird und das Gerät im Azure-Portal angezeigt wird.

### <a name="exercise-2-implement-azure-ad-based-conditional-access-for-azure-virtual-desktop"></a>Übung 2: Azure AD-basierten bedingten Zugriff für Azure Virtual Desktop zu implementieren.

Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen einer Azure AD-basierten Richtlinie für bedingten Zugriff für alle Azure Virtual Desktop-Verbindungen
1. Testen der Azure AD-basierten Richtlinie für bedingten Zugriff für alle Azure Virtual Desktop-Verbindungen
1. Ändern der Azure AD-basierten Richtlinie für bedingten Zugriff, um in Azure AD eingebundene Hybridcomputer aus der MFA-Anforderung auszuschließen
1. Testen der geänderten Azure AD-basierten Richtlinie für bedingten Zugriff

#### <a name="task-1-create-an-azure-ad-based-conditional-access-policy-for-all-azure-virtual-desktop-connections"></a>Aufgabe 1: Erstellen einer Azure AD-basierten Richtlinie für bedingten Zugriff für alle Azure Virtual Desktop-Verbindungen

>**Hinweis:** In dieser Aufgabe konfigurieren Sie eine Azure AD-basierte Richtlinie für bedingten Zugriff, die MFA für die Anmeldung bei einer Azure Virtual Desktop-Sitzung erfordert. Die Richtlinie erzwingt auch die erneute Authentifizierung nach den ersten vier Stunden nach erfolgreicher Authentifizierung.

1. Navigieren Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Übersicht** des Azure AD-Mandanten, und klicken Sie im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** auf **Sicherheit**.
1. Klicken Sie auf dem Blatt **Sicherheit \| Erste Schritte** im vertikalen Menü auf der linken Seite im Abschnitt **Schützen** auf **Bedingter Zugriff**.
1. Klicken Sie auf dem Blatt **Bedingter Zugriff \| Richtlinien** auf der Symbolleiste auf **+ Neue Richtlinie**, und wählen Sie im Kontextmenü die Option **Neue Richtlinie erstellen** aus.
1. Konfigurieren Sie auf dem Blatt **Neu** die folgenden Einstellungen:

   - Geben Sie in das Textfeld **Name** den Namen **az140-31-wvdpolicy1** ein.
   - Wählen Sie im Abschnitt **Zuweisungen** die Option **Benutzer- oder Workloadidentitäten** aus. Vergewissern Sie sich in der Dropdownliste **Wofür gilt diese Richtlinie?** , dass **Benutzer und Gruppen** aktiviert ist. Aktivieren Sie im Abschnitt **Benutzer und Gruppen auswählen** das Kontrollkästchen **Benutzer und Gruppen**, und klicken Sie auf dem Blatt **Auswählen** auf **aduser5** und anschließend auf **Auswählen**.
   - Klicken Sie im Abschnitt **Zuweisungen** auf **Cloudanwendungen oder -aktionen** . Vergewissern Sie sich, dass unter **Wählen Sie aus, worauf diese Richtlinie angewendet werden soll.** die Option **Cloud-Apps** aktiviert ist, und klicken Sie auf die Option **Apps auswählen**. Aktivieren Sie auf dem Blatt **Auswählen** das Kontrollkästchen neben dem Eintrag **Azure Virtual Desktop**, und klicken Sie auf **Auswählen**. 
   - Klicken Sie im Abschnitt **Zuweisungen** auf **Bedingungen** > **Client-Apps**. Legen Sie auf dem Blatt **Client-Apps** den Schalter **Konfigurieren** auf **Ja** fest, vergewissern Sie sich, dass die Kontrollkästchen **Browser** und **Mobile Apps und Desktopclients** aktiviert sind, und klicken Sie auf **Fertig**.
   - Klicken Sie im Abschnitt **Zugriffssteuerung** auf **Gewähren**. Vergewissern Sie sich auf dem Blatt **Gewähren**, dass die Option **Zugriff gewähren** aktiviert ist, aktivieren Sie das Kontrollkästchen **Multi-Faktor-Authentifizierung erfordern**, und klicken Sie auf **Auswählen**.
   - Klicken Sie im Abschnitt **Zugriffssteuerungen** auf **Sitzung**, und aktivieren Sie auf dem Blatt **Sitzung** das Kontrollkästchen **Anmeldehäufigkeit**. Geben Sie im ersten Textfeld **4** ein. Wählen Sie in der Dropdownliste **Einheiten auswählen** den Eintrag **Stunden** aus, lassen Sie das Kontrollkästchen **Beständige Browsersitzung** deaktiviert, und klicken Sie auf **Auswählen**.
   - Legen Sie den Schalter **Richtlinie aktivieren** auf **Ein** fest.

1. Klicken Sie auf dem Blatt **Neu** auf **Erstellen**. 

#### <a name="task-2-test-the-azure-ad-based-conditional-access-policy-for-all-azure-virtual-desktop-connections"></a>Aufgabe 2: Testen der Azure AD-basierten Richtlinie für bedingten Zugriff für alle Azure Virtual Desktop-Verbindungen

1. Öffnen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ Folgendes aus, um die virtuellen Azure-Computer mit den Azure Virtual Desktop-Sitzungshosts zu starten, die Sie in diesem Lab verwenden:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Hinweis:** Warten Sie, bis der Befehl ausgeführt wurde und alle virtuellen Azure-Computer in der Ressourcengruppe **az140-21-RG** ausgeführt werden. 

1. Öffnen Sie auf Ihrem Labcomputer eine Webbrowsersitzung vom Typ **InPrivate**, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an, indem Sie den Benutzerprinzipalnamen für **aduser5**, den Sie weiter oben in dieser Übung ermittelt haben, und das beim Erstellen dieses Kontos festgelegte Kennwort eingeben.

   > **Hinweis:** Stellen Sie sicher, dass Sie nicht zur Authentifizierung per MFA aufgefordert werden.

1. Navigieren Sie in der **InPrivate**-Webbrowsersitzung zu der HTML5-Webclientseite von Azure Virtual Desktop unter [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Hinweis:** Vergewissern Sie sich, dass dadurch automatisch die Authentifizierung per MFA ausgelöst wird.

1. Geben Sie im Bereich **Code eingeben** den Code aus der Textnachricht oder der Authentifizierungs-App ein, die Sie registriert haben, und wählen Sie **Überprüfen** aus.
1. Klicken Sie auf der Seite **Alle Ressourcen** auf **Eingabeaufforderung**. Deaktivieren Sie im Bereich **Auf lokale Ressourcen zugreifen** das Kontrollkästchen **Drucker**, und klicken Sie auf **Zulassen**.
1. Geben Sie bei entsprechender Aufforderung unter **Geben Sie Ihre Anmeldeinformationen ein** in das Textfeld **Benutzername** den Benutzerprinzipalnamen von **aduser5** und in das Textfeld **Kennwort** das Kennwort ein, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben, und klicken Sie auf **Übermitteln**.
1. Vergewissern Sie sich, dass die Remote-App **Eingabeaufforderung** erfolgreich gestartet wurde.
1. Geben Sie im Fenster der Remote-App **Eingabeaufforderung** an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**.
1. Navigieren Sie zurück zur Seite **Alle Ressourcen**, klicken Sie in der oberen rechten Ecke auf **aduser5** und im Dropdownmenü auf **Abmelden**, und schließen Sie das **InPrivate**-Webbrowserfenster.

#### <a name="task-3-modify-the-azure-ad-based-conditional-access-policy-to-exclude-hybrid-azure-ad-joined-computers-from-the-mfa-requirement"></a>Aufgabe 3: Ändern der Azure AD-basierten Richtlinie für bedingten Zugriff, um in Azure AD eingebundene Hybridcomputer aus der MFA-Anforderung auszuschließen

>**Hinweis:** In dieser Aufgabe ändern Sie die Azure AD-basierte Richtlinie für bedingten Zugriff, die MFA für die Anmeldung bei einer Azure Virtual Desktop-Sitzung erfordert, so, dass für Verbindungen von in Azure AD eingebundenen Computern keine MFA erforderlich ist.

1. Klicken Sie auf Ihrem Labcomputer in dem Browserfenster, in dem das Azure-Portal angezeigt wird, auf dem Blatt **Bedingter Zugriff | Richtlinien** auf den Eintrag, der die Richtlinie **az140-31-wvdpolicy1** darstellt.
1. Klicken Sie auf dem Blatt **az140-31-wvdpolicy1** im Abschnitt **Zugriffssteuerungen** auf **Gewähren**. Aktivieren Sie auf dem Blatt **Gewähren** die Kontrollkästchen **Multi-Faktor-Authentifizierung erfordern** und **In Azure AD eingebundenes Hybridgerät erforderlich**, vergewissern Sie sich, dass die Option **Eine der ausgewählten Steuerungen anfordern** aktiviert ist, und klicken Sie auf **Auswählen**.
1. Klicken Sie auf dem Blatt **az140-31-wvdpolicy1** auf **Speichern**.

>**Hinweis:** Es dauert möglicherweise einige Minuten, bis die Richtlinie wirksam wird.

#### <a name="task-4-test-the-modified-azure-ad-based-conditional-access-policy"></a>Aufgabe 4: Testen der geänderten Azure AD-basierten Richtlinie für bedingten Zugriff

1. Suchen Sie auf Ihrem Labcomputer in dem Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-cl-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen an, und wählen **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11** Microsoft Edge, und navigieren Sie zur HTML5-Webclientseite von Azure Virtual Desktop unter [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Hinweis:** Stellen Sie sicher, dass Sie dieses Mal nicht zur Authentifizierung per MFA aufgefordert werden. Das liegt daran, dass **az140-cl-vm11** als Hybridgerät in Azure AD eingebunden ist.

1. Doppelklicken Sie auf der Seite **Alle Ressourcen** auf **Eingabeaufforderung**. Deaktivieren Sie im Bereich **Auf lokale Ressourcen zugreifen** das Kontrollkästchen **Drucker**, und klicken Sie auf **Zulassen**.
1. Geben Sie bei entsprechender Aufforderung unter **Geben Sie Ihre Anmeldeinformationen ein** in das Textfeld **Benutzername** den Benutzerprinzipalnamen von **aduser5** und in das Textfeld **Kennwort** das Kennwort ein, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben, und klicken Sie auf **Übermitteln**.
1. Vergewissern Sie sich, dass die Remote-App **Eingabeaufforderung** erfolgreich gestartet wurde.
1. Geben Sie im Fenster der Remote-App **Eingabeaufforderung** an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**.
1. Navigieren Sie zurück zur Seite **Alle Ressourcen**, und klicken Sie in der Ecke oben rechts auf **aduser5** und im Dropdownmenü auf **Abmelden**.
1. Klicken Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11** auf **Start**. Klicken Sie auf der vertikalen Leiste direkt über der Schaltfläche **Start** auf das Symbol, das das angemeldete Benutzerkonto darstellt, und klicken Sie dann im Popupmenü auf **Abmelden**.

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>Übung 3: Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab bereitgestellten und verwendeten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### <a name="task-1-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten und verwendeten Azure-VMs

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
