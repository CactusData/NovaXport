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

Der er ikke noget, der kan installeres i e-conomic, da den er web-baseret, men der er en række punkter, der skal konfigureres korrekt af en bruger med administratorrettigheder.

Disse er beskrevet i: [Konfiguration][Configuration].

### Netværk

Serveren, der afvikler NOVAX-systemet, er typisk installeret i et lukket netværk, der hører til NOVAX, og som kun NOVAX har adgang til, og som NOVAX administrerer. Det er blandt andet derfor, at brugeren betjener NOVAX-systemet via *Fjernskrivebord*.

For at give **NovaXport** adgang til serveren, er der tre muligheder:

- opsætte en separat server, som **NovaXport** installeres på, inde i NOVAX' netværk
- etablere en VPN-forbindelse mellem NOVAX' netværk og jeres eget, så **NovaXport** kan køre på en maskine hos jer selv
- installere **NovaXport** på den eksisterende NOVAX-server

Ingen af de tre muligheder kan realiseres uden en aftale med NOVAX.

Den første mulighed kan være kostbar. Den sidste vil NOVAX formentlig ikke tillade, fordi de har ansvaret for maskinens drift. Den nemmeste og mest sandsynlige mulighed er derfor VPN-forbindelsen, som kan etableres ved en aftale mellem klinkkens IT-folk og NOVAX. Når den er oprettet, kan en Windows-maskine under klinikkens kontrol og administration sættes op, og **NovaXport** installeres på den.

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
- anbefalet: Windows Server 2022 eller senere, *Core* edition eller *Desktop* edition

Opdatering, sikkerhedskopiering og vedligeholdelse bør som minimum følge samme procedurer som for klinikkens øvrige maskiner.


### NovaXport

#### Filer

Der er ikke nogen installationsrutine til **NovaXport**. De nødvendig filer leveres i en zip-fil og en kommandofil, der kan bruges til at pakke den zip-filen ud:

- `NovaXport.zip`
- `NovaXport_Unpack.cmd`

De kan med fordel kopieres til brugernes fællesmappe _Delte filer_ (typisk _C:\Users\Public_).

Zip-filen indeholder disse to mapper:

- Program Files
- ProgramData

Efter login på maskinen med en administratorkonto kan filerne herfra kopieres ind i de tilsvarende Windows systemmapper:

- %ProgramFiles%
- %ProgramData%

som typisk vil være disse fysiske mapper:

- `C:\Program Files`
- `C:\ProgramData`

Det kan gøres enten manuelt med _Stifinder_ eller fx ved at åbne et Terminal-vindue i mappen og kalde kommandofilen således:

    .\NovaXport_Unpack.cmd

Herefter skal der som minimum være disse filer i mapperne:

- `C:\Program Files\Novax Export`
  - `appsettings.json`
  - `NovaXport.exe`
- `C:\ProgramData\Novax Export`
  - `Credentials.xml`
  - `NovaxData.db`

Der vil i programmappen også være to undermapper, `Command` og `Desktop Shortcuts`, med disse hjælpefiler:

- `C:\Program Files\Novax Export`
  - `Command\NovaXport_Create.cmd`
  - `Command\NovaXport_Remove.cmd`
  - `Command\NovaXport_Start.cmd`
  - `Command\NovaXport_Stop.cmd`
  - `Command\NovaXport_Status.cmd`
  - `Desktop Shortcuts\NovaXport Service Prompt.lnk`
  - `Desktop Shortcuts\NovaXport Credentials.lnk`
  - `Desktop Shortcuts\NovaXport Database.lnk`
  - `Desktop Shortcuts\NovaXport Log.lnk`

Desuden er der undermapper med hjælpeprogrammer til visning af databasen og logbogen:

  - `SQLiteBrowser`
  - `Nirsoft`

  Brugen af disse omtales under [Konfiguration][Configuration] og [Kontrol og vedligeholdelse][Maintenance].

  Genvejene fra undermappen `Desktop Shortcuts` kan med fordel kopieres til _Skrivebord_, da man så har stort set alt for hånden til pasning af **NovaXport**.

  > TIP: Genvejene kan også kaldes med _PowerShell_ fx under _Windows Server Core Edition_, blot skal det fulde filnavn angives, fx:
  >
  > `cmd /r "NovaXport Service Prompt.lnk"`

#### Tjenesten

*De fem* kommando-filer indeholder de nødvendige kommandoer til - hvad deres navne siger - at:

- oprette tjenesten
- fjerne tjenesten
- starte tjenesten
- stoppe tjenesten
- vise tjenestens status

> NB: Alle kommandofiler skal køres fra en **Kommandoprompt** åbnet med administratorrettigheder.
>
> Derfor er der også (se ovenfor) inkluderet en *genvej*, `NovaXport Service Prompt`, der åbner `cmd.exe` med administratorrettigheder. Den kan umiddelbart kopieres til *Skrivebord*.

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

Endelig skal `DB Browser (SQLite)` installeres, da den skal bruges til at opsætte **NovaXport**s database, vedligeholde den og justere **NovaXport**s funktion (se [Konfiguration][Configuration] og [Kontrol og vedligeholdelse][Maintenance]).

Installationsfilen hertil, `DB.Browser.for.SQLite` ligger i undermappen `SQLiteBrowser`.

installationen kan gennemføres med standardindstillinger og -valg. Herefter kan genvejen `NovaXport Database` (se ovenfor) bruges til at åbne **NovaXport**s database direkte.

Bruger man genvejen, åbnes *DB Browser for SQLite* straks og viser tabellen *Company*, og de øvrige tabeller kan man uden videre vælge også at få vist:

![NovaxData Company][Display table Company] 


#### Log

Det medfølgende program _Nirsoft FullEventLogView_ er til visning af _Windows logbog_, som **NovaXport** skriver til ved hver kørsel.

Det er et selvstændigt program, som ikke kræver installation, men bør åbnes direkte med genvejen _NovaXport Log_, som er omtalt ovenfor.

<hr>

[Tilbage til forsiden][Main page]

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
[Configuration]: https://cactusdata.github.io/NovaXport/Configuration
[Maintenance]: https://cactusdata.github.io/NovaXport/Maintenance
[Main page]: https://cactusdata.github.io/NovaXport/
