---
lab:
  title: 'Lab: Implementieren der Überwachung mithilfe von Azure Virtual Desktop Insights'
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Lab – Implementieren der Überwachung mithilfe von Azure Virtual Desktop Insights
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzerrolle oder Teilnehmerrolle in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den ausreichenden Berechtigungen, um Geräte dem mit diesem Azure-Abonnement verbundenen Entra-Mandanten beizutreten.
- Das Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals* wurde abgeschlossen
- Das Lab *Verwalten von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)* wurde abgeschlossen

## Geschätzte Dauer

25 Minuten

## Labszenario

Sie verfügen über eine vorhandene Azure Virtual Desktop-Umgebung. Sie möchten den Status und die Aktivitäten der Umgebung überwachen.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Implementieren der Überwachung einer Azure Virtual Desktop-Umgebung

## Labdateien

- Keine

## Anweisungen

### Übung 1: Implementieren der Überwachung einer Azure Virtual Desktop-Umgebung
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Überprüfen, ob das Abonnement beim Microsoft.Insights-Ressourcenanbieter registriert ist.
1. Azure Log Analytics-Arbeitsbereich erstellen
1. Einrichten der Arbeitsmappe zur Konfiguration von Virtual Desktop Insights

#### Aufgabe 1: Registrieren Sie das Azure-Abonnement mit dem Microsoft.Insights-Ressourcenanbieter

> **Hinweis**: Da Azure Virtual Desktop Insights auf den Microsoft.Insights-Ressourcenanbieter angewiesen ist, müssen Sie diesen zunächst innerhalb des Azure-Abonnements registrieren, das Sie für dieses Lab verwenden. Im Lab *Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (Entra-ID)* haben Sie diese Aufgabe mithilfe von Azure PowerShell ausgeführt. In dieser Übung werden Sie dies mithilfe der Azure-Portal erreichen (beide Methoden werden unterstützt und sind verfügbar).

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich an, indem Sie die Anmeldedaten eines Benutzerkontos mit der Besitzerrolle in dem Abonnement angeben, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Abonnements**, wählen Sie diese aus, wählen Sie auf der Seite **Abonnements** das Azure-Abonnement aus, das Sie in diesem Lab verwenden, und wählen Sie im vertikalen Navigationsmenü im Abschnitt **Einstellungen** **Ressourcenanbieter** aus.
1. Geben Sie auf der Registerkarte **Ressourcenanbieter** im Suchtextfeld **Microsoft.Insights** ein, wählen Sie in der Ergebnisliste den kleinen Kreis links neben dem Eintrag **Microsoft.Insights** aus und wählen Sie dann **Registrieren** aus.

    > **Hinweis:** Warten Sie, bis der Registrierungsprozess abgeschlossen ist. Dies dauert normalerweise 1 Minute. Verwenden Sie die Symbolleistenschaltfläche **Aktualisieren**, um den aktuellen Wert des Registrierungsstatus anzuzeigen.

#### Aufgabe 2: Erstellen eines Azure Log Analytics-Arbeitsbereichs

> **Hinweis**: Azure Virtual Desktop Insights ist ein Dashboard, das auf Azure Monitor Workbooks basiert und die Überwachung von Azure Virtual Desktop-Umgebungen erleichtert. 

> **Hinweis**: Sie können Autoskalierungsvorgänge nur mit Erkenntnissen mit gepoolten Hostpools überwachen. 

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Log Analytics-Arbeitsbereichen**, und wählen Sie diese aus. Wählen Sie auf der Seite **Log Analytics-Arbeitsbereiche** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** der Seite **Erstellen von Log Analytics-Arbeitsbereich** die folgenden Einstellungen an und wählen Sie **Überprüfen + erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
    |Resource group|der Name einer neuen Ressourcengruppe **az140-411e-RG**|
    |Name|**az140-laworkspace41e**|
    |Region|der Namen der Azure-Region, in der Sie die Azure Virtual Desktop-Umgebung bereitgestellt haben|

1. Wählen Sie auf der Seite **Bewerten + erstellen** die Option **Erstellen**.

    > **Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Dies dauert normalerweise 1 Minute.

    > **Hinweis**: Als Nächstes müssen Sie die Sammlung von Daten im neu bereitgestellten Log Analytics-Arbeitsbereich für Diagnosen aus der Azure Virtual Desktop-Umgebung, Leistungsindikatoren von den Sitzungshosts und Windows-Ereignisprotokolle von den Azure Virtual Desktop-Sitzungshosts aktivieren.

#### Aufgabe 3: Einrichten der Arbeitsmappe zur Konfiguration von Virtual Desktop Insights

> **Hinweis**: Wenn Sie Azure Virtual Desktop Insights zum ersten Mal öffnen, müssen Sie Azure Virtual Desktop Insights so einrichten, dass es auf Ihre Azure Virtual Desktop-Umgebung ausgerichtet ist.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** im vertikalen Navigationsmenü im Abschnitt **Überwachung** die Option **Arbeitsmappen** aus.
1. Wählen Sie in der Liste der **Windows Virtual Desktop**-Arbeitsmappen im Abschnitt **Windows Virtual Desktop** die Arbeitsmappe **Insights** aus.
1. Überprüfen Sie auf der Seite Insights von **Azure Virtual Desktop \|-Arbeitsmappe \|** die Warnmeldungen, die darauf hinweisen, dass der Arbeitsbereich und die Sitzungshosts keine Daten an den Arbeitsbereich senden, und wählen Sie dann den Link **Konfigurationsarbeitsmappe** aus, um das Problem zu beheben.
1. Wählen Sie auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Ressourcendiagnose-Einstellungen** in der Dropdown-Liste **Log Analytics-Arbeitsbereich** den Eintrag **az140-laworkspace41e** aus.
1. Auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Ressourcendiagnose-Einstellungen** im Abschnitt **Hostpool az140-21-hp1** wird eine Warnmeldung angezeigt, die darauf hinweist, dass keine vorhandene Diagnosekonfiguration für den ausgewählten Hostpool gefunden wurde. Wählen Sie dann **Hostpool konfigurieren** aus.
1. Wählen Sie im Bereich **Vorlage bereitstellen** die Option **Bereitstellen** aus.

    > **Hinweis**: Dadurch werden die folgenden Diagnosetabellen im Ziel-Log Analytics-Arbeitsbereich effektiv aktiviert:
    - Verwaltungsaktivitäten
    - Feed
    - Verbindungen
    - Errors
    - Prüfpunkte
    - HostRegistration
    - AgentHealthStatus

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dieser Schritt dauert normalerweise weniger als eine Minute.

1. Auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Ressourcendiagnose-Einstellungen** das Symbol **Aktualisieren** (ein kreisförmiger Pfeil) in der Symbolleiste auswählen.
1. Überprüfen Sie den Abschnitt **Host-Pool az140-21-hp1** und stellen Sie sicher, dass die Diagnoseeinstellungen für **allLogs** aktiviert sind.
1. Scrollen Sie auf der Registerkarte **Ressourcendiagnose-Einstellungen** nach unten zum Abschnitt **Arbeitsbereich az140-21-ws1** und wählen Sie dann **Arbeitsbereich konfigurieren**.
1. Wählen Sie im Bereich **Vorlage bereitstellen** die Option **Bereitstellen** aus.

    > **Hinweis**: Dadurch wird der Arbeitsbereich für **allLogs** effektiv konfiguriert.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dieser Schritt dauert normalerweise weniger als eine Minute.

1. Auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Ressourcendiagnose-Einstellungen** das Symbol **Aktualisieren** (ein kreisförmiger Pfeil) in der Symbolleiste auswählen.
1. Überprüfen Sie den Abschnitt **Arbeitsbereich az140-21-ws1** und stellen Sie sicher, dass die Diagnoseeinstellungen für **allLogs** aktiviert sind und dass keine Warnmeldungen mehr angezeigt werden.
1. Navigieren Sie zum Anfang der Seite **CheckAMAConfiguration** und wechseln Sie zur Registerkarte **Host-Dateneinstellungen auswählen**.
1. Wählen Sie auf der Registerkarte **Einstellungen für Hostdaten auswählen** im Abschnitt **DCR erstellen** in der Dropdown-Liste **Arbeitsbereichsziel** **az140-laworkspace41e** aus und wählen Sie dann **Regel für die Datenerfassung erstellen** aus.
1. Wählen Sie im Bereich **Vorlage bereitstellen** die Option **Bereitstellen** aus.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dieser Schritt dauert normalerweise weniger als eine Minute.

1. Wählen Sie auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Einstellungen für Hostdaten auswählen** das Symbol **Aktualisieren** (ein kreisförmiger Pfeil) in der Symbolleiste aus.

    > **Hinweis**: Bevor Sie fortfahren, stellen Sie sicher, dass der neu erstellte DCR im Unterabschnitt **Verfügbare DCRs** des Abschnitts **DCR erstellen** aufgeführt ist. Wenn dies nicht der Fall ist, warten Sie eine weitere Minute, und aktualisieren Sie die Seite erneut.

1. Wählen Sie auf der Registerkarte **Einstellungen für Hostdaten auswählen** in der Dropdown-Liste **Ausgewählte DCR** den Eintrag aus, der mit dem Präfix **microsoft-avdi-** beginnt.
1. Wählen Sie auf der Registerkarte **Einstellungen für Hostdaten auswählen** im Abschnitt **DCR-Zuordnungen** die Option **Zuordnung bereitstellen** aus.
1. Wählen Sie im Bereich **Vorlage bereitstellen** die Option **Bereitstellen** aus.

    > **Hinweis**: Dadurch wird der neu erstellte DCR den Sitzungshosts im Host-Pool **az140-21-hp1** zugeordnet.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dieser Schritt dauert normalerweise weniger als eine Minute.

1. Wählen Sie auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Einstellungen für Hostdaten auswählen** das Symbol **Aktualisieren** (ein kreisförmiger Pfeil) in der Symbolleiste aus.
1. Wählen Sie auf der Registerkarte **Einstellungen für Hostdaten auswählen** im Abschnitt **Sitzungshosts ohne Azure Monitor-Erweiterung** die Option **Erweiterung hinzufügen** aus.
1. Wählen Sie im Bereich **Vorlage bereitstellen** die Option **Bereitstellen** aus.

    > **Hinweis**: Dadurch wird die Azure Monitor-Erweiterung effektiv auf den Sitzungshosts im **az140-21-hp1-Hostpool** installiert.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa eine Minute dauern.

1. Wählen Sie auf der Seite **CheckAMAConfiguration** auf der Registerkarte **Einstellungen für Hostdaten auswählen** das Symbol **Aktualisieren** (ein kreisförmiger Pfeil) in der Symbolleiste aus.
1. Stellen Sie sicher, dass keine Fehlermeldungen oder Warnmeldungen angezeigt werden. 
1. Navigieren Sie zum Anfang der Seite **CheckAMAConfiguration**, wählen Sie die Registerkarte **Generierte Daten** aus und klicken Sie dann in der Symbolleiste auf das Symbol **Aktualisieren** (ein kreisförmiger Pfeil).
1. Sehen Sie sich die Abschnitte mit den Diagrammen an, die die gesammelten Daten darstellen, einschließlich **Abgerechnete Daten der letzten 24 Stunden**, **Leistungsindikatoren** und **Ereignisse**.

    > **Hinweis**: Verwenden Sie den Abschnitt **Abgerechnete Daten der letzten 24 Stunden**, um die Datenerfassung zu überwachen. Sie sind für Log Analytics-Gebühren für die Datenspeicherung und -erfassung verantwortlich.

1. Navigieren Sie im Webbrowser, der das Azure-Portal anzeigt, zurück zur Seite **Azure Virtual Desktop** und wählen Sie im Abschnitt **Überwachung** des vertikalen Navigationsmenüs **Insights** aus.
1. Sehen Sie sich auf der Seite **Azure Virtual Desktop \| Insights** den Inhalt der Registerkarte **Übersicht** an, einschließlich des Abschnitts **Kapazität**, **Verbindungsdiagnose: % der Benutzenden können eine Verbindung herstellen**, **Verbindungsleistung: Zeit bis zur Verbindung (neue Sitzungen)** und **Nutzung** Telemetrie. 
1. Überprüfen Sie als Nächstes alle verbleibenden Registerkarten auf der Seite **Azure Virtual Desktop \| Insights**, einschließlich **Verbindungszuverlässigkeit**, **Verbindungsdiagnose**, **Verbindungsleistung**, **Benutzende**, **Nutzung**, **Clients**, und **Benachrichtigungen**.

    > **Hinweis**: Sehen Sie sich diese Registerkarten der Seite Insights erneut an, sobald Sie die nachfolgenden Labs abgeschlossen haben, um die Diagramme mit der gesammelten Telemetrie zu überprüfen.
