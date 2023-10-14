# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 

## Fuldautomatisk rutine til eksport af fakturaer i NOVAX til e-conomic

### Introduktion

**NovaXport** henter fakturaer oprettet og gemt i NOVAX-systemet og eksporterer dem til et regnskab oprettet i e-conomic og bogfører dem dér - helt automatisk.

Er kunden/klienten ikke oprettet i e-conomic, oprettes den. Også fakturerede ydelser og produkter, som ikke er oprettet i e-conomic, oprettes automatisk.

**NovaXport** er et Windows-program, der kører kontinuert på en Windows-maskine, der skal have adgang til NOVAX-systemet og dets oprettede fakturafiler. Disse hentes løbende, registreres og konverteres, hvorefter de sendes til og bogføres i e-conomic.

Flowet er ligefremt:

![NovaXport Flow][Data flow] 

Én installation af **NovaXport** kan håndtere flere klinikker og flere selskaber i e-conomic.

### Funktionen i detaljer

NOVAX-systemet gemmer alle fakturaer i en mappe på serveren, der afvikler NOVAX-systemet for klinikken, i standard OIOUBL-dokumentformatet.

Efter en tidsplan, som i flere intervaller kan justeres fra ét minut til mange timer hen over døgnet, gør **NovaXport** følgende:

- logger ind på filserveren
- henter den aktuelle fakturaliste
- opdaterer sin egen fakturaliste, som gemmes i en lokal database
- logger ud af serveren

Dette forløb gentages for hver yderligere server, hvis flere klinikker håndteres.

Dernæst eksporteres de fakturaer, der ikke allerede er eksporteret:

- henter fra sin fakturaliste en liste over fakturaer, der ikke er eksporteret
- logger ind på filserveren
- læser den første faktura og eksporterer den til e-conomic
- bogfører fakturaen i e-conomic
- markerer fakturaen i fakturalisten som eksporteret
- fortsætter med at læse, eksportere og bogføre fakturaer, indtil der ikke er flere
- logger ud af filserveren

Dette forløb gentages for hver yderligere server, hvis flere klinikker håndteres.

Til sidst:

- forløbet logges i Windows Logbog
- holder pause til næste kørsel

De eksporterede fakturaer vil i e-conomic kunne ses på sædvanlig måde, dels i Arkiv under Salg, dels i loggen.

## Installation

### NOVAX

Kun NOVAX har administratorrettighed til serveren, der afvikler NOVAX-systemet. Det skal derfor aftales med NOVAX' support, at de gør følgende:

- deler mappen med fakturafilerne som et Windows SMBv3 share, fx med navnet *Faktura*
- opretter en ny bruger, fx med navnet *NovaxEksport*, med læseadgang til den delte mappe

### E-conomic



### Netværk og maskine

Serveren, der afvikler NOVAX-systemet, er typisk installeret i et lukket netværk, der hører til NOVAX, og som kun NOVAX har adgang til. Det er blandt andet derfor, at NOVAX-systemet betjenes via Windows Fjernskrivebord.

For at give **NovaXport** adgang til serveren, er der tre muligheder:

- opsætte en separat server ind i deres netværk, hvor **NovaXport** kan installeres
- etablere en VPN-forbindelse mellem NOVAX' netværk og jeres eget, så **NovaXport** kan køre på en maskine hos jer selv
- installere **NovaXport** på den eksisterende NOVAX-server

Ingen af de tre muligheder kan realiseres uden en aftale med NOVAX.

Den første mulighed kan være kostbar. Den sidste vil NOVAX formentlig ikke tillade, fordi de har ansvaret for maskinens drift. Den nemmeste og mest sandsynlige mulighed er derfor VPN-forbindelsen, som kan etableres ved en aftale mellem jeres IT-folk og NOVAX. Når den er oprettet, kan en Windows-maskine under jeres kontrol sættes op, og **NovaXport** installeres på den.



## Konfiguration

### Programdata

Datafilerne til **NovaXport** skal ligge i mappen `Novax Export` i systemmappen `%ProgramData%`. 

Typisk vil det være mappen:

- `C:\ProgramData\Novax Export`

Her findes to datafiler:

- `Credentials.xml`
- `NovaxData.db`

Det håndteres således:

#### Credentials

Den første indeholder brugernavn og adgangskode til den delte mappe med fakturafilerne på den eller de filservere, der afvikler NOVAX.

> Det anbefales kraftigt ikke at bruge en normal brugerkonto, men at oprette en speciel brugerkonto kun med læseadgang til den delte mappe.
>
> Dette skal gøres af NOVAX, hvis de administrerer serveren.

Adgangen styres af de tre felter:

- `Domain`
- `Username`
- `Password`

Det fjerde felt:

- `AppSecretToken`

indeholder det token, **NovaXport** er tildelt af e-conomic. Det må ikke rettes eller slettes.

NOVAX' domæne hedder typisk `HOSTING`, og hvis brugerkontoen hedder `NovaxEksport`, og adgangskoden er `VoresMegetLangeAdgangskode`, skal filen rettes til således:

```htm
<?xml version="1.0" encoding="utf-8"?>
<ReaderAccount>
    <Domain>HOSTING</Domain>
    <Username>NovaxEksport</Username>
    <Password>VoresMegetLangeAdgangskode</Password>
    <AppSecretToken>2RIukREbtPIvuN90ry33My1cazJtx3LUX9bFsCMVBA81</AppSecretToken>
</ReaderAccount>
```



## Tilføj selskab (klinik)

For at NovaXport kan sende fakturaer til e-conomic, skal to ting være på plads.

For det første skal appen NovaXport i e-conomic <i>tilknyttes selskabet</i> ved at blive tilføjet listen over apps, som selskabet kan kommunikere med.

Dernæst skal selskabet i NovaXPorts <i>tilføjes listen over selskaber</i>, der kan eksporteres fakturaer til.

Når appen (NovaXport) tilføjes i e-conomic, generes en <i>unik nøgle</i> (Adgangs-ID/token), som identificerer de to over for hinanden. Med dette Adgangs-ID:

- accepterer e-conomic, at dette selskab kan kommunikere med NovaXport
- fortæller NovaXport, når den kontakter e-conomic, hvilket selskab der skal kommunikeres med 

### Tilknyt app i e-conomic

Du skal have en superbruger/administrator-brugerkonto til e-conomic til rådighed.

1. Log ind med denne brugerkonto på [Visma Home](https://connect.visma.com/)
3. Hvis I har flere selskaber, så vælg det, der skal eksporteres til fra NOVAX
4. Vælg fra topmenuen <b>Indstillinger</b> punktet <u>Alle indstillinger</u>
5. Vælg i menuen til venstre punktet <u>Apps</u>
6. Nu vises enten en liste med de apps, der tidligere er aktiveret, eller siden: <i>Sådan tilføjer du apps</i>

7.  1. <i>Kopiér</i> nu [dette link](https://secure.e-conomic.com/secure/api1/requestaccess.aspx?appPublicToken=ToVYPF4QxTW73TcmtKPZtQCTwjKJlAwu0cPn3LEOE201)
    2. Åbn en ny side i browseren
    3. <i>Indsæt</i> derefter linket i den nye side og tryk <i>Enter</i>
    4. Denne side vises:

    <br>![Tilføj app][Attach app]<br>
    
    5. <b>Kontrollér, at aftalenummeret er korrekt</b>
    6. Klik på knappen <b>Tilføj app</b>
    7. Denne side vises nu:

    <br>![App tilføjet][Attached app]<br>

    8. Det viste ID er det <b>Adgangs-ID</b>, som skal bruges ved konfigurationen af NovaXport
    9. Luk siden

8. Opdatér [siden med dine apps i e-conomic][EC extensions] for at få vist og bekræftet tilknytningen af appen:

![App-liste][App list]

E-conomic vil nu tillade dit selskab og NovaXport at kommunikere indbyrdes.

Næste trin er at muliggøre dette for NovaXport.

### Tilknyt selskab til NovaXport



<hr>

### Information og support

![Cactus Data ApS][Cactus Data logo]



[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps