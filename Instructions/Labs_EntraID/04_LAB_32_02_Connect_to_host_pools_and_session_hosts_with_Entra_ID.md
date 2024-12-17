---
lab:
  title: 'Lab: Verbindung zu Sitzungshosts herstellen (Entra ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Lab – Verbindung zu Sitzungshosts (Entra ID)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle „Besitzer“ oder „Teilnehmer“ in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den ausreichenden Berechtigungen, um Geräte dem mit diesem Azure-Abonnement verbundenen Entra-Tenant beizutreten.
- Das Lab *Bereitstellen von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)* ist abgeschlossen
- Das Lab *Verwaltung von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)* ist abgeschlossen

## Geschätzte Dauer

20 Minuten

## Labszenario

Sie verfügen über eine bestehende Azure Virtual Desktop-Umgebung, die mit Entra verbundene Sitzungshosts enthält. Sie müssen deren Funktionalität überprüfen, indem Sie eine Verbindung zu ihnen von einem Windows 11-Client aus herstellen, der nicht mit Microsoft Entra verbunden oder registriert ist.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- validieren Sie die Funktionalität von Microsoft Entra-verbundenen Azure Virtual Desktop Sitzungshosts, indem Sie sich mit ihnen von einem Windows-Client aus verbinden, der nicht Microsoft Entra-verbunden oder registriert ist.

## Labdateien

- Keine

## Anweisungen

### Übung 1: Überprüfen Sie die Funktionalität der mit Microsoft Entra verbundenen Azure Virtual Desktop-Sitzungshosts, indem Sie sich von einem Windows 11-Client aus mit ihnen verbinden
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Anpassen der RDP-Eigenschaften des Azure Virtual Desktop-Hostpools
1. Installieren des Microsoft-Remotedesktop-Clients auf einem Windows 11-Computer
1. Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs
1. Testen von Azure Virtual Desktop-Apps


#### Aufgabe 1: Anpassen der RDP-Eigenschaften des Azure Virtual Desktop-Hostpools

> **Hinweis**: Die RDP-Einstellungen, die Sie im vorangegangenen Lab implementiert haben, bieten die optimale Benutzererfahrung (über die Unterstützung für Single Sign-On). Dies erfordert jedoch zusätzliche Änderungen, die in [Konfigurieren der Single Sign-On für Azure Virtual Desktop mit Microsoft Entra ID-Authentifizierung](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on) beschrieben werden. Ohne diese Änderungen wird die Authentifizierung standardmäßig unterstützt, vorausgesetzt, der Client-Computer erfüllt eines der folgenden Kriterien:

- Es handelt sich um Microsoft Entra, das mit demselben Microsoft Entra-Mandanten verbunden ist wie der Sitzungshost.
- Es handelt sich um Microsoft Entra Hybrid, das mit demselben Microsoft Entra-Mandanten wie der Sitzungshost verbunden ist.
- Es handelt sich um Microsoft Entra, das im selben Microsoft Entra-Mandanten wie der Sitzungshost registriert ist.

Da keines dieser Kriterien auf den Labor-Computer zutrifft, müssen Sie `targetisaadjoined:i:1` als benutzerdefinierte RDP-Eigenschaft zum Host-Pool hinzufügen.

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich mit den Anmeldedaten eines Benutzerkontos mit der Besitzerrolle des Abonnements an, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldedaten des Kontos `User1-`, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters der Lab-Sitzung aufgeführt ist.

1. Wählen Sie in dem Webbrowser, der das Azure-Portal anzeigt, auf der Seite **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Einstellungen** den Eintrag **RDP-Eigenschaften**.
1. Wählen Sie auf der Seite **az140-21-hp1 \| RDP Eigenschaften** die Registerkarte **Erweitert**. 
1. Fügen Sie auf der Registerkarte **Erweitert** der Seite **az140-21-hp1 \|RDP-Eigenschaften** im Textfeld **RDP-Eigenschaften** die folgende Zeichenfolge an den vorhandenen Inhalt an (fügen Sie bei Bedarf ein führendes Semikolon (`;`) hinzu, um diese Zeichenfolge von der vorhergehenden zu trennen:

    ```txt
    targetisaadjoined:i:1
    ```

1. Entfernen Sie im Textfeld **RDP-Eigenschaften** die folgende Zeichenkette (falls vorhanden) aus dem vorhandenen Inhalt (mit dem nachgestellten Semikolonzeichen):

    ```txt
    enablerdsaadauth:i:value
    ```

1. Auf der Seite **az140-21-hp1 \| RDP-Eigenschaften** wählen Sie **Speichern**.

#### Aufgabe 2: Installieren Sie den Microsoft-Remotedesktop Client auf einem Windows 11-Computer

1. Starten Sie auf dem Lab-Computer einen Webbrowser, navigieren Sie zur Seite [Herstellen einer Verbindung mit Azure Virtual Desktop mithilfe des Remotedesktop-Clients für Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), scrollen Sie nach unten zum Abschnitt **Herunterladen und Installieren des Remotedesktop-Clients (MSI)** und wählen Sie den Link [Windows 64 Bit](https://go.microsoft.com/fwlink/?linkid=2139369) aus. 
1. Öffnen Sie den Datei-Explorer, navigieren Sie zu dem Ordner **Downloads** und starten Sie die Installation der neu heruntergeladenen MSI-Datei. 
1. Wenn Sie im Fenster **Remotedesktop Setup** dazu aufgefordert werden, akzeptieren Sie die Bedingungen der Lizenzvereinbarung und wählen Sie die Option **Für alle Benutzenden dieses Rechners installieren**. Wenn Sie dazu aufgefordert werden, akzeptieren Sie die Aufforderung der Benutzerkontensteuerung, um mit der Installation fortzufahren.
1. Sobald die Installation abgeschlossen ist, vergewissern Sie sich, dass das Kontrollkästchen **Remotedesktop starten, wenn Setup beendet wird** aktiviert ist und wählen Sie **Beenden**, um den Microsoft-Remotedesktop Client zu starten.

   > **Hinweis**: Die [Remotedesktop Store App](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store) für Windows unterstützt keine Verbindungen zu Microsoft Entra-joined Session Hosts.

#### Aufgabe 3: Abonnieren Sie einen Azure Virtual Desktop-Arbeitsbereich

1. Wechseln Sie auf dem Lab-Computer zum **Remotedesktop**-Client-Fenster, wählen Sie **Abmelden** aus und melden Sie sich nach Aufforderung mit den Anmeldedaten des Entra ID-Benutzerkontos `User1` an, die Sie auf der Registerkarte **Ressourcen** im rechten Bereich des Fensters der Lab-Oberfläche finden.

   > **Hinweis**: Wählen Sie das Benutzerkonto, das Mitglied der Entra-Gruppe mit dem Präfix **AVD-DAG** ist.

   > **Hinweis**: Alternativ dazu wählen Sie im Client-Fenster **Remotedesktop** die Option **Abonnement mit URL**, im Bereich **Abonnement eines Arbeitsbereichs** geben Sie in das Feld **E-Mail oder Arbeitsbereich-URL** **https://client.wvd.microsoft.com/api/arm/feeddiscovery** ein, wählen **Weiter** und melden sich nach Aufforderung mit den Microsoft Entra-Anmeldedaten an.

1. Stellen Sie sicher, dass auf der Seite **Remotedesktop** nur das Symbol **SessionDesktop** angezeigt wird.

   > **Hinweis**: Dies wird erwartet, da das von Ihnen gewählte Microsoft Entra-Benutzerkonto im ersten Lab *Bereitstellen von Hostpools und Sitzungshosts über das Azure-Portal (Entra-ID)* der automatisch generierten **az140-21-hp1-DAG**-Desktop-Anwendungsgruppe zugewiesen wurde.

1. Klicken Sie auf der Seite **Remotedesktop** mit der rechten Maustaste auf das Symbol **SessionDesktop** und wählen Sie im Popup-Menü **Einstellungen**.
1. Deaktivieren Sie im Bereich **SessionDesktop** den Schalter **Standardeinstellungen verwenden**.
1. Wählen Sie im Abschnitt **Anzeigeeinstellungen** im Dropdown-Menü den Eintrag **Anzeigen auswählen** und wählen Sie die Anzeigen, die Sie für die Sitzung verwenden möchten.
1. Überprüfen Sie im Bereich **SessionDesktop** die verbleibenden Optionen, einschließlich **Auf aktuelle Anzeigen maximieren**, **Einzelne Anzeige im Fenstermodus** und **Sitzung an Fenster anpassen**, ohne irgendwelche Änderungen vorzunehmen. 
1. Schließen Sie den Bereich **SessionDesktop**. 
1. Doppelklicken Sie auf der Seite **Remotedesktop** auf das Symbol **SessionDesktop**.
1. Wenn Sie aufgefordert werden, sich anzumelden, geben Sie im Dialogfeld **Windows-Sicherheit** das Kennwort des ersten Microsoft Entra-Benutzerkontos ein, das Sie in dieser Aufgabe für die Verbindung mit der Zielumgebung Azure Virtual Desktop verwendet haben.

   > **Hinweis**: Azure Virtual Desktop unterstützt es nicht, sich mit einem Benutzerkonto bei Microsoft Entra ID anzumelden und sich dann mit einem anderen Benutzerkonto bei Windows anzumelden. Die gleichzeitige Anmeldung mit zwei unterschiedlichen Konten kann dazu führen, dass Benutzer wieder eine Verbindung mit dem falschen Sitzungshost herstellen, dass falsche oder fehlende Informationen im Azure-Portal angezeigt werden und dass während der Verwendung von App Attach oder MSIX App Attach Fehlermeldungen erscheinen.

   > **Hinweis**: Es wird automatisch das Fenster **SessionDesktop** angezeigt.

1. Vergewissern Sie sich im Fenster der Remotedesktop-Sitzung, dass Sie vollen administrativen Zugriff auf die Sitzung haben (wählen Sie z. B. das **Windows**-Logo-Symbol in der Taskleiste und wählen Sie dann den Eintrag **Windows PowerShell(Admin)** aus dem Popupmenü.
1. Wählen Sie im Fenster der Remotedesktop-Sitzung das Windows-Logo-Symbol in der Taskleiste aus, wählen Sie das Avatar-Symbol aus, das das Microsoft Entra-Benutzerkonto repräsentiert, mit dem Sie sich angemeldet haben, und wählen Sie im Popupmenü **Abmelden** aus.

   > **Hinweis**: Dadurch wird die Remotedesktop-Sitzung automatisch beendet. 

1. Zurück im Fenster **Remotedesktop**, wählen Sie den Auslassungspunkt (`...`) rechts neben dem Eintrag **az140-21-ws1** Arbeitsbereich, wählen Sie **Abmelden** und, wenn Sie zur Bestätigung aufgefordert werden, wählen Sie **Fortfahren**.
1. Wählen Sie im Fenster **Remotedesktop** die Option **Abmelden** aus und melden Sie sich, wenn Sie dazu aufgefordert werden, mit den Anmeldedaten des zweiten Entra ID-Benutzerkontos an, die Sie auf der Registerkarte **Ressourcen** im rechten Bereich des Fensters der Lab-Benutzeroberfläche finden.

   > **Hinweis**: Wählen Sie das Benutzerkonto, das Mitglied der Entra-Gruppe mit dem Präfix **AVD-RemoteApp** ist.

1. Vergewissern Sie sich, dass auf der Seite **Remotedesktop** vier Symbole angezeigt werden, darunter Eingabeaufforderung, Microsoft Word, Microsoft Excel, Microsoft PowerPoint. 

   > **Hinweis**: Dies wird erwartet, da das von Ihnen gewählte Microsoft Entra-Benutzerkonto im ersten Lab *Bereitstellen von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)* den Anwendungsgruppen **az140-21-hp1-Office365-RAG** und **az140-21-hp1-Utilities-RAG** zugewiesen wurde.

1. Doppelklicken Sie auf das Symbol der Eingabeaufforderung. 
1. Wenn Sie aufgefordert werden, sich anzumelden, geben Sie im Dialogfeld **Windows-Sicherheit** das Kennwort des zweiten Microsoft Entra-Benutzerkontos ein, das Sie für die Verbindung mit der Zielumgebung Azure Virtual Desktop verwendet haben.
1. Überprüfen Sie, ob kurz darauf ein **Eingabeaufforderung**-Fenster erscheint. 
1. Geben Sie im Fenster der Eingabeaufforderung **Hostname** ein und drücken Sie die Taste **Eingabe**, um den Namen des Computers anzuzeigen, auf dem die Eingabeaufforderung läuft.

   > **Hinweis**: Stellen Sie sicher, dass der angezeigte Name mit dem Präfix **sh-** beginnt.

1. Geben Sie an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.
1. Doppelklicken Sie auf die übrigen Symbole auf der Seite **Remotedesktop**, um Microsoft Word, Microsoft Excel und Microsoft PowerPoint zu starten.
1. Schließen Sie jedes Sitzungsfenster.
