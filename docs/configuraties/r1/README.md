[← Terug naar documentatieoverzicht](../../README.md)

# Configuratie van R1

> **Status:** In uitvoering
> **Apparaat:** Cisco 1841
> **Hostname:** R1
> **Functie:** Routering tussen personeel en klanten

## 1. Doel

`R1` verbindt de twee afzonderlijke IPv4-netwerken van het gamecenter:

- personeel: `192.168.10.0/24`;
- klanten: `192.168.20.0/24`.

Iedere fysieke routerinterface dient als default gateway voor zijn eigen subnet. De router heeft beide subnetten rechtstreeks aangesloten. Daarom zijn er voor deze twee netwerken geen extra statische routes nodig.

## 2. Consoleverbinding

| Onderdeel | Waarde |
|---|---|
| Adapter | StarTech/Prolific USB-naar-serieeladapter |
| Terminalprogramma | PuTTY |
| Verbindingstype | Serial connection |
| Windows COM-poort | `COM6` |
| Seriële snelheid | `9600` |
| Data bits | `8` |
| Stop bits | `1` |
| Parity | `None` |
| Flow control | `None` |

De consolekabel werd aangesloten op de `CONSOLE`-poort van de router. Deze kabel hoort niet in een Ethernet- of AUX-poort.

## 3. Probleem: onbekend enable-wachtwoord

De Cisco 1841 bevatte aanvankelijk een bestaande configuratie met een onbekend enable-wachtwoord. De router heeft geen fysieke fabrieksresetknop. Daarom werd password recovery via ROMMON gebruikt.

Er worden geen echte wachtwoorden in deze documentatie opgenomen.

## 4. ROMMON openen

1. PuTTY-sessie openen op `COM6`.
2. Router uitschakelen.
3. Router opnieuw inschakelen.
4. Tijdens het opstarten via het PuTTY-menu kiezen voor `Special Command → Break`.
5. De router kwam in ROMMON:

```text
rommon 1 >
```

## 5. Startup-config tijdelijk negeren

Gebruikte ROMMON-commando's:

```text
confreg 0x2142
reset
```

`0x2142` zorgt ervoor dat de bestaande startup-config tijdens de volgende boot tijdelijk wordt genegeerd. `reset` herstart de router. Dit is een password-recoveryprocedure en wist geen IOS of flashgeheugen.

Gebruik hierbij geen commando's zoals:

```text
erase flash:
format flash:
```

## 6. Configuratieregister herstellen

Na de recovery moet het configuratieregister opnieuw naar de normale waarde worden teruggezet:

```text
configure terminal
config-register 0x2102
end
```

De normale registerwaarde `0x2102` moet nog worden bevestigd met `show version`.

## 7. Basisconfiguratie

Alle wachtwoorden zijn om veiligheidsredenen vervangen door placeholders.

```text
enable
configure terminal

hostname R1

enable secret <ENABLE_SECRET>

line console 0
password <CONSOLE_WACHTWOORD>
login
exit

line vty 0 4
password <VTY_WACHTWOORD>
login
exit

service password-encryption

banner motd #HYDRATE#
```

| Onderdeel | Uitleg |
|---|---|
| `hostname R1` | Stelt de routerhostname in. |
| `enable secret <ENABLE_SECRET>` | Beveiligt privileged EXEC mode. |
| `line console 0` | Opent de consolelijnconfiguratie. |
| `password <CONSOLE_WACHTWOORD>` en `login` | Vraagt bij een nieuwe consolesessie een consolewachtwoord. |
| `line vty 0 4` | Opent de VTY-lijnen voor beheer op afstand. |
| `password <VTY_WACHTWOORD>` en `login` | Beveiligt de VTY-lijnen met een wachtwoord. |
| `service password-encryption` | Versleutelt lijnwachtwoorden in de configuratieweergave. |
| `banner motd #HYDRATE#` | Toont een melding bij het verbinden met het apparaat. |

## 8. Routerinterfaces configureren

```text
interface FastEthernet0/0
description NAAR-SW-P-D-FA0-24
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

interface FastEthernet0/1
description NAAR-SW-K-D-FA0-24
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
```

`FastEthernet0/0` is de personeelskant. `FastEthernet0/1` is de klantenkant. `no shutdown` activeert de interface. De IP-adressen eindigen op `.1` en zijn de default gateways van beide subnetten.

Op deze router worden de IP-adressen rechtstreeks op de fysieke routerinterfaces ingesteld. Gebruik hier dus geen `interface vlan 1` of `ip default-gateway`.

## 9. Configuratie controleren

Veilige controlecommando's:

```text
show ip interface brief
show ip route
show interfaces description
show version
```

Verwachte toestand:

```text
FastEthernet0/0    192.168.10.1    up    up
FastEthernet0/1    192.168.20.1    up    up
```

`show ip route` moet beide `/24`-netwerken als rechtstreeks verbonden routes tonen. Voeg pas volledige command output toe wanneer die veilig beschikbaar is en geen wachtwoorden bevat.

## 10. Troubleshooting: dubbel IP-adres

Tijdens het labo werd deze foutmelding gezien:

```text
%IP-4-DUPADDR: Duplicate address 192.168.10.1 on FastEthernet0/0
```

Situatie:

- `R1 FastEthernet0/0` gebruikte correct `192.168.10.1`;
- een ander toestel in hetzelfde personeelsnetwerk had per ongeluk eveneens `192.168.10.1`;
- daardoor detecteerde de router een dubbel IP-adres.

Oorzaak: `192.168.10.1` was op meer dan één toestel ingesteld.

Oplossing:

- het foutief ingestelde toestel werd opgespoord;
- het dubbele adres werd daar aangepast;
- alleen `R1 FastEthernet0/0` bleef `192.168.10.1` gebruiken.

Resultaat:

- de duplicate-addressmelding verdween;
- de netwerkcommunicatie werkte daarna opnieuw.

Leerpunt: `Ieder IPv4-adres moet uniek zijn binnen hetzelfde subnet.`

| Onderdeel | Inhoud |
|---|---|
| Symptoom | Duplicate-addressmelding |
| Interface | `FastEthernet0/0` |
| Correct router-IP | `192.168.10.1` |
| Oorzaak | Zelfde IP op een ander toestel |
| Oplossing | IP-adres op het foutieve toestel aangepast |
| Resultaat | Verbinding hersteld |

## 11. Ping- en connectiviteitstests

Controleplan vanaf `R1`:

```text
ping 192.168.10.2
ping 192.168.20.2
```

Deze testen controleren:

- `R1` naar `SW-P-D`;
- `R1` naar `SW-K-D`.

Latere eindtests zijn mogelijk naar:

```text
192.168.10.5
192.168.10.6
192.168.10.7
192.168.10.8

192.168.20.5
192.168.20.6
192.168.20.7
192.168.20.8
192.168.20.9
```

Markeer individuele pingresultaten alleen als geslaagd wanneer er concrete output beschikbaar is. De algemene melding dat het netwerk na het oplossen van het IP-conflict opnieuw werkte, is wel als opgelost resultaat opgenomen.

## 12. Configuratie opslaan

```text
copy running-config startup-config
```

Dit schrijft de actieve configuratie naar NVRAM.

De save-stap moet nog definitief worden bevestigd of gecontroleerd.

## 13. Veilig volledig configuratieoverzicht

Alle wachtwoorden zijn om veiligheidsredenen vervangen door placeholders.

```text
enable
configure terminal

hostname R1

enable secret <ENABLE_SECRET>

line console 0
password <CONSOLE_WACHTWOORD>
login
exit

line vty 0 4
password <VTY_WACHTWOORD>
login
exit

service password-encryption

banner motd #HYDRATE#

interface FastEthernet0/0
description NAAR-SW-P-D-FA0-24
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

interface FastEthernet0/1
description NAAR-SW-K-D-FA0-24
ip address 192.168.20.1 255.255.255.0
no shutdown
exit

end

show ip interface brief
show ip route

copy running-config startup-config
```

## 14. Resterende controles

- [ ] `show version` bevestigt configuratieregister `0x2102`
- [ ] beide interfaces staan `up/up`
- [ ] beide rechtstreeks verbonden routes zijn aanwezig
- [ ] ping naar `SW-P-D` werkt
- [ ] ping naar `SW-K-D` werkt
- [ ] communicatie tussen beide subnetten werkt
- [ ] `copy running-config startup-config` is met `[OK]` bevestigd
