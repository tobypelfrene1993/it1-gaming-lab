[← Terug naar documentatieoverzicht](../../README.md)

# Fabrieksreset van Cisco-router en switches

> **Status:** Bevestigde laboprocedure
> **Toepassing:** Cisco 1841-router en Cisco Catalyst WS-C3560-24PS-S-switches
> **Waarschuwing:** Deze procedures verwijderen de opgeslagen configuratie.

## 1. Doel

Deze procedure wordt gebruikt om de gebruikte Cisco-toestellen na het labo terug te zetten naar een lege configuratie.

Een gewone herstart behoudt de configuratie wanneer de actieve configuratie eerst naar de startup-config werd gekopieerd. Een fabrieksreset verwijdert de opgeslagen startup-config, waardoor het toestel opnieuw leeg opstart.

## 2. Belangrijke waarschuwingen

> **Waarschuwing**
>
> - Maak alleen een reset wanneer de leerkracht of verantwoordelijke toestemming geeft.
> - Controleer eerst of belangrijke configuraties gedocumenteerd zijn.
> - Wis nooit het IOS-image.
> - Gebruik nooit `erase flash:` of `format flash:`.
> - Onderbreek de stroom niet tijdens kritieke schrijfbewerkingen.
> - Bewaar geen echte wachtwoorden in documentatie of screenshots.

## 3. Gewone herstart zonder configuratieverlies

Voor router en switch:

```text
enable
copy running-config startup-config
reload
```

De configuratie blijft behouden wanneer ze eerst met `copy running-config startup-config` naar de startup-config werd gekopieerd.

## 4. Cisco 1841-router terugzetten naar fabrieksconfiguratie

### Situatie A: toegang tot privileged EXEC mode

```text
enable
configure terminal
config-register 0x2102
end

erase startup-config
reload
```

Wanneer gevraagd wordt om de gewijzigde configuratie op te slaan, antwoord:

```text
no
```

Bij deze melding druk je op Enter:

```text
Proceed with reload? [confirm]
```

Na de herstart verschijnt mogelijk:

```text
Would you like to enter the initial configuration dialog? [yes/no]:
```

Antwoord:

```text
no
```

### Controle achteraf

```text
enable
show version
show startup-config
show ip interface brief
```

De normale configuratieregisterwaarde moet zijn:

```text
0x2102
```

Claim alleen dat dit bevestigd is wanneer er concrete uitvoer beschikbaar is.

## 5. Cisco 1841-router resetten zonder enable-wachtwoord

Deze procedure gebruikt password recovery via ROMMON.

### Consoleverbinding

```text
COM-poort: COM6
Snelheid: 9600
Data bits: 8
Stop bits: 1
Parity: None
Flow control: None
```

Tijdens het labo werd een StarTech/Prolific USB-naar-serieeladapter gebruikt.

### ROMMON openen

1. PuTTY-seriële sessie openen.
2. Router uitschakelen.
3. Router opnieuw inschakelen.
4. Tijdens het opstarten in het geopende PuTTY-venster kiezen voor `Special Command → Break`.
5. Wachten op:

```text
rommon 1 >
```

### Bestaande startup-config tijdelijk negeren

```text
confreg 0x2142
reset
```

`0x2142` zorgt ervoor dat de bestaande startup-config tijdens de volgende boot tijdelijk wordt genegeerd.

### Router leegmaken en register herstellen

Na het opstarten:

```text
no
enable
erase startup-config

configure terminal
config-register 0x2102
end

reload
```

Wanneer gevraagd wordt om de huidige configuratie op te slaan:

```text
no
```

Na de definitieve herstart moet de router normaal opstarten met configuratieregister `0x2102`.

### Controle

```text
show version
show startup-config
```

Deze procedure verwijdert de startup-config en niet het IOS-image.

## 6. Cisco Catalyst WS-C3560-24PS-S resetten

```text
enable
erase startup-config
delete flash:vlan.dat
reload
```

Bij deze vraag druk je op Enter:

```text
Delete filename [vlan.dat]?
```

Bij deze vraag druk je opnieuw op Enter:

```text
Delete flash:/vlan.dat? [confirm]
```

Wanneer gevraagd wordt om de huidige configuratie op te slaan:

```text
no
```

Na de herstart verschijnt mogelijk:

```text
Would you like to enter the initial configuration dialog? [yes/no]:
```

Antwoord:

```text
no
```

`erase startup-config` verwijdert de opgeslagen switchconfiguratie. `delete flash:vlan.dat` verwijdert opgeslagen VLAN-informatie. Wanneer `vlan.dat` niet bestaat, kan de reset gewoon worden voortgezet. Het IOS-image mag niet verwijderd worden.

## 7. Controle na switchreset

```text
enable
show startup-config
show vlan brief
show ip interface brief
show running-config
```

De switch kan opnieuw een standaardhostname zoals `Switch` tonen en hoort geen eerder management-IP meer te hebben.

## 8. Resetchecklist

### Vooraf

- [ ] Toestemming gekregen
- [ ] Configuratie gedocumenteerd
- [ ] Geen belangrijke wijzigingen meer nodig
- [ ] Juiste consoleverbinding actief
- [ ] Juiste router of switch geselecteerd

### Router

- [ ] Startup-config gewist
- [ ] Configuratieregister terug op `0x2102`
- [ ] Reload uitgevoerd
- [ ] Configuratie niet opnieuw opgeslagen
- [ ] Setupwizard overgeslagen
- [ ] Lege configuratie gecontroleerd

### Switch

- [ ] Startup-config gewist
- [ ] `vlan.dat` verwijderd of niet aanwezig
- [ ] Reload uitgevoerd
- [ ] Configuratie niet opnieuw opgeslagen
- [ ] Setupwizard overgeslagen
- [ ] Lege configuratie gecontroleerd

## 9. Veelvoorkomende fouten

| Probleem | Mogelijke oorzaak | Oplossing |
|---|---|---|
| Oud wachtwoord wordt nog gevraagd | Startup-config werd opnieuw opgeslagen | Reset opnieuw uitvoeren en bij save-vraag `no` antwoorden |
| Router blijft startup-config negeren | Register staat nog op `0x2142` | Terugzetten naar `0x2102` |
| VLAN's blijven bestaan | `vlan.dat` werd niet verwijderd | Bestand verwijderen en opnieuw herstarten |
| Toestel start niet normaal op | IOS-image mogelijk beschadigd of verwijderd | Stoppen en leerkracht/netwerkbeheerder inschakelen |
| Geen console-uitvoer | Verkeerde COM-poort of seriële instellingen | COM-poort en `9600 8N1` controleren |
| Break-signaal werkt niet | Break werd niet tijdens boot verzonden | Router opnieuw starten en PuTTY `Special Command → Break` gebruiken |
