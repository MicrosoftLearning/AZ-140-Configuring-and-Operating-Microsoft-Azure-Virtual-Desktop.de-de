---
lab:
  title: "Lab: Konfigurieren von Richtlinien für bedingten Zugriff für AVD (AD\_DS)"
  module: 'Module 3: Manage Access and Security'
---

# Lab: Konfigurieren von Richtlinien für bedingten Zugriff für AVD (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder ein Microsoft Entra-Konto mit der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten, der dem Azure-Abonnement zugeordnet ist, und mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**
- Das abgeschlossene Lab **Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)**

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen den Zugriff auf eine Bereitstellung von Azure Virtual Desktop in einer Active Directory Domain Services-Umgebung (AD DS) mithilfe des bedingten Zugriffs von Microsoft Entra steuern.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Vorbereiten des bedingten Zugriffs von Microsoft Entra für Azure Virtual Desktop
- Implementieren des bedingten Zugriffs von Microsoft Entra für Azure Virtual Desktop

## Labdateien

- Keine 

## Anweisungen

>**Wichtig:** Microsoft hat **Azure Active Directory** (**Azure AD**) in **Microsoft Entra ID** umbenannt. Ausführliche Informationen zu dieser Änderung finden Sie unter [Neuer Name für Azure Active Directory](https://learn.microsoft.com/en-us/entra/fundamentals/new-name). Das ist ein laufendes Projekt, daher kann es immer noch vorkommen, dass die Labanweisungen und die Elemente auf der Benutzerfläche voneinander abweichen, während Sie einzelne Übungen absolvieren. Berücksichtigen Sie das (in diesem Lab ist **Microsoft Entra Connect** der neue Name von **Azure Active Directory Connect**, und der Begriff **Azure Active Directory** wird bei der Konfiguration des Dienstverbindungspunkts in Aufgabe 4 von Übung 1 noch verwendet).

>**Wichtig:** Für die Aktivierung einer Microsoft Entra ID P2-Testversion müssen Kreditkarteninformationen angegeben werden. Aus diesem Grund ist diese Übung optional. Stattdessen können Kursleiter diese Funktionalität für die Lernenden demonstrieren.

### Übung 1: Vorbereiten des bedingten Zugriffs von Microsoft Entra für Azure Virtual Desktop

Die Hauptaufgaben für diese Übung sind Folgende:

1. Konfigurieren der Microsoft Entra Premium P2-Lizenzierung
1. Konfigurieren der Microsoft Entra Multi-Faktor-Authentifizierung (MFA)
1. Registrieren eines Benutzers bzw. einer Benutzerin für Microsoft Entra MFA
1. Konfigurieren der hybriden Microsoft Entra-Einbindung
1. Auslösen der Microsoft Azure Active Directory Connect-Deltasynchronisierung

#### Aufgabe 1: Konfigurieren der Microsoft Entra Premium P2-Lizenzierung

>**Hinweis:** Die Premium P1- oder P2-Lizenzierung von Microsoft Entra ist erforderlich, um den bedingten Zugriff von Microsoft Entra zu implementieren. Für dieses Lab verwenden Sie eine 30-Tage-Testversion.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com) und melden Sie sich an. Verwenden Sie dabei die Microsoft Entra-Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Microsoft Entra-Mandanten, der diesem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.

    >**Wichtig:** Stellen Sie sicher, dass Sie ein Geschäfts‑, Schul‑ oder Unikonto und **kein** Microsoft-Konto verwenden.

1. Suchen Sie im Azure-Portal nach **Azure Active Directory** und wählen Sie diese Option aus, um zu dem Microsoft Entra-Mandanten zu navigieren, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist.
1. Klicken Sie auf dem Blatt „Azure Active Directory“ auf der vertikalen Menüleiste links im Abschnitt **Verwalten** auf **Benutzer**. 
1. Wählen Sie auf dem Blatt **Benutzer | Alle Benutzer (Vorschau)** die Option **aduser5** aus.
1. Klicken Sie auf dem Blatt **aduser5 | Profil** auf der Symbolleiste auf **Bearbeiten**. Wählen Sie im Abschnitt **Einstellungen** in der Dropdownliste **Nutzungsstandort** das Land aus, in dem sich die Labumgebung befindet, und klicken Sie auf der Symbolleiste auf **Speichern**.
1. Ermitteln Sie auf dem Blatt **aduser5 | Profil** im Abschnitt **Identität** den Benutzerprinzipalnamen des Kontos **aduser5**.

    >**Hinweis**: Notieren Sie sich diesen Wert. Sie benötigen diese später in diesem Lab.

1. Wählen Sie auf dem Blatt **Benutzer | Alle Benutzer (Vorschau)** das Benutzerkonto aus, mit dem Sie sich am Anfang dieser Aufgabe angemeldet haben, und wiederholen Sie den vorherigen Schritt, falls Ihrem Konto kein **Nutzungsstandort** zugewiesen ist. 

    >**Hinweis:** Die Eigenschaft **Nutzungsstandort** muss festgelegt werden, damit Benutzerkonten eine Microsoft Entra Premium P2-Lizenz zugewiesen werden kann.

1. Wählen Sie auf dem Blatt **Benutzer | Alle Benutzer (Vorschau)** das Benutzerkonto **aadsyncuser** aus, und ermitteln Sie seinen Benutzerprinzipalnamen.

    >**Hinweis**: Notieren Sie sich diesen Wert. Sie benötigen diese später in diesem Lab.

1. Navigieren Sie im Azure-Portal zurück zum Blatt **Übersicht** des Microsoft Entra-Mandanten, und wählen Sie auf der vertikalen Menüleiste auf der linken Seite im Abschnitt **Verwalten** **Lizenzen** aus.
1. Klicken Sie auf dem Blatt **Lizenzen \| Übersicht** auf der vertikalen Menüleiste links im Abschnitt **Verwalten** auf **Alle Produkte**.
1. Klicken Sie auf dem Blatt **Lizenzen \| Alle Produkte** auf der Symbolleiste auf **+ Ausprobieren/Kaufen**.
1. Klicken Sie auf dem Blatt **Aktivieren** im Abschnitt **MICROSOFT ENTRA ID P2** auf **Kostenlose Testversion**, und klicken Sie dann auf **Aktivieren**. Befolgen Sie dann die Anweisungen, um den Aktivierungsprozess abzuschließen.
1. Wählen Sie auf dem Blatt **Lizenzen – Alle Produkte** den Eintrag **Enterprise Mobility + Security E5** aus. 
1. Klicken Sie auf dem Blatt **Enterprise Mobility + Security E5** auf der Symbolleiste auf **+ Zuweisen**.
1. Klicken Sie auf dem Blatt **Lizenz zuweisen** auf **Benutzer und Gruppen hinzufügen**, wählen Sie auf dem Blatt **Benutzer und Gruppen hinzufügen** den Eintrag **aduser5** und Ihr Benutzerkonto aus, und klicken Sie auf **Auswählen**.
1. Navigieren Sie zurück zum Blatt **Lizenzen zuweisen**, und klicken Sie auf **Zuweisungsoptionen**. Stellen Sie auf dem Blatt **Zuweisungsoptionen** sicher, dass alle Optionen aktiviert sind, und klicken Sie auf **Überprüfen + zuweisen** und auf **Zuweisung**.

#### Aufgabe 2: Konfigurieren der Microsoft Entra Multi-Faktor-Authentifizierung (MFA)

1. Navigieren Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal geöffnet ist, zurück zum Blatt **Übersicht** des Microsoft Entra-Mandanten, und wählen Sie im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** **Sicherheit** aus.
1. Klicken Sie auf dem Blatt **Sicherheit | Erste Schritte** im vertikalen Menü auf der linken Seite im Abschnitt **Schützen** auf **Identity Protection**.
1. Wählen Sie auf dem Blatt **Identitätsschutz | Übersicht** im vertikalen Menü auf der linken Seite im Abschnitt **Schützen** **Richtlinie für die Registrierung der Multi-Faktor-Authentifizierung** aus. (Aktualisieren Sie ggf. die Webbrowserseite.)
1. Wählen Sie auf dem Blatt **Identitätsschutz | Richtlinie für die Registrierung der Multi-Faktor-Authentifizierung** im Abschnitt **Zuweisungen** der **Richtlinie für die Registrierung der Multi-Faktor-Authentifizierung** **Alle Benutzer*innen** aus. Wählen Sie dann auf der Registerkarte **Einschließen** die Option **Einzelne Benutzer*innen und Gruppen auswählen**. Wählen Sie unter **Benutzer*innen auswählen** **aduser5** und dann **Auswählen** aus. Stellen Sie unten auf dem Blatt den Schalter für **Richtlinie erzwingen** auf **Ein** und wählen Sie **Speichern** aus.

#### Aufgabe 3: Registrieren eines Benutzers bzw. einer Benutzerin für Microsoft Entra MFA

1. Öffnen Sie auf Ihrem Labcomputer eine Webbrowsersitzung vom Typ **InPrivate**, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an, indem Sie den Benutzerprinzipalnamen für **aduser5**, den Sie weiter oben in dieser Übung ermittelt haben, und das beim Erstellen dieses Kontos festgelegte Kennwort eingeben.
1. Wenn Sie die Meldung **Weitere Informationen erforderlich** angezeigt wird, klicken Sie auf **Weiter**. Dadurch werden Sie im Browser automatisch zur Seite **Microsoft Authenticator** umgeleitet.
1. Wählen Sie auf der Seite **Zusätzliche Sicherheitsüberprüfung** im Abschnitt **Schritt 1: Auf welchem Weg sollen wir Sie kontaktieren?** Ihre bevorzugte Authentifizierungsmethode aus, und befolgen Sie die Anweisungen zum Abschließen des Registrierungsprozesses. 
1. Klicken Sie im Azure-Portal in der oberen rechten Ecke auf das Symbol für den Benutzeravatar und dann auf **Abmelden**, und schließen Sie das **InPrivate**-Browserfenster. 

#### Aufgabe 4: Konfigurieren der hybriden Microsoft Entra-Einbindung

> **Hinweis:** Mit dieser Funktion kann zusätzliche Sicherheit implementiert werden, wenn Sie bedingten Zugriff für Geräte basierend auf Ihrem Microsoft Entra-Einbindungsstatus einrichten.

1. Suchen Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-dc-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-dc-vm11** **Verbindung herstellen** aus und wählen Sie im Dropdownmenü **Verbindung über Bastion herstellen** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Erweitern Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Menü **Start** den Ordner **Azure AD Connect**, und wählen Sie **Azure AD Connect** aus.

   > **Hinweis:** Wenn ein Fenster mit dem Fehler angezeigt wird, dass der Synchronisierungsdienst nicht ausgeführt wird, navigieren Sie zum PowerShell-Befehlsfenster, und geben Sie **Start-Service "ADSync"** ein. Wiederholen Sie letzten Schritt.

1. Klicken Sie auf der Seite **Willkommen bei Azure AD Connect** des Fensters **Microsoft Azure Active Directory Connect** auf **Konfigurieren**.
1. Wählen Sie auf der Seite **Weitere Aufgaben** im Fenster **Microsoft Azure Active Directory Connect** die Option **Geräteoptionen konfigurieren** und anschließend **Weiter** aus.
1. Überprüfen Sie auf der Seite **Übersicht** im Fenster **Microsoft Azure Active Directory Connect** die Information zu **Hybrid-Microsoft Entra-Einbindung** und **Geräterückschreiben**, und wählen Sie **Weiter** aus.
1. Authentifizieren Sie sich auf der Seite **Mit Microsoft Entra verbinden** im Fenster **Microsoft Azure Active Directory Connect** mit den Anmeldeinformationen des Benutzerkontos **aadsyncuser**, das Sie in einem früheren Lab erstellt haben, und wählen Sie anschließend **Weiter** aus.  
1. Vergewissern Sie sich auf der Seite **Geräteoptionen** im Fenster **Microsoft Azure Active Directory Connect**, dass die Option **Hybrid-Azure AD-Einbindung konfigurieren** aktiviert ist, und wählen Sie **Weiter** aus. 
1. Aktivieren Sie auf der Seite **Gerätebetriebssysteme** im Fenster **Microsoft Azure Active Directory Connect** das Kontrollkästchen **In die Domäne eingebundene Geräte mit Windows 10 oder höher**, und wählen Sie **Weiter** aus. 
1. Aktivieren Sie auf der Seite **SCP-Konfiguration** im Fenster **Microsoft Azure Active Directory Connect** das Kontrollkästchen neben dem Eintrag **adatum.com**, und wählen Sie in der Dropdownliste **Authentifizierungsdienst** den Eintrag **Azure Active Directory** und anschließend **Hinzufügen** aus. 
1. Wenn Sie dazu aufgefordert werden, geben Sie im Dialogfeld **Unternehmensadministrator-Anmeldeinformationen** die folgenden Anmeldeinformationen ein, und wählen Sie **OK** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**ADATUM\Student**|
   |Kennwort|**Pa55w.rd1234**|

1. Navigieren Sie zurück zur Seite **SCP-Konfiguration** im Fenster **Microsoft Azure Active Directory Connect**, und wählen Sie **Weiter** aus.
1. Wählen Sie auf der Seite **Bereit zur Konfiguration** im Fenster **Microsoft Azure Active Directory Connect** die Option **Konfigurieren** und nach Abschluss der Konfiguration die Option **Beenden** aus.
1. Starten Sie innerhalb der Bastion-Sitzung auf **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator*in.
1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um das Computerkonto **az140-cl-vm11** in die Organisationseinheit **WVDClients** zu verschieben:

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. Erweitern Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Menü **Start** den Ordner **Azure AD Connect**, und wählen Sie **Azure AD Connect** aus.
1. Klicken Sie auf der Seite **Willkommen bei Azure AD Connect** des Fensters **Microsoft Azure Active Directory Connect** auf **Konfigurieren**.
1. Wählen Sie auf der Seite **Weitere Aufgaben** im Fenster **Microsoft Azure Active Directory Connect** die Option **Synchronisierungsoptionen anpassen** und anschließend **Weiter** aus.
1. Authentifizieren Sie sich auf der Seite **Mit Microsoft Entra verbinden** im Fenster **Microsoft Azure Active Directory Connect** mit den Anmeldeinformationen des Benutzerkontos **aadsyncuser**, das Sie in der vorherigen Übung erstellt haben, und wählen Sie anschließend **Weiter** aus. 
1. Wählen Sie auf der Seite **Verzeichnisse verbinden** im Fenster **Microsoft Azure Active Directory Connect** die Option **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Filtern von Domänen und Organisationseinheiten** im Fenster **Microsoft Azure Active Directory Connect**, dass die Option **Ausgewählte Domänen und Organisationseinheiten synchronisieren** aktiviert ist, und erweitern Sie den Knoten **adatum.com**. Stellen Sie sicher, dass das Kontrollkästchen neben der Organisationseinheit **ToSync** aktiviert ist, aktivieren Sie das Kontrollkästchen neben der Organisationseinheit **WVDClients**, und wählen Sie **Weiter** aus.
1. Übernehmen Sie auf der Seite **Optionale Features** im Fenster **Microsoft Azure Active Directory Connect** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Vergewissern Sie sich auf der Seite **Bereit zur Konfiguration** im Fenster **Microsoft Azure Active Directory Connect**, dass das Kontrollkästchen **Starten Sie den Synchronisierungsvorgang, nachdem die Konfiguration abgeschlossen wurde.** aktiviert ist, und klicken Sie auf **Konfigurieren**.
1. Überprüfen Sie die Informationen auf der Seite **Konfiguration abgeschlossen**, und wählen Sie **Beenden** aus, um das Fenster **Microsoft Azure Active Directory Connect** zu schließen.

#### Aufgabe 5: Auslösen der vollständigen Microsoft Azure Active Directory Connect-Synchronisierung

1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **VMs**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **VMs** den Eintrag **az140-cl-vm11** aus. Daraufhin wird das Blatt **az140-cl-vm11** geöffnet.
1. Wählen Sie im Blatt **az140-cl-vm11** **Neustarten** aus, und warten Sie dann, bis die Benachrichtigung **VM erfolgreich neu gestartet** angezeigt wird.
1. Wechseln Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** zum Fenster **Administrator: Windows PowerShell ISE**.
1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** Folgendes aus, um die vollständige Microsoft Azure Active Directory Connect-Synchronisierung auszulösen:

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mithilfe der Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das im Microsoft Entra-Mandanten, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist, über die Rolle „Globale*r Administrator*in“ verfügt.
1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach **Microsoft Entra ID**, und wählen Sie diese Option aus, um zu dem Microsoft Edge-Mandanten zu navigieren, der dem in diesem Lab verwendeten Azure-Abonnement zugeordnet ist.
1. Wählen Sie auf dem Blatt „Microsoft Entra ID“ auf der vertikalen Menüleiste links im Abschnitt **Verwalten** **Geräte** aus. 
1. Überprüfen Sie auf dem Blatt **Geräte | Alle Geräte** die Liste der Geräte, und vergewissern Sie sich, dass das Gerät **az140-cl-vm11** mit dem Eintrag **Hybride Microsoft Entra-Einbindung** in der Spalte **Einbindungstyp** aufgeführt ist.

   > **Hinweis:** Möglicherweise müssen Sie einige Minuten warten, bis die Synchronisierung wirksam wird und das Gerät im Azure-Portal angezeigt wird.

### Übung 2: Implementieren des bedingten Zugriffs von Microsoft Entra für Azure Virtual Desktop

Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen einer Richtlinie für bedingten Zugriff von Microsoft Entra für alle Azure Virtual Desktop-Verbindungen
1. Testen der Richtlinie für bedingten Zugriff von Microsoft Entra für alle Azure Virtual Desktop-Verbindungen
1. Ändern der Richtlinie für bedingten Zugriff von Microsoft Entra, um hybrid in Microsoft Entra eingebundene Computer aus der MFA-Anforderung auszuschließen
1. Testen der geänderten Richtlinie für bedingten Zugriff von Microsoft Entra

#### Aufgabe 1: Erstellen einer Richtlinie für bedingten Zugriff von Microsoft Entra für alle Azure Virtual Desktop-Verbindungen

>**Hinweis:** In dieser Aufgabe konfigurieren Sie eine Richtlinie für bedingten Zugriff von Microsoft Entra, die MFA für die Anmeldung bei einer Azure Virtual Desktop-Sitzung erfordert. Die Richtlinie erzwingt auch die erneute Authentifizierung nach den ersten vier Stunden nach erfolgreicher Authentifizierung.

1. Navigieren Sie auf Ihrem Labcomputer in dem Webbrowser, in dem das Azure-Portal geöffnet ist, zurück zum Blatt **Übersicht** des Microsoft Entra-Mandanten, und wählen Sie im vertikalen Menü auf der linken Seite im Abschnitt **Verwalten** **Sicherheit** aus.
1. Klicken Sie auf dem Blatt **Sicherheit \| Erste Schritte** im vertikalen Menü auf der linken Seite im Abschnitt **Schützen** auf **Bedingter Zugriff**.
1. Wählen Sie auf dem Blatt **Bedingter Zugriff \| Richtlinien** in der Symbolleiste **+ Neue Richtlinie** aus.
1. Konfigurieren Sie auf dem Blatt **Neu** die folgenden Einstellungen:

   - Geben Sie in das Textfeld **Name** den Namen **az140-31-wvdpolicy1** ein.
   - Wählen Sie im Abschnitt **Zuweisungen** die Option **Benutzer- oder Workloadidentitäten** aus. Vergewissern Sie sich in der Dropdownliste **Wofür gilt diese Richtlinie?**, dass **Benutzer und Gruppen** aktiviert ist. Aktivieren Sie im Abschnitt **Benutzer und Gruppen auswählen** das Kontrollkästchen **Benutzer und Gruppen**, und klicken Sie auf dem Blatt **Auswählen** auf **aduser5** und anschließend auf **Auswählen**.
   - Klicken Sie im Abschnitt **Zuweisungen** auf **Cloud-Apps oder -Aktionen**, und stellen Sie sicher, dass unter **Wählen Sie aus, worauf diese Richtlinie angewendet werden soll** die Option **Cloud-Apps** ausgewählt ist. Wählen Sie **Apps auswählen** auf dem Blatt **Auswählen** aus, und geben Sie im Textfeld **Suchen** **Azure Virtual Desktop** ein. Aktivieren Sie in der Liste der Ergebnisse das Kontrollkästchen neben dem Eintrag **Azure Virtual Desktop**. Geben Sie im Textfeld **Suchen** dann **Microsoft-Remotedesktop** ein, aktivieren Sie das Kontrollkästchen neben dem Eintrag **Microsoft-Remotedesktop**, und wählen Sie **Auswählen** aus. 

   > **Hinweis:** Azure Virtual Desktop (App-ID 9cdead84-a844-4324-93f2-b2e6bb768d07) wird verwendet, wenn ein*e Benutzer*in einen Feed abonniert und sich während einer Verbindung beim Azure Virtual Desktop Gateway authentifiziert. Microsoft Remote Desktop (App-ID a4a365df-50f1-4397-bc59-1a1564b8bb9c) wird verwendet, wenn ein*e Benutzer*in sich beim Sitzungshost authentifiziert, wenn einmaliges Anmelden aktiviert ist.

   - Klicken Sie im Abschnitt **Zuweisungen** auf **Bedingungen** > **Client-Apps**. Legen Sie auf dem Blatt **Client-Apps** den Schalter **Konfigurieren** auf **Ja** fest, vergewissern Sie sich, dass die Kontrollkästchen **Browser** und **Mobile Apps und Desktopclients** aktiviert sind, und klicken Sie auf **Fertig**.
   - Klicken Sie im Abschnitt **Zugriffssteuerung** auf **Gewähren**. Vergewissern Sie sich auf dem Blatt **Gewähren**, dass die Option **Zugriff gewähren** aktiviert ist, aktivieren Sie das Kontrollkästchen **Multi-Faktor-Authentifizierung erfordern**, und klicken Sie auf **Auswählen**.
   - Klicken Sie im Abschnitt **Zugriffssteuerungen** auf **Sitzung**, und aktivieren Sie auf dem Blatt **Sitzung** das Kontrollkästchen **Anmeldehäufigkeit**. Geben Sie im ersten Textfeld **4** ein. Wählen Sie in der Dropdownliste **Einheiten auswählen** den Eintrag **Stunden** aus, lassen Sie das Kontrollkästchen **Beständige Browsersitzung** deaktiviert, und klicken Sie auf **Auswählen**.
   - Legen Sie den Schalter **Richtlinie aktivieren** auf **Ein** fest.

1. Klicken Sie auf dem Blatt **Neu** auf **Erstellen**. 

#### Aufgabe 2: Testen der Richtlinie für bedingten Zugriff von Microsoft Entra für alle Azure Virtual Desktop-Verbindungen

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

#### Aufgabe 3: Ändern der Richtlinie für bedingten Zugriff von Microsoft Entra, um hybrid in Microsoft Entra eingebundene Computer aus der MFA-Anforderung auszuschließen

>**Hinweis:** In dieser Aufgabe verändern Sie die Richtlinie für bedingten Zugriff von Microsoft Entra, die MFA für die Anmeldung bei einer Azure Virtual Desktop-Sitzung erfordert, so, dass für Verbindungen von in Microsoft Entra eingebundenen Computern keine MFA erforderlich ist.

1. Klicken Sie auf Ihrem Labcomputer in dem Browserfenster, in dem das Azure-Portal angezeigt wird, auf dem Blatt **Bedingter Zugriff | Richtlinien** auf den Eintrag, der die Richtlinie **az140-31-wvdpolicy1** darstellt.
1. Wählen Sie auf dem Blatt **az140-31-wvdpolicy1** im Abschnitt **Zugriffssteuerungen** **Gewähren** aus. Aktivieren Sie auf dem Blatt **Gewähren** die Kontrollkästchen **Multi-Faktor-Authentifizierung erfordern** und **Hybride Microsoft Entra-Einbindung erforderlich**. Vergewissern Sie sich, dass die Option **Eine der ausgewählten Steuerungen erfordern** aktiviert ist, und wählen Sie **Auswählen** aus.
1. Klicken Sie auf dem Blatt **az140-31-wvdpolicy1** auf **Speichern**.

>**Hinweis:** Es dauert möglicherweise einige Minuten, bis die Richtlinie wirksam wird.

#### Aufgabe 4: Testen der geänderten Richtlinie für bedingten Zugriff von Microsoft Entra

1. Suchen Sie auf Ihrem Labcomputer in dem Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-cl-vm11** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11** Microsoft Edge, und navigieren Sie zur HTML5-Webclientseite von Azure Virtual Desktop unter [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Hinweis:** Stellen Sie sicher, dass Sie dieses Mal nicht zur Authentifizierung per MFA aufgefordert werden. Das liegt daran, dass **az140-cl-vm11** hybrid in Microsoft Entra eingebunden ist.

1. Doppelklicken Sie auf der Seite **Alle Ressourcen** auf **Eingabeaufforderung**. Deaktivieren Sie im Bereich **Auf lokale Ressourcen zugreifen** das Kontrollkästchen **Drucker**, und klicken Sie auf **Zulassen**.
1. Geben Sie bei entsprechender Aufforderung unter **Geben Sie Ihre Anmeldeinformationen ein** in das Textfeld **Benutzername** den Benutzerprinzipalnamen von **aduser5** und in das Textfeld **Kennwort** das Kennwort ein, das Sie beim Erstellen dieses Benutzerkontos festgelegt haben, und klicken Sie auf **Übermitteln**.
1. Vergewissern Sie sich, dass die Remote-App **Eingabeaufforderung** erfolgreich gestartet wurde.
1. Geben Sie im Fenster der Remote-App **Eingabeaufforderung** an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**.
1. Navigieren Sie zurück zur Seite **Alle Ressourcen**, und klicken Sie in der Ecke oben rechts auf **aduser5** und im Dropdownmenü auf **Abmelden**.
1. Wählen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11** **Start** aus. Wählen Sie auf der vertikalen Leiste direkt über der Schaltfläche **Start** das Symbol für das angemeldete Benutzerkonto und dann im Popupmenü **Abmelden** aus.

### Übung 3: Beenden der im Lab bereitgestellten und verwendeten Azure-VMs und Aufheben ihrer Zuordnung

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
