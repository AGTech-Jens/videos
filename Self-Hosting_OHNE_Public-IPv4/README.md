# Self-Hosting OHNE öffentliche IPv4 Adresse
Egal ob DS-Lite, CG-NAT oder mobile Internetlösungen wie z.B. Starlink, in diesem Video zeig ich dir wie du trotzdem selbst gehostete Dienste wie z.B. Nextcloud raus ins Internet öffnen kannst. Alles via einem kleinen VPS im Rechenzentrum, der dich gerade einmal wenige Euro im Monat kostet. Außerdem installieren wir auch gleich einen Reverse Proxy, um alles sicher mit SSL Zertifikat erreichbar zu machen.

## Voraussetzungen

- VPS im Rechenzentrum mit ausreichend monatlichem Datenvolumen & Bandbreite
- On-Prem pfSense Firewall, konfiguriert wie in diesem Video gezeigt: https://youtu.be/7VWu2qDCjHw

*Funktioniert wie am Ende des Videos erklärt auch ohne pfSense, mit beispielsweise einem LXC für WG auf deinem Proxmox Server, würde aber ganz klar zu einer ordentlichen Firewall raten!* 

## Notizen zu diesem Video

### Befehl um Private/Public SSH Keys zu generieren

```bash
umask 077; wg genkey | tee privatekey | wg pubkey > publickey
```

### wg0.conf Template

```bash
[Interface] 
PrivateKey = <soeben ausgelesener private-key aus .txt>
Address = 10.100.90.0/31
SaveConfig = true 
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
ListenPort = 51820

[Peer]
PublicKey = <public Key der pfSense>
AllowedIPs = 10.100.90.0/31, 192.168.69.0/24
PersistentKeepalive = 25
```


### wg0.conf Zusatz für Port-Forwarding an pfSense/HAProxy

```bash
PreUp = iptables -t nat -A PREROUTING -i ens3 -p tcp --dport 80 -j DNAT --to-destination 10.100.90.1
PreUp = iptables -t nat -A PREROUTING -i ens3 -p tcp --dport 443 -j DNAT --to-destination 10.100.90.1
PreUp = iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
PostDown = iptables -t nat -D PREROUTING -i ens3 -p tcp --dport 80 -j DNAT --to-destination 10.100.90.1
PostDown = iptables -t nat -D PREROUTING -i ens3 -p tcp --dport 443 -j DNAT --to-destination 10.100.90.1
PostDown = iptables -t nat -D POSTROUTING -o wg0 -j MASQUERADE
```
