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

Det er en standard XML-fil, der kan redigeres med fx Notesblok, og ser således ud:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ReaderAccount>
    <Domain>HOSTING</Domain>
    <Username>NovaxEksport</Username>
    <Password>VoresMegetLangeAdgangskode</Password>
    <AppSecretToken>2RIukREatPIvuN92ry89My1cazJtx3LUX9bFsCMVBA81</AppSecretToken>
</ReaderAccount>
```

Adgangen til den delte mappe styres af de tre felter:

- `Domain`
- `Username`
- `Password`

NOVAX' domæne hedder typisk `HOSTING`. Brugerkontoen (her `NovaxEksport`) og adgangskoden (her `VoresMegetLangeAdgangskode`) skal rettes til som aftalt med NOVAX.

Det fjerde felt:

- `AppSecretToken`

indeholder det token, **NovaXport** i sin egenskab af en "e-conomic app" er tildelt af e-conomic. Det er normalt allerede skrevet ind i filen og må ikke rettes eller slettes.

### Tilføj selskab (klinik)

For at NovaXport kan sende fakturaer til e-conomic, skal to ting være på plads.

For det første skal i e-conomic appen NovaXport *tilknyttes selskabet* ved at blive tilføjet listen over apps, som selskabet kan kommunikere med.

Dernæst skal selskabet i NovaXPorts database *tilføjes listen over selskaber*, der kan eksporteres fakturaer til.

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

E-conomic vil nu tillade dit selskab og NovaXport at kommunikere indbyrdes.

Næste trin er at muliggøre dette for NovaXport.

#### Tilknyt selskab i NovaXport

Åbn databasen med databasemanageren. Bruges genvejen (anbefalet under Installation), vises straks tabellen *Company*, som indeholder listen med selskaber. Hvis en anden tabel vises, så vælg *Company* i kombinationsfeltet *Table*.

Gå til en ny post og indsæt i felterne *Cvr* og *AgreementGrantToken*:
- *Cvr*: Selskabets *CVR-nummer*
- *AgreementGrantToken*: Det *Adgangs-ID*, der blev oprettet i e-conomic - se afsnittet ovenfor.

Feltet *Name* kan også udfyldes med selskabets navn, men det behøves ikke; **NovaXport** vil selv slå navnet op i CVR-registeret og indsætte det, hvis feltet er tomt.

Felterne *Id* og *Inactive* vil allerede være udfyldte og må ikke ændres.

Resultatet skal (hvis uden selskabsnavn) ligne dette:

![Tilknyt selskab][New company]




<hr>

[Cactus Data logo]: images/cactuslogopale.png
[Title logos]: images/Novax-e-conomic%20200.png
[Attach app]: images/ec-apps-001.png
[Attached app]: images/ec-apps-002.png
[App list]: images/ec-apps-003.png
[Data flow]: images/NovaXport%20Diagram.drawio%2024.png
[New company]: images/NewCompany.png
[EC extensions]: https://secure.e-conomic.com/settings/extensions/apps
[App link]: https://secure.e-conomic.com/secure/api1/requestaccess.aspx?appPublicToken=ToVYPF4QxTW73TcmtKPZtQCTwjKJlAwu0cPn3LEOE201
[Visma home]: https://connect.visma.com/