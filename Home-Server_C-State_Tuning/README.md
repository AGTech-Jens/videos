# Home Server Stromverbrauch optimieren mit C-States
Du erreichtst mit deinem Server nicht die in meinen Build Videos gezeigten Stromverbrauchswerte obwohl du die Hardware IDENT nachgekauft hast? Oder willst noch das letzte Bisschen Effizienz aus deinem aktuellen Home Server rausholen? Dann wird's auf jeden Fall mal Zeit einen Blick auf die C-States von seinem Server zu werfen!

## Voraussetzungen

- Geeignete Hardware, für genauere Infos siehe mein "Effiziente Home Server bauen" Video! (LINK HIER SOBALD ONLINE)
- Linux basiertes Betriebssytem auf deinem Server installiert

## Status Quo in Erfahrung bringen

### C-States mittels Powertop anzeigen

```bash
#Befehl um Powertop in Debian Linux basiertem System zu installieren
apt install powertop
#Befehl um Powertop in unRaid zu installieren
kein Befehl notwendig -> Powertop über das "NerdTools" Plugin aktivieren!

#Die aktiellen C-State Werte anzeigen
powertop
```
*Mittels Tab in Powertop dann in den Tab "Idle stats" springen*

### Befehl um zu prüfen ob Komponenten in deinem System aktuell kein ASPM nutzen

```bash
lspci -vv | awk '/ASPM/{print $0}' RS= | grep --color -P '(^[a-z0-9:.]+|ASPM )'
```

*JEDES hier als “ASPM Disabled” gelistete Device ist dein FEIND! :D*

## Ziel
Je höher die Prozentzahl in höheren C-States, desto effizienter läuft dein System im Idle. Heißt siehst du den Großteil in der linken "Pkg(HW)" Spalte in beispielsweise C2, bedeutet das dein Server nutzt quasi keine dieser "semi-Idle" States, die dir im Leerlauf einiges an Verbrauch ersparen könnten ohne das du bei der Benutzung irgendwas davon mitbekommst!

### Idle Features im BIOS aktivieren
Grundsätzlich sollte man natürlich erstmal im BIOS überprüfen ob C-States, Package C-States und ASPM L1 überhaupt im BIOS aktiviert sind.

### Beispiel Onboard NIC beim Asrock N100m ("unRaid Server Build 2024" Video)
In meinem kürzlich erschienenen unRaid Home Build Guide für 2024 hatt ich beispielsweise den Fall das die Onboard Realtek Netzwerkkarte das System in C3 gehalten hat und nach Ausführen dieses Befehls gings plötzlich runter in C8.
```bash
echo 1 | sudo tee /sys/bus/pci/drivers/r8169/0000\:01\:00.0/link/l1_aspm
```

### Die Quick&Dirty Lösung für unRaid
In unRaid reichts meiner Erfahrung nach oftmals schon eifach diesen einen Befehl auszuführen, und schon sieht man bei allen Geräten mit dem Befehl von Oben “ASPM Enabled”.
```bash
echo -n powersave > /sys/module/pcie_aspm/parameters/policy
```
