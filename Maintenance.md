# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 

## Kontrol og vedligeholdelse

### Rutinekontrol

#### 1. Loggen

**NovaXport**s aktivitet logges i _Windows Logbog_. 

Det lille program _Logbog_, der følger med Windows kan bruges ved heri at slå op under _Windows-logger_ og _Program_ og se efter *Kilde* NovaXport. Men det er langsomt, så det anbefales at bruge *Nirsoft FullEventLogView*, der ligger i undermappen _Nirsoft_ og _genvejen_ hertil, `NovaXPort Log` (begge omtalt under [Installation][Installation]).

Åbnes denne genvej vises straks loggen for **NovaXport** med alt andet filtreret fra.


Ved start og stop af tjenesten logges følgende:

| Niveau      | Hændelses-id | Generelt                  |
| :---------- | -----------: | :------------------------ |
| Oplysninger | 0      | Service started successfully    |
| *Kørsler*   |        | *Se næste tabel*                                |          
| Oplysninger | 0      | Application is shutting down... |
| Oplysninger | 0      | Service stopped successfully    |

Når **NovaXport** kører planmæssigt, sker følgende ved hver kørsel:

1. Tjenesten logger ind og ud af serverne, der læses fakturafiler fra
2. Fakturalisten i databasen ajourføres
3. Fakturaer eksporteres til e-conomic
4. Kørslen logges
5. Pause holdes til næste kørsel

Hver kørsel logges i maskinens *Logbog* under *Windows-logger/Program* med NovaXport angivet som *Kilde*.

Der vil typisk være disse hændelser (kolonnerne *Dato/tid* og *Kilde* er udeladt), her vist for to fakturaer:

| Niveau      | Hændelses-id | Generelt                                                    |
| :---------- | -----------: | :---------------------------------------------------------- |
| Oplysninger | 2960         | Dato/tid - Læser start.                                     |
| Oplysninger | 2970         | Dato/tid - Læser selskab 11223344 faktura 123456: OK.       |
| Oplysninger | 2970         | Dato/tid - Læser selskab 11223344 faktura 123457: OK.       |
| Oplysninger | 2961         | Dato/tid - Læser slut.                                      |
| Oplysninger | 2962         | Dato/tid - Eksporterer start.                               |
| Oplysninger | 2980         | Dato/tid - Eksporterer selskab 11223344 faktura 123456: OK. |
| Oplysninger | 2980         | Dato/tid - Eksporterer selskab 11223344 faktura 123457: OK. |
| Oplysninger | 2963         | Dato/tid - Eksporterer slut.                                |
| Oplysninger | 2964         | Dato/tid - Søger manglende start.                           |
| Oplysninger | 2990         | Dato/tid - Søger selskab 11223344 faktura 123454: Mangler.  |
| Oplysninger | 2965         | Dato/tid - Søger manglende slut.                            |
| Oplysninger | 2966         | Dato/tid - Fakturamangelliste sendt: OK.                    |
| Oplysninger | 2950         | Dato/tid - Pause i 15 minutter.                             |

Hvis *Læsning* og/eller *Eksport* skulle fejle, logges følgende for Hændelses-id 2970 hhv. 2990:

| Niveau      | Hændelses-id | Generelt                                                      |
| :---------- | -----------: | :------------------------------------------------------------ |
| Advarsel    | 2979         | Dato/tid - Læser selskab 11223344 faktura 123456: Fejl.       |
| Advarsel    | 2989         | Dato/tid - Eksporterer selskab 11223344 faktura 123456: Fejl. |

Årsagen hertil må naturligvis undersøges. Den typiske fejlkilde er manglende adgang eller rettigheder til enten filserveren eller e-conomic.


#### 2. Serverne

På den enkelte server bliver login og logout registreret i Windows' *Logbog* under *Windows-logger/Security*.

Mislykkede loginforsøg registreres også, hvilket kan være nyttigt ved fejlfinding.


#### 3. Fakturalisten

Åbn databasen med databasemanageren og vælg tabellen *Invoice* i kombinationsfeltet *Table*.

Den har disse felter:

| Felt       | Indhold         | Udfyldes | Indtastes |
| :--------- | :-------------- | :------- | :-------- |
| Id         | Løbenummer      | Nej      | Ingenting |
| ServerId   | Serverens Id    | Nej      | Ingenting |
| FileName   | Fakturafilnavn  | Nej      | Ingenting |
| FileDate   | Fakturafildato  | Nej      | Ingenting |
| ExportedAt | Eksportdatoive  | Valgfri  | Dato      |

Fakturafilnavnet har dette format:

`Novax_12345678_11223344_Faktura.xml` 

eller for en kreditnota:

`Novax_12345678_11223344_Kreditnote.xml`

hvor det første tal er fakturanummeret, og det andet er CVR-nummeret på selskabet, klinikken er knyttet til.

Feltet *FileDate* er fildatoen, som kan være en anden end fakturadatoen. Det gælder især for en faktura, der genudskrives i Novax-systemet, hvor fildatoen da vil være dags dato.

Sidste felt er eksportdatoen, *ExportedAt*. Den kan have én af disse tre mulige værdier:

| Værdi i ExportedAt      | Status                                                                |
| :---------------------- | :-------------------------------------------------------------------- |
| Senere dato end fildato | Fakturaen er eksporteret til e-conomic på denne dato/tid              |
| Samme dato som fildato  | Fakturaen er tidligere eksporteret eller manuelt oprettet i e-conomic |
| Tom (null)              | Fakturaen er ikke eksporteret. Afventer eksport                       | 

Den første er den normale. De to næste bør man undersøge årsagen til.

> Hvis en faktura ikke er eksporteret og af en eller anden grund heller ikke skal, skrives en dato (fx fakturadatoen) ind i feltet *ExportedAt*. **NovaXport** tror så, at den er faktureret og vil ignorere den fremover.


#### 4. E-conomic

Eksporterede fakturaer vil være bogført i e-conomic. De kan derfor ses under *Salg/Arkiv* og følges under *Indstillinger/Log* på sædvanlig måde.


#### 5. Fakturamangelliste

Listen med tilsyneladende manglende fakturafiler udsendes dagligt. Modtagerne står opført i tabellen *Recipient* i databasen. Listen kan ændres efter behov:

- Åbn databasen med databasemanageren og vælg tabellen *Recipient* i kombinationsfeltet *Table* og tilret listen.

En modtager kan sættes på pause ved at ændre værdien i felt *Inactive* til **1**. Pausen afsluttes ved at ændre værdien tilbage til **0**. Er pausen permanent, kan man i stedet slette hele posten. 


### Tilknytning af ekstra server eller selskab

**NovaXport** kan håndtere flere servere og selskaber. Følg vejledningen i afsnit **Konfiguration**:

- [Tilføj selskab][Configuration company]
- [Tilføj server][Configuration server]


### Fjern tilknytning af server eller selskab

> NB: Data bør ikke slettes fra databasen og slet ikke uden først at have oprettet en sikkerhedskopi.

Skal en server eller et selskab af den ene eller anden grund ikke længere håndteres af **NovaXport**, gøres det nemmest ved _deaktivere_ den/det.

#### Deaktivér server

- Åbn **NovaXport**s database med genvejen _NovaXport Database_
- I kombinationsboksen _Table_ vælg _Server_
- Vælg den server, der skal deaktiveres
- Indtast i feltet _Inactive_ til højre: **1**
- Klik øverst på _Write Changes_
- Luk vinduet

Herefter vil **NovaXport** ignorere serveren ved kommende kørsler.

Serveren kan genaktiveres på tilsvarende måde ved at indtaste **0** i stedet for **1**.

#### Deaktivér selskab

- Åbn **NovaXport**s database med genvejen _NovaXport Database_
- I kombinationsboksen _Table_ vælg _Company_
- Vælg det selskab, der skal deaktiveres
- Indtast i feltet _Inactive_ til højre: **1**
- Klik øverst på _Write Changes_
- Luk vinduet

Herefter vil **NovaXport** ignorere selskabet ved kommende kørsler.

Selskabet kan genaktiveres på tilsvarende måde ved at indtaste **0** i stedet for **1**.


### Sikkerhedskopiering

Der er to datafiler at sikkerhedskopiere. De ligger begge i mappen `%ProgramData%\Novax Export`:

- `Credentials.xml`
- `NovaxData.db`

Den første er låst, mens **NovaXport** kører, men ændres ikke, så efter installation og konfiguration kan der blot gemmes en kopi af den et passende sted.

Dene anden vil normalt kun være åbnet i få sekunder ad gangen, så enhver gængs metode til sikkerhedskopiering kan benyttes.

**NovaXport** programfilerne behøver ikke at sikkerhedskopieres, da de nemt kan installeres igen, i fald Windows skal retableres, eller maskinen udskiftes.


### Opdatering af NovaXport

Hvis en ny version skal tages i brug, gøres følgende:

1. Stop tjenesten
2. Kopiér den nye fil, `NovaXport.exe`, til mappen `%ProgramFiles%\Novax Export`
3. Start tjenesten

Brug evt. kommandofilerne:

  - `NovaXport_Stop.cmd`
  - `NovaXport_Start.cmd`

til hhv. pkt 1. og 3.


### Sletning af NovaXport

Hvis **NovaXport** skal fjernes fra maskinen, overvej så først, om de to datafiler skal arkiveres. Det er filerne i mappen `%ProgramData%\Novax Export`.

Herefter gøres følgende med administratorrettigheder:

1. Stop tjenesten
2. Fjern tjenesten
3. Slet mappen med datafilerne: `%ProgramData%\Novax Export`
4. Slet mappen med programfilerne: `%ProgramFiles%\Novax Export`

Brug til pkt. 1. og 2. evt. kommandofilerne:

  - `NovaXport_Stop.cmd`
  - `NovaXport_Remove.cmd`

Pkt. 3. og 4. må udføres manuelt.

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
[Installation]: https://cactusdata.github.io/NovaXport/Installation
[Maintenance]: https://cactusdata.github.io/NovaXport/Maintenance
[Main page]: https://cactusdata.github.io/NovaXport/
[Configuration]: https://cactusdata.github.io/NovaXport/Configuration
[Configuration company]: https://cactusdata.github.io/NovaXport/Configuration#tilf%C3%B8j-selskab-klinik
[Configuration server]: https://cactusdata.github.io/NovaXport/Configuration#tilf%C3%B8j-server
[Main page]: https://cactusdata.github.io/NovaXport/#supplerende-information