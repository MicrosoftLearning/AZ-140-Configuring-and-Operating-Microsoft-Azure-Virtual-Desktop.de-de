---
lab:
  title: 'Lab: Benutzerdefinierte Sitzungshostbilder mithilfe von Bildvorlagen erstellen'
  module: 'Module 1.5: Create and manage session host images'
---

# Lab – Benutzerdefinierte Sitzungshostbilder mithilfe von Bildvorlagen erstellen
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft Entra-Benutzerkonto mit der Rolle Besitzende in dem Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit den erforderlichen Berechtigungen, um Geräte mit dem Entra-Mandanten zu verbinden, der diesem Azure-Abonnement zugeordnet ist.

## Geschätzte Dauer

90 Minuten (ca. 45 Minuten Wartezeit bis zur Fertigstellung des Image Builds)

## Labszenario

Sie planen die Implementierung einer Azure Virtual Desktop-Umgebung. Bei der Bereitstellung von Azure Virtual Desktop-Sitzungshosts müssen Sie benutzerdefinierte Images virtueller Maschinen verwenden.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Erstellen benutzerdefinierter Sitzungshost-Images für Azure Virtual Desktop mithilfe von Bildvorlagen

## Labdateien

- Keine

## Anweisungen

### Übung 1: Erstellen von benutzerdefinierten Sitzungshostbildern mithilfe von Bildvorlagen

Die Hauptaufgaben für diese Übung sind Folgende:

1. Registrieren der erforderlichen Ressourcenanbieter
1. Erstellen einer benutzerseitig zugewiesenen verwalteten Identität
1. Erstellen einer benutzerdefinierten Azure rollenbasierten Zugriffssteuerung (RBAC) Rolle
1. Festlegen von Berechtigungen für die mit der Bereitstellung von Host-Images verbundenen Ressourcen
1. Erstellen einer Azure Compute Gallery-Instanz und einer Bilddefinition
1. Erstellen einer benutzerdefinierten Imagevorlage
1. Erstellen eines benutzerdefinierten Bildes
1. Bereitstellen von Sitzungshosts mit einem benutzerdefinierten Bild

> **Hinweis**: Bevor Sie eine benutzerdefinierte Bildvorlage erstellen können, müssen Sie eine Reihe von Voraussetzungen erfüllen, darunter:

- Registrieren Sie alle erforderlichen Ressourcenanbieter
- Erstellen einer benutzerseitig zugewiesenen verwalteten Identität
- Erteilung von Berechtigungen, die für die dem Benutzer zugewiesene verwaltete Identität erforderlich sind, unter Verwendung einer benutzerdefinierten rollenbasierten Azure-Zugriffssteuerungsrolle (RBAC)
- Wenn Sie beabsichtigen, das Bild mit Hilfe der Azure Compute Gallery zu verteilen, müssen Sie die Instanz zusammen mit einer Image-Definition erstellen

#### Aufgabe 1: Registrieren der erforderlichen Ressourcenanbieter

1. Starten Sie bei Bedarf vom Lab-Computer aus einen Webbrowser, navigieren Sie zum Azure-Portal und melden Sie sich an, indem Sie die Anmeldedaten eines Benutzerkontos mit der Besitzerrolle in dem Abonnement angeben, das Sie in diesem Lab verwenden werden.

    > **Hinweis**: Verwenden Sie die Anmeldeinformationen des `User1-` Kontos, das auf der Registerkarte Ressourcen auf der rechten Seite des Fensters Lab-Sitzung aufgeführt ist.

1. Starten Sie im Azure-Portal eine PowerShell-Sitzung in der Azure Cloud Shell.

    > **Hinweis**: Wenn eine Eingabeaufforderung angezeigt wird, wählen Sie im Bereich **Erste Schritte** in der Dropdownliste **Abonnement** den Namen des Azure-Abonnements aus, das Sie in diesem Lab verwenden, und wählen Sie dann ‚**Anwenden** aus.

1. Führen Sie in der PowerShell-Sitzung im Bereich Azure Cloud Shell den folgenden Befehl aus, um den Ressourcenanbieter **Microsoft.DesktopVirtualization** zu registrieren:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **Hinweis:** Warten Sie nicht, bis die Registrierung abgeschlossen ist. Dies kann etwa fünf Minuten dauern.

1. Schließen Sie den Azure Cloud Shell-Bereich.

#### Aufgabe 2: Erstellen einer benutzerseitigen zugewiesenen verwalteten Identität

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Verwaltete Identitäten** und wählen Sie diese aus.
1. Wählen Sie auf der Seite **Verwaltete Identitäten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Alggemeine Informationen** der Seite **Benutzerseitig zugewiesene verwaltete Identität erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    > **Hinweis**: Wechseln Sie beim Festlegen des Werts für **Name** zur Registerkarte Ressourcen auf der rechten Seite des Fensters Lab und identifizieren Sie die Zeichenfolge zwischen *„User1-*“ und dem Zeichen *@*. Verwenden Sie diese Zeichenfolge, um den *zufälligen* Platzhalter zu ersetzen.

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe: **az140-15a-RG**|
    |Region|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Name|**az140**-*random*-**uami**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.

    >**Hinweis**: Warten Sie nicht, bis die Bereitstellung der Benutzenden zugewiesenen verwalteten Identität abgeschlossen ist. Dies sollte nur ein paar Sekunden dauern.

#### Aufgabe 3: Erstellen einer benutzerdefinierten Azure-RBAC-Rolle (rollenbasierte Zugriffssteuerung)

>**Hinweis**: Die benutzerdefinierte rollenbasierte Azure-Zugriffssteuerungsrolle (RBAC) wird verwendet, um der in der vorherigen Aufgabe erstellten verwalteten Identität der benutzenden Person entsprechende Berechtigungen zuzuweisen.

1. Starten Sie vom Lab-Computer aus im Webbrowser, der das Azure-Portal anzeigt, eine PowerShell-Sitzung in der Azure Cloud Shell.

1. Führen Sie in der PowerShell-Sitzung im Bereich Azure Cloud Shell den folgenden Befehl aus, um den Wert der Eigenschaft **ID** des für dieses Lab verwendeten Azure-Abonnements zu ermitteln und in der Variablen **$subscriptionId** zu speichern:

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. Führen Sie den folgenden Befehl aus, um die Rollendefinition der neuen benutzerdefinierten Rolle einschließlich des zuweisbaren Bereichswerts zu erstellen und in der Variablen **$jsonContent** zu speichern (ersetzen Sie den *zufälligen* Platzhalter durch dieselbe Zeichenfolge, die Sie in der vorherigen Aufgabe identifiziert haben):

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. Führen Sie den folgenden Befehl aus, um den Inhalt der Variablen **$jsonContent** in einer Datei mit dem Namen **CustomRole.json** zu speichern:

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. Führen Sie den folgenden Befehl aus, um die benutzerdefinierte Rolle zu erstellen:

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Schließen Sie den Azure Cloud Shell-Bereich.

#### Aufgabe 4: Festlegen von Berechtigungen für die mit der Bereitstellung von Host-Images verbundenen Ressourcen

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Ressourcengruppen**, wählen Sie diese aus und klicken Sie auf der Seite **Ressourcengruppen** auf **+ Erstellen**.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** der Seite **Ressourcengruppe erstellen** die folgenden Einstellungen an und wählen Sie dann **Überprüfen + erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe **az140-15b-RG**|
    |Region|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.
1. Aktualisieren Sie die Seite **Ressourcengruppen** und wählen Sie in der Liste der Ressourcengruppen **az140-15b-RG** aus.
1. Wählen Sie auf der Seite **az140-15b-RG** im vertikalen Navigationsmenü **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf der Seite **az140-15b-RG\|Zugriffssteuerung (IAM)** auswählen **+ Hinzufügen** und im Dropdown-Menü **Rollenzuweisung hinzufügen** aus.
1. Stellen Sie sicher, dass auf der Registerkarte **Rolle** auf der Seite **Rollenzuweisung hinzufügen** die Registerkarte **Stellenfunktionsrollen** ausgewählt ist. Geben Sie im Textfeld Suchen **Desktop Virtualization Image Creator** (*random*) ein, wählen Sie in der Ergebnisliste **Desktop Virtualization Image Creator** (*random*) aus und wählen Sie dann **Weiter**.

    >**Hinweis**: Achten Sie darauf, den *zufälligen* Platzhalter durch dieselbe Zeichenfolge zu ersetzen, die Sie bei der Definition der neuen benutzerdefinierten RBAC-Rolle verwendet haben.

1. Wählen Sie auf der Registerkarte **Mitgliedschaft** auf der Seite **Rollenzuweisung hinzufügen** die Option **Verwaltete Identität** aus, klicken Sie auf **+ Mitgliedschaft auswählen**, wählen Sie im Bereich **Verwaltete Identitäten auswählen** der Dropdownliste **Verwaltete Identität** die Option **Benutzenden zugewiesene verwaltete Identität** aus, in der Liste der Benutzenden zugewiesenen verwalteten Identitäten die Option **az140**-*random*-**uami** (wobei der *zufällige* Platzhalter dieselbe Zeichenfolge darstellt, die Sie bei der Definition der neuen benutzerdefinierten RBAC-Rolle verwendet haben), und klicken Sie dann auf **Auswählen**.
1. Zurück auf der Registerkarte „**Mitgliedschaft** auf der Seite **Rollenzuweisung hinzufügen** **Überprüfen + zuweisen**“ auswählen.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus. 

#### Aufgabe 5: Erstellen einer Azure Compute Gallery-Instanz und einer Bilddefinition

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Compute Galeries**, wählen Sie diese aus und klicken Sie auf der Seite **Azure Compute Galeries** auf **+ Erstellen**.
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Azure Compute Gallery erstellen** die folgenden Einstellungen an und wählen Sie dann **Weiter: Freigabemethode** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-15b-RG**|
    |Name|**az14015computegallery**|
    |Region|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|

1. Lassen Sie auf der Registerkarte **Freigabe** der Seite **Azure Compute Gallery erstellen** die Standardoption **Rollenbasierte Zugriffssteuerung (RBAC)** ausgewählt und wählen Sie dann **Überprüfen + erstellen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.

    >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Das sollte weniger als eine Minute dauern.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Compute Galleries**, wählen Sie diese aus und wählen Sie auf der Seite **Azure Compute Galleries** **az14015computegallery** aus. 
1. Wählen Sie auf der Seite **az14015computegallery** die Option **+ Hinzufügen** aus und wählen Sie im Dropdown-Menü die Option **+ VM-Image-Definition** aus. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **VM-Image-Definition erstellen** die folgenden Einstellungen an (belassen Sie die anderen Einstellungen bei ihren Standardwerten) und wählen Sie dann **Weiter: Version**:

    |Einstellung|Wert|
    |---|---|
    |Region|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Name der VM-Imagedefinition|**az14015imagedefinition**|
    |Betriebssystemtyp|**Windows**|
    |Sicherheitstyp|**Vertrauenswürdiger Start unterstützt**|
    |Betriebssystemstatus|**Generalisiert**|
    |Herausgeber|**MicrosoftWindowsDesktop**|
    |Angebot|**Windows-11**|
    |SKU|**win11-23h2-avd-m365**|

    > **Hinweis**: Die VM-Generation wird automatisch auf Gen2 gesetzt, da virtuelle Maschinen der Generation 1 nicht mit dem Sicherheitstyp Vertrauenswürdig und Vertraulich unterstützt werden.

1. Lassen Sie auf der Registerkarte **Version** der Seite **VM-Image-Definition erstellen** die Einstellungen unverändert und wählen Sie **Weiter: Veröffentlichungsoptionen** aus.

    > **Hinweis**: Sie sollten die VM-Imageversion in dieser Phase nicht erstellen. Dies erfolgt durch Azure Virtual Desktop.

1. Lassen Sie auf der Registerkarte **Veröffentlichungsoptionen** der Seite **VM-Image-Definition erstellen** die Einstellungen unverändert und wählen Sie **Überprüfen + erstellen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** auf der Seite **VM-Image-Definition erstellen** **Erstellen** auswählen.

    > **Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Dieser Schritt dauert normalerweise weniger als eine Minute.

#### Aufgabe 6: Erstellen einer benutzerdefinierten Imagevorlage

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus, und wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Benutzerdefinierte Imagevorlagen** und auf der Seite **Azure Virtual Desktop \| Benutzerdefinierte Imagevorlagen** **+ Benutzerdefinierte Imagevorlage hinzufügen** auswählen. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** auf der Seite **Benutzerdefinierte Bildvorlage erstellen** die folgenden Einstellungen an und wählen Sie **Weiter** aus:

    > **Hinweis**: Achten Sie beim Festlegen der Eigenschaft **Verwaltete Identität** darauf, den *zufälligen* Platzhalter durch dieselbe Zeichenfolge zu ersetzen, die Sie zuvor in dieser Übung identifiziert haben.

    |Einstellung|Wert|
    |---|---|
    |Vorlagenname|**az140-15b-imagetemplate**|
    |Importieren aus einer vorhandenen Vorlage|**Nein**|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-15b-RG**|
    |Location|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Verwaltete Identität|**az140**-*random*-**uami**|

1. Geben Sie auf der Registerkarte **Quellbild** der Seite **Benutzerdefiniertes Bild erstellen** die folgenden Einstellungen an und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Quelltyp|**Plattformimage (Marketplace)**|
    |Image auswählen|**Windows 11 Enterprise für mehrere Sitzungen, Version 23H2 + Microsoft 365 Apps**|

1. Geben Sie auf der Registerkarte **Verteilungsziele** auf der Seite **Benutzerdefinierte Bildvorlage erstellen** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten) und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Azure Compute Gallery|enabled|
    |Katalogname|**az14015computegallery**|
    |Definition des Katalogimages|**az14015imagedefinition**|
    |Katalogimageversion|**1.0.0**|
    |Name der Ausführungsausgabe|**az140-15-image-1.0.0**|
    |Replikationsregionen|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Aus Neueste ausschließen|**Nein**|
    |Speicherkontotyp|**Standard_LRS**|

    > **Hinweis**: Sie können die Eigenschaft **Replikatregionen** verwenden, um Builds mit mehreren Regionen zu erstellen. Wenn Sie **Von der neuesten ausschließen** auf **Ja** einstellen, wird verhindert, dass diese Bildversion verwendet wird, wenn bei der VM-Erstellung **Neueste** als Version des Elements**ImageReference**angegeben wird.

1. Geben Sie auf der Registerkarte **Eigenschaften erstellen** auf der Seite **Benutzerdefinierte Bildvorlage erstellen** die folgenden Einstellungen an (lassen Sie die anderen Einstellungen auf ihren Standardwerten) und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Buildtimeout|**120**|
    |Größe des Builds des VM|**Standard_DC2s_v3**|
    |Größe des Betriebssystemdatenträgers (GB)|**127**|
    |Staginggruppe|**az140-15c-RG**|
    |VNET|Nicht festgelegt lassen|

    > **Hinweis:**** Staging group** ist die Ressourcengruppe, die zum Staging von Ressourcen für die Erstellung des Bildes und zum Speichern von Protokollen verwendet wird. Wenn Sie den Namen nicht angeben, wird er automatisch generiert. Wenn der Name **VNet** nicht festgelegt ist, wird ein temporärer Name erstellt, zusammen mit einer öffentlichen IP-Adresse für die VM, die zum Erstellen des Builds verwendet wird.

    > **Wichtig**: Stellen Sie sicher, dass Sie über ausreichende Anzahl verfügbarer vCPUs für die angegebene Build-VM-Größe verfügen. Wenn nicht, wählen Sie entweder eine andere Größe oder fordern Sie eine Kontingenterhöhung an.

1. Auf der Registerkarte **Anpassung** der Seite **Benutzerdefinierte Bildvorlage erstellen** **+ Integriertes Skript hinzufügen** auswählen. 
1. Überprüfen Sie im Bereich **Integrierte Skripte auswählen** die verfügbaren Optionen, die in betriebssystemspezifische Skripte, Azure Virtual Desktop-Skripte, MSIX-App-Attach-Skripte, Anwendungsskripte und Windows-Updates-bezogene Skripte unterteilt sind, und wählen Sie dann die folgenden Einträge aus:

   - **Zeitzonenumleitung**: ermöglicht es dem Client, seine Zeitzone innerhalb einer Sitzung auf Sitzungshosts zu verwenden
   - **Deaktivieren der Speicheroptimierung**: Verhindert, dass die Speicheroptimierung Sitzungshosts durch die falsche Erkennung von Bedingungen mit geringem freiem Speicherplatz beeinträchtigt.
   - **Aktivieren des Schutzes vor Bildschirmaufnahmen** mit der Funktion **Bildschirmaufnahme blockieren auf Client und Server**: blockiert oder verbirgt Remote-Inhalte in Screenshots und bei der Bildschirmfreigabe.

1. Im Bereich **Ausgewählte integrierte Skripte** die Option **Speichern** auswählen.

    > **Hinweis**: Sie haben die Möglichkeit, eigene Skripts hinzuzufügen. Als Beispiele können Sie auf die integrierten Skripts verweisen, wie etwa [Zeitzonenumleitung](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1), [Speicheroptimierung deaktivieren](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1) oder [Schutz vor Bildschirmaufnahmen aktivieren](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1).

1. Zurück auf der Registerkarte **Anpassung** der Seite **Benutzerdefinierte Bildvorlage erstellen** **Weiter** auswählen.
1. Auf der Registerkarte **Tags** der Seite **Benutzerdefinierte Bildvorlage erstellen** **Weiter** auswählen.
1. Auf der Registerkarte **Überprüfen + erstellen** auf der Seite **Benutzerdefinierte Bildvorlage erstellen****Erstellen** auswählen.

    > **Note**: Warten Sie, bis die Vorlage erstellt wurde. Dieser Vorgang kann einige Minuten in Anspruch nehmen. Aktualisieren Sie die Seite **Azure Virtual Desktop \| Benutzerdefinierte Bildvorlagen** um den Status der Vorlage zu überprüfen.

#### Aufgabe 7: Erstellen eines benutzerdefinierten Bildes

> **Hinweis:** Die restlichen Aufgaben dieses Labs sind optional, da sie eine ziemlich lange Wartezeit mit sich bringen. 

1. Wählen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, auf der Seite **Azure Virtual Desktop \| Benutzerdefinierte Bildvorlage** die Vorlage **az140-15b-imagetemplate** aus.
1. Wälen Sie auf der Seite **az140-15b-imagetemplate** **Build starten** aus.

    > **Hinweis:** Warten Sie, bis der Build erstellt wurde. Die tatsächliche Zeit bis zum Abschluss des Build Prozesses kann variieren, aber mit den in den Lab-Anweisungen angegebenen Einstellungen sollte er innerhalb von 45 Minuten abgeschlossen sein. Aktualisieren Sie die Seite alle paar Minuten und überwachen Sie den Wert **Buildausführungsstatus** im Abschnitt **Essentials** auf der Seite **az140-15b-imagetemplate**. 

    > **Hinweis**: Der Buildausführungsstatus sollte sich irgendwann von **Läuft – Erstellung** zu **Läuft – Verteilung** und schließlich zu **Erfolgreich** ändern.

    > **Hinweis**: Während Sie auf den Abschluss des Builds warten, überprüfen Sie den Inhalt der Staging-Ressourcengruppe **az140-15c-RG**, in der die Build-Ressourcen, einschließlich der virtuellen Maschine, eines virtuellen Netzwerks, einer Netzwerksicherheitsgruppe, eines Schlüsseltresors, eines Snapshots, einer Containerinstanz und eines Speicherkontos, automatisch bereitgestellt werden. 

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Ressourcengruppen**, wählen Sie diese aus und wählen Sie auf der Seite **Ressourcengruppen** **az140-15c-RG** aus.
1. Auf der Seite **az140-15c-RG** im Abschnitt **Ressourcen** finden Sie die automatisch bereitgestellten Ressourcen.
1. Kehren Sie zur Seite **az140-15b-imagetemplate** zurück und überwachen Sie den Fortschritt des Buildfortschritts. 

    > **Hinweis**: Alternativ können Sie auch das **Aktivitätsprotokoll** verwenden, um den Abschluss des Buildprozesses zu verfolgen. Die Aktion, auf die Sie sich konzentrieren sollten, ist **Ausführen einer VM-Bildvorlage, um die Ausgabe zu erzeugen**. Sein Status sollte sich irgendwann von **Akzeptiert** zu **Erfolgreich** ändern.

1. Sobald der Build abgeschlossen ist, suchen Sie vom Lab-Computer aus im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Compute Galleries** und wählen Sie diese aus. Wählen Sie auf der Seite **Azure Compute Galleries** **az14015computegallery** aus. 
1. Wählen Sie auf der **az14015computegallery** auf der Registerkarte **Definitionen** **az14015imagedefinition** aus.
1. Auf der Seite **az14015imagedefinition** auf der Registerkarte **Versionen** finden Sie Informationen zum Bild **1.0.0 (neueste Version)**.

#### Aufgabe 8: Bereitstellen von Sitzungshosts mithilfe eines benutzerdefinierten Images

> **Hinweis**: Optional können Sie die ersten Stufen der Bereitstellung von Sitzungshosts für Azure Virtual Desktop mithilfe des von Ihnen erstellten benutzerdefinierten Images durchlaufen. 

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Virtuelle Netzwerke** und wählen Sie es aus. Wählen Sie auf der Seite **Virtuelle Netzwerke** **Erstellen +** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtuelles Netzwerk erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer neuen Ressourcengruppe **az140-15d-RG**|
    |Name des virtuellen Netzwerks|**az140-vnet15d**|
    |Region|Den Namen der Azure-Region, in der Sie die Azure Virtual Desktop-Umgebung bereitstellen möchten|

1. Übernehmen Sie auf der Registerkarte **Sicherheit** die Standardeinstellungen, und wählen Sie **Weiter** aus.
1. Geben Sie auf der Registerkarte **IP-Adressen** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |IP-Adressbereich|**10.30.0.0/16**|

1. Wählen Sie das Bearbeiten-Symbol (Stift) neben dem Eintrag **Standard** Subnetz im Bereich **Bearbeiten** aus, legen Sie die folgenden Einstellungen fest (lassen Sie die anderen mit den vorhandenen Werten unverändert) und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**hp1-Subnet**|
    |Startadresse|**10.30.1.0**|
    |Privates Subnetz aktivieren (kein standardmäßiger ausgehender Zugriff)|Disabled|

1. Wählen Sie zurück auf der Registerkarte **IP-Adressen** **Überprüfen + erstellen** und dann **Erstellen** aus.

    > **Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Dieser Schritt dauert normalerweise weniger als eine Minute.

1. Suchen Sie auf dem Lab-Computer im Webbrowser, der das Azure-Portal anzeigt, nach **Azure Virtual Desktop**, wählen Sie es aus, wählen Sie auf der Seite **Azure Virtual Desktop** im Abschnitt **Verwalten** des vertikalen Navigationsmenüs **Hostpools** auswählen und auf der Seite **Azure Virtual Desktop \| Hostpools** **+ Erstellen** auswählen. 
1. Geben Sie auf der Registerkarte **Allgemeine Informationen** der Seite **Hostpool erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Sitzungshosts >** (belassen Sie die anderen Einstellungen auf ihren Standardwerten):

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**az140-15d-RG**|
    |Hostpoolname|**az140-15-hp1**|
    |Location|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Überprüfungsumgebung|**Nein**|
    |Bevorzugter App-Gruppentyp|**Desktop**|
    |Hostpooltyp|**In einem Pool zusammengefasst**|
    |Sitzungshostkonfiguration erstellen|**Nein**|
    |Lastenausgleichsalgorithmus|**Breitensuche**|

    > **Hinweis**: Bei Verwendung des breitenorientierten Algorithmus für den Lastenausgleich ist der Parameter für die maximale Anzahl von Sitzungen optional.

1. Geben Sie auf der Registerkarte **Sitzungshosts** der Seite **Hostpool erstellen** die folgenden Einstellungen an (belassen Sie die anderen Einstellungen bei ihren Standardwerten):

    > **Hinweis**: Wenn Sie den Wert für **Präfixname** einstellen, wechseln Sie zur Registerkarte Ressourcen auf der rechten Seite des Lab-Sitzungsfensters. Suchen Sie dort die Zeichenfolge zwischen *User1-* und dem Zeichen *@*. Verwenden Sie diese Zeichenfolge, um den *zufälligen* Platzhalter zu ersetzen.

    |Einstellung|Wert|
    |---|---|
    |Virtuelle Computer hinzufügen|**Ja**|
    |Resource group|**Standardmäßig mit der Ressourcengruppe des Hostpools identisch.**|
    |Namenspräfix|**sh0**_random_|
    |Typ des virtuellen Computers|**Virtueller Azure-Computer**|
    |Virtueller Computer Standort|Den Namen der Azure-Region, in der Sie Ihre Azure Virtual Desktop-Umgebung bereitstellen möchten|
    |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
    |Sicherheitstyp|**Virtuelle Computer mit vertrauenswürdigem Start**|

1. Wählen Sie auf der Registerkarte **Virtuelle Maschinen** der Seite**Einen Hostpool erstellen** unter der Dropdownliste **Image** **Alle Bilder anzeigen**.
1. Wählen Sie auf der Seite **Bild auswählen** die Option **Freigegebene Bilder** aus und wählen Sie in der Liste der Bilder die Option **az14015imagedefinition** aus. 
1. Zurück auf der Registerkarte **Virtuelle Computer** auf der Seite **Hostpool erstellen** die folgenden Einstellungen angeben und **Weiter: Arbeitsbereich >** auswählen (andere Einstellungen mit ihren Standardwerten belassen):

    |Einstellung|Wert|
    |---|---|
    |Größe des virtuellen Computers|**Standard DC2s_v3**|
    |Number of VMs (Anzahl von VMs)|**1**|
    |Typ des Betriebssystemdatenträgers|**SSD Standard**|
    |Größe des Betriebssystemdatenträgers|**Standardgröße**|
    |Startdiagnose|**Mit verwaltetem Speicherkonto aktivieren (empfohlen)**|
    |Virtuelles Netzwerk|az140-vnet15d|
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

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa 10–15 Minuten dauern.
