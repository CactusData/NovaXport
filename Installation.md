# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 

## Installation

### NOVAX

Kun NOVAX har administratorrettighed til serveren, der afvikler NOVAX-systemet. Det skal derfor aftales med NOVAX' support, at de gør følgende:

- deler mappen med fakturafilerne som et *Windows SMBv3 share*, helst med navnet *Faktura*
- opretter en ny bruger med et beskrivende navn, fx *NovaxEksport*, med læseadgang til denne delte mappe

Eksempler på den fysiske mappe på serveren, hvor fakturafilerne bliver gemt:
- `F:\App8\Novax\Finans`
- `F:\App8\Økonomifiler`
- `F:\C5`

### E-conomic

Der er ikke noget, der kan installeres i e-conomic, som er web-baseret, men der er en række punkter, der skal konfigureres korrekt af en bruger med administratorrettigheder.

Disse er beskrevet i: [Konfiguration][Configuration].

### Netværk

Serveren, der afvikler NOVAX-systemet, er typisk installeret i et lukket netværk, der hører til NOVAX, og som kun NOVAX har adgang til, og som NOVAX administrerer. Det er blandt andet derfor, at brugeren betjener NOVAX-systemet via *Fjernskrivebord*.

For at give **NovaXport** adgang til serveren, er der tre muligheder:

- opsætte en separat server, som **NovaXport** installeres på, inde i NOVAX' netværk
- etablere en VPN-forbindelse mellem NOVAX' netværk og jeres eget, så **NovaXport** kan køre på en maskine hos jer selv
- installere **NovaXport** på den eksisterende NOVAX-server

Ingen af de tre muligheder kan realiseres uden en aftale med NOVAX.

Den første mulighed kan være kostbar. Den sidste vil NOVAX formentlig ikke tillade, fordi de har ansvaret for maskinens drift. Den nemmeste og mest sandsynlige mulighed er derfor VPN-forbindelsen, som kan etableres ved en aftale mellem jeres IT-folk og NOVAX. Når den er oprettet, kan en Windows-maskine under jeres kontrol og administration sættes op, og **NovaXport** installeres på den.

#### VPN-forbindelse

Vælges denne løsning, skal **SMBv3**-protokollen kunne passere firewallen. Derfor skal denne port være åben:

```console
TCP 445
```

### Maskine

Der er ingen særlige krav til maskinen, der skal afvikle **NovaXport**, ud over hvad Windows kræver. Da maskinen i det daglige vil passe sig selv, kan følgende krav formuleres:

#### Krav til maskine

- minimum: Stabil maskine af anerkendt fabrikat
- anbefalet: Maskine af servertype af anerkendt fabrikat

#### Krav til Windows-version

- minimum: Windows 10 Pro (64-bit)
- anbefalet: Windows Server 2022 eller senere

Opdatering, sikkerhedskopiering og vedligeholdelse bør som minimum følge samme procedurer som for jeres øvrige maskiner.


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
  - `NovaXport_Create.cmd`
  - `NovaXport_Remove.cmd`
  - `NovaXport_Start.cmd`
  - `NovaXport_Stop.cmd`
  - `NovaXport_Status.cmd`
  - `NovaXport Service Prompt.lnk`
  - `DB.Browser.for.SQLite.msi`

#### Tjenesten

*De fem* kommando-filer indeholder de nødvendige kommandoer til - hvad deres navne siger - at:

- oprette tjenesten
- fjerne tjenesten
- starte tjenesten
- stoppe tjenesten
- vise tjenestens status

> NB: Alle kommandofiler skal køres fra en **Kommandoprompt** åbnet med administratorrettigheder.
>
> Derfor er der også inkluderet en *genvej*, `NovaXport Service Prompt`, der åbner `cmd.exe` med administratorrettigheder. Den kan med fordel kopieres til *Skrivebord*.

*Den første* kommandofil er den kritiske, for det er den, der ved gentagne kald af `sc.exe` bruges til at registrere `NovaXport.exe` som en tjeneste med den korrekte konfiguration. Den ser således ud:

```cmd
: Command file for registering NovaXport as a service.
: V 1.0.1
: 2023-10-15. Gustav Brock, Cactus Data ApS, CPH.

@echo off

: Variables.
setlocal
set novaxport=NovaXport
set displayname=Novax eksport
set description=Novax fakturaeksport til e-conomic

: Header block.
echo Opret %novaxport% tjenesten.
echo V 1.0.
echo 2023-10-15. Gustav Brock, Cactus Data ApS, CPH.
echo ----------------------------------------------
echo.

: Register the service.
echo Opretter %novaxport% som tjeneste ...
echo.
sc.exe create %novaxport% binpath="%ProgramFiles%\Novax Export\NovaXport.exe"

: Configure the names of the service and specify delayed auto start.
sc.exe config %novaxport% displayname="%displayname%"
sc.exe config %novaxport% start=delayed-auto
: Add the description to the service.
sc.exe description %novaxport% "%description%"
echo.

: Display the result.
sc.exe queryex %novaxport%
echo.

: Clean up and await a key press.
endlocal
echo Et tastetryk afslutter.
pause > nul

: EOF
```
Køres den, registreres **NovaXport** som en tjeneste, og det vises således:

```txt
C:\Program Files\Novax Export>NovaXport_Create
Opret NovaXport tjenesten.
V 1.0.
2023-10-15. Gustav Brock, Cactus Data ApS, CPH.
----------------------------------------------

Opretter NovaXport som tjeneste ...

[SC] CreateService SUCCESS
[SC] ChangeServiceConfig SUCCESS
[SC] ChangeServiceConfig SUCCESS
[SC] ChangeServiceConfig2 SUCCESS


SERVICE_NAME: NovaXport
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 0
        FLAGS              :

Et tastetryk afslutter.
```

> Bemærk, at tjenesten ikke vil være startet, da den øvrige konfigurationen måske ikke er klar, og det derfor ikke vil give mening, at starte tjenesten inden da.

Den korrekte installationen af **NovaXport** bør verificeres ved at vise Windows' liste over  installerede Windows Tjenester:

![NovaXport Service][Service installed] 

Yderligere bør **NovaXport** egenskaber verificeres. 

Disse skal vise de to navne, dens beskrivelse, om den kører, og at den startes med forsinkelse:

![NovaXport Service Properties][Service properties] 

*De fire sidste* kommandofiler er trivielle. De bruges til at *starte*, *stoppe* eller *fjerne* tjenesten eller få vist dens *status*.

Fx kan tjenesten startes med `NovaXport_Start.cmd`, og det vil give dette output:

```txt
C:\Program Files\Novax Export>NovaXport_Start.cmd
Start NovaXport tjenesten.
V 1.0.1
2023-10-15. Gustav Brock, Cactus Data ApS, CPH.
----------------------------------------------

Kalder start af NovaXport ...

Tjenesten Novax eksport starter.
Tjenesten Novax eksport er startet.


SERVICE_NAME: NovaXport
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

Et tastetryk afslutter.
```

På tilsvarende måde kan tjenesten stoppes, fjernes og få vist sin status.


#### Database-manager

Endelig skal `DB Browser (SQLite)` installeres, da den skal bruges til at studere **NovaXport**s database, vedligeholde den og justere **NovaXport**s funktion (se afsnit Vedligeholdelse).

*Den sidste* fil - *msi*-filen - er installationsfilen hertil.

Køres installationen med standardindstillinger og -valg, oprettes en genvej på Skrivebord (*DB Browser (SQLite)*). Den kan med fordel trimmes til at have disse indstillinger:

```
Destination: 
    "%ProgramFiles%\DB Browser for SQLite\DB Browser for SQLite.exe" -t Company NovaxData.db

Start i:
    "%ProgramData%\Novax Export"
```
og omdøbes til fx: **NovaXport Database**. 

Åbnes genvej med denne tilpasning, viser *DB Browser for SQLite* straks tabellen *Company*, og de øvrige tabeller kan man uden videre vælge også at få vist:

![NovaxData Company][Display table Company] 

Tabellerne styrer den daglige funktion af **NovaXport**. Hvordan er beskrevet under 
[Konfiguration][Configuration].

<hr>

[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[Service installed]: images/NovaXport%20Service.png
[Service properties]: images/NovaXport%20Service%20Properties.png
[Display table Company]: images/NovaxDataCompany.png
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps
[Configuration]: https://github.com/CactusData/NovaXport/blob/main/Configuration.md