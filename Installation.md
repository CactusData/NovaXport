# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 

## Installation

### NOVAX

Kun NOVAX har administratorrettighed til serveren, der afvikler NOVAX-systemet. Det skal derfor aftales med NOVAX' support, at de gør følgende:

- deler mappen med fakturafilerne som et Windows SMBv3 share, fx med navnet *Faktura*
- opretter en ny bruger, fx med navnet *NovaxEksport*, med læseadgang til den delte mappe

### E-conomic

Der er ikke noget, der kan installeres i e-conomic, som er web-baseret, men der er en række punkter, der skal konfigureres korrekt af en bruger med administratorrettigheder.

Disse er beskrevet i: [Konfiguration](https://github.com/CactusData/NovaXport/blob/main/Configuration.md).

### Netværk

Serveren, der afvikler NOVAX-systemet, er typisk installeret i et lukket netværk, der hører til NOVAX, og som kun NOVAX har adgang til. Det er blandt andet derfor, at NOVAX-systemet betjenes via Windows Fjernskrivebord.

For at give **NovaXport** adgang til serveren, er der tre muligheder:

- opsætte en separat server ind i deres netværk, hvor **NovaXport** kan installeres
- etablere en VPN-forbindelse mellem NOVAX' netværk og jeres eget, så **NovaXport** kan køre på en maskine hos jer selv
- installere **NovaXport** på den eksisterende NOVAX-server

Ingen af de tre muligheder kan realiseres uden en aftale med NOVAX.

Den første mulighed kan være kostbar. Den sidste vil NOVAX formentlig ikke tillade, fordi de har ansvaret for maskinens drift. Den nemmeste og mest sandsynlige mulighed er derfor VPN-forbindelsen, som kan etableres ved en aftale mellem jeres IT-folk og NOVAX. Når den er oprettet, kan en Windows-maskine under jeres kontrol sættes op, og **NovaXport** installeres på den.

#### VPN-forbindelse

Vælges denne løsning, skal **SMBv3**-protokollen kunne passere firewallen:

```console
TCP 445
```

### Maskine

Der er ingen særlige krav til maskinen, der skal afvikle **NovaXport**, ud over hvad Windows kræver. Da maskinen i det daglige vil passe sig selv, kan følgende krav formuleres:

#### Krav, maskine

- minimum: Stabil maskine af anerkendt fabrikat
- anbefalet: Maskine af servertype af anerkendt fabrikat

#### Krav, Windows

- minimum: Windows 10 Pro (64-bit)
- anbefalet: Windows Server 2022 eller senere

Sikkerhedskopiering og vedligeholdelse bør følge samme procedurer som for øvrige servere.


### NovaXport

#### Filer

Der er ikke nogen installationsrutine til **NovaXport**. De nødvendig filer leveres i en zip-fil, som indeholder disse to mapper:

- ProgramFiles
- ProgramData

Efter login på maskinen med en administratorkonto kan filerne herfra kopieres ind i de tilsvarende Windows systemmapper:

- %ProgramFiles%
- %ProgramData%

som typisk vil være disse fysiske mapper:

- `C:\Program Files`
- `C:\ProgramData`

Herefter skal der som minimum være disse filer i mapperne:

- `C:\Program Files\Novax Export`
  - `appsettings.json`
  - `NovaXport.exe`
- `C:\ProgramData\Novax Export`
  - `Credentials.xml`
  - `NovaxData.db`

Der vil i programmappen også være disse hjælpefiler:

- `C:\Program Files\Novax Export`
  - `NovaXport Create.ps1`
  - `NovaXport Delete.ps1`
  - `NovaXport Start.ps1`
  - `NovaXport Stop.ps1`
  - `DB.Browser.for.SQLite.msi`

#### Installation af NovaXport tjenesten

De fire PowerShell-filer indeholder de nødvendige kommandoer til - hvad deres navne siger - at:

- oprette tjenesten
- slette tjenesten
- starte tjenesten
- stoppe tjenesten



Den femte er installationsfilen til `DB Browser (SQLite)`, som - hvis den installeres - kan bruges til at studere **NovaXport**s database.



<hr>

[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps