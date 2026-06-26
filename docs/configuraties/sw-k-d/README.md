[← Terug naar documentatieoverzicht](../../README.md)

# Configuratie van SW-K-D

> **Status:** In uitvoering
> **Apparaat:** SW-K-D
> **Laatste fase:** basisconfiguratie en management-IP
> **Praktijktests:** nog verder aan te vullen

## 1. Apparaatgegevens

| Eigenschap | Waarde |
|---|---|
| Hostname | `SW-K-D` |
| Type | Cisco Catalyst `WS-C3560-24PS-S` |
| Functie | Distributieswitch van de klantenkant |
| Management-IP | `192.168.20.2` |
| Subnetmasker | `255.255.255.0` |
| Default gateway | `192.168.20.1` |
| Netwerk | `192.168.20.0/24` |

Geplande fysieke verbindingen:

| Poort | Verbinding naar | Status tijdens controle |
|---|---|---|
| `FastEthernet0/1` | `SW-K-1` | `up/up` |
| `FastEthernet0/2` | `SW-K-2` | `up/up` |
| `FastEthernet0/24` | Router `R1` | Nog niet bevestigd tijdens deze configuratiestap |

## 2. Setupwizard afsluiten

De automatische Cisco System Configuration Dialog werd niet gebruikt. Na het afsluiten verscheen:

```text
Switch>
```

## 3. Privileged EXEC-modus openen

Commando:

```text
enable
```

Resultaat:

```text
Switch#
```

## 4. Global configuration mode openen

Commando:

```text
configure terminal
```

Resultaat:

```text
Switch(config)#
```

## 5. Hostname instellen

Commando:

```text
hostname SW-K-D
```

Resultaat:

```text
SW-K-D(config)#
```

## 6. Consoletoegang beveiligen

De consolelijn werd geopend en beveiligd met een wachtwoord.

```text
line console 0
password <CONSOLE_WACHTWOORD>
login
exit
```

`login` zorgt ervoor dat het ingestelde consolewachtwoord bij een nieuwe consolesessie wordt gevraagd.

## 7. Enable secret instellen

```text
enable secret <ENABLE_SECRET>
```

`enable secret` beveiligt de toegang tot privileged EXEC mode.

## 8. Wachtwoordweergave beveiligen

Commando:

```text
service password-encryption
```

Dit voorkomt dat lijnwachtwoorden als gewone leesbare tekst in de configuratie worden weergegeven. `enable secret` wordt afzonderlijk gehasht opgeslagen.

## 9. Management-IP instellen

Het management-IP werd ingesteld op interface VLAN 1.

```text
interface vlan 1
ip address 192.168.20.2 255.255.255.0
no shutdown
exit
```

Interface VLAN 1 werd gebruikt voor het management-IP van deze switch.

## 10. Default gateway instellen

Commando:

```text
ip default-gateway 192.168.20.1
```

Dit adres verwijst naar router `R1` aan de klantenkant.

## 11. Configuratie controleren

Gebruikt controlecommando:

```text
do show ip interface brief
```

De relevante uitvoer bevestigde:

```text
Vlan1              192.168.20.2    up    up
FastEthernet0/1     unassigned      up    up
FastEthernet0/2     unassigned      up    up
```

Hieruit bleek:

- `Vlan1` had het correcte management-IP;
- `Vlan1` stond op `up/up`;
- `FastEthernet0/1` stond op `up/up`;
- `FastEthernet0/2` stond op `up/up`;
- de overige niet-aangesloten poorten konden nog `down/down` tonen.

## 12. Configuratie opslaan

Het voorziene commando om de actieve configuratie permanent te bewaren is:

```text
copy running-config startup-config
```

De configuratie moest als laatste stap nog worden opgeslagen of gecontroleerd met `copy running-config startup-config`.

## 13. Gemaakte fouten en correcties

### Onvolledig enable-secretcommando

Foutieve invoer:

```text
enable secret
```

Melding:

```text
% Incomplete command.
```

Correcte vorm:

```text
enable secret <ENABLE_SECRET>
```

### Interfacecommando in verkeerde modus

`interface vlan 1` werd eerst buiten global configuration mode geprobeerd.

Correcte werkwijze:

```text
configure terminal
interface vlan 1
```

### Foutieve schrijfwijze van default gateway

Foutieve invoer:

```text
ip default gateway 192.168.20.1
```

Correcte invoer:

```text
ip default-gateway 192.168.20.1
```

Cisco IOS vereist hier een koppelteken.

### Niet-bestaand commando

Het woord `next` werd per ongeluk in Cisco IOS ingevoerd en gaf:

```text
% Invalid input detected at '^' marker.
```

`next` werd alleen gebruikt als bericht in de begeleiding en is geen Cisco IOS-commando.

## 14. Veilig configuratieoverzicht

Wachtwoorden zijn om veiligheidsredenen vervangen door placeholders.

```text
enable
configure terminal

hostname SW-K-D

line console 0
password <CONSOLE_WACHTWOORD>
login
exit

enable secret <ENABLE_SECRET>

service password-encryption

interface vlan 1
ip address 192.168.20.2 255.255.255.0
no shutdown
exit

ip default-gateway 192.168.20.1

do show ip interface brief

end
copy running-config startup-config
```

## 15. Nog aan te vullen

- bevestigen of `copy running-config startup-config` succesvol werd voltooid;
- status van `FastEthernet0/24` controleren;
- pingtest naar gateway `192.168.20.1` uitvoeren;
- pingtest naar toestellen in het klantennetwerk uitvoeren;
- eventuele controle-uitvoer zonder wachtwoorden toevoegen.
