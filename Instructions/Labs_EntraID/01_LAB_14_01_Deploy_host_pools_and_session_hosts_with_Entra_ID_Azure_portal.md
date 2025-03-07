---
lab:
  title: 'Lab: Bereitstellen von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)'
  module: 'Module 1.4: Implement host pools and session hosts'
---

# Lab – Bereitstellen von Hostpools und Sitzungshosts über das Azure-Portal (Entra ID)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzende in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den erforderlichen Berechtigungen, um Geräte mit dem Entra-Mandanten zu verbinden, der diesem Azure-Abonnement zugeordnet ist.

## Geschätzte Dauer

30 Minuten

## Labszenario

Sie haben ein Microsoft Azure-Abonnement. Sie müssen eine Azure Virtual Desktop-Umgebung bereitstellen, die Microsoft Entra joined session hosts verwendet.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Bereitstellen von Microsoft Entra für Azure Virtual Desktop-Sitzungshosts

## Labdateien

- Keine

## Anweisungen

### Übung 1: Implementieren einer Azure Virtual Desktop-Umgebung mit in Microsoft Entra eingebundenen Sitzungshosts
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools
1. Bereitstellen eines Azure Virtual Desktop-Hostpools
1. Erstellen einer Azure Virtual Desktop-Anwendungsgruppe
1. Erstellen eines Azure Virtual Desktop-Arbeitsbereichs
1. Zugriff auf Azure Virtual Desktop-Hostpools gewähren

#### Aufgabe 1: Vorbereiten des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools

1. Starten Sie auf dem Lab-Computer einen Webbrowser, navigieren Sie zum Azure-Portal unter [https://portal.azure.com](https://portal.azure.com) und melden Sie sich mit den Anmeldeinformationen eines Benutzerkontos mit der Besitzerrolle in dem Abonnement an, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Starten Sie im Azure-Portal eine PowerShell-Sitzung in der Azure Cloud Shell.

    > **Hinweis**: Wenn eine Eingabeaufforderung angezeigt wird, wählen Sie im Bereich **Erste Schritte** in der Dropdownliste **Abonnement** den Namen des Azure-Abonnements aus, das Sie in diesem Lab verwenden, und wählen Sie dann ‚**Anwenden** aus.

1. Führen Sie in der PowerShell-Sitzung im Bereich Azure Cloud Shell den folgenden Befehl aus, um den Ressourcenanbieter **Microsoft.DesktopVirtualization** zu registrieren:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    > **Hinweis:** Warten Sie nicht, bis die Registrierung abgeschlossen ist. Dieser Vorgang kann einige Minuten in Anspruch nehmen.

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Webbrowser, der das Azure-Portal anzeigt, nach **Virtuelle Netzwerke**, wählen Sie es aus und wählen Sie auf der Seite **Virtuelle Netzwerke** **Erstellen +** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtuelles Netzwerk erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe **az140-11e-RG**|
    |Name des virtuellen Netzwerks|**az140-vnet11e**|
    |Region|Den Namen der Azure-Region, in der Sie die Azure Virtual Desktop-Umgebung bereitstellen möchten|

1. Übernehmen Sie auf der Registerkarte **Sicherheit** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Geben Sie auf der Registerkarte **IP-Adressen** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |IP-Adressbereich|**10.20.0.0/16**|

1. Wählen Sie das Bearbeiten-Symbol (Stift) neben dem Eintrag **Standard** Subnetz im Bereich **Bearbeiten** aus, legen Sie die folgenden Einstellungen fest (lassen Sie die anderen mit den vorhandenen Werten unverändert) und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**hp1-Subnet**|
    |Startadresse|**10.20.1.0**|
    |Privates Subnetz aktivieren (kein standardmäßiger ausgehender Zugriff)|Disabled|

1. Zurück auf der Registerkarte **IP-Adressen** **Überprüfen + erstellen** auswählen und dann auf der Registerkarte **Überprüfen + erstellen** **Erstellen** auswählen.

    > **Hinweis:** Warten Sie nicht, bis der Bereitstellungsprozess abgeschlossen ist. Dieser Schritt dauert normalerweise weniger als eine Minute.

1. Suchen Sie im Webbrowser, der das Azure-Portal anzeigt, nach **Microsoft Entra ID** und wählen Sie diese aus.
1. Wählen Sie auf der Seite **Übersicht** des Ihrem Abonnement zugeordneten Microsoft Entra-Mandanten im Abschnitt „**Verwalten“** des vertikalen Navigationsmenüs **Benutzende** aus.
1. Geben Sie auf der Seite **Benutzende** im Textfeld **Suchen** den Namen des `User1-` Kontos ein, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab aufgeführt ist.
1. Wählen Sie in der Ergebnisliste der Suche den Benutzerkontoeintrag mit dem passenden Namen aus.
1. Auf der Seite mit den Eigenschaften des Benutzerkontos im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Gruppen** auswählen.
1. Notieren Sie auf der Seite **Gruppen** den Namen der Gruppe, beginnend mit dem Präfix **AVD-DAG** (Sie werden ihn später in diesem Lab benötigen).
1. Navigieren Sie zurück zur Seite **Benutzende**. Geben Sie im Textfeld **Suchen** den Namen des `User2-` Kontos ein, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Labs aufgeführt ist.
1. Wählen Sie in der Ergebnisliste der Suche den Benutzerkontoeintrag mit dem passenden Namen aus.
1. Auf der Seite mit den Eigenschaften des Benutzerkontos im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Gruppen** auswählen.
1. Notieren Sie auf der Seite **Gruppen** den Namen der Gruppe, beginnend mit dem Präfix **AVD-RemoteApp** (Sie werden ihn später in diesem Lab benötigen).

#### Aufgabe 2: Bereitstellen eines Azure Virtual Desktop-Hostpools

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus, wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Hostpools** auswählen und auf der Seite **Azure Virtual Desktop \| Hostpools** **+ Erstellen** auswählen. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** der Seite **Hostpool erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Sitzungshosts >** (belassen Sie die anderen Einstellungen auf ihren Standardwerten):

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe **az140-21e-RG**|
    |Hostpoolname|**az140-21-hp1**|
    |Location|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Überprüfungsumgebung|**Nein**|
    |Bevorzugter App-Gruppentyp|**Desktop**|
    |Hostpooltyp|**In einem Pool zusammengefasst**|
    |Sitzungshostkonfiguration erstellen|**Nein**|
    |Lastenausgleichsalgorithmus|**Breitensuche**|

    > **Hinweis**: Bei Verwendung des breitenorientierten Algorithmus für den Lastenausgleich ist der Parameter für die maximale Anzahl von Sitzungen optional.

1. Geben Sie auf der Registerkarte **Sitzungshosts** auf der Seite **Hostpool erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Arbeitsbereich >** aus (lassen Sie die anderen Einstellungen auf ihren Standardwerten):

    > **Hinweis**: Wenn Sie den Wert für **Präfixname** einstellen, wechseln Sie zur Registerkarte Ressourcen auf der rechten Seite des Lab-Sitzungsfensters. Suchen Sie dort die Zeichenfolge zwischen *User1-* und dem Zeichen *@*. Verwenden Sie diese Zeichenfolge, um den *zufälligen* Platzhalter zu ersetzen.

    |Einstellung|Wert|
    |---|---|
    |Virtuelle Computer hinzufügen|**Ja**|
    |Resource group|**Standardmäßig mit der Ressourcengruppe des Hostpools identisch.**|
    |Namenspräfix|**sh**-*random*|
    |Typ des virtuellen Computers|**Virtueller Azure-Computer**|
    |Virtueller Computer Standort|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
    |Sicherheitstyp|**Virtuelle Computer mit vertrauenswürdigem Start**|
    |Abbildung|**Windows 11 Enterprise für mehrere Sitzungen, Version 23H2 + Microsoft 365 Apps**|
    |Größe des virtuellen Computers|**Standard DC2s_v3**|
    |Number of VMs (Anzahl von VMs)|**2**|
    |Typ des Betriebssystemdatenträgers|**SSD Standard**|
    |Größe des Betriebssystemdatenträgers|**Standardgröße (128 GiB)**|
    |Startdiagnose|**Mit verwaltetem Speicherkonto aktivieren (empfohlen)**|
    |Virtuelles Netzwerk|**az140-vnet11e**|
    |Subnet|**hp1-Subnet**|
    |Netzwerksicherheitsgruppe|**Grundlegend**|
    |Öffentliche Eingangsports|**Nein**|
    |Wählen Sie das gewünschte Verzeichnis für die Einbindung aus.|**Microsoft Entra ID**|
    |VM bei Intune registrieren|**Nein**|
    |Benutzername|**Kursteilnehmer**|
    |Kennwort|Eine hinreichend komplexe Zeichenfolge, die als Kennwort für das integrierte Administratorkonto verwendet werden soll|
    |Kennwort bestätigen|Die gleiche Zeichenfolge, die Sie zuvor eingegeben haben|

    > **Hinweis**: Das Kennwort sollte mindestens 12 Zeichen lang sein und aus einer Kombination von Kleinbuchstaben, Großbuchstaben, Ziffern und Sonderzeichen bestehen. Ausführliche Informationen finden Sie unter den Informationen zu [den Kennwortanforderungen beim Erstellen einer Azure-VM](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Auf der Registerkarte **Arbeitsbereich** auf der Seite **Hostpool erstellen**„ die folgende Einstellung bestätigen und **Überprüfen + erstellen‘** auswählen:

    |Einstellung|Wert|
    |---|---|
    |Desktop-App-Gruppe registrieren|**Nein**|

1. Auf der Registerkarte **Überprüfen + erstellen** der Seite **Hostpool erstellen** die Option **Erstellen** auswählen.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa 20 Minuten dauern.

#### Aufgabe 3: Erstellen einer Azure Virtual Desktop-Anwendungsgruppe

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Anwendungsgruppen** aus.
1. Beachten Sie auf der Seite **Azure Virtual Desktop \| Anwendungsgruppen** die vorhandene, automatisch generierte **az140-21-hp1-DAG**-Desktop-Anwendungsgruppe und wählen Sie sie aus. 
1. Auf der Seite **az140-21-hp1-DAG** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Zuweisungen** auswählen.
1. Auf der Seite **az140-21-hp1-DAG \| Zuweisungen** auswählen: **+ Hinzufügen**.
1. Wählen Sie auf der Seite **Microsoft Entra-Benutzende oder -Benutzendengruppen** auswählen die Option **Gruppen** aus. Geben Sie im Suchfeld den vollständigen Namen der Gruppe **AVD-DAG** ein, die Sie in der ersten Aufgabe dieser Übung identifiziert haben. Aktivieren Sie das Kontrollkästchen neben dem Gruppennamen und klicken Sie auf **Auswählen**.
1. Navigieren Sie zurück zur Seite **Azure Virtual Desktop \| Anwendungsgruppen** und wählen Sie **+ Erstellen** aus. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Anwendungsgruppe erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Anwendungen >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-21a-RG**|
    |Hostpool|**az140-21-hp1**|
    |Anwendungsgruppentyp|**Remote-App**|
    |Name der Anwendungsgruppe|**az140-21-hp1-Office365-RAG**|

1. Auf der Registerkarte **Anwendungen** auf der Seite **Anwendungsgruppe erstellen** **+ Anwendungen hinzufügen** auswählen.
1. Auf der Seite **Anwendung hinzufügen** die folgenden Einstellungen angeben und **Überprüfen + hinzufügen** auswählen, dann **Hinzufügen** auswählen:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsquelle|**Startmenü**|
    |Application|**Word**|
    |Anzeigename|**Microsoft Word**|
    |Beschreibung|**Microsoft Word**|
    |Befehlszeile erforderlich|**Nein**|

1. Zurück auf der Registerkarte **Anwendungen** auf der Seite **Anwendungsgruppe erstellen** **+ Anwendungen hinzufügen** auswählen.
1. Auf der Seite **Anwendung hinzufügen** die folgenden Einstellungen angeben und **Überprüfen + hinzufügen** auswählen, dann **Hinzufügen** auswählen:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsquelle|**Startmenü**|
    |Application|**Excel**|
    |Anzeigename|**Microsoft Excel**|
    |Beschreibung|**Microsoft Excel**|
    |Befehlszeile erforderlich|**Nein**|

1. Zurück auf der Registerkarte **Anwendungen** auf der Seite **Anwendungsgruppe erstellen** **+ Anwendungen hinzufügen** auswählen.
1. Auf der Seite **Anwendung hinzufügen** die folgenden Einstellungen angeben und **Überprüfen + hinzufügen** auswählen, dann **Hinzufügen** auswählen:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsquelle|**Startmenü**|
    |Application|**PowerPoint**|
    |Anzeigename|**Microsoft PowerPoint**|
    |Beschreibung|**Microsoft PowerPoint**|
    |Befehlszeile erforderlich|**Nein**|

1. Zurück auf der Registerkarte **Anwendungen** auf der Seite **Anwendungsgruppe erstellen****Weiter: Zuweisungen >** auswählen.
1. Auf der Registerkarte **Aufgaben** auf der Seite **Anwendungsgruppe erstellen** **+ Microsoft Entra-Benutzende oder -Benutzendengruppen** hinzufügen auswählen.
1. Wählen Sie auf der Seite **Microsoft Entra-Benutzende oder -Benutzendengruppen** auswählen die Option **Gruppen** aus, geben Sie den vollständigen Namen der Gruppe **AVD-RemoteApp** ein, die Sie in der ersten Aufgabe dieser Übung identifiziert haben, aktivieren Sie das Kontrollkästchen neben dem Gruppennamen und klicken Sie auf **Auswählen**.
1. Zurück auf der Registerkarte **Aufgaben** auf der Seite **Anwendungsgruppe erstellen****Weiter: Arbeitsbereich >** auswählen.
1. Geben Sie auf der Registerkarte **Arbeitsbereich** auf der Seite **Arbeitsbereich erstellen** die folgende Einstellung an und wählen Sie **Überprüfen + erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsgruppen registrieren|**Nein**|

1. Auf der Registerkarte **Überprüfen + erstellen** der Seite **Anwendungsgruppe erstellen** die Option **Erstellen** auswählen.

    > **Hinweis:** Warten Sie, bis die Anwendungsgruppe erstellt wurde. Das sollte weniger als eine Minute dauern. 

    > **Hinweis:** Als Nächstes erstellen Sie eine Anwendungsgruppe, die auf dem Dateipfad als Anwendungsquelle basiert.

1. Suchen Sie im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Anwendungsgruppen** aus.
1. Auf der Seite **Azure Virtual Desktop \| Anwendungsgruppen** auswählen **+ Erstellen**. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Anwendungsgruppe erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Anwendungen >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-21a-RG**|
    |Hostpool|**az140-21-hp1**|
    |Anwendungsgruppentyp|**RemoteApp**|
    |Name der Anwendungsgruppe|**az140-21-hp1-Utilities-RAG**|

1. Auf der Registerkarte **Anwendungen** auf der Seite **Anwendungsgruppe erstellen** **+ Anwendungen hinzufügen** auswählen.
1. Auf der Seite **Anwendung hinzufügen** auf der Registerkarte **Allgemeine Informationen** die folgenden Einstellungen angeben und **Weiter** auswählen:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsquelle|**Dateipfad**|
    |Anwendungspfad|**C:\Windows\system32\cmd.exe**|
    |Anwendungskennung|**Eingabeaufforderung**|
    |Anzeigename|**Eingabeaufforderung**|
    |Beschreibung|**Windows-Eingabeaufforderung**|
    |Befehlszeile erforderlich|**Nein**|

1. Geben Sie auf der Registerkarte **Symbol** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Hinzufügen** und dann **Hinzufügen** aus:

    |Einstellung|Wert|
    |---|---|
    |Symbolpfad|**C:\Windows\system32\cmd.exe**|
    |Symbolindex|0|

1. Zurück auf der Registerkarte **Anwendungen** auf der Seite **Anwendungsgruppe erstellen****Weiter: Zuweisungen >** auswählen.
1. Auf der Registerkarte **Aufgaben** auf der Seite **Anwendungsgruppe erstellen** **+ Microsoft Entra-Benutzende oder -Benutzendengruppen** hinzufügen auswählen.
1. Wählen Sie auf der Seite **Microsoft Entra-Benutzende oder -Benutzendengruppen** auswählen die Option **Gruppen** aus, geben Sie den vollständigen Namen der Gruppe **AVD-RemoteApp** ein, die Sie in der ersten Aufgabe dieser Übung identifiziert haben, aktivieren Sie das Kontrollkästchen neben dem Gruppennamen und klicken Sie auf **Auswählen**.
1. Zurück auf der Registerkarte **Aufgaben** auf der Seite **Anwendungsgruppe erstellen****Weiter: Arbeitsbereich >** auswählen.
1. Geben Sie auf der Registerkarte **Arbeitsbereich** auf der Seite **Arbeitsbereich erstellen** die folgende Einstellung an und wählen Sie **Überprüfen + erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsgruppen registrieren|**Nein**|

1. Auf der Registerkarte **Überprüfen + erstellen** der Seite **Anwendungsgruppe erstellen** die Option **Erstellen** auswählen.

    > **Hinweis:** Warten Sie, bis die Anwendungsgruppe erstellt wurde. Das sollte weniger als eine Minute dauern. 

#### Aufgabe 4: Erstellen eines Azure Virtual Desktop-Arbeitsbereichs

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus und wählen Sie auf der Seite **Azure Virtual Desktop** **Arbeitsbereiche** aus.
1. Auf der Seite **Azure Virtual Desktop \| Arbeitsbereiche** **+ Erstellen** auswählen. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Arbeitsbereich erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Anwendungsgruppen >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-21a-RG**|
    |Arbeitsbereichsname|**az140-21-ws1**|
    |Anzeigename|**az140-21-ws1**|
    |Standort|Der Name der Azure-Region, in der Sie Ressourcen in der ersten Übung dieses Labs bereitgestellt haben, oder einer Region in deren Nähe|

1. Geben Sie auf der Registerkarte **Anwendungsgruppen** der Seite **Arbeitsbereich erstellen** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |Anwendungsgruppen registrieren|**Ja**|

1. Auf der Registerkarte **Arbeitsbereich** auf der Seite **Arbeitsbereich erstellen** **+ Anwendungsgruppen registrieren** auswählen.
1. Wählen Sie auf der Seite **Anwendungsgruppen hinzufügen** das Pluszeichen neben den Einträgen **az140-21-hp1-DAG**, **az140-21-hp 1-Office365-RAG** und **az140-21-hp1-Utilities-RAG** aus und klicken Sie auf **Auswählen**. 
1. Zurück auf der Registerkarte **Anwendungsgruppen** der Seite **Arbeitsbereich erstellen** **Überprüfen + erstellen** auswählen.
1. Auf der Registerkarte **Überprüfen + erstellen** der Seite **Arbeitsbereich erstellen** die Option **Erstellen** auswählen.

#### Aufgabe 5: Gewähren des Zugriffs auf Azure Virtual Desktop-Hostpools

> **Hinweis**: Wenn Sie Sitzungshosts von Microsoft Entra verwenden, müssen Sie den Benutzenden und Administrierenden von Azure Virtual Desktop entsprechende rollenbasierte Zugriffssteuerungsrollen (RBAC) von Azure zuweisen. Insbesondere ist die Rolle *Virtual Machine Anmeldung als Benutzende* erforderlich, um sich bei Sitzungshosts anzumelden, und die Rolle *Virtual Machine Anmeldung als Administrierende* ist für die lokalen administrativen Berechtigungen erforderlich. 

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Ressourcengruppen**, wählen Sie diese aus und wählen Sie auf der Seite **Ressourcengruppen** **az140-21e-RG** aus.
1. Auf der Seite **az140-21e-RG** im vertikalen Navigationsmenü **Zugriffssteuerung (IAM)** auswählen.
1. Wählen Sie auf der Seite **az140-21e-RG\| Access Control (IAM)** die Option **+ Hinzufügen** aus und wählen Sie im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus.
1. Stellen Sie sicher, dass auf der Registerkarte **Rolle** auf der Seite **Rollenzuweisung hinzufügen** die Registerkarte **Stellenfunktionsrollen** ausgewählt ist. Geben Sie im Textfeld Suchen die Zeichenfolge **Virtual Machine User Login** ein, wählen Sie in der Ergebnisliste **Virtual Machine User Login** aus und wählen Sie dann **Weiter**.
1. Stellen Sie sicher, dass auf der Registerkarte **Mitglieder** auf der Seite **Rollenzuweisung hinzufügen** die Option **Benutzende, Gruppe oder Dienstprinzipal** ausgewählt ist. Klicken Sie auf **+ Mitglieder auswählen**, suchen Sie im Bereich **Mitglieder auswählen** die Gruppe **AVD-RemoteApp**, die Sie in der ersten Aufgabe dieser Übung identifiziert haben, und klicken Sie auf **Auswählen**.
1. Zurück auf der Registerkarte **Mitglieder** der Seite **Rollenzuweisung hinzufügen** **Weiter** auswählen.
1. Auf der Registerkarte **Auftragstyp** der Seite **Rollenzuweisung hinzufügen** den **Auftragstyp** auf **Aktiv** setzen und dann **Überprüfen + zuweisen** auswählen.
1. Auf der Registerkarte **Überprüfen + zuweisen** der Seite **Rollenzuweisung hinzufügen** **Überprüfen + zuweisen** auswählen. 
1. Zurück auf der Seite **az140-21e-RG\|Zugriffssteuerung (IAM)** wählen Sie **+ Hinzufügen** und im Dropdownmenü **Rollenzuweisung hinzufügen**.
1. Stellen Sie sicher, dass auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** die Registerkarte **Stellenfunktionen** ausgewählt ist. Geben Sie im Textfeld Suchen Folgendes ein: **Virtual Machine Administrator Login** ein, wählen Sie in der Ergebnisliste **Virtual Machine Administrator Login** aus und klicken Sie dann auf **Weiter**.
1. Stellen Sie sicher, dass auf der Registerkarte **Mitglieder** auf der Seite **Rollenzuweisung hinzufügen** die Option **Benutzende, Gruppe oder Dienstprinzipal** ausgewählt ist. Klicken Sie auf **+ Mitglieder auswählen**,suchen Sie im Bereich **Mitglieder auswählen** die Gruppe **AVD-DAG**, die Sie in der ersten Aufgabe dieser Übung identifiziert haben, und klicken Sie auf **Auswählen**.
1. Zurück auf der Registerkarte **Mitglieder** der Seite **Rollenzuweisung hinzufügen**, legen Sie den **Zuweisungstyp** auf **Aktiv** fest, wählen Sie **Überprüfen + zuweisen** und wählen Sie dann erneut **Überprüfen + zuweisen**. 
