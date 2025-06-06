# NovaXport

![NOVAX(R) e-conomic(R)][Title logos] 


## Konfiguration

### Datafiler

Datafilerne til **NovaXport** skal ligge i mappen `Novax Export` i systemmappen `%ProgramData%`. 

Typisk vil det være mappen:

- `C:\ProgramData\Novax Export`

Her findes to datafiler:

- `Credentials.xml`
- `NovaxData.db`

Den første indeholder brugernavn og adgangskode til NOVAX-serverne og nøgler til mailserveren.

Den anden, databasen, indeholder *fem lister*, der styrer den daglige funktion af **NovaXport**:

1. Serverne, der skal læses fakturaer fra - minimum én
2. Selskaberne hos e-conomic, der skal eksporteres fakturaer til - minimum ét
3. Fakturaerne (numre og udskrivningsdato) og deres status
4. Ugetidsplan for hvornår og hvor tit, **NovaXport** skal genlæse fakturalisten på serverne
5. Modtagerne af den daglige e-mail med listen over eventuelle manglende fakturafiler


#### Adgang til servere og e-conomic

Filen `Credentials.xml` indeholder brugernavn og adgangskode for brugerkontoen, der har adgang til den delte mappe med fakturafilerne på den eller de filservere, der afvikler NOVAX.

> Det anbefales kraftigt ikke at bruge en normal brugerkonto, men at oprette en speciel brugerkonto, der kun bruges af **NovaXport**, kun med læseadgang til den delte mappe.
>
> Dette skal gøres af NOVAX, da de administrerer serverne. Det aktuelle brugernavn og tilhørende adgangskode skal derfor aftales med NOVAX og skal være kendt af NOVAX' support.

Det er en standard XML-fil, der kan redigeres med fx _Notesblok_, og det gøres nemmest ved at bruge _genvejen_ `NovaXport Credentials` omtalt under [Installation][Installation]. 

> NB: Filen er låst og kan ikke ændres, mens **NovaXport** kører. Så skal den ændres, skal **NovaXport** stoppes, ændringerne udføres og gemmes, og **NovaXport** startes igen.

Filen ser således ud:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ReaderAccount>
    <Domain>HOSTING</Domain>
    <Username>NovaxEksport</Username>
    <Password>VoresMegetLangeAdgangskode</Password>
    <AppSecretToken>2RIukREatPIvuN92ry89My1cazJtx3LUX9bFsCMVBA81</AppSecretToken>
    <MailjetPublicToken>8d50a951512e1d9f051efbd1f336b55b</MailjetPublicToken>
    <MailjetPrivateToken>28849ccda2b3c50790f580aedf7d50fe</MailjetPrivateToken>
</ReaderAccount>
```

Adgangen til de delte mapper på serverne styres af de tre felter:

- `Domain`
- `Username`
- `Password`

NOVAX' domæne hedder typisk `HOSTING`. Brugerkontoen (her `NovaxEksport`) og adgangskoden (her `VoresMegetLangeAdgangskode`) skal rettes til som aftalt med NOVAX.

Det fjerde felt:

- `AppSecretToken`

indeholder det token, **NovaXport** i sin egenskab af en "e-conomic app" er tildelt af e-conomic. Det er normalt allerede skrevet ind i filen og må ikke rettes eller slettes.

De to sidste felter:

- `MailjetPublicToken`
- `MailjetPrivateToken`

er nøglerne til e-mailtjenesten hos Mailjet, der bruges til at udsende listen med manglende fakturaer. De må ikke ændres eller slettes.


### Tilføj selskab (klinik)

For at NovaXport kan sende fakturaer til et selskab i e-conomic, skal tre ting være på plads:

1. For det første skal i e-conomic appen "NovaXport" *tilknyttes selskabet* ved at blive tilføjet listen over apps, som selskabet kan kommunikere med. Det skal gøre inde i e-conomic

2. Dernæst skal nogle grupper og eventuelt afdelinger *oprettes* i e-conomic

3. Endelig skal selskabet i **NovaXPort**s database *tilføjes listen over selskaber*, der kan eksporteres fakturaer til

Når appen (NovaXport) tilføjes i e-conomic, generes en *unik nøgle* (Adgangs-ID/token), som identificerer de to over for hinanden. Med dette Adgangs-ID:

- accepterer e-conomic, at dette selskab kan kommunikere med **NovaXport**
- fortæller **NovaXport**, når den kontakter e-conomic, hvilket selskab der skal kommunikeres med 


#### 1. Tilknyt app i e-conomic

Du skal have en *superbruger/administrator*-brugerkonto til e-conomic til rådighed.

1. Log ind med denne brugerkonto på [Visma Home][Visma home]
2. Hvis I har flere selskaber, så vælg det, der skal eksporteres til fra NOVAX
3. Vælg fra topmenuen *Indstillinger* punktet *Alle indstillinger*
4. Vælg i menuen til venstre punktet _Apps_
5. Nu vises enten en liste med de apps, der tidligere er aktiveret, eller siden med titlen: *Sådan tilføjer du apps*
6. **Notér dig aftaltenummeret** (oppe til højre), fx 1234567

7.  1. *Kopiér* nu [dette link][App link]
    2. Åbn en ny side i browseren
    3. *Indsæt* derefter linket (kopieret ovenfor) i browserens søgelinje (URL) og tryk *Enter*
    4. Denne side vises nu:

    <br>![Tilføj app][Attach app]<br>
    
    5. **Kontrollér, at aftalenummeret er korrekt**, her er vist 1234567
    6. Klik på knappen *Tilføj app*
    7. **NovaXport**-siden vises som bekræftelse på tilføjelsen
    8. Klik to gange _tilbage_ i browseren (venstrepil øverst til venstre)
    9. Denne side vises nu:

    <br>![App-liste][App list]</br>
    
    10. Det viste **Adgangs-ID** er det, som skal bruges ved konfigurationen af NovaXport (se næste afsnit)

Nu vil _e-conomic_ tillade dit selskab og **NovaXport** at kommunikere indbyrdes.

Derfor skal **NovaXport** også vide, at dette Adgangs-ID skal bruges ved eksport til dette selskab i e-conomic. Det gøres som beskrevet under [pkt. 3][Punkt 3] herunder.


#### 2. Justér og kontrollér selskabets opsætning i e-conomic

Før **NovaXport** kan eksportere fakturaer, skal følgende være oprettet:

**Kundegruppe 100**:

> _Vigtigt_: Vælg layout som vist.

![Kundegruppe 100][Kundegruppe 100]

**Varegruppe 100**:

> _Vigtigt_: Kontrollér kontonumrene.

![Varegruppe 100][Varegruppe 100]

**Afdeling 10000** (hvis afdelinger/dimensioner er i brug i e-conomic):

![Afdeling 10000][Afdeling 10000]

**Afdeling for ydernummer** (hvis afdelinger/dimensioner er i brug i e-conomic):

> VIGTIGT: For hvert _ydernummer_, der faktureres for, skal en afdeling oprettes med _ydernummeret som afdelingsnummer_.
>
> Er afdelingsnummeret ikke oprettet, vil **NovaXport** ikke eksportere fakturaer fra denne klinik til selskabet.

##### Bogføring af fakturaer i e-conomic 
For at en faktura/kreditnota kan oprettes og bogføres i e-conomic skal den have disse oplysninger:

1. En kunde (klient/patient)
2. En kundegruppe, hvis kunden er ny
3. En momszone, hvis kunden er ny
4. Et eller flere varenumre (produkter/ydelser)
5. En eller flere varegrupper, hvis et eller flere varenumre er nye
6. En betalingsbetingelse af typen med forfaldsdato
7. Et layout

Det sker på to forskellige måder afhængig af, om kunden er en privatperson eller er en myndighed eller en erhvervsvirksomhed

###### Erhverv og myndigheder

Disse har altid et CVR- eller EAN-nummer og eksporteres således:

- For hver faktura, der skal eksporteres, vil **NovaXport** forsøge at finde kunden, først via CVR-nummeret for erhvervskunder, dernæst via navnet. Findes ikke et eksakt match, oprettes kunden og knyttes til kundegruppe 100
- Momszone vælges ud fra kundens data som angivet i Novax
- Nye varenumre knyttes til varegruppe 100
- Hvis en betalingsbetingelse med forfaldsdato ikke findes i e-conomic, oprettes den med navnet "Standard"
- Layout hentes fra kundens kundegruppe
- Kunden vil fremstå som adressat på fakturaudskriften
- klientens/patientens CPR-nummer angives på fakturaen som en reference

###### Privatpersoner

Disse har altid et CPR-nummer, men oprettes ikke som separate kunder. I stedet knyttes de til en "samlekunde" knyttet til kundegruppe 100. Denne samlekunde oprettes (automatisk) med klinikkens ydernummer som kundenummer, fx:

`33221 Lægehuset Nyborg`

Hvis flere klinikker er knyttet til det samme selskab, vil der derfor findes en samlekunde for hver klinik.

Disse fakturaer eksporteres dermed således:

- For hver faktura, der skal eksporteres, vil **NovaXport** finde klinikkens samlekunde ud fra ydernummeret
- Momszone og layout hentes fra samlekundens kundegruppe
- Nye varenumre knyttes til varegruppe 100
- Hvis en betalingsbetingelse med forfaldsdato ikke findes i e-conomic, oprettes den med navnet "Standard"
- Klienten/patienten vil fremstå som adressat på fakturaudskriften
- klientens/patientens CPR-nummer angives på fakturaen som en reference

> Hvis ikke et af disse to flows kan følges, kan fakturaen ikke bogføres, og den kan måske heller ikke oprettes som en kladde. I begge tilfælde vil **NovaXport** rapportere en fejl i loggen.


#### 3. Tilknyt selskab i NovaXport

Åbn databasen med databasemanageren. Bruges genvejen (anbefalet under [Installation][Installation]), vises straks tabellen *Company*, som indeholder listen med selskaber. Hvis en anden tabel vises, så vælg *Company* i kombinationsfeltet *Table*.

Gå til en ny post og indsæt i felterne *Cvr* og *AgreementGrantToken* hhv. selskabets *CVR-nummer* og det *Adgangs-ID*, der blev oprettet i e-conomic - se [pkt. 1][Punkt 1] ovenfor.

> Tip: Brug *kopiér/indsæt* med mus eller tastatur til at indsætte *Adgang-ID*.

Feltet *Name* kan også udfyldes med selskabets navn, men det behøves ikke; **NovaXport** vil selv slå navnet op i CVR-registeret og indsætte det, hvis feltet er tomt.

Feltet *FirstDate* angiver *den tidligste dato* en faktura/kreditnota kan have for at blive eksporteret til e-conomic. Udfyldes det ikke, bruges første dato i det aktuelle regnskabsår.

> Fakturaer/kreditnotaer med ældre dato vil blive registreret i databasen og vist som eksporteret, da de må antages allerede at være blevet eksporteret til e-conomic enten manuelt eller på anden måde. 

Felterne *Id* og *Inactive* vil allerede være udfyldte og må ikke ændres. 

> **VIGTIGT**: Ændringer/tilføjelser gemmes først, når man klikker på *Write Changes* eller taster *Ctrl+S*.

Feltoversigt:

| Felt                | Indhold      | Udfyldes | Indtastes               |
| :------------------ | :----------- | :------- | :---------------------- |
| Id                  | Løbenummer   | Nej      | Ingenting               |
| Cvr                 | CVR-nummer   | Ja       | Otte cifre              |
| Name                | Selskabsnavn | Valgfrit | Tekst                   |
| AgreementGrantToken | Adgangs-ID   | Ja       | 44 cifre/bogstaver      |
| Inactive            | Status       | Nej      | Ingenting               |
| FirstDate           | Dato         | Valgfrit | En dato eller ingenting |

Resultatet skal (hvis uden selskabsnavn) ligne dette:

![Tilknyt selskab][New company]

I eksemplet her vil **NovaXport** kun eksportere fakturaer fra 2. kvartal 2023 og frem.

### Tilføj server

For hver server, **NovaXport** skal skanne og læse NOVAX-fakturaer fra, skal bruges:

- Hostname, som er serverens navn på NOVAX' netværk
- ShareName, som er navnet på den delte mappe med fakturafilerne

> Begge navne oplyses af NOVAX' support og skal kendes for at kunne fortsætte.

Åbn databasen med databasemanageren og vælg tabellen *Server* i kombinationsfeltet *Table*.

Gå til en ny post og indsæt i felterne *Hostname* og *ShareName* hhv. serverens *Hostname* og navnet på den *delte mappe* med fakturafilerne oprettet af NOVAX.

Felterne *Id* og *Inactive* vil allerede være udfyldte og må ikke ændres. 

Feltet *FirstDate* kan være tomt, og **NovaXport** vil da hente fakturafiler fra det aktuelle regnskabsårs start. Ønsker man kun at hente fakturafiler fra en senere dato - fx fordi fakturaer ældre end denne dato allerede på anden vis er bogført i e-conomic - skrives datoen ind her.

> **VIGTIGT**: Ændringer/tilføjelser gemmes først, når man klikker på *Write Changes* eller taster *Ctrl+S*.

Feltoversigt:

| Felt      | Indhold         | Udfyldes | Indtastes               |
| :-------- | :-------------- | :------- | :---------------------- |
| Id        | Løbenummer      | Nej      | Ingenting               |
| Hostname  | Servernavn      | Ja       | Tekst                   |
| ShareName | Delt mappe-navn | Ja       | Tekst                   |
| Inactive  | Status          | Nej      | Ingenting               |
| FirstDate | Dato            | Valgfrit | En dato eller ingenting |

Resultatet skal ligne dette:

![Tilknyt server][New server]

I eksemplet her vil **NovaXport** kun hente fakturafiler fra 4. kvartal 2023 og senere.

> Når **NovaXport** kører, vil feltet *FirstDate* løbende blive opdateret med den seneste dato på de hentede fakturafiler.


### Justér tidsplan

Når **NovaXport** tjenesten kører, sker det efter en tidsplan. Efter hver kørsel ser den i tidsplanen, hvor mange minutters pause, den skal holde, før næste kørsel.

Som udgangspunkt er oprettet 24 timeintervaller, hvor pausen er sat til 15 minutter fra kl. 07.00 frem til aften.

Både tidsintervaller og pauser kan justeres til det aktuelle behov, og tidsplanen kan helt eller delvist sættes til kun at være aktiv på hverdage.

Når en kørsel er afsluttet, slås op i tabellen, det nærmeste tidligere starttidspunkt findes, pausen angivet her læses, og tjenesten holder pause i dette antal minutter.

> Justering af tidsplanen er ikke kritisk, da **NovaXport** bruger meget få ressourcer, og en kørsel, hvor der ingen fakturaer er at eksportere, afvikles på få sekunder.

Åbn databasen med databasemanageren og vælg tabellen *Schedule* i kombinationsfeltet *Table*.

Her kan starttidspunktet for tidsintervallerne, pausen (feltet *Interval*) og weekendkørsel justeres. 

Er *Interval* angivet til 0 (nul) eller *Weekend* til 1 (ét), gør tjenesten intet andet end at vente til næste starttidspunkt.

Feltoversigt:

| Felt      | Indhold     | Udfyldes | Indtastes                    |
| :-------- | :---------- | :------- | :--------------------------- |
| StartTime | Dato og tid | Ja       | En dato og ønsket starttid   |
| Interval  | Minutter    | Ja       | 0 eller et positivt tal      |
| Weekend   | 0 eller 1   | Ja       | 1 for ingen kørsel i weekend |

> Fra feltet *StartTime* læses kun tidsdelen, men undlad alligevel at indtaste andre datoer end 0001-01-01.

Standardtidsplanen kan se således ud:

![Tidsplan][New schedule]


### Tilføj modtagere af fakturamangellisten

Efter dagens første kørsel kan **NovaXport** udsende en e-mail med en liste over tilsyneladende manglende filer. Modtagerne skrives ind i databasen:

Åbn databasen med databasemanageren og vælg tabellen *Recipient* i kombinationsfeltet *Table*.

Den har disse felter:

| Felt      | Indhold         | Udfyldes | Indtastes            |
| :-------- | :-------------- | :------- | :------------------- |
| Id        | Løbenummer      | Nej      | Ingenting            |
| Name      | Modtagernavn    | Valgfri  | Tekst                |
| Email     | Modtageradresse | Ja       | E-mailadresse        |
| Inactive  | 0 eller 1       | Ja       | 0 for aktiv modtager |

Indtast de ønskede modtageres e-mailadresser. Navn er valgfrit, men anbefalet om ikke andet så fordi, det er nemmere at identificere modtagerne, når modtagerlisten skal justeres.

En modtager kan sættes på pause ved at ændre værdien i felt *Inactive* til **1** som vist her: 

![Modtagere][New recipient]

Pausen afsluttes ved at ændre værdien tilbage til **0**.

Er pausen permanent, kan man i stedet slette hele posten.

Afsenderen vil altid være: `Novax Eksport <novaxport@cactusdata.dk>`

Den modtagne e-mail vil ligne denne:

![Liste med manglende fakturaer][Missing invoice list email]


### Start NovaXport

Kører **NovaXport** ikke, kan den nu startes - enten manuelt under *Tjenester* eller med kommandofilen `NovaXport_Start.cmd` som beskrevet i afsnit [Installation][Installation]. Administratorrettigheder kræves i begge tilfælde.

<hr>

[Tilbage til forsiden][Main page]

[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[New company]: images/NewCompany.png
[New server]: images/NewServer.png
[New schedule]: images/NewSchedule.png
[New recipient]: images/NewRecipient.png
[Missing invoice list email]: images/MissingInvoices.png
[Installation]: https://github.com/CactusData/NovaXport/blob/main/Installation.md
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps
[App link]: https://secure.e-conomic.com/secure/api1/requestaccess.aspx?appPublicToken=F1sRys0ygqsIdNgfd2bcaaJODr3MM0KSGsOBBggjiKM1&redirectUrl=https%3A%2F%2Fcactusdata.github.io%2FNovaXport%2F
[Visma home]: https://connect.visma.com/
[Kundegruppe 100]: images/ec-kundegruppe100.png
[Varegruppe 100]: images/ec-varegruppe100.png
[Afdeling 10000]: images/ec-afdeling10000.png
[Installation]: https://cactusdata.github.io/NovaXport/Installation
[Maintenance]: https://cactusdata.github.io/NovaXport/Maintenance
[Main page]: https://cactusdata.github.io/NovaXport/#supplerende-information
[Punkt 1]: https://cactusdata.github.io/NovaXport/Configuration#1-tilknyt-app-i-e-conomic
[Punkt 3]: https://cactusdata.github.io/NovaXport/Configuration#3-tilknyt-selskab-i-novaxport
