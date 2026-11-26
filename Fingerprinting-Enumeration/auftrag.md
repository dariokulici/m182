# Enumeration & Fingerprinting – Metasploitable 2

## 1. Ziel des Auftrags

Ziel dieses Auftrags war es, den Ablauf einer Schwachstellenanalyse in einer kontrollierten Lab-Umgebung kennenzulernen.

Dazu wurde mit Kali Linux ein absichtlich verwundbares Metasploitable-2-System untersucht. Zuerst wurde das Lab-Netzwerk auf erreichbare Hosts überprüft. Anschliessend wurden offene Ports, laufende Dienste und deren Versionen ermittelt.

Auf Basis dieser Informationen wurden mögliche Schwachstellen recherchiert und zwei mögliche Angriffsvarianten verglichen. Eine davon wurde anschliessend innerhalb der isolierten Lab-Umgebung durchgeführt.

Zum Schluss wurden Massnahmen definiert, mit denen ein reales System gegen einen vergleichbaren Angriff geschützt werden könnte.

---

# 2. Lab-Umgebung

Für die Durchführung wurden zwei virtuelle Maschinen verwendet:

|System|Funktion|IP-Adresse|
|---|---|---|
|Kali Linux|Angriffssystem|192.168.56.10|
|Metasploitable 2|Zielsystem|192.168.56.20|

Die beiden Systeme befanden sich in einem isolierten virtuellen Netzwerk:

`192.168.56.0/24`

Die Umgebung wurde bewusst vom produktiven Netzwerk getrennt.

## Netzwerkaufbau

**Kali Linux**  
Angriffssystem  
`192.168.56.10`

↓

**Isoliertes Lab-Netz**  
`192.168.56.0/24`

↓

**Metasploitable 2**  
Zielsystem  
`192.168.56.20`

---

# 3. Code of Conduct

Bevor mit der Analyse begonnen wurde, wurden einige grundlegende Regeln festgelegt.

Die Scans und Angriffstests werden ausschliesslich gegen Systeme durchgeführt, für welche eine entsprechende Berechtigung besteht.

Für dieses Lab gelten deshalb folgende Bedingungen:

- ausschliesslich eigene virtuelle Maschinen
- isoliertes Lab-Netzwerk
- keine Scans gegen öffentliche IP-Adressen
- keine Scans gegen das Schul- oder Firmennetzwerk
- keine Systeme anderer Personen
- keine produktiven Daten
- Angriffe ausschliesslich zu Ausbildungszwecken

Dadurch können Enumeration und Exploitation kontrolliert durchgeführt werden, ohne fremde Systeme zu gefährden.

---

# 4. Enumeration

## 4.1 Ziel

Vor einem möglichen Angriff müssen möglichst viele Informationen über das Zielsystem gesammelt werden.

Interessant sind insbesondere:

- vorhandene Hosts
- IP-Adressen
- offene Ports
- verwendete Protokolle
- laufende Dienste
- Softwareversionen
- Betriebssystem
- mögliche Netzwerkfreigaben

Diese Phase wird als **Enumeration** bezeichnet.

---

# 5. Netzwerkscan

Zuerst wurde überprüft, welche Hosts im Lab-Netzwerk erreichbar sind.

Dazu wurde folgender Nmap-Befehl verwendet:

`nmap -sn 192.168.56.0/24`

Ausgabe:

`Nmap scan report for 192.168.56.10`  
`Host is up.`

`Nmap scan report for 192.168.56.20`  
`Host is up.`

`Nmap done: 256 IP addresses scanned, 2 hosts up`

Damit wurden zwei aktive Systeme erkannt.

|IP-Adresse|System|
|---|---|
|192.168.56.10|Kali Linux|
|192.168.56.20|Metasploitable 2|

Das Zielsystem für die weiteren Scans ist:

`192.168.56.20`


---

# 6. TCP-Portscan

Als Nächstes wurde untersucht, welche TCP-Ports auf dem Zielsystem geöffnet sind.

Dafür wurde ein SYN-Scan über alle TCP-Ports durchgeführt:

`sudo nmap -sS -p- 192.168.56.20`

Das Flag `-sS` aktiviert einen TCP-SYN-Scan.

Mit `-p-` werden alle TCP-Ports von 1 bis 65535 geprüft.

Der Scan ergab mehrere offene Ports. Unter anderem wurden folgende Dienste erkannt:

|Port|Status|Dienst|
|---|---|---|
|21|open|FTP|
|22|open|SSH|
|23|open|Telnet|
|25|open|SMTP|
|53|open|DNS|
|80|open|HTTP|
|111|open|RPC|
|139|open|NetBIOS|
|445|open|SMB|
|512|open|rexec|
|513|open|rlogin|
|514|open|rsh|
|1099|open|Java RMI|
|1524|open|bindshell|
|2049|open|NFS|
|2121|open|FTP|
|3306|open|MySQL|
|5432|open|PostgreSQL|
|5900|open|VNC|
|6000|open|X11|
|6667|open|IRC|
|8009|open|AJP13|
|8180|open|HTTP|

Bereits dieser Scan zeigt eine sehr grosse Angriffsfläche, da zahlreiche Netzwerkdienste erreichbar sind.

Ein offener Port bedeutet allerdings noch nicht automatisch, dass eine Schwachstelle vorhanden ist. Deshalb müssen im nächsten Schritt die jeweiligen Dienste genauer identifiziert werden.



---

# 7. Service- und OS-Fingerprinting

Für die Identifikation der Dienste und des Betriebssystems wurde ein weiterer Nmap-Scan durchgeführt.

`sudo nmap -sS -sV -O 192.168.56.20`

Die wichtigsten Optionen sind:

|Option|Bedeutung|
|---|---|
|`-sS`|TCP-SYN-Scan|
|`-sV`|Erkennung der Dienstversionen|
|`-O`|Betriebssystemerkennung|

Folgende Dienste und Versionen wurden erkannt:

|Port|Dienst|erkannte Version|
|---|---|---|
|21|FTP|vsftpd 2.3.4|
|22|SSH|OpenSSH 4.7p1|
|23|Telnet|Linux telnetd|
|25|SMTP|Postfix smtpd|
|53|DNS|ISC BIND 9.4.2|
|80|HTTP|Apache httpd 2.2.8|
|139|SMB|Samba smbd 3.X|
|445|SMB|Samba smbd 3.X|
|3306|MySQL|MySQL 5.0.51a|
|5432|PostgreSQL|PostgreSQL 8.3|
|5900|VNC|VNC protocol 3.3|
|6667|IRC|UnrealIRCd|
|8180|HTTP|Apache Tomcat|

Zusätzlich deutete das OS-Fingerprinting auf ein älteres Linux-System hin.

`Running: Linux 2.6.X`

`OS details: Linux 2.6.9 - 2.6.33`

---

# 8. Bewertung des Fingerprintings

Das Fingerprinting zeigt, dass auf dem Zielsystem zahlreiche alte Dienste betrieben werden.

Besonders auffällig sind:

- FTP über vsftpd 2.3.4
- ein alter Apache-Webserver
- Samba
- Telnet
- MySQL und PostgreSQL
- IRC
- VNC
- mehrere Remote-Administration-Dienste

Die grosse Anzahl erreichbarer Dienste erhöht die Angriffsfläche deutlich.

Besonders Telnet ist problematisch, da die Kommunikation unverschlüsselt erfolgt. Für Remote-Administration sollte stattdessen beispielsweise SSH eingesetzt werden.

Auch Dienste, die für den eigentlichen Betrieb eines Servers nicht benötigt werden, sollten deaktiviert werden.

---

# 9. UDP-Scan

Neben TCP wurden auch UDP-Dienste untersucht.

`sudo nmap -sU 192.168.56.20`

Folgende UDP-Ports wurden erkannt:

|Port|Status|Dienst|
|---|---|---|
|53/udp|open|domain|
|111/udp|open|rpcbind|
|137/udp|open|netbios-ns|
|2049/udp|open|NFS|

UDP-Scans müssen vorsichtiger interpretiert werden als TCP-Scans.

Da UDP keinen TCP-Handshake besitzt, kann Nmap bei ausbleibenden Antworten teilweise nicht eindeutig bestimmen, ob ein Port tatsächlich offen oder lediglich gefiltert ist.

---

# 10. Zusätzliche Enumeration

Da auf dem System SMB erkannt wurde, wurde zusätzlich `enum4linux` zur Enumeration verwendet.

Befehl:

`enum4linux -a 192.168.56.20`

Damit können unter anderem Informationen über Samba beziehungsweise SMB gesammelt werden.

Interessante Informationen sind beispielsweise:

- Benutzer
- Gruppen
- Freigaben
- Hostname
- Domain- oder Workgroup-Informationen
- Samba-Konfiguration

Unter anderem wurden folgende Informationen ermittelt:

`Target: 192.168.56.20`

`Workgroup: WORKGROUP`

`Server: METASPLOITABLE`

Verfügbare Shares:

- `print$`
- `tmp`
- `opt`
- `IPC$`

Damit konnten weitere Informationen über das Zielsystem gesammelt werden.

---

# 11. Zusammenfassung der Enumeration

Nach Abschluss der Enumeration waren folgende wesentliche Informationen bekannt:

### Zielhost

`192.168.56.20`

### Betriebssystem

Älteres Linux-System aus der Linux-2.6-Generation.

### Besonders interessante Dienste

- FTP
- SSH
- Telnet
- HTTP
- SMB
- NFS
- MySQL
- PostgreSQL
- VNC
- IRC

Diese Informationen bilden die Grundlage für die anschliessende Schwachstellenanalyse.

---

# 12. Vulnerability Assessment

Anhand der ermittelten Softwareversionen wurde geprüft, welche Dienste bekannte Schwachstellen besitzen könnten.

Dabei ist wichtig:

**Ein offener Port ist nicht automatisch eine Schwachstelle.**

Relevant ist die Kombination aus:

**Port → Dienst → Software → Version → bekannte Schwachstelle**

Beispiel:

**21/tcp → FTP → vsftpd → Version 2.3.4 → Recherche nach bekannten Schwachstellen**

Dadurch kann die Recherche gezielt auf die tatsächlich vorhandenen Softwareversionen beschränkt werden.

---

# 13. Mögliche Angriffsvariante 1 – vsftpd 2.3.4

Beim Fingerprinting wurde auf TCP-Port 21 folgende Version erkannt:

`vsftpd 2.3.4`

Diese Version ist im Kontext von Metasploitable 2 besonders interessant, da die dort verwendete kompromittierte Version eine bekannte Backdoor enthält.

### Bewertung

**Vorteile**

- Dienst wurde direkt durch Nmap erkannt
- Version konnte eindeutig identifiziert werden
- bekannte Schwachstelle vorhanden
- gut dokumentiertes Angriffsszenario
- im Lab einfach nachvollziehbar

**Risiko**

Ein erfolgreicher Angriff kann einem Angreifer Zugriff auf das Zielsystem ermöglichen.

---

# 14. Mögliche Angriffsvariante 2 – Samba

Eine zweite interessante Angriffsfläche stellt der erkannte Samba-Dienst dar.

Nmap zeigte SMB-Dienste auf:

`139/tcp`

`445/tcp`

Zusätzlich konnten über die SMB-Enumeration Informationen über das System und vorhandene Freigaben gesammelt werden.

Eine Schwachstelle in einem veralteten Samba-Dienst kann unter bestimmten Voraussetzungen die Ausführung von Befehlen auf dem Zielsystem ermöglichen.

### Bewertung

**Vorteile**

- SMB ist aus dem Netzwerk erreichbar
- zusätzliche Informationen konnten enumeriert werden
- ältere Samba-Versionen besitzen bekannte Sicherheitsprobleme

**Nachteil**

Vor dem Einsatz eines konkreten Exploits muss die exakte Version beziehungsweise die Verwundbarkeit eindeutig bestätigt werden.

---

# 15. Vergleich der Angriffsvarianten

|Kriterium|vsftpd|Samba|
|---|---|---|
|Dienst erkannt|Ja|Ja|
|Angriffsfläche gut erkennbar|Sehr gut|Gut|
|Rechercheaufwand|gering|mittel|
|Durchführung im Lab|einfach|mittel|
|Nachvollziehbarkeit|sehr gut|gut|

Aufgrund der eindeutig erkannten Version und der bekannten Schwachstelle wurde die **vsftpd-Variante** ausgewählt.

---

# 16. Durchführung des Exploits

## Auswahl

Für den Angriff wurde die zuvor identifizierte FTP-Software verwendet:

`vsftpd 2.3.4`

Auf dem Kali-System wurde zunächst Metasploit gestartet:

`msfconsole`

Anschliessend wurde innerhalb des Frameworks nach einem passenden Modul gesucht:

`search vsftpd`

Dabei wurde ein passendes Modul für die erkannte Version identifiziert.

Das entsprechende Modul wurde anschliessend ausgewählt und die Zieladresse des Metasploitable-Systems als `RHOSTS` gesetzt:

`RHOSTS = 192.168.56.20`

Danach wurde der Exploit ausgeführt.

Nach der Ausführung konnte eine Sitzung auf dem Zielsystem hergestellt werden.

Zur Überprüfung der erhaltenen Berechtigungen wurde ausgeführt:

`id`

Ausgabe:

`uid=0(root) gid=0(root)`

Anschliessend wurde der Hostname überprüft:

`hostname`

Ausgabe:

`metasploitable`

Damit konnte bestätigt werden, dass über den verwundbaren Netzwerkdienst Zugriff auf das Zielsystem möglich war und die erhaltene Sitzung über Root-Berechtigungen verfügte.

Die Sitzung wurde danach beendet und nicht für weitere Aktivitäten im System verwendet.


---

# 17. Bewertung des Angriffs

Der Angriff war erfolgreich, weil mehrere Sicherheitsprobleme gleichzeitig vorhanden waren.

Besonders entscheidend war die Verwendung einer verwundbaren Softwareversion.

Zusätzlich war der betroffene Dienst direkt über das Netzwerk erreichbar.

Dadurch war folgender Ablauf möglich:

1. Zielsystem im Netzwerk finden
2. offenen Port identifizieren
3. Dienst und Softwareversion bestimmen
4. nach bekannten Schwachstellen recherchieren
5. passenden Exploit identifizieren
6. Exploit gegen das Zielsystem durchführen
7. erhaltenen Zugriff überprüfen

Gerade das Fingerprinting war deshalb ein wichtiger Bestandteil des Angriffs. Ohne die Informationen über die verwendeten Dienste und Versionen wäre eine gezielte Auswahl eines passenden Exploits deutlich schwieriger gewesen.

---

# 18. Resilienzanalyse

Der erfolgreiche Angriff hätte durch verschiedene Schutzmassnahmen verhindert oder zumindest deutlich erschwert werden können.

## Massnahme 1 – Patch- und Vulnerability-Management

Veraltete oder verwundbare Software sollte regelmässig aktualisiert oder vollständig ersetzt werden.

Im Lab war insbesondere die identifizierte FTP-Version problematisch.

In einer produktiven Umgebung sollten deshalb regelmässig:

- Sicherheitsupdates installiert,
- bekannte CVEs geprüft,
- nicht mehr unterstützte Software ersetzt und
- Vulnerability-Scans durchgeführt

werden.

Damit wäre die ausgenutzte Schwachstelle nicht mehr vorhanden beziehungsweise frühzeitig erkannt worden.

---

## Massnahme 2 – Angriffsfläche reduzieren

Auf dem Zielsystem waren sehr viele Netzwerkdienste gleichzeitig erreichbar.

Beispiele:

- FTP
- Telnet
- SSH
- SMB
- NFS
- VNC
- Datenbanken
- Webserver
- IRC

Ein produktiver Server sollte nur diejenigen Dienste bereitstellen, welche tatsächlich benötigt werden.

Nicht benötigte Dienste sollten deaktiviert oder über Firewall-Regeln blockiert werden.

Das Prinzip dahinter lautet:

**Je kleiner die Angriffsfläche, desto weniger Möglichkeiten besitzt ein Angreifer.**

Wäre der FTP-Dienst nicht benötigt und deshalb deaktiviert worden, hätte die untersuchte Schwachstelle nicht über das Netzwerk ausgenutzt werden können.

---

## Massnahme 3 – Netzwerksegmentierung und Firewall

Der Zugriff auf administrative oder interne Dienste sollte zusätzlich durch Netzwerksegmentierung und Firewall-Regeln eingeschränkt werden.

Beispielsweise sollte ein FTP-, SMB-, Datenbank- oder Administrationsdienst nicht automatisch für jedes Gerät im Netzwerk erreichbar sein.

Mögliche Schutzmassnahmen:

- Firewall-Regeln
- separate VLANs
- Management-Netz
- Zugriff nur von definierten Hosts
- Default-Deny-Regeln für nicht benötigte Verbindungen

Selbst wenn auf einem System eine Schwachstelle vorhanden ist, kann eine korrekte Netzwerksegmentierung verhindern, dass ein Angreifer den betroffenen Dienst überhaupt erreicht.

---

# 19. Weitere mögliche Schutzmassnahmen

Zusätzlich wären folgende Massnahmen sinnvoll:

- regelmässige Vulnerability-Scans
- zentrale Protokollierung
- IDS/IPS
- Monitoring ungewöhnlicher Netzwerkaktivitäten
- sichere SSH-Konfiguration
- Telnet deaktivieren
- unverschlüsselte Protokolle vermeiden
- starke Authentifizierung
- Least-Privilege-Prinzip
- regelmässige Überprüfung offener Ports

Ein produktives System sollte grundsätzlich nach dem Prinzip **Secure by Default** betrieben werden.

---

# 20. Fazit

Durch diesen Auftrag wurde deutlich, welche Bedeutung Enumeration und Fingerprinting für einen Penetrationstest haben.

Mit einem einfachen Netzwerkscan konnte zunächst festgestellt werden, welche Systeme im Netzwerk erreichbar sind.

Anschliessend konnten mit Nmap offene Ports sowie die dahinterliegenden Dienste und Softwareversionen identifiziert werden.

Der grundlegende Ablauf war:

**Netzwerk entdecken**

↓

**Hosts identifizieren**

↓

**Ports scannen**

↓

**Dienste identifizieren**

↓

**Versionen bestimmen**

↓

**Schwachstellen recherchieren**

↓

**Risiko bewerten**

↓

**Exploit im Lab durchführen**

↓

**Schutzmassnahmen definieren**

Besonders interessant war, dass ein offener Port allein noch nicht ausreicht, um eine Schwachstelle festzustellen. Erst durch das Fingerprinting der entsprechenden Softwareversionen konnten bekannte Schwachstellen gezielt recherchiert und bewertet werden.

Der durchgeführte Angriff zeigte ausserdem, warum Patch-Management, eine möglichst kleine Angriffsfläche und Netzwerksegmentierung wichtige Bestandteile der IT-Sicherheit sind.

---

# 21. Quellen

## Nmap

Nmap Project: **Nmap Reference Guide**

https://nmap.org/book/man.html

Abruf: 26.11.2026

---

## Rapid7 – Metasploitable 2

Rapid7: **Metasploitable 2 Exploitability Guide**

https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide

Abruf: 26.11.2026

---

## CVE-Datenbank

CVE: **Common Vulnerabilities and Exposures**

https://www.cve.org/

Abruf: 26.11.2026

---

## Ethical Hacking / Code of Conduct

Panmore Institute: **Ethical Hacking Code of Ethics: Security Risk Issues**

https://panmore.com/ethical-hacking-code-of-ethics-security-risk-issues

Abruf: 26.11.2026

---

# 22. Verwendete Werkzeuge

|Werkzeug|Zweck|
|---|---|
|Kali Linux|System für die Sicherheitsanalyse|
|Metasploitable 2|absichtlich verwundbares Zielsystem|
|Nmap|Host-, Port- und Service-Enumeration|
|enum4linux|SMB-/Samba-Enumeration|
|Metasploit Framework|Durchführung des Exploits|

---

# 23. Kurzreflexion

Der Auftrag hat mir gezeigt, dass man bei einem Penetrationstest nicht direkt mit einem Exploit beginnt. Zuerst müssen möglichst viele Informationen über das Ziel gesammelt werden.

Besonders Nmap war dabei hilfreich, da sich damit nicht nur offene Ports finden lassen, sondern auch Informationen über Dienste, Versionen und das Betriebssystem gesammelt werden können.

Anhand dieser Informationen kann anschliessend wesentlich gezielter nach bekannten Schwachstellen gesucht werden.

Besonders interessant fand ich den Zusammenhang zwischen Enumeration und Exploitation. Erst durch das Erkennen von `vsftpd 2.3.4` konnte gezielt nach einer passenden Schwachstelle und einem möglichen Exploit gesucht werden.

Gleichzeitig wurde deutlich, dass viele erfolgreiche Angriffe durch grundlegende Sicherheitsmassnahmen verhindert oder zumindest stark erschwert werden könnten. Vor allem regelmässige Updates, das Deaktivieren unnötiger Dienste und eine saubere Netzwerksegmentierung hätten die Angriffsfläche des untersuchten Systems deutlich reduziert.