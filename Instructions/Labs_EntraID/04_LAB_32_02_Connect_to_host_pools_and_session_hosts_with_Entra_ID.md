---
lab:
  title: 'Lab: Verbinden mit Sitzungshosts (Entra-ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Lab: Verbinden mit Sitzungshosts (Entra-ID)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzerrolle oder Teilnehmerrolle in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den ausreichenden Berechtigungen, um Geräte dem mit diesem Azure-Abonnement verbundenen Entra-Mandanten beizutreten.
- Das Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals* wurde abgeschlossen
- Das Lab *Verwalten von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)* wurde abgeschlossen

## Geschätzte Dauer

20 Minuten

## Labszenario

Sie verfügen über eine vorhandene Azure Virtual Desktop-Umgebung, die in Entra eingebundene Sitzungshosts enthält. Sie müssen ihre Funktionalität überprüfen, indem Sie eine Verbindung zu ihnen von einem Windows 11-Client aus herstellen, der nicht mit Microsoft Entra verbunden oder registriert ist.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Überprüfen der Funktionalität von Microsoft Entra-verbundenen Azure Virtual Desktop-Sitzungshosts, indem Sie sich mit ihnen von einem Windows-Client aus verbinden, der nicht Microsoft Entra-verbunden oder registriert ist.

## Labdateien

- Keine

## Anweisungen

### Übung 1: Überprüfen der Funktionalität der mit Microsoft Entra verbundenen Azure Virtual Desktop-Sitzungshosts, indem Sie von einem Windows 11-Client aus eine Verbindung zu ihnen herstellen
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Anpassen der RDP-Eigenschaften des Azure Virtual Desktop-Hostpools
1. Installieren des Microsoft Remote Desktop-Clients auf einem Windows 11-Computer
1. Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs
1. Testen von Azure Virtual Desktop-Apps


#### Aufgabe 1: Anpassen von RDP-Eigenschaften des Azure Virtual Desktop-Hostpools

> **Hinweis**: Die RDP-Einstellungen, die Sie im vorherigen Lab implementiert haben, bieten die optimale Benutzererfahrung (durch Unterstützung für Single Sign-On). Dies erfordert jedoch zusätzliche Änderungen, die in [Single Sign-On für Azure Virtual Desktop mit Microsoft Entra ID-Authentifizierung](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on) konfigurieren beschrieben sind. Ohne diese Änderungen wird standardmäßig die Authentifizierung unterstützt, wenn der Clientcomputer einem der folgenden Kriterien entspricht:

- Es handelt sich um Microsoft Entra, das mit demselben Microsoft Entra-Mandanten verbunden ist wie der Sitzungshost
- Es handelt sich um Microsoft Entra Hybrid, das mit demselben Microsoft Entra-Mandanten wie der Sitzungshost verbunden ist
- Es handelt sich um Microsoft Entra, das im gleichen Microsoft Entra-Mandanten wie der Sitzungshost registriert ist

Da keines dieser Kriterien auf den Lab-Computer zutrifft, muss `targetisaadjoined:i:1` als benutzerdefinierte RDP-Eigenschaft zum Hostpool hinzugefügt werden.

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich an, indem Sie die Anmeldedaten eines Benutzerkontos mit der Besitzerrolle in dem Abonnement angeben, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Wählen Sie im Webbrowser, der das Azure-Portal anzeigt, auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Einstellungen** den Eintrag **RDP-Eigenschaften** aus.
1. Auf der Seite **az140-21-hp1 \| RDP Eigenschaften** die Registerkarte **Erweitert** auswählen. 
1. Fügen Sie auf der Registerkarte **Erweitert** der Seite „**az140-21-hp1 \| RDP-Eigenschaften** im Textfeld „**RDP-Eigenschaften** die folgende Zeichenfolge an den vorhandenen Inhalt an (fügen Sie bei Bedarf ein führendes Semikolon (`;`) hinzu, um diese Zeichenfolge von der vorherigen zu trennen:

    ```txt
    targetisaadjoined:i:1
    ```

1. Entfernen Sie im Textfeld **RDP Eigenschaften** die folgende Zeichenfolge (falls vorhanden) aus dem vorhandenen Inhalt (mit dem abschließenden Semikolon):

    ```txt
    enablerdsaadauth:i:value
    ```

1. Wählen Sie auf der Seite „**az140-21-hp1 \| RDP Eigenschaften** **Speichern** aus.

#### Aufgabe 2: Installieren Sie den Microsoft Remote Desktop-Client auf einem Windows 11-Computer

1. Starten Sie auf dem Lab-Computer einen Webbrowser, navigieren Sie zur Seite [Mit Azure Virtual Desktop mit dem Remotedesktop-Client für Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows) verbinden, scrollen Sie nach unten zum Abschnitt **Remotedesktop-Client (MSI) herunterladen und installieren** und wählen Sie den Link [Windows 64-Bit](https://go.microsoft.com/fwlink/?linkid=2139369) aus. 
1. Öffnen Sie den Datei-Explorer, navigieren Sie zum Ordner **Downloads** und starten Sie die Installation der neu heruntergeladenen MSI-Datei. 
1. Akzeptieren Sie im Fenster **Remotedesktop Setup**, bei entsprechender Eingabeaufforderung die Bedingungen der Lizenzvereinbarung und wählen Sie die Option **Für alle Benutzenden dieses Computers installieren**. Wenn Sie dazu aufgefordert werden, akzeptieren Sie die Eingabeaufforderung zur Benutzerkontensteuerung, um mit der Installation fortzufahren.
1. Stellen Sie nach Abschluss der Installation sicher, dass das Kontrollkästchen **Remotedesktop starten, wenn das Setup beendet ist** aktiviert ist und wählen Sie **Beenden** um den Microsoft Remotedesktop-Client zu starten.

   > **Hinweis**: Die [App Remotedesktop Store](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store) für Windows unterstützt keine Verbindung zu Microsoft Entra-verbundenen Sitzungshosts.

#### Aufgabe 3: Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs

1. Wechseln Sie auf dem Lab-Computer zum **Remotedesktop**-Clientfenster, wählen Sie **Abonnieren** aus und melden Sie sich bei der Eingabeaufforderung mit den Anmeldeinformationen des `User1` Entra ID-Benutzerkontos an, das Sie auf der Registerkarte **Ressourcen** im rechten Bereich des Fensters der Benutzeroberfläche des Labs finden.

   > **Hinweis**: Wählen Sie das Benutzerkonto aus, das Mitglied der Entra-Gruppe mit dem Präfix **AVD-DAG** ist.

   > **Hinweis**: Alternativ wählen Sie im **Remotedesktop** Clientfenster **Abonnieren mit URL**, im Bereich **einen Arbeitsbereich abonnieren**, in der **E-Mail oder Arbeitsbereich URL**, geben Sie **https://client.wvd.microsoft.com/api/arm/feeddiscovery** ein, wählen Sie **Weiter**, und melden Sie sich nach der Eingabeaufforderung mit den Anmeldeinformationen für Microsoft Entra an.

1. Stellen Sie sicher, dass auf der **Remotedesktop-Seite** nur das Symbol **SessionDesktop** angezeigt wird.

   > **Hinweis**: Dies ist zu erwarten, da das von Ihnen ausgewählte Microsoft Entra-Benutzerkonto im ersten Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (Entra-ID)* der automatisch generierten **az140-21-hp1-DAG**-Desktopanwendungsgruppe zugewiesen wurde.

1. Klicken Sie auf der Seite **Remotedesktop** mit der rechten Maustaste auf das Symbol **SessionDesktop** und wählen Sie im Popupmenü **Einstellungen** aus.
1. Deaktivieren Sie im Bereich **SessionDesktop** den Schalter **Standardeinstellungen verwenden**.
1. Wählen Sie im Abschnitt **Anzeigeeinstellungen** im Dropdownmenü den Eintrag **Anzeigen auswählen** aus und wählen Sie die Anzeigen aus, die Sie für die Sitzung verwenden möchten.
1. Überprüfen Sie im Bereich **SessionDesktop** die verbleibenden Optionen, einschließlich „**Auf aktuelle Anzeigen maximieren**, **Einzelanzeige im Fenstermodus** und **Sitzung an Fenster anpassen**, ohne Änderungen vorzunehmen. 
1. Schließen Sie den Bereich **SessionDesktop** . 
1. Doppelklicken Sie auf der Seite **Remotedesktop** auf das Symbol **SessionDesktop**.
1. Wenn Sie im Dialogfeld **Windows-Sicherheit** zur Eingabeaufforderung aufgefordert werden, drücken Sie die Eingabetaste und geben Sie das Kennwort des ersten Microsoft Entra-Benutzerkontos ein, das Sie in dieser Aufgabe für die Verbindung mit der Azure Virtual Desktop-Zielumgebung verwendet haben.

   > **Hinweis**: Azure Virtual Desktop unterstützt nicht die Anmeldung bei Microsoft Entra ID mit einem Benutzerkonto und die anschließende Anmeldung bei Windows mit einem separaten Benutzerkonto. Die gleichzeitige Anmeldung mit zwei unterschiedlichen Konten kann dazu führen, dass Benutzer wieder eine Verbindung mit dem falschen Sitzungshost herstellen, dass falsche oder fehlende Informationen im Azure-Portal angezeigt werden und dass während der Verwendung von App Attach oder MSIX App Attach Fehlermeldungen erscheinen.

   > **Hinweis**: Das Fenster **SessionDesktop** wird automatisch angezeigt.

1. Vergewissern Sie sich im Fenster der Remotedesktop-Sitzung, dass Sie innerhalb der Sitzung vollen Administratorzugriff haben (wählen Sie beispielsweise das **Windows**-Logo in der Taskleiste aus und wählen Sie dann das Element **Windows PowerShell(Admin)** aus dem Pop-up-Menü aus).
1. Wählen Sie im Fenster der Remotedesktop-Sitzung das Windows-Logo in der Taskleiste aus, wählen Sie das Avatar-Symbol aus, das das Microsoft Entra-Benutzerkonto darstellt, mit dem Sie sich angemeldet haben, und wählen Sie im Pop-up-Menü **Abmelden** aus.

   > **Hinweis**: Dadurch wird die Remotedesktopsitzung automatisch beendet. 

1. Zurück im Fenster **Remotedesktop** das Symbol mit den Auslassungspunkten (`...`) rechts neben dem Arbeitsbereichseintrag **az140-21-ws1** auswählen, **Nict mehr abonnieren** auswählen und bei der Eingabeaufforderung zur Bestätigung **Weiter** auswählen.
1. Wählen Sie im **Remote Desktop**-Clientfenster **Abonnieren** aus und melden Sie sich bei der Eingabeaufforderung mit den Anmeldeinformationen des zweiten Entra ID-Benutzerkontos an, das Sie auf der Registerkarte **Ressourcen** im rechten Bereich des Lab-Schnittstellenfensters finden.

   > **Hinweis**: Wählen Sie das Benutzerkonto aus, das Mitglied der Entra-Gruppe mit dem Präfix **AVD-RemoteApp** ist.

1. Stellen Sie sicher, dass auf der Seite **Remotedesktop** vier Symbole angezeigt werden, darunter Eingabeaufforderung, Microsoft Word, Microsoft Excel und Microsoft PowerPoint. 

   > **Hinweis**: Dies ist zu erwarten, da das von Ihnen ausgewählte Microsoft Entra-Benutzerkonto im ersten Lab*Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (Entra-ID)* dem **az140-21-hp1-Office365-RAG** und **az140-21-hp1-Utilities-RAG** Anwendungsgruppen zugewiesen wurde.

1. Doppelklicken Sie auf das Eingabeaufforderungssymbol. 
1. Geben Sie bei der Eingabeaufforderung zur Anmeldung im Dialogfeld **Windows-Sicherheit** das Kennwort des zweiten Microsoft Entra-Benutzerkontos ein, das Sie für die Verbindung mit der Zielumgebung von Azure Virtual Desktop verwendet haben.
1. Überprüfen Sie, ob kurz darauf ein **Eingabeaufforderungsfenster** angezeigt wird. 
1. Geben Sie im Eingabeaufforderungsfenster **hostname** ein und drücken Sie die **Eingabetaste**, um den Namen des Computers anzuzeigen, auf dem die Eingabeaufforderung ausgeführt wird.

   > **Hinweis**: Vergewissern Sie sich, dass der angezeigte Name mit dem Präfix **sh-** beginnt.

1. Geben Sie an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.
1. Doppelklicken Sie auf die übrigen Symbole auf der Seite **Remotedesktop**, um Microsoft Word, Microsoft Excel und Microsoft PowerPoint zu starten.
1. Schließen Sie jedes Sitzungsfenster.
