# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 


## Fuldautomatisk rutine til eksport af fakturaer i NOVAX til e-conomic

### Introduktion

**NovaXport** henter fakturaer oprettet og gemt i NOVAX-systemet og eksporterer dem til et regnskab oprettet i e-conomic og bogfører dem dér - helt automatisk.

Er kunden/klienten ikke oprettet i e-conomic, oprettes den. Også fakturerede ydelser og produkter, som ikke er oprettet i e-conomic, oprettes automatisk.

**NovaXport** er et Windows-program, der kører kontinuert på en Windows-maskine, der skal have adgang til NOVAX-systemet og dets oprettede fakturafiler. Disse hentes løbende, registreres og konverteres, hvorefter de sendes til og bogføres i e-conomic.

Flowet er ligefremt:

![NovaXport Flow][Data flow] 

Én installation af **NovaXport** kan uden videre håndtere flere klinikker og flere selskaber i e-conomic.

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

- logger forløbet i Windows Logbog
- holder pause til næste kørsel

De eksporterede fakturaer vil i e-conomic kunne ses på sædvanlig måde, dels i Arkiv under Salg, dels i loggen.

> Bemærk, at fakturaerne, som i de fleste tilfælde vil indeholde personfølsomme oplysninger, kun læses og eksporteres. De gemmes på intet tidspunkt på maskinen, der afvikler tjenesten. Kun fakturanummer og -dato og selskabet, fakturaerne hører til, opbevares i fakturalisten.


## Supplerende artikler

De nærmere detaljer i installation, konfiguration mv. af **NovaXport** findes her:

- [Installation][Installation]
- [Konfiguration][Configuration]
- [Kontrol og vedligeholdelse][Maintenance]


## Information og support

Kontakt [Cactus Data](mailto:cactus@cactus.dk?subject=NovaXport) for yderligere information om **NovaXport**.

For hjælp til **NovaXport**, kontakt service@cactus.dk.

![Cactus Data ApS][Cactus Data logo]
<hr>


[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps
<!-- [Configuration]: https://github.com/CactusData/NovaXport/blob/main/Configuration.md -->
[Configuration]: https://cactuscata/github.io/NovaXport/Configuration
[Installation]: https://github.com/CactusData/NovaXport/blob/main/Installation.md
[Maintenance]: https://github.com/CactusData/NovaXport/blob/main/Maintenance.md