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

Den første indeholder brugernavn og adgangskode til NOVAX-serverne.

Den anden, databasen, indeholder *fire lister*, der styrer den daglige funktion af **NovaXport**:

1. Serverne, der skal læses fakturaer fra - minimum én
2. Selskaberne hos e-conomic, der skal eksporteres fakturaer til - minimum ét
3. Fakturaerne (numre og dato) og deres status
4. Ugetidsplan for hvornår og hvor tit, **NovaXport** skal genlæse fakturalisten på serveren


#### Adgang til servere og e-conomic

Filen `Credentials.xml` indeholder brugernavn og adgangskode for brugerkontoen, der har adgang til den delte mappe med fakturafilerne på den eller de filservere, der afvikler NOVAX.

> Det anbefales kraftigt ikke at bruge en normal brugerkonto, men at oprette en speciel brugerkonto, der kun bruges af **NovaXport**, kun med læseadgang til den delte mappe.
>
> Dette skal gøres af NOVAX, da de administrerer serveren. Det aktuelle brugernavn og tilhørende adgangskode skal derfor aftales med NOVAX og skal være kendt af NOVAX' support.

Det er en standard XML-fil, der kan redigeres med fx _Notesblok_, og det gøres nemmest ved at bruge _genvejen_ `NovaXport Credentials` omtalt under [Installation][Installation]. 

Filen ser således ud:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ReaderAccount>
    <Domain>HOSTING</Domain>
    <Username>NovaxEksport</Username>
    <Password>VoresMegetLangeAdgangskode</Password>
    <AppSecretToken>2RIukREatPIvuN92ry89My1cazJtx3LUX9bFsCMVBA81</AppSecretToken>
</ReaderAccount>
```

Adgangen til den delte mappe på serveren styres af de tre felter:

- `Domain`
- `Username`
- `Password`

NOVAX' domæne hedder typisk `HOSTING`. Brugerkontoen (her `NovaxEksport`) og adgangskoden (her `VoresMegetLangeAdgangskode`) skal rettes til som aftalt med NOVAX.

Det fjerde felt:

- `AppSecretToken`

indeholder det token, **NovaXport** i sin egenskab af en "e-conomic app" er tildelt af e-conomic. Det er normalt allerede skrevet ind i filen og må ikke rettes eller slettes.

### Tilføj selskab (klinik)

For at NovaXport kan sende fakturaer til e-conomic, skal to ting være på plads.

For det første skal "e-conomic appen NovaXport" *tilknyttes selskabet* ved at blive tilføjet listen over apps, som selskabet kan kommunikere med. Det skal gøre inde i e-conomic.

Dernæst skal selskabet i **NovaXPort**s database *tilføjes listen over selskaber*, der kan eksporteres fakturaer til.

Når appen (NovaXport) tilføjes i e-conomic, generes en *unik nøgle* (Adgangs-ID/token), som identificerer de to over for hinanden. Med dette Adgangs-ID:

- accepterer e-conomic, at dette selskab kan kommunikere med **NovaXport**
- fortæller **NovaXport**, når den kontakter e-conomic, hvilket selskab der skal kommunikeres med 


#### Tilknyt app i e-conomic

Du skal have en superbruger/administrator-brugerkonto til e-conomic til rådighed.

1. Log ind med denne brugerkonto på [Visma Home][Visma home]
3. Hvis I har flere selskaber, så vælg det, der skal eksporteres til fra NOVAX
4. Vælg fra topmenuen *Indstillinger* punktet *Alle indstillinger*
5. Vælg i menuen til venstre punktet *Apps*
6. Nu vises enten en liste med de apps, der tidligere er aktiveret, eller denne side: *Sådan tilføjer du apps*

7.  1. *Kopiér* nu [dette link][App link]
    2. Åbn en ny side i browseren
    3. *Indsæt* derefter linket (kopieret i pkt. 1) i den nye side og tryk *Enter*
    4. Denne side vises nu:

    <br>![Tilføj app][Attach app]<br>
    
    5. **Kontrollér, at aftalenummeret er korrekt**
    6. Klik på knappen *Tilføj app*
    7. Denne side vises nu:

    <br>![App tilføjet][Attached app]<br>

    8. Det viste ID er det **Adgangs-ID**, som skal bruges ved konfigurationen af NovaXport (se næste afsnit)
    9. Luk siden

8. Opdatér [siden med dine apps i e-conomic][EC extensions] for at få vist og bekræftet tilknytningen af appen:

![App-liste][App list]


#### Kontrollér selskabets opsætning i e-conomic

For at en faktura/kreditnota kan oprettes og bogføres i e-conomic skal den have disse oplysninger:

1. En kunde (klient/patient)
2. En kundegruppe, hvis kunden er ny
3. En momszone, hvis kunden er ny
4. Et eller flere varenumre (produkter/ydelser)
5. En eller flere varegrupper, hvis et eller flere varenumre er nye
6. En betalingsbetingelse af typen med forfaldsdato
7. Et layout

Det sker ved at:

- For hver faktura, der skal eksporteres, vil **NovaXport** forsøge at finde kunden, først via CVR-nummeret for erhvervskunder, dernæst via navnet. Findes ikke et eksakt match, oprettes kunden og knyttes til den første indenlandske kundegruppe og den første indenlandske momszone, der er oprettet i e-conomic
- Nye varenumre knyttes til varegruppe 1 i e-conomic
- Hvis en betalingsbetingelse med forfaldsdato ikke findes i e-conomic, oprettes den med navnet "Standard"
- Som layout bruges det første, der kan findes i e-conomic

> Hvis ikke dette flow kan følges, kan fakturaen ikke bogføres, og den kan måske heller ikke oprettes som en kladde. I begge tilfælde vil **NovaXport** rapportere en fejl i loggen.

Er de ovennævnte punkter på plads, vil e-conomic nu tillade dit selskab og NovaXport at kommunikere indbyrdes.

Næste trin er at muliggøre dette for **NovaXport**.


#### Tilknyt selskab i NovaXport

Åbn databasen med databasemanageren. Bruges genvejen (anbefalet under [Installation][Installation]), vises straks tabellen *Company*, som indeholder listen med selskaber. Hvis en anden tabel vises, så vælg *Company* i kombinationsfeltet *Table*.

Gå til en ny post og indsæt i felterne *Cvr* og *AgreementGrantToken* hhv. selskabets *CVR-nummer* og det *Adgangs-ID*, der blev oprettet i e-conomic - se afsnittet ovenfor.

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
| FirstDate           | Datous       | Valgfrit | En dato eller ingenting |

Resultatet skal (hvis uden selskabsnavn) ligne dette (feltet _FirstDate_ er dog ikke vist):

![Tilknyt selskab][New company]


### Tilføj server

For hver server, **NovaXport** skal skanne og læse NOVAX-fakturaer fra, skal bruges:

- Hostname, som er serverens navn på NOVAX' netværk
- ShareName, som er navnet på den delte mappe med fakturafilerne

> Begge navne oplyses af NOVAX' support og skal kendes for at kunne fortsætte.

Åbn databasen med databasemanageren og vælg tabellen *Server* i kombinationsfeltet *Table*.

Gå til en ny post og indsæt i felterne *Hostname* og *ShareName* hhv. serverens *Hostname* og navnet på dens den *delte mappe* med fakturafilerne oprettet af NOVAX.

Felterne *Id* og *Inactive* vil allerede være udfyldte og må ikke ændres. 

> **VIGTIGT**: Ændringer/tilføjelser gemmes først, når man klikker på *Write Changes* eller taster *Ctrl+S*.

Feltoversigt:

| Felt      | Indhold         | Udfyldes | Indtastes |
| :-------- | :-------------- | :------- | :-------- |
| Id        | Løbenummer      | Nej      | Ingenting |
| Hostname  | Servernavn      | Ja       | Tekst     |
| ShareName | Delt mappe-navn | Ja       | Tekst     |
| Inactive  | Status          | Nej      | Ingenting |

Resultatet skal ligne dette:

![Tilknyt server][New server]


### Justér tidsplan

Når **NovaXport** tjenesten kører, sker det efter en tidsplan. Efter hver kørsel ser den i tidsplanen, hvor mange minutters pause, den skal holde, før næste kørsel.

Som udgangspunkt er oprettet 24 timeintervaller, hvor pausen er sat til 15 minutter fra kl. 06.00.

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
| Interval  | Minutter    | Ja       | 0 eller et positivt tal               |
| Weekend   | 0 eller 1   | Ja       | 1 for ingen kørsel i weekend |

> Fra feltet *StartTime* læses kun tidsdelen, men undlad alligevel at indtaste andre datoer end 0001-01-01.

Standardtidsplanen ser således ud:

![Tidsplan][New schedule]


### Start NovaXport

Kører **NovaXport** ikke, kan den nu startes - enten manuelt under *Tjenester* eller med kommandofilen `NovaXport_Start.cmd` som beskrevet i afsnit [Installation][Installation]. Administratorrettigheder kræves for begge muligheder.

<hr>

[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[New company]: images/NewCompany.png
[New server]: images/NewServer.png
[New schedule]: images/NewSchedule.png
[Installation]: https://github.com/CactusData/NovaXport/blob/main/Installation.md
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps
[App link]: https://secure.e-conomic.com/secure/api1/requestaccess.aspx?appPublicToken=ToVYPF4QxTW73TcmtKPZtQCTwjKJlAwu0cPn3LEOE201
[Visma home]: https://connect.visma.com/