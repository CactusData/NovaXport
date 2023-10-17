# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 

## Kontrol og vedligeholdelse

### Rutinekontrol

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

Der vil typisk være disse hændelser (*Dato/tid* og *Kilde* er udeladt), her vist for to fakturaer:

| Niveau      | Hændelses-id | Generelt                                                    |
| :---------- | -----------: | :---------------------------------------------------------- |
| Oplysninger | 2960         | Dato/tid - Læser start.                                     |
| Oplysninger | 2970         | Dato/tid - Læser selskab 11223344 faktura 123456: OK.       |
| Oplysninger | 2970         | Dato/tid - Læser selskab 11223344 faktura 123457: OK.       |
| Oplysninger | 2961         | Dato/tid - Læser slut.                                      |
| Oplysninger | 2980         | Dato/tid - Eksporterer start.                               |
| Oplysninger | 2990         | Dato/tid - Eksporterer selskab 11223344 faktura 123456: OK. |
| Oplysninger | 2990         | Dato/tid - Eksporterer selskab 11223344 faktura 123457: OK. |
| Oplysninger | 2981         | Dato/tid - Eksporterer slut.                                |
| Oplysninger | 2950         | Dato/tid - Pause i 15 minutter.                             |

Hvis *Læsning* og/eller *Eksport* skulle fejle, logges følgende for Hændelses-id 2970 hhv. 2990:

| Niveau      | Hændelses-id | Generelt                                                      |
| :---------- | -----------: | :------------------------------------------------------------ |
| Advarsel    | 2970         | Dato/tid - Læser selskab 11223344 faktura 123456: Fejl.       |
| Advarsel    | 2990         | Dato/tid - Eksporterer selskab 11223344 faktura 123456: Fejl. |

Årsagen hertil må naturligvis undersøges.


#### 1. Serverne

På den enkelte server bliver login og logout registreret i Windows' *Logbog* under *Windows-logger/Security*.

Mislykkede loginforsøg registreres også, hvilket kan være nyttigt ved fejlfinding.


#### 2. Fakturalisten

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

`Novax_12345678_11223344_Faktura.xml` eller for en kreditnota:
`Novax_12345678_11223344_Kreditnote.xml`

hvor det første tal er fakturanummeret, og det andet er CVR-nummeret.

Feltet *FileDate* er fildatoen, som kan være en anden end fakturadatoen.

Sidste felt er eksportdatoen, *ExportedAt*. Den kan have én af disse tre mulige værdier:

| Værdi i ExportedAt      | Status                                                                |
| :---------------------- | :-------------------------------------------------------------------- |
| Senere dato end fildato | Fakturaen er eksporteret til e-conomic på denne dato/tid              |
| Samme dato som fildato  | Fakturaen er tidligere eksporteret eller manuelt oprettet i e-conomic |
| Tom (null)              | Fakturaen er ikke eksporteret. Afventer eksport                       | 

Den første er den normale. De to næste bør man undersøge årsagen til.

> Hvis en faktura ikke er eksporteret og af en eller anden grund heller ikke skal, skrives en dato (fx fakturadatoen) ind i feltet *ExportedAt*. **NovaXport** tror så, at den er faktureret og vil ignorere den fremover.


#### 3. e-conomic

Eksporterede fakturaer vil være bogført i e-conomic. De kan derfor ses under *Salg/Arkiv* og følges under *Indstillinge/Log* på sædvanlig måde.


### Tilknytning af ekstra server eller selskab

**NovaXport** kan håndtere flere servere og selskaber. Følg vejledningen i afsnit **Konfiguration**:

- [Tilføj selskab][Configuration company]
- [Tilføj server][Configuration server]


### Sikkerhedskopiering

Der er to datafiler at sikkerhedskopiere. De ligger begge i mappen `%ProgramData%\Novax Export`:

- `Credentials.xml`
- `NovaxData.db`

De vil normalt kun være åbnet i få sekunder ad gangen, så enhver gængs metode til sikkerhedskopiering kan benyttes.

**NovaXPort** selv behøver ikke at sikkerhedskopieres, da den nemt installeres igen, i fald Windows skal retableres eller maskinen udskiftes.


### Opdatering af NovaXport

Hvis en ny version skal installeres, gøres følgende:

1. Stop tjenesten
2. Kopiér den nye fil, `NovaXport.exe`, til mappen `%ProgramFiles%\Novax Export`
3. Start tjenesten

Brug evt. kommandofilerne:

  - `NovaXport_Stop.cmd`
  - `NovaXport_Start.cmd`


### Sletning af NovaXport

Hvis **NovaXport** skal fjernes fra maskinen, overvej så først, om de to datafiler skal arkiveres. Det er filerne i mappen `%ProgramData%\Novax Export`.

Herefter gøres følgende:

1. Stop tjenesten
2. Fjern tjenesten
3. Slet mappen med datafilerne: `%ProgramData%\Novax Export`
4. Slet mappen med programfilerne: `%ProgramFiles%\Novax Export`

Brug evt. kommandofilerne:

  - `NovaXport_Stop.cmd`
  - `NovaXport_Remove.cmd`

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
[Configuration company]: https://github.com/CactusData/NovaXport/blob/main/Configuration.md#Tilføj-selskab-klinik
[Configuration server]: https://github.com/CactusData/NovaXport/blob/main/Configuration.md#Tilføj-selskab