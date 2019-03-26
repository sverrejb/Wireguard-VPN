[English version 🇬🇧](english.md)
# Sett opp din egen VPN-server med WireGuard!
**Tidsbruk:** En time  
**Nivå:** Grunnleggende terminalbruk er en fordel.  
**Forberedelser/utstyr:**
-   Laptop (Mac eller Linux)
-   Tilgang til en ekstern server med Ubuntu/tilsvarende* og statisk IPv4 og IPv6** aktivert. Digital Ocean til 5 USD/mnd eller tilsvarende fungerer bra.
  -   Android/iPhone med appen installert (valgfritt).

<sub>* Denne guiden tar utgangspunkt i Ubuntu. Hjelp/støtte kan ikke garanteres for andre OS.</sub>
<sub>** Dette er valgfritt</sub>
## Installasjon på server
### 1) SSH deg inn på serveren du ønsker å bruke, og installer WireGuard:
```
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt-get update
sudo apt-get install wireguard
```
### 2) Generer privat og offentlig nøkkel
```
wg genkey | tee privatekey | wg pubkey > publickey
```
Verifiser at du nå har to filer `privatekey` og `publickey`.

### 3) Aktiver packet forwarding
Rediger filen  `/etc/sysctl.conf` og skru på IPV4 packet forwarding. 
Den aktuelle linjen er `net.ipv4.ip_forward=1`.
Kjør i tillegg `echo 1 > /proc/sys/net/ipv4/ip_forward`. * 

<sub>* Dette sparer oss for en reboot.</sub>

### 4) Opprett WireGuard-config

#### Finn navnet på ditt vanlige nettverks-interface
Vi trenger å vite hvilket interface som trafikken som kommer inn på WireGuard skal rutes til. Dette heter typisk `eth0` eller `ens<tall>`. Sjekk hva ditt heter med `ifconfig`.

#### Konfigurer WireGuard
Opprett filen `/etc/wireguard/wg0.conf` og fyll den med følgende data. Legg merke til der vi setter inn din `privatekey` og ditt interface fra steget over.
```bash
[Interface]
Address = 192.168.2.1
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o <interface name> -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o <interface name> -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o <interface name> -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o <interface name> -j MASQUERADE
PrivateKey = <insert key here>
ListenPort = 51820

[Peer]
PublicKey = <insert key here>
AllowedIPs = 192.168.2.2/32
```
Sett inn innholdet av privatekey-filen din i `PrivateKey`-feltet.

### 5) Aktiver WireGuard-servicen
Aktiver servicen med `sudo systemctl enable wg-quick@wg0-client.service` 


## Installasjon på mobil
### 1) Installer appen for [Android](https://play.google.com/store/apps/details?id=com.wireguard.android) eller [iOS](https://itunes.apple.com/us/app/wireguard/id1441195209?ls=1&mt=8).

### 2) Opprett en profil
* Trykk +-knappen og `Create from scratch``
* Gi profilen et valgfritt navn
* Generer private key og public key
* Sett `Addresses` til 192.168.2.2/32
* Legg til en `Peer`, dette skal da vere serveren din.
* Legg til serverens `Public key` (Bruks Slack eller noe til å sende den til deg selv).
* 
### 3) Konfigurer serveren
Send `public key`-en din fra mobilen og tilbake til

## Installasjon på laptop
### 1) Installer Wireguard
#### Windows: Nei.
#### Mac: Last ned fra [appstore](https://itunes.apple.com/us/app/wireguard/id1451685025?ls=1&mt=12).
#### Linux: Se installasjon under serveroppsett
### 2) Konfigurer klient
#### Mac: Opprett ny tunell og sett opp slik som her:![Mac config](https://i.imgur.com/W60ldFj.jpg)
Dien konfig må selvsagt peke på din server.

#### Linux: Konfigurer tilsvarende som i server-oppsettet.