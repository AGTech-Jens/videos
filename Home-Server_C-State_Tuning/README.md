# Home Server Stromverbrauch optimieren mit C-States
Du erreichtst mit deinem Server nicht die in meinen Build Videos gezeigten Stromverbrauchswerte obwohl du die Hardware IDENT nachgekauft hast? Oder willst noch das letzte Bisschen Effizienz aus deinem aktuellen Home Server rausholen? Dann wird's auf jeden Fall mal Zeit einen Blick auf die C-States von seinem Server zu werfen!

## Voraussetzungen

- Geeignete Hardware, für genauere Infos siehe mein "Effiziente Home Server bauen" Video! (LINK HIER SOBALD ONLINE)
- Linux basiertes Betriebssytem auf deinem Server

## Status Quo in Erfahrung bringen

### C-States mittels Powertop anzeigen

```bash
#Befehl um Powertop in Debian Linux basiertem System zu installieren
apt install powertop
#Befehl um Powertop in unRaid zu installieren
kein Befehl notwendig -> Powertop über das "NerdTools" Plugin aktivieren!

#Die aktuellen Package C-State Werte anzeigen
powertop
```
*mittels Tab-Taste auf deiner Tastatur in Powertop dann in den "Idle stats" Tab springen*

### Befehl um zu prüfen welche PCI Devices in deinem System aktuell ASPM nutzen

```bash
lspci -vv | awk '/ASPM/{print $0}' RS= | grep --color -P '(^[a-z0-9:.]+|ASPM )'
```

*JEDES hier als “ASPM Disabled” gelistete Device ist dein FEIND! :D*

## Ziel
Je höher die Prozentzahl in höheren C-States, desto effizienter läuft dein System im Idle. Heißt siehst du den Großteil in der linken "Pkg(HW)" Spalte in beispielsweise C2, bedeutet das dein Server nutzt quasi keine dieser "semi-Idle" States, die dir im Leerlauf einiges an Verbrauch ersparen könnten ohne das du bei der Benutzung irgendwas davon mitbekommst!

### Idle Features im BIOS aktivieren
Grundsätzlich sollte man natürlich erstmal im BIOS überprüfen ob C-States, Package C-States und ASPM L1 überhaupt im BIOS aktiviert sind.

### Beispiel: Onboard NIC beim Asrock N100m ("unRaid Server Build 2024" Video)
In meinem kürzlich erschienenen unRaid Home Build Guide für 2024 hatt ich beispielsweise den Fall das die Onboard Realtek Netzwerkkarte das System in C3 gehalten hat und mittels folgendem Befehl um L1_ASPM zu erzwingen gings plötzlich runter bis C8.
```bash
echo 1 | sudo tee /sys/bus/pci/drivers/r8169/0000\:01\:00.0/link/l1_aspm
```

### Enable-ASPM Skript
Einfacher geht's aber definitiv mittels aspm-enable.sh Skript- die "Problemkinder" raussuchen, Adresse notieren und anschleßend mit Hilfe des Skripts versuchen auf L1 oder L0s ASPM zu forcen!
1. aspm-enable.sh Skript aus meinem Git auf dem Server in /usr/sbin platzieren
2. aspm-enable.sh Skript mittels "chmod +x aspm-enable.sh" ausführbar machen
3. Skript ausführen:
```bash
/usr/sbin/enable-aspm.sh [PCI Device] [ASPM Setting]
```
*Mögliche ASPM Settings: 1=L02; 2=L1; 3=L1&L0s*

### Enable-ASPM Skript in unRaid nutzen
unRaid ist in der Hinsicht besonders, dass alles abseits von /boot nicht persistent ist. Heißt das Skript an sich funktioniert zwar wie auch in jedem anderen Linux basierten OS, man muss allerdings paar Dinge beachten:
1. aspm-enable.sh Skript aus meinem Git auf dem Server in /boot/scripts platzieren
2. aspm-enable.sh Skript mittels "chmod +x aspm-enable.sh" ausführbar machen
3. User Scripts Plugin installieren und ein neues User Script erstellen:
```bash
#!/bin/bash
bash /boot/scripts/enable-aspm.sh [PCI Device] [ASPM Setting]
```
4. Dieses Skript lässt man dann über das User Scripts Plugin automatisch bei jedem Server-Reboot ausführen und muss sich somit keine Gedanken mehr über ASPM machen wenn man seinen Server neustertet.

### Quick&Dirty Lösung für unRaid
In unRaid ists auch einen Versuch Wert mal den Powersave Mode zu versuchen, die entsprechende Policy lässt sich mit folgendem Befehl triggern:
```bash
echo -n powersave > /sys/module/pcie_aspm/parameters/policy
```
