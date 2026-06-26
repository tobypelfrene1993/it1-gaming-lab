# IT1 Gaming Lab

Documentatie van een fysiek Cisco-netwerklabo voor een klein gamecenter.

## Status

**Documentatie in uitvoering**

Deze repository wordt in de loop van het labo verder aangevuld en aangepast.

## Netwerkarchitectuur

![Netwerkarchitectuur van het gamecenter](docs/assets/gamecenter-netwerkarchitectuur.svg)

## Netwerken

- Personeelsnetwerk: `192.168.10.0/24`
- Klantennetwerk: `192.168.20.0/24`

Beide netwerken worden via één fysieke router met elkaar verbonden.

## Volledige documentatie

[Open de volledige documentatie](docs/gamecenter-netwerklabo.md)

[Open het documentatieoverzicht](docs/README.md)

## Belangrijke regels

- Wijzig geen IP-adressen.
- Wijzig geen subnetmaskers.
- Wijzig geen poorten.
- Wijzig de fysieke topologie niet.
- Voeg geen VLAN's, trunks, DHCP, NAT, ACL's of andere onbevestigde technieken toe.
- Verwijder geen bestaande correcte documentatie.
- Publiceer geen wachtwoorden, tokens of andere gevoelige informatie.
- Gebruik geen force push.
- Herschrijf geen bestaande Git-geschiedenis.
