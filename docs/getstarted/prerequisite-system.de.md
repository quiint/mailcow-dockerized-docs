Bevor Sie **mailcow: dockerized** ausführen, sollten Sie einige Voraussetzungen überprüfen:

!!! warning "Achtung"
    Versuchen Sie **nicht**, mailcow auf einem Synology/QNAP-Gerät (jedes NAS), OpenVZ, LXC oder anderen Container-Plattformen zu installieren. KVM, ESX, Hyper-V und andere vollständige Virtualisierungsplattformen werden unterstützt.

!!! info
    - mailcow: dockerized erfordert, dass [einige Ports](#standard-ports) für eingehende Verbindungen offen sind, also stellen Sie sicher, dass Ihre Firewall diese nicht blockiert.
    - Stellen Sie sicher, dass keine andere Anwendung die Konfiguration von mailcow stört, wie z.B. ein anderer Maildienst
    - Ein korrektes DNS-Setup ist entscheidend für jedes gute Mailserver-Setup, also stellen Sie bitte sicher, dass Sie zumindest die [basics](../getstarted/prerequisite-dns.de.md#die-minimale-dns-konfiguration) abgedeckt haben, bevor Sie beginnen!
    - Stellen Sie sicher, dass Ihr System ein korrektes Datum und eine korrekte [Zeiteinstellung](#datum-und-uhrzeit) hat. Dies ist entscheidend für verschiedene Komponenten wie die Zwei-Faktor-TOTP-Authentifizierung.

## Minimale Systemressourcen

!!! success "Kompatibilität hergestellt"
    Seit XXX ist mailcow endlich auch auf ARM64 Plattformen verfügbar! Komplett! Ohne Einschränkungen der Funktionalität!

Bitte stellen Sie sicher, dass Ihr System mindestens über die folgenden Ressourcen verfügt:

| Ressource | Minimale Anforderung |
| ----------------------- | ------------------------------------------------ |
| CPU | 1 GHz |
| RAM | **Minimum** 6 GiB + 1 GiB Swap (Standardkonfiguration) |
| Festplatte | 20 GiB (ohne Emails) |
| Architektur | x86_64, ARM64 :warning:{ title="Frisch Released, Fehler können noch existieren"} |

!!! failure "Nicht unterstützt"
	**OpenVZ, Virtuozzo und LXC**

ClamAV und Solr können sehr viel Arbeitspeicher verbrauchen. Sie können diese in der `mailcow.conf` durch die Einstellungen `SKIP_CLAMD=y` und `SKIP_SOLR=y` jedoch auch deaktivieren.

!!! info 
	Wir sind uns bewusst, dass ein reiner MTA auf 128 MiB RAM laufen kann. 
	mailcow ist eine ausgewachsene und gebrauchsfertige Groupware mit vielen Extras, die das Leben einfacher machen. 
	Diese kommt mit einem Webserver, Webmailer, ActiveSync (MS), Antivirus, Antispam, Indexierung (Solr), Dokumentenscanner (Oletools), SQL (MariaDB), Cache (Redis), MDA, MTA, verschiedenen Webdiensten etc.

Ein einzelner SOGo-Worker **kann** ~350 MiB RAM belegen, bevor er geleert wird. Je mehr ActiveSync-Verbindungen Sie verwenden möchten, desto mehr RAM wird benötigt. In der Standardkonfiguration werden 20 Arbeiter erzeugt.

#### Beispiele für die RAM Planung

Ein Unternehmen mit 15 Smartphones (EAS aktiviert) und etwa 50 gleichzeitigen IMAP-Verbindungen sollte 16 GiB RAM einplanen.

6 GiB RAM + 1 GiB Swap sind für die meisten privaten Installationen ausreichend, während 8 GiB RAM für ~5 bis 10 Benutzer empfohlen werden.

Im Rahmen unseres Supports können wir Ihnen bei der korrekten Planung Ihres Setups helfen.

### Unterstützte Betriebssysteme
Grundsätzlich kann mailcow auf jeder Distribution verwendet werden, die von Docker CE unterstützt wird (siehe https://docs.docker.com/install/).
Es kann jedoch in vereinzelten Fällen zu einer Inkompatibilität der Betriebssysteme und den mailcow Komponenten kommen.

Die folgende Tabelle enthält alle von uns offiziell unterstützten und getesteten Betriebssysteme (*Stand Juni 2023*):

| Betriebssystem                | Kompatibilität                            |
| ----------------------- | ------------------------------------------------ |
| Alpine 3.16 und älter            | [⚠️](https://www.alpinelinux.org/ "Eingeschränkt Kompatibel") |
| Centos 7              | [✅](https://www.centos.org/ "Vollständig Kompatibel") |
| Debian 10, 11, 12              | [✅](https://www.debian.org/index.de.html "Vollständig Kompatibel") |
| Ubuntu 18.04, 20.04, 22.04                   | [✅](https://ubuntu.com/ "Vollständig Kompatibel")|
| Alma Linux 8 | [✅](https://almalinux.org/ "Vollständig Kompatibel") |
| Rocky Linux 9 | [✅](https://rockylinux.org/ "Vollständig Kompatibel") |


!!! info "Legende"
        ✅ = Funktioniert **out of the box** anhand der Anleitung.<br>
        ⚠️ = Erfordert einige **manuelle Anpassungen**, sonst aber nutzbar.<br>
        ❌ = Generell **NICHT Kompatibel**.<br>
        ❔ = Ausstehend.

!!! warning "Achtung"
    **Andere (nicht genannte Betriebssysteme) können auch funktionieren, sind jedoch nicht offiziell getestet worden.**

## Firewall & Ports

Bitte überprüfen Sie, ob alle Standard-Ports von mailcow offen sind und nicht von anderen Anwendungen genutzt werden:

```
ss -tlpn | grep -E -w '25|80|110|143|443|465|587|993|995|4190'
# oder:
netstat -tulpn | grep -E -w '25|80|110|143|443|465|587|993|995|4190'
```

!!! danger "Vorsicht"
    Es gibt einige Probleme mit dem Betrieb von mailcow auf einem Firewalld/ufw aktivierten System. <br>
	Sie sollten es deaktivieren (wenn möglich) und stattdessen Ihren Regelsatz in die DOCKER-USER-Kette verschieben, die nicht durch einen Neustart des Docker-Dienstes gelöscht wird. <br>
	Siehe [diese (blog.donnex.net)](https://blog.donnex.net/docker-and-iptables-filtering/) oder [diese (unrouted.io)](https://unrouted.io/2017/08/15/docker-firewall/) Anleitung für Informationen darüber, wie man iptables-persistent mit der DOCKER-USER Kette benutzt. <br>
    Da mailcow im Docker-Modus läuft, haben INPUT-Regeln keinen Effekt auf die Beschränkung des Zugriffs auf mailcow. <br>
	Verwenden Sie stattdessen die FORWARD-Kette.

Wenn dieser Befehl irgendwelche Ergebnisse liefert, entfernen oder stoppen Sie bitte die Anwendung, die auf diesem Port läuft. Sie können mailcows Ports auch über die Konfigurationsdatei `mailcow.conf` anpassen.

### Standard Ports

Wenn Sie eine Firewall vor mailcow haben, stellen Sie bitte sicher, dass diese Ports für eingehende Verbindungen offen sind:

| Dienst | Protokoll | Port | Container | Variable |
| --------------------|:--------:|:-------|:------------------|----------------------------------|
| Postfix SMTP | TCP | 25 | postfix-mailcow | `${SMTP_PORT}` |
| Postfix SMTPS | TCP | 465 | postfix-mailcow | `${SMTPS_PORT}` |
| Postfix Submission | TCP | 587 | postfix-mailcow | `${SUBMISSION_PORT}` |
| Dovecot IMAP | TCP | 143 | dovecot-mailcow | `${IMAP_PORT}` |
| Dovecot IMAPS | TCP | 993 | dovecot-mailcow | `${IMAPS_PORT}` |
| Dovecot POP3 | TCP | 110 | dovecot-mailcow | `${POP_PORT}` |
| Dovecot POP3S | TCP | 995 | dovecot-mailcow | `${POPS_PORT}` |
| Dovecot ManageSieve | TCP | 4190 | dovecot-mailcow | `${SIEVE_PORT}` |
| HTTP(S) | TCP | 80/443 | nginx-mailcow | `${HTTP_PORT}` / `${HTTPS_PORT}` |

Um einen Dienst an eine IP-Adresse zu binden, können Sie die IP-Adresse wie folgt voranstellen: `SMTP_PORT=1.2.3.4:25`

**Wichtig**: Sie können keine IP:PORT-Bindungen in HTTP_PORT und HTTPS_PORT verwenden. Bitte verwenden Sie stattdessen `HTTP_PORT=1234` und `HTTP_BIND=1.2.3.4`.

### Wichtig für Hetzner Firewalls

Ich zitiere https://github.com/chermsen über https://github.com/mailcow/mailcow-dockerized/issues/497#issuecomment-469847380 (DANKE!):

Für alle, die mit der Hetzner-Firewall zu kämpfen haben:

Port 53 ist in diesem Fall für die Firewall-Konfiguration unwichtig. Laut Dokumentation verwendet unbound den Portbereich 1024-65535 für ausgehende Anfragen.
Da es sich bei der Hetzner Robot Firewall um eine statische Firewall handelt (jedes eingehende Paket wird isoliert geprüft) - müssen die folgenden Regeln angewendet werden:

**Für TCP**
```
SRC-IP: ---
DST-IP: ---
SRC-Port: ---
DST-Port: 1024-65535
Protokoll: tcp
TCP-Flags: ack
Aktion:      Akzeptieren
```

**Für UDP**
```
SRC-IP: ---
DST-IP: ---
SRC-Port: ---
DST-Port: 1024-65535
Protokoll: udp
Aktion:      Akzeptieren
```

Wenn man einen restriktiveren Portbereich anwenden will, muss man zuerst die Konfiguration von unbound ändern (nach der Installation):

{mailcow-dockerized}/data/conf/unbound/unbound.conf:
```
ausgehender-Port-vermeiden: 0-32767
```

Nun können die Firewall-Regeln wie folgt angepasst werden:

```
[...]
DST Port: 32768-65535
[...]
```

## Datum und Uhrzeit

Um sicherzustellen, dass Sie das richtige Datum und die richtige Zeit auf Ihrem System eingestellt haben, überprüfen Sie bitte die Ausgabe von `timedatectl status`:

```
$ timedatectl status
      Lokale Zeit: Sat 2017-05-06 02:12:33 CEST
  Weltzeit: Sa 2017-05-06 00:12:33 UTC
        RTC-Zeit: Sa 2017-05-06 00:12:32
       Zeitzone: Europa/Berlin (MESZ, +0200)
     NTP aktiviert: ja
NTP synchronisiert: ja
 RTC in lokaler TZ: nein
      Sommerzeit aktiv: ja
 Letzte DST-Änderung: Sommerzeit begann am
                  Sonne 2017-03-26 01:59:59 MEZ
                  So 2017-03-26 03:00:00 MESZ
 Nächste Sommerzeitänderung: Die Sommerzeit endet (die Uhr springt eine Stunde rückwärts) am
                  Sun 2017-10-29 02:59:59 MESZ
                  Sun 2017-10-29 02:00:00 MEZ
```

Die Zeilen `NTP aktiviert: ja` und `NTP synchronisiert: ja` zeigen an, ob Sie NTP aktiviert haben und ob es synchronisiert ist.

Um NTP zu aktivieren, müssen Sie den Befehl `timedatectl set-ntp true` ausführen. Sie müssen auch Ihre `/etc/systemd/timesyncd.conf` bearbeiten:

```
# vim /etc/systemd/timesyncd.conf
[Zeit]
NTP=0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
```

## Hetzner Cloud (und wahrscheinlich andere)

Prüfen Sie `/etc/network/interfaces.d/50-cloud-init.cfg` und ändern Sie die IPv6-Schnittstelle von eth0:0 auf eth0:

```
# Falsch:
auto eth0:0
iface eth0:0 inet6 static
# Richtig:
auto eth0
iface eth0 inet6 static
```

Starten Sie die Schnittstelle neu, um die Einstellungen zu übernehmen.
Sie können außerdem die [cloud-init Netzwerkänderungen deaktivieren.](https://wiki.hetzner.de/index.php/Cloud_IP_static/en#disable_cloud-init_network_changes)

## MTU

Besonders relevant für OpenStack-Benutzer: Überprüfen Sie Ihre MTU und setzen Sie sie entsprechend in docker-compose.yml. Siehe [Problebehandlungen](../getstarted/install.de.md#benutzer-mit-einer-mtu-ungleich-1500-zb-openstack) in unseren Installationsanleitungen.
