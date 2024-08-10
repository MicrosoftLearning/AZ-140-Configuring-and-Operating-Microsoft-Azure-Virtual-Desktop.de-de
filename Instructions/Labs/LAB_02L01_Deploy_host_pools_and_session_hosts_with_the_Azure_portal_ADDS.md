---
lab:
  title: "Lab: Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Lab: Bereitstellen von Hostpools und Sitzungshosts mithilfe des Azure-Portals (AD DS)
# Lab-Handbuch für Kursteilnehmer

## Lababhängigkeiten

- Ein Azure-Abonnement, das Sie in diesem Lab verwenden werden
- Ein Microsoft-Konto oder Microsoft Entra-Konto mit der Rolle „Besitzer*in“ oder „Mitwirkende*r“ im Azure-Abonnement, das Sie in diesem Lab verwenden werden, und mit der Rolle „Globale*r Administrator*in“ im Microsoft Entra-Mandanten, der diesem Azure-Abonnement zugeordnet ist.
- Das abgeschlossene Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)**

## Geschätzte Dauer

60 Minuten

## Labszenario

Sie müssen Hostpools und Sitzungshosts in einer Umgebung von Active Directory Domain Services (AD DS) erstellen und konfigurieren.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Implementieren einer Azure Virtual Desktop-Umgebung in einer AD DS-Domäne
- Überprüfen einer Azure Virtual Desktop-Umgebung in einer AD DS-Domäne

## Labdateien

- Keine

## Anweisungen

### Übung 1: Implementieren einer Azure Virtual Desktop-Umgebung in einer AD DS-Domäne
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Vorbereiten einer AD DS-Domäne und des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools
1. Bereitstellen eines Azure Virtual Desktop-Hostpools
1. Verwalten der Sitzungshosts der Azure Virtual Desktop-Hostpools
1. Konfigurieren von Azure Virtual Desktop-Anwendungsgruppen
1. Konfigurieren von Azure Virtual Desktop-Arbeitsbereichen

#### Aufgabe 1: Vorbereiten einer AD DS-Domäne und des Azure-Abonnements für die Bereitstellung eines Azure Virtual Desktop-Hostpools

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [Azure-Portal]( ), und melden Sie sich an. Verwenden Sie dabei die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ verfügt.
1. Suchen Sie im Azure-Portal nach **Virtuelle Computer**, und wählen Sie diese Option aus. Klicken Sie auf dem Blatt **Virtuelle Computer** auf **az140-dc-vm11**.
1. Klicken Sie auf dem Blatt **az140-dc-vm11** auf **Verbinden**. Wählen Sie im Dropdownmenü die Option **Bastion** und auf der Registerkarte **Bastion** des Blatts **az140-dc-vm11 \| Verbinden** die Option **Bastion verwenden** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|

1. Starten Sie innerhalb der Bastion-Sitzung auf **az140-dc-vm11** die **Windows PowerShell ISE** als Administrator*in.
1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um eine Organisationseinheit zu erstellen, in der die Computerobjekte der Azure Virtual Desktop-Hosts gehostet werden:

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Benutzerprinzipalnamen des Kontos **aduser1** zu ermitteln:

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **Hinweis:** Notieren Sie sich den in diesem Schritt ermittelten Benutzerprinzipalnamen. Sie benötigen diese später in diesem Lab.

1. Führen Sie über den Skriptbereich **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um den Ressourcenanbieter **Microsoft.DesktopVirtualization** zu registrieren:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** Microsoft Edge, und navigieren Sie zum [Azure-Portal](https://portal.azure.com). Wenn Sie dazu aufgefordert werden, melden Sie sich mit den Microsoft Entra-Anmeldeinformationen des Benutzerkontos an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer*in“ verfügt.
1. Suchen Sie innerhalb der Bastion-Sitzung für **az140-dc-vm11** im Azure-Portal oben auf der Seite über das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** nach **Virtuelle Netzwerke**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Netzwerke** die Option **az140-adds-vnet11** aus. 
1. Wählen Sie auf dem Blatt **az140-adds-vnet11** die Option **Subnetze** und auf dem Blatt **Subnetze** die Option **+ Subnetz** aus. Geben Sie auf dem Blatt **Subnetz hinzufügen** die folgenden Einstellungen an (übernehmen Sie bei allen anderen Einstellungen die Standardwerte), und klicken Sie auf **Speichern**:

   |Einstellung|Wert|
   |---|---|
   |Name|**hp1-Subnet**|
   |Startadresse|**10.0.1.0**|
   |Größe|**/24 (256 Adressen)**|

#### Aufgabe 2: Bereitstellen eines Azure Virtual Desktop-Hostpools

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Hostpools** und auf dem Blatt **Azure Virtual Desktop \| Hostpools** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Virtuelle Computer >** aus (behalten Sie für die anderen Einstellungen die Standardwerte bei):

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|erstellen Sie eine **neue** Ressourcengruppe mit dem Namen **az140-21-RG**|
   |Hostpoolname|**az140-21-hp1**|
   |Standort|Der Name der Azure-Region, in der Sie Ressourcen in der ersten Übung dieses Labs bereitgestellt haben, oder einer Region in deren Nähe |
   |Überprüfungsumgebung|**Nein**|
   |Bevorzugter App-Gruppentyp|**Desktop**|
   |Hostpooltyp|**In einem Pool zusammengefasst**|
   |Lastenausgleichsalgorithmus|**Breitensuche**|
   |Maximale Anzahl von Sitzungen|**12**|

   > **Hinweis**: Wenn ein Benutzer oder eine Benutzerin sowohl RemoteApp- als auch Desktop-Apps veröffentlicht hat, bestimmt der bevorzugte App-Gruppentyp, welcher dieser Apps in ihrem Feed angezeigt wird.

1. Geben Sie auf dem Blatt **Hostpool erstellen** auf der Registerkarte **Virtuelle Computer** die folgenden Einstellungen an, und wählen Sie **Weiter: Arbeitsbereiche >** aus (behalten Sie für die anderen Einstellungen die Standardwerte bei):

   |Einstellung|Wert|
   |---|---|
   |Virtuelle Computer hinzufügen|**Ja**|
   |Resource group|**Standardmäßig mit der Ressourcengruppe des Hostpools identisch.**|
   |Namenspräfix|**az140-21-p1**|
   |Typ des virtuellen Computers|Virtueller Azure-Computer|
   |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie Ressourcen im letzten Lab bereitgestellt haben.|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Sicherheitstyp|**Virtuelle Computer mit vertrauenswürdigem Start**|
   |Abbildung|**Windows 11 Enterprise (mehrere Sitzungen) + Microsoft 365-Apps, Version 22H2**|
   |Größe des virtuellen Computers|**Standard DC2s_v3**|
   |Number of VMs (Anzahl von VMs)|**2**|
   |Typ des Betriebssystemdatenträgers|**SSD Standard**|
   |Größe des Betriebssystemdatenträgers|**Standardgröße (128 GiB)**|
   |Startdiagnose|**Mit verwaltetem Speicherkonto aktivieren (empfohlen)**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnetz|**hp1-Subnet (10.0.1.0/24)**|
   |Netzwerksicherheitsgruppe|**Grundlegend**|
   |Öffentliche Eingangsports|**Nein**|
   |Wählen Sie das gewünschte Verzeichnis für die Einbindung aus.|**Active Directory**|
   |UPN für AD-Domänenbeitritt|**student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|
   |Kennwort bestätigen|**Pa55w.rd1234**|
   |Domäne oder Einheit angeben|**Ja**|
   |Domäne für den Beitritt|**adatum.com**|
   |Pfad der Organisationseinheit|**OU=WVDInfra,DC=adatum,DC=com**|
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|
   |Kennwort bestätigen|**Pa55w.rd1234**|

1. Bestätigen Sie auf der Registerkarte **Arbeitsbereich** auf dem Blatt **Hostpool erstellen** die folgenden Einstellungen, und wählen Sie **Überprüfen + Erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Desktop-App-Gruppe registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Hostpool erstellen** die Option **Erstellen** aus.

   > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa 10–15 Minuten dauern.

#### Aufgabe 3: Verwalten der Sitzungshosts der Azure Virtual Desktop-Hostpools

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** auf der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Hostpools** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools den Eintrag **az140-21-hp1** aus.
1. Wählen Sie auf dem Blatt **az140-21-hp1** auf der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Sitzungshosts** aus, und überprüfen Sie, ob der Pool aus zwei Hosts besteht. 
1. Wählen Sie auf dem Blatt **az140-21-hp1 \| Sitzungshosts** die Option **+ Hinzufügen** aus.
1. Überprüfen Sie auf dem Blatt **VMs zu Hostpool hinzufügen** auf der Registerkarte **Grundeinstellungen** die vorkonfigurierten Einstellungen, und wählen Sie **Weiter: Virtuelle Computer** aus.
1. Geben Sie auf der Registerkarte **Virtuelle Computer** des Blatts **VMs zu Hostpool hinzufügen** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Resource group|**az140-21-RG**|
   |Namenspräfix|**az140-21-p1**|
   |Virtueller Computer Standort|Der Name der Azure-Region, in der Sie die ersten zwei Sitzungshost-VMs bereitgestellt haben|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Sicherheitstyp|**Virtuelle Computer mit vertrauenswürdigem Start**|
   |Abbildung|**Windows 11 Enterprise (mehrere Sitzungen) + Microsoft 365-Apps, Version 22H2**|
   |Number of VMs (Anzahl von VMs)|**1**|
   |Typ des Betriebssystemdatenträgers|**SSD Standard**|
   |Größe des Betriebssystemdatenträgers|**Standardgröße (128 GiB)**|
   |Startdiagnose|**Mit verwaltetem Speicherkonto aktivieren (empfohlen)**|
   |Virtuelles Netzwerk|**az140-adds-vnet11**|
   |Subnetz|**hp1-Subnet (10.0.1.0/24)**|
   |Netzwerksicherheitsgruppe|**Grundlegend**|
   |Öffentliche Eingangsports|**Nein**|
   |Wählen Sie das gewünschte Verzeichnis für die Einbindung aus.|**Active Directory**|
   |UPN für AD-Domänenbeitritt|**student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|
   |Kennwort bestätigen|**Pa55w.rd1234**|
   |Domäne oder Einheit angeben|**Ja**|
   |Domäne für den Beitritt|**adatum.com**|
   |Pfad der Organisationseinheit|**OU=WVDInfra,DC=adatum,DC=com**|   
   |Benutzername|**Kursteilnehmer**|
   |Kennwort|**Pa55w.rd1234**|
   |Kennwort bestätigen|**Pa55w.rd1234**|

   > **Hinweis:** Wie Sie wahrscheinlich bereits festgestellt haben, ist es möglich, das Image und das Präfix der VMs zu ändern, wenn Sie Sitzungshosts zum vorhandenen Pool hinzufügen. Im Allgemeinen wird dies nur empfohlen, wenn Sie alle VMs im Pool ersetzen möchten. 

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **VMs zu Hostpool hinzufügen** die Option **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren. Dies kann etwa zehn Minuten dauern. 

#### Aufgabe 4: Konfigurieren von Azure Virtual Desktop-Anwendungsgruppen

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** in dem Webbrowserfenster, in dem das Azure-Portal geöffnet ist, nach **Azure Virtual Desktop** und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Anwendungsgruppen** aus.
1. Auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** wird die vorhandene, automatisch generierte Desktopanwendungsgruppe **az140-21-hp1-DAG** angezeigt. Wählen Sie diese Gruppe aus. 
1. Wählen Sie auf dem Blatt **az140-21-hp1-DAG** die Option **Zuweisungen** aus.
1. Wählen Sie auf dem Blatt **az140-21-hp1-DAG \| Zuweisungen** die Option **+ Hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Microsoft Entra-Benutzerinnen und -Benutzer oder -Benutzergruppen auswählen** **Gruppen** aus und dann **az140-wvd-pooled** und **Auswählen** aus.
1. Navigieren Sie zurück zum Blatt **Azure Virtual Desktop \| Anwendungssicherheitsgruppen**, und wählen Sie **+ Erstellen** aus. 
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Anwendungsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-21-RG**|
   |Hostpool|**az140-21-hp1**|
   |Anwendungsgruppentyp|**Remote-App**|
   |Name der Anwendungsgruppe|**az140-21-hp1-Office365-RAG**|

1. Wählen Sie auf der Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + Hinzufügen** und dann **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Startmenü**|
   |Application|**Word**|
   |Beschreibung|**Microsoft Word**|
   |Befehlszeile erforderlich|**Nein**|

1. Kehren Sie zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + Hinzufügen** und dann **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Startmenü**|
   |Application|**Excel**|
   |Beschreibung|**Microsoft Excel**|
   |Befehlszeile erforderlich|**Nein**|

1. Kehren Sie zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + Hinzufügen** und dann **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsquelle|**Startmenü**|
   |Application|**PowerPoint**|
   |Beschreibung|**Microsoft PowerPoint**|
   |Befehlszeile erforderlich|**Nein**|

1. Kehren Sie zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie **Weiter: Zuweisungen >** aus.
1. Wählen Sie auf der Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Microsoft Entra-Benutzer*innen oder -Benutzergruppen hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Microsoft Entra-Benutzerinnen und -Benutzer oder -Benutzergruppen auswählen** **Gruppen** aus und dann **az140-wvd-remote-app** und **Auswählen** aus.
1. Kehren Sie zur Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie **Weiter: Arbeitsbereich >**.
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die folgende Einstellung an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsgruppen registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Anwendungsgruppe erstellen** die Option **Erstellen** aus.

   > **Hinweis:** Warten Sie, bis die Anwendungsgruppe erstellt wurde. Das sollte weniger als eine Minute dauern. 

   > **Hinweis:** Als Nächstes erstellen Sie eine Anwendungsgruppe, die auf dem Dateipfad als Anwendungsquelle basiert.

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Anwendungsgruppen** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Anwendungsgruppen** die Option **+ Erstellen** aus. 
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Anwendungsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-21-RG**|
   |Hostpool|**az140-21-hp1**|
   |Anwendungsgruppentyp|**RemoteApp**|
   |Name der Anwendungsgruppe|**az140-21-hp1-Utilities-RAG**|

1. Wählen Sie auf der Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Anwendungen hinzufügen** aus.
1. Geben Sie auf dem Blatt **Anwendung hinzufügen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellung an und wählen Sie **Weiter** aus:

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

1. Kehren Sie zur Registerkarte **Anwendungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie **Weiter: Zuweisungen >** aus.
1. Wählen Sie auf der Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** die Option **+ Microsoft Entra-Benutzer*innen oder -Benutzergruppen hinzufügen** aus.
1. Wählen Sie auf dem Blatt **Microsoft Entra-Benutzerinnen und -Benutzer oder -Benutzergruppen auswählen** **Gruppen** aus und dann **az140-wvd-remote-app** und **az140-wvd-admins** und anschließend **Auswählen** aus.
1. Kehren Sie zur Registerkarte **Zuweisungen** des Blatts **Anwendungsgruppe erstellen** zurück, und wählen Sie **Weiter: Arbeitsbereich >**.
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die folgende Einstellung an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsgruppen registrieren|**Nein**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Anwendungsgruppe erstellen** die Option **Erstellen** aus.

#### Aufgabe 5: Konfigurieren von Azure Virtual Desktop-Arbeitsbereichen

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** in dem Microsoft Edge-Fenster, in dem das Azure-Portal geöffnet ist, nach **Azure Virtual Desktop** und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** die Option **Arbeitsbereiche** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Arbeitsbereiche** die Option **+ Erstellen** aus. 
1. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Anwendungsgruppen >** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az140-21-RG**|
   |Arbeitsbereichsname|**az140-21-ws1**|
   |Anzeigename|**az140-21-ws1**|
   |Standort|Der Name der Azure-Region, in der Sie Ressourcen in der ersten Übung dieses Labs bereitgestellt haben, oder einer Region in deren Nähe|

1. Geben Sie auf der Registerkarte **Anwendungsgruppen** des Blatts **Arbeitsbereich erstellen** die folgenden Einstellungen an:

   |Einstellung|Wert|
   |---|---|
   |Anwendungsgruppen registrieren|**Ja**|

1. Wählen Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Arbeitsbereich** die Option **+ Anwendungsgruppen registrieren** aus.
1. Wählen Sie auf dem Blatt **Anwendungsgruppen hinzufügen** das Plussymbol neben den Einträgen **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** und **az140-21-hp1-Utilities-RAG** aus, und klicken Sie auf **Auswählen**. 
1. Wählen Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Anwendungsgruppen** die Option **Überprüfen + erstellen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Arbeitsbereich erstellen** die Option **Erstellen** aus.

### Übung 2: Überprüfen der Azure Virtual Desktop-Umgebung
  
Die Hauptaufgaben für diese Übung sind Folgende:

1. Installieren des Microsoft-Remotedesktopclients (MSRDC) auf einem Windows 10-Computer
1. Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs
1. Testen von Azure Virtual Desktop-Apps

#### Aufgabe 1: Installieren des Microsoft-Remotedesktopclients (MSRDC) auf einem Windows 10-Computer

1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** im Browserfenster, in dem das Azure-Portal angezeigt wird, nach **VMs**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **VMs** den Eintrag **az140-cl-vm11** aus.
1. Scrollen Sie auf dem Blatt **az140-cl-vm11** nach unten zum Abschnitt **Vorgänge**, und wählen Sie **Befehl ausführen** aus. 
1. Wählen Sie auf dem Blatt **az140-cl-vm11 \| Befehl ausführen** die Option **EnableRemotePS** und dann **Ausführen** aus. 

   > **Hinweis:** Warten Sie, bis der Befehl ausgeführt wurde, bevor Sie mit dem nächsten Schritt fortfahren. Dies kann etwa eine Minute dauern. Sie erhalten möglicherweise Fehlermeldungen mit rotem Text, die sich auf das verwendete öffentliche Profil und nicht auf das Domänenprofil beziehen. Sie können diese Meldungen ignorieren und mit dem nächsten Schritt fortfahren.

1. Führen Sie innerhalb der Bastionsitzung für **az140-dc-vm11** über die Konsole **Administrator: Windows PowerShell ISE** den folgenden Befehl aus, um alle Mitglieder von **ADATUM\\az140-wvd-users** der lokalen Gruppe **Remotedesktopbenutzer** auf dem virtuellen Azure-Computer **az140-cl-vm11** unter Windows 10 hinzuzufügen, den Sie im Lab **Vorbereiten der Bereitstellung von Azure Virtual Desktop (AD DS)** bereitgestellt haben.

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. Wechseln Sie zu Ihrem Labcomputer, suchen Sie im Browserfenster, in dem das Azure-Portal angezeigt wird, nach **Virtuelle Computer**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Virtuelle Computer** den Eintrag **az140-cl-vm11** aus.
1. Wählen Sie auf dem Blatt **az140-cl-vm11** **Verbindung herstellen** aus und wählen Sie im Dropdownmenü **Verbindung über Bastion herstellen** aus.
1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Anmeldeinformationen ein, und klicken Sie auf **Verbinden**:

   |Einstellung|Wert|
   |---|---|
   |Benutzername|**Student@adatum.com**|
   |Kennwort|**Pa55w.rd1234**|

   > **Hinweis** Die erste Anmeldung kann 5–10 Minuten dauern.

1. Starten Sie innerhalb der Bastion-Sitzung für **az140-cl-vm11** Microsoft Edge, und navigieren Sie zur [Downloadseite für den Windows-Desktopclient](https://go.microsoft.com/fwlink/?linkid=2068602). Wählen Sie bei entsprechender Aufforderung **Ausführen** aus, um die Installation zu starten. Wählen Sie auf der Seite **Installationsbereich** des Assistenten **Remotedesktopsetup** die Option **Für alle Benutzer auf diesem Computer installieren** aus, und klicken Sie auf **Installieren**. Wenn Sie von der Benutzerkontensteuerung zur Eingabe von Administratoranmeldeinformationen aufgefordert werden, authentifizieren Sie sich mit dem Benutzernamen **ADATUM\\Student** und dem Kennwort **Pa55w.rd1234**.
1. Vergewissern Sie sich nach Abschluss der Installation, dass das Kontrollkästchen **Launch Remote Desktop when setup exits** (Remotedesktop starten, wenn das Setup abgeschlossen ist) aktiviert ist, und klicken Sie auf **Fertig stellen**, um den Remotedesktopclient zu starten.

#### Aufgabe 2: Abonnieren eines Azure Virtual Desktop-Arbeitsbereichs

1. Wählen Sie im Fenster des **Remotedesktopclients** die Option **Abonnieren** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aduser1** an. Geben Sie dabei das weiter oben in diesem Lab ermittelten Benutzerprinzipalname Attribut userPrincipalName und das beim Erstellen dieses Kontos festgelegte Kennwort an.

   > **Hinweis:** Wählen Sie alternativ im Fenster des **Remotedesktopclients** die Option **Mit URL abonnieren** aus. Geben Sie im Bereich **Arbeitsbereich abonnieren** unter **E-Mail oder Arbeitsbereichs-URL** die URL **https://client.wvd.microsoft.com/api/arm/feeddiscovery** ein, wählen Sie **Weiter** aus, und melden Sie sich bei entsprechender Aufforderung mit den Anmeldeinformationen für **aduser1** an. (Verwenden Sie das userPrincipalName-Attribut als Benutzername und das beim Erstellen dieses Kontos festgelegte Kennwort.) 

1. Überprüfen Sie, ob auf der Seite **Remotedesktop** der SessionDesktop angezeigt wird, der in der automatisch generierten az140-21-hp1-DAG-Desktop-Anwendungsgruppe enthalten ist, die im Arbeitsbereich veröffentlicht und über die Gruppenmitgliedschaft mit dem Benutzerkonto **aduser1** verbunden ist. 

   > **Hinweis**: Dies wird erwartet, da der **Bevorzugte App-Gruppentyp** des Hostpools derzeit auf **Desktop** festgelegt ist.

#### Aufgabe 3: Testen von Azure Virtual Desktop-Apps

1. Doppelklicken Sie in der Bastion-Sitzung für **az140-cl-vm11** im Clientfenster des **Remotedesktop** in der Liste der Anwendungen auf **SessionDesktop**, und vergewissern Sie sich, dass eine Remotedesktopsitzung gestartet wird. 

   > **Hinweis:** Es kann anfangs einige Minuten dauern, bis die Anwendung gestartet wird. Danach sollte der Anwendungsstart aber viel schneller erfolgen.

   > **Hinweis:** Wenn Ihnen die Anmeldeaufforderung **Willkommen bei Microsoft Teams** angezeigt wird, schließen Sie sie.

1. Klicken Sie in der Sitzung **SessionDesktop** mit der rechten Maustaste auf **Start**, und wählen Sie **Ausführen** aus. Geben Sie in das Textfeld **Öffnen** des Dialogfelds **Ausführen** **cmd** ein, und wählen Sie **OK** aus. 
1. Geben Sie in der Sitzung **SessionDesktop** in der Eingabeaufforderung **hostname** ein, und drücken Sie die **Eingabetaste**, um den Namen des Computers anzuzeigen, auf dem die Remotedesktopsitzung ausgeführt wird.
1. Vergewissern Sie sich, dass der angezeigte Name **az140-21-p1-0**, **az140-21-p1-1** oder **az140-21-p1-2** lautet.
1. Geben Sie in der Eingabeaufforderung **logoff** ein, und drücken Sie die **Eingabetaste**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.

   > **Hinweis**: Als nächstes ändern Sie den **Bevorzugten App-Gruppentyp**, indem Sie ihn auf **RemoteApp** festlegen.

1. Wechseln Sie von Ihrem Labcomputer zur Bastion-Sitzung für **az140-dc-vm11**.
1. Suchen Sie in der Bastion-Sitzung für **az140-dc-vm11** im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, nach **Azure Virtual Desktop**, und wählen Sie diese Option aus. Wählen Sie auf dem Blatt **Azure Virtual Desktop** auf der vertikalen Menüleiste im Abschnitt **Verwalten** die Option **Hostpools** aus.
1. Wählen Sie auf dem Blatt **Azure Virtual Desktop \| Hostpools** in der Liste der Hostpools den Eintrag **az140-21-hp1** aus.
1. Wählen Sie auf dem Blatt **az140-21-hp1** in der vertikalen Menüleiste im Abschnitt **Einstellungen** die Option **Eigenschaften**, im Feld **Bevorzugter App-Gruppentyp** die Option **Remote-App** und anschließend die Option **Speichern**. 
1. Wechseln Sie von Ihrem Labcomputer in die Bastion-Sitzung zu **az140-cl-vm11**.
1. Wählen Sie innerhalb der Bastion-Sitzung zu **az140-cl-vm11** im Client-Fenster **Remotedesktop** das Auslassungszeichen in der oberen rechten Ecke und wählen Sie im Dropdown-Menü **Aktualisieren**.
1. Überprüfen Sie, ob auf der Seite **Remotedesktop** einzelne Anwendungen angezeigt werden, die zu den beiden Anwendungsgruppen gehören, die Sie erstellt und im Arbeitsbereich veröffentlicht haben und die über die Gruppenmitgliedschaft auch mit dem Benutzerkonto **aduser1** verbunden sind. 

   > **Hinweis**: Dies wird erwartet, da der **Bevorzugte App-Gruppentyp** des Hostpools jetzt auf **RemoteApp** festgelegt ist.

1. Doppelklicken Sie in der Bastion-Sitzung für **az140-cl-vm11** im Clientfenster **Remotedesktop** in der Liste der Anwendungen auf **Eingabeaufforderung**, und vergewissern Sie sich, dass ein Fenster vom Typ **Eingabeaufforderung** geöffnet wird. Geben Sei bei der Aufforderung zur Authentifizierung das Kennwort ein, das Sie beim Erstellen des Benutzerkontos **aduser1** festgelegt haben, aktivieren Sie das Kontrollkästchen **Anmeldedaten speichern**, und wählen Sie **OK** aus.
1. Geben Sie an der Eingabeaufforderung **hostname** ein, und drücken Sie die **EINGABETASTE**, um den Namen des Computers anzuzeigen, auf dem die Eingabeaufforderung ausgeführt wird.

   > **Hinweis:** Vergewissern Sie sich, dass der angezeigte Name **az140-21-p1-0**, **az140-21-p1-1** oder **az140-21-p1-2** und nicht **az140-cl-vm11** lautet.

1. Geben Sie an der Eingabeaufforderung **logoff** ein, und drücken Sie die **EINGABETASTE**, um sich bei der aktuellen Remote-App-Sitzung abzumelden.

### Übung 3: Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

Die Hauptaufgaben für diese Übung sind Folgende:

1. Beenden der im Lab bereitgestellten Azure-VMs und Aufheben ihrer Zuordnung

>**Hinweis:** In dieser Übung heben Sie die Zuordnung der in diesem Lab bereitgestellten Azure-VMs auf, um die entsprechenden Computegebühren zu minimieren.

#### Aufgabe 1: Aufheben der Zuordnung von im Lab bereitgestellten Azure-VMs

1. Wechseln Sie zum Labcomputer, und öffnen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, die **PowerShell**-Shellsitzung im Bereich **Cloud Shell**.
1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs aufzulisten:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Führen Sie in der PowerShell-Sitzung im Bereich „Cloud Shell“ den folgenden Befehl aus, um alle in diesem Lab erstellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Hinweis:** Der Befehl wird (wie über den Parameter „-NoWait“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss einen weiteren PowerShell-Befehl in derselben PowerShell-Sitzung ausführen können, es jedoch einige Minuten dauert, bis die Azure-VMs tatsächlich beendet werden und ihre Zuordnung aufgehoben wird.
