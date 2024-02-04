---
lab:
  title: "Lab: Implementieren und Verwalten von Speicher für Azure Virtual Desktop (Microsoft Entra\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Lab – Implementieren und Verwalten von Speicher für Azure Virtual Desktop (Microsoft Entra DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement
- Ein Microsoft-Konto oder ein Microsoft Entra-Konto mit der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten, der dem Azure-Abonnement zugeordnet ist, und mit der Rolle „Besitzer“ oder „Mitwirkender“ im Azure-Abonnement
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (Microsoft Entra DS)**

## Geschätzte Dauer

30 Minuten

## Labszenario

Sie müssen Speicher für eine Azure Virtual Desktop-Bereitstellung in einer Microsoft Entra DS-Umgebung implementieren und verwalten.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Konfigurieren von Azure Files, um Profilcontainer für Azure Virtual Desktop in einer Microsoft Entra DS-Umgebung zu speichern

## Labdateien

- Keine

## Anweisungen

### Übung 1: Konfigurieren von Azure Files für das Speichern von Profilcontainern für Azure Virtual Desktop

Die Hauptaufgaben für diese Übung sind Folgende:

1. Erstellen eines Azure-Speicherkontos
1. Erstellen einer Azure Files-Freigabe
1. Aktivieren der Microsoft Entra DS-Authentifizierung für das Azure Storage-Konto 
1. Konfigurieren der Azure Files-Freigabeberechtigungen
1. Konfigurieren von Azure Files-Verzeichnis und Berechtigungen auf Dateiebene

#### Aufgabe 1: Erstellen eines Azure-Speicherkontos

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11a** aus. Daraufhin wird das Blatt **az140-cl-vm11a** geöffnet.
1. Wählen Sie auf dem Blatt **az140-cl-vm11a** die Option **Verbinden** aus. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-cl-vm11a \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**aadadmin1@adatum.com**|
   |Kennwort|Bereits festgelegtes Kennwort|

1. Starten Sie in der Bastion-Sitzung für die Azure-VM **az140-cl-vm11a** Microsoft Edge, navigieren Sie zum [Azure-Portal](https://portal.azure.com) und melden Sie sich an. Geben Sie dabei den Benutzernamen des Benutzerkontos **aadadmin1** und das Kennwort an, das Sie beim Erstellen dieses Kontos festgelegt haben.

   >**Hinweis:** Sie können den Benutzerprinzipalnamen (User Principal Name, UPN) des Kontos **aadadmin1** ermitteln, indem Sie sich in der Benutzer*innen- und Computer-Konsole von Active Directory das zugehörige Eigenschaftendialogfeld ansehen oder indem Sie zu Ihrem Labcomputer zurückwechseln und sich dessen Eigenschaften im Azure-Portal auf dem Blatt Microsoft Entra-Mandant ansehen.

1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Speicherkonten** und wählen Sie diese Option aus. Klicken Sie dann auf dem Blatt **Speicherkonten** auf **+ Erstellen**.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Speicherkonto erstellen** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|Der Name einer neuen Ressourcengruppe: **az140-22a-RG**|
   |Speicherkontoname|Global gültiger Name zwischen 3 und 15 Zeichen, der aus Kleinbuchstaben und Ziffern besteht und mit einem Buchstaben beginnt|
   |Standort|Name einer Azure-Region, in der die Azure Virtual Desktop-Labumgebung gehostet wird|
   |Leistung|**Standard**|
   |Replikation|**Lokal redundanter Speicher (LRS)**|

   >**Hinweis:** Stellen Sie sicher, dass die Länge des Speicherkontonamens 15 Zeichen nicht überschreitet. Der Name wird zum Erstellen eines Computerkontos in der Active Directory Domain Services-Domäne (AD DS) verwendet, die mit dem Microsoft Entra-Mandanten integriert ist, der wiederum dem Azure-Abonnement mit dem Speicherkonto zugeordnet ist. Dadurch wird die AD DS-basierte Authentifizierung beim Zugriff auf Dateifreigaben möglich, die in diesem Speicherkonto gehostet werden.

1. Klicken Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Speicherkonto erstellen** auf **Überprüfen + erstellen**, warten Sie, bis der Überprüfungsprozess abgeschlossen ist, und klicken Sie dann auf **Erstellen**.

   >**Hinweis**: Warten Sie, bis das Speicherkonto erstellt wurde. Dies sollte etwa zwei Minuten dauern.

#### Aufgabe 2: Erstellen einer Azure Files-Freigabe

1. Navigieren Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, zurück zum Blatt **Speicherkonten**, und wählen Sie den Eintrag aus, der das neu erstellte Speicherkonto darstellt.
1. Klicken Sie auf dem Blatt für das Speicherkonto im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** auf **Dateifreigaben** und anschließend auf **+ Dateifreigabe**.
1. Legen Sie auf dem Blatt **Neue Dateifreigabe** die folgenden Einstellungen fest, und klicken Sie auf **Erstellen** (Standardwerte für die anderen Einstellungen übernehmen):

   |Einstellung|Wert|
   |---|---|
   |Name|**az140-22a-profiles**|

#### Aufgabe 3: Aktivieren der Microsoft Entra DS-Authentifizierung für das Azure Storage-Konto

1. Gehen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster im Azure-Portal zu dem Blatt, auf dem die Eigenschaften des Speicherkontos angezeigt werden, das Sie in der vorherigen Aufgabe erstellt haben. Wählen Sie dort im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** die Option **Dateifreigaben** aus. 
1. Wählen Sie im Abschnitt **Dateifreigabeeinstellungen** neben der Bezeichnung **Active Directory** den Link **Nicht konfiguriert** aus.
1. Wählen Sie im Abschnitt **Active Directory-Quelle aktivieren** im Rechteck mit der Bezeichnung **Azure Active Directory Domain Services** die Option **Einrichten** aus.
1.  Wählen Sie auf dem Blatt **Identitätsbasierter Zugriff** die Option **Aktiviert** und anschließend **Speichern** aus.

#### Aufgabe 4: Konfigurieren der RBAC-basierten Azure Files-Berechtigungen

1. Gehen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** im Microsoft Edge-Fenster im Azure-Portal zu dem Blatt, auf dem die Eigenschaften des in dieser Übung erstellten Speicherkontos angezeigt werden. Wählen Sie dort im vertikalen Menü auf der linken Seite im Abschnitt **Datenspeicher** die Option **Dateifreigaben** und in der Liste mit den Freigaben den Eintrag **az140-22a-profiles** aus.
1. Wählen Sie auf dem Blatt **az140-22a-profiles** im vertikalen Menü auf der linken Seite **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf dem Blatt **az140-22a-profiles \| Zugriffssteuerung (IAM)** die Option **+ Hinzufügen** und im Dropdownmenü den Eintrag **Rollenzuweisung hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Rollenzuweisung hinzufügen** die Option **Mitwirkender für Speicherdateidaten-SMB-Freigabe** und dann **Weiter** aus:
1. Wählen Sie auf dem Blatt **Mitglieder** die Option **Zugriff zuweisen zu** aus, und klicken Sie auf **+ Mitglieder auswählen**.
1. Geben Sie auf dem Blatt **Mitglieder auswählen** im Textfeld **Auswählen** den Text **az140-wvd-ausers** ein, und klicken Sie auf **Auswählen**.
1. Wählen Sie auf dem Blatt **Mitglieder** die Option **Überprüfen + zuweisen** zweimal aus.
1. Wiederholen Sie die oben beschriebenen Schritte 3–8, und geben Sie dabei die folgenden Einstellungen an:

   |Einstellung|Wert|
   |---|---|
   |Rolle|**Speicherdateidaten-SMB-Freigabemitwirkender mit erhöhten Rechten**|
   |Auswählen|**az140-wvd-aadmins**|

   > **Hinweis:** Verwenden Sie zum Konfigurieren der Berechtigungen für die Dateifreigabe das Benutzerkonto **aadadmin1**. Dieses Konto ist Mitglied der Gruppe **az140-wvd-aadmins**. 

#### Aufgabe 5: Konfigurieren von Azure Files-Verzeichnis und Berechtigungen auf Dateiebene

1. Rufen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** die **Eingabeaufforderung** auf, und führen Sie im Fenster **Eingabeaufforderung** den folgenden Befehl aus, um der Zielfreigabe ein Laufwerk zuzuordnen (ersetzen Sie den Platzhalter `<storage-account-name>` durch den Namen des Speicherkontos):

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. Öffnen Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11a** den Datei-Explorer. Navigieren Sie zum neu zugeordneten Laufwerk Z:. Rufen Sie das entsprechende Dialogfeld **Eigenschaften** auf. Wählen Sie auf der Registerkarte **Sicherheit** **Bearbeiten** und dann **Hinzufügen** aus. Vergewissern Sie sich im Dialogfeld **Benutzer*innen, Computer, Dienstkonten und Gruppen auswählen**, dass das Textfeld **Suchpfad** den Eintrag **adatum.com** enthält. Geben Sie im Textfeld **Namen des auszuwählenden Objekts eingeben** den Namen **az140-wvd-ausers** ein, und klicken Sie auf **OK**.
1. Kehren Sie zur Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks zurück, und vergewissern Sie sich, dass der Eintrag **az140-wvd-ausers** ausgewählt ist. Aktivieren Sie das Kontrollkästchen **Ändern** in der Spalte **Zulassen**. Klicken Sie auf **OK**. Lesen Sie die Meldung, die im Textfeld **Windows-Sicherheit** angezeigt wird, und klicken Sie auf **Ja**. 
1. Kehren Sie zur Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks zurück, und wählen Sie **Bearbeiten** und **Hinzufügen** aus. Vergewissern Sie sich im Dialogfeld **Select Users, Computers, Service Accounts, and Groups** (Benutzer, Computer, Dienstkonten und Gruppen auswählen), dass das Textfeld **Suchpfad** den Eintrag **adatum.com** enthält. Geben Sie im Textfeld **Namen des auszuwählenden Objekts eingeben** den Namen **az140-wvd-aadmins** ein, und klicken Sie auf **OK**.
1. Kehren Sie zur Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks zurück, und vergewissern Sie sich, dass der Eintrag **az140-wvd-aadmins** ausgewählt wurde. Aktivieren Sie das Kontrollkästchen **Vollzugriff** in der Spalte **Zulassen**, und klicken Sie auf **OK**. 
1. Wählen Sie auf der Registerkarte **Sicherheit** des Dialogfelds mit den Berechtigungen des zugeordneten Laufwerks **Bearbeiten**, in der Liste mit Gruppen und Benutzernamen den Eintrag **Authentifizierte Benutzer** und dann **Entfernen** aus.
1. Wählen Sie auf dem Bildschirm „Bearbeiten“ in der Liste mit Gruppen und Benutzernamen den Eintrag **Benutzer** und dann **Entfernen** aus. Klicken Sie auf **OK**, und klicken Sie dann zweimal auf **OK**, um den Vorgang abzuschließen. 

   >**Hinweis:** Alternativ können Sie Berechtigungen auch mithilfe des Befehlszeilenprogramms **icacls** festlegen. 
