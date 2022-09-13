---
lab:
  title: 'Lab: Implementieren und Verwalten von Speicher für virtuelle Android-Geräte (Azure AD DS)'
  module: 'Module 2: Implement a AVD Infrastructure'
---

# <a name="lab---implement-and-manage-storage-for-avd-azure-ad-ds"></a>Lab: Implementieren und Verwalten von Speicher für virtuelle Android-Geräte (Azure AD DS)
# <a name="student-lab-manual"></a>Lab-Handbuch für Kursteilnehmer

## <a name="lab-dependencies"></a>Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder Azure AD-Konto mit der Rolle „Globaler Administrator“ im Azure AD-Mandanten, das dem Azure-Abonnement zugeordnet ist und im Azure-Abonnement die Rolle „Besitzer“ oder „Mitwirkender“ innehat.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (Azure AD DS)**

## <a name="estimated-time"></a>Geschätzte Dauer

30 Minuten

## <a name="lab-scenario"></a>Labszenario

Sie müssen Speicher für eine Azure Virtual Desktop-Bereitstellung in einer Azure Active Directory Domain Services-Umgebung (Azure AD DS) implementieren und verwalten.

## <a name="objectives"></a>Ziele
  
In diesem Lab lernen Sie Folgendes:

- Azure Files zu konfigurieren, um Profilcontainer für Azure Virtual Desktop in einer Azure AD DS-Umgebung zu speichern

## <a name="lab-files"></a>Lab-Dateien

- Keine

## <a name="instructions"></a>Anweisungen

### <a name="exercise-1-configure-azure-files-to-store-profile-containers-for-azure-virtual-desktop"></a>Übung 1: Konfigurieren von Azure Files für das Speichern von Profilcontainern für Azure Virtual Desktop

Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen eines Azure-Speicherkontos
1. Erstellen einer Azure Files-Freigabe
1. Aktivieren der Azure AD DS-Authentifizierung für das Azure Storage-Speicherkonto 
1. Konfigurieren der Azure Files-Freigabeberechtigungen
1. Konfigurieren von Azure Files-Verzeichnis und Berechtigungen auf Dateiebene

#### <a name="task-1-create-an-azure-storage-account"></a>Aufgabe 1: Erstellen eines Azure-Speicherkontos

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11a** aus. Daraufhin wird das Blatt **az140-cl-vm11a** geöffnet.
1. Wählen Sie auf dem Blatt **az140-cl-vm11a** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11a \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen an, und wählen **Verbinden** aus:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**aadadmin1@adatum.com**|
   |Kennwort|Bereits festgelegtes Kennwort|

1. Starten Sie im Remotedesktop für den virtuellen Azure-Computer **az140-cl-vm11a** Microsoft Edge, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Geben Sie dabei den Benutzernamen des Benutzerkontos **aadadmin1** und das Kennwort an, das Sie beim Erstellen dieses Kontos festgelegt haben.

   >**Hinweis:** Sie können das UPN-Attribut (User Principal Name, Benutzerprinzipalname) des Kontos **aadadmin1** ermitteln, indem Sie sich in der Konsole „Active Directory-Benutzer und -Computer“ das zugehörige Eigenschaftendialogfeld ansehen oder indem Sie zu Ihrem Labcomputer zurückwechseln und sich dessen Eigenschaften im Azure-Portal auf dem Azure AD-Mandantenblatt ansehen.

1. Suchen Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Speicherkonten**, und wählen Sie die Option aus. Klicken Sie dann auf dem Blatt **Speicherkonten** auf **+ Erstellen**.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Speicherkonto erstellen** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|Der Name einer neuen Ressourcengruppe: **az140-22a-RG**|
   |Speicherkontoname|Global gültiger Name zwischen 3 und 15 Zeichen, der aus Kleinbuchstaben und Ziffern besteht und mit einem Buchstaben beginnt|
   |Standort|Name einer Azure-Region, in der die Azure Virtual Desktop-Labumgebung gehostet wird|
   |Leistung|**Standard**|
   |Replikation|**Lokal redundanter Speicher (LRS)**|

   >**Hinweis:** Stellen Sie sicher, dass die Länge des Speicherkontonamens 15 Zeichen nicht überschreitet. Der Name wird zum Erstellen eines Computerkontos in der Active Directory Domain Services-Domäne (AD DS) verwendet, die mit dem Azure AD-Mandanten integriert ist, der wiederum dem Azure-Abonnement mit dem Speicherkonto zugeordnet ist. Dadurch wird die AD DS-basierte Authentifizierung beim Zugriff auf Dateifreigaben möglich, die in diesem Speicherkonto gehostet werden.

1. Klicken Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Speicherkonto erstellen** auf **Überprüfen + erstellen**, warten Sie, bis der Überprüfungsprozess abgeschlossen ist, und klicken Sie dann auf **Erstellen**.

   >**Hinweis**: Warten Sie, bis das Storage-Konto erstellt wurde. Dies sollte etwa zwei Minuten dauern.

#### <a name="task-2-create-an-azure-files-share"></a>Aufgabe 2: Erstellen einer Azure Files-Freigabe

1. Navigieren Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Speicherkonten**, und wählen Sie den Eintrag aus, der das neu erstellte Speicherkonto darstellt.
1. Klicken Sie auf dem Blatt für das Speicherkonto im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** auf **Dateifreigaben** und anschließend auf **+ Dateifreigabe**.
1. Legen Sie auf dem Blatt **Neue Dateifreigabe** die folgenden Einstellungen fest, und klicken Sie auf **Erstellen** (Standardwerte für die anderen Einstellungen übernehmen):

   |Einstellung|Wert|
   |---|---|
   |Name|**az140-22a-profiles**|

#### <a name="task-3-enable-azure-ad-ds-authentication-for-the-azure-storage-account"></a>Aufgabe 3: Aktivieren der Azure AD DS-Authentifizierung für das Azure Storage-Speicherkonto

1. Wählen Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster im Azure-Portal auf dem Blatt, auf dem die Eigenschaften des Speicherkontos angezeigt werden, das Sie in der vorherigen Aufgabe erstellt haben, im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** die Option **Dateifreigaben** aus. 
1. Wählen Sie im Abschnitt **Dateifreigabeeinstellungen** neben der Bezeichnung **Active Directory** den Link **Nicht konfiguriert** aus.
1. Wählen Sie im Abschnitt **Active Directory-Quelle aktivieren** im Rechteck mit der Bezeichnung **Azure Active Directory Domain Services** die Option **Einrichten** aus.
1  Wählen Sie auf dem Blatt **Identitätsbasierter Zugriff** die Option **Aktiviert** und anschließend **Speichern** aus.

#### <a name="task-4-configure-the-azure-files-rbac-based-permissions"></a>Aufgabe 4: Konfigurieren der RBAC-basierten Azure Files-Berechtigungen

1. Wählen Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster mit dem Azure-Portal auf dem Blatt, auf dem die Eigenschaften des in dieser Übung erstellten Speicherkontos angezeigt werden, im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** die Option **Dateifreigaben** und in der Liste mit den Freigaben den Eintrag **az140-22a-profiles** aus.
1. Wählen Sie auf dem Blatt **az140-22a-profiles** im vertikalen Menü auf der linken Seite **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf dem Blatt **az140-22a-profiles \| Zugriffssteuerung (IAM)** die Option **+ Hinzufügen** und im Dropdownmenü den Eintrag **Rollenzuweisung hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** die Option **Mitwirkender für Speicherdateidaten-SMB-Freigabe** und dann **Weiter** aus:
1. Wählen Sie auf dem Blatt **Mitglieder** die Option **Zugriff zuweisen zu** aus, und klicken Sie auf **+ Mitglieder auswählen**.
1. Geben Sie auf dem Blatt **Mitglieder auswählen** im Textfeld **Auswählen** den Text **az140-wvd-ausers** ein, und klicken Sie auf **Auswählen**.
1. Wählen Sie auf dem Blatt **Mitglieder** die Option **Überprüfen + zuweisen** zweimal aus.
1. Wiederholen Sie die oben beschriebenen Schritte 3–8, und geben Sie dabei die folgenden Einstellungen an:

   |Einstellung|Wert|
   |---|---|
   |Role|**Speicherdateidaten-SMB-Freigabemitwirkender mit erhöhten Rechten**|
   |Auswählen|**az140-wvd-aadmins**|

   > **Hinweis:** Verwenden Sie zum Konfigurieren der Berechtigungen für die Dateifreigabe das Benutzerkonto **aadadmin1**. Dieses Konto ist Mitglied der Gruppe **az140-wvd-aadmins**. 

#### <a name="task-5-configure-the-azure-files-directory-and-file-level-permissions"></a>Aufgabe 5: Konfigurieren von Azure Files-Verzeichnis und Berechtigungen auf Dateiebene

1. Rufen Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** die **Eingabeaufforderung** auf, und führen Sie im Fenster **Eingabeaufforderung** den folgenden Befehl aus, um der Zielfreigabe ein Laufwerk zuzuordnen (ersetzen Sie den Platzhalter `<storage-account-name>` durch den Namen des Speicherkontos):

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. Öffnen Sie innerhalb der Remotedesktopsitzung für **az140-cl-vm11a** Explorer. Navigieren Sie zum neu zugeordneten Laufwerk Z:. Rufen Sie das entsprechende Dialogfeld **Eigenschaften** auf. Wählen Sie die Registerkarte **Sicherheit**, **Bearbeiten**, **Hinzufügen** aus. Vergewissern Sie sich im Dialogfeld **Select Users, Computers, Service Accounts, and Groups** (Benutzer, Computer, Dienstkonten und Gruppen auswählen), dass das Textfeld **Suchpfad** den Eintrag **adatum.com** enthält. Geben Sie im Textfeld **Namen des auszuwählenden Objekts eingeben** den Namen **az140-wvd-ausers** ein, und klicken Sie auf **OK**.
1. Kehren Sie zur Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks zurück, und vergewissern Sie sich, dass der Eintrag **az140-wvd-ausers** ausgewählt ist. Aktivieren Sie das Kontrollkästchen **Ändern** in der Spalte **Zulassen**. Klicken Sie auf **OK**. Lesen Sie die Meldung, die im Textfeld **Windows-Sicherheit** angezeigt wird, und klicken Sie auf **Ja**. 
1. Kehren Sie zur Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks zurück, und wählen Sie **Bearbeiten** und **Hinzufügen** aus. Vergewissern Sie sich im Dialogfeld **Select Users, Computers, Service Accounts, and Groups** (Benutzer, Computer, Dienstkonten und Gruppen auswählen), dass das Textfeld **Suchpfad** den Eintrag **adatum.com** enthält. Geben Sie im Textfeld **Namen des auszuwählenden Objekts eingeben** den Namen **az140-wvd-aadmins** ein, und klicken Sie auf **OK**.
1. Kehren Sie zur Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks zurück, und vergewissern Sie sich, dass der Eintrag **az140-wvd-aadmins** ausgewählt wurde. Aktivieren Sie das Kontrollkästchen **Vollzugriff** in der Spalte **Zulassen**, und klicken Sie auf **OK**. 
1. Wählen Sie auf der Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks **Bearbeiten**, in der Liste mit Gruppen und Benutzernamen den Eintrag **Authentifizierte Benutzer** und dann **Entfernen** aus.
1. Wählen Sie auf dem Bildschirm „Bearbeiten“ in der Liste mit Gruppen und Benutzernamen den Eintrag **Benutzer** und dann **Entfernen** aus. Klicken Sie auf **OK**, und klicken Sie dann zweimal auf **OK**, um den Vorgang abzuschließen. 

   >**Hinweis:** Alternativ können Sie Berechtigungen auch mithilfe des Befehlszeilenprogramms **icacls** festlegen. 
