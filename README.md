# Gy11 och Gy25 – kurs- och ämnesdatabas

En fristående, återanvändbar grunddatabas för bland annat kursinformation,
utbildningsappar och framtida examensplanering. Underlaget är hämtat från
Skolverkets öppna Syllabus API. Databasen innehåller inga elevuppgifter.

## Återskapa från GitHub med ett kommando

Om den färdiga databasfilen saknas kan den återskapas från underlaget i detta repo.
Efter att ha hämtat eller klonat projektet, kör från projektmappen:

```sh
python src/rebuild.py
```

Python 3.10 eller senare med SQLite/FTS5 räcker. Inga extra Python-paket eller
anrop till Skolverket behövs. Byggsteget lägger den färdiga databasen,
CSV-exporten och den sökbara HTML-filen i `output/` och skriver ut sökvägarna.
Även om API:et hos Skolverket är otillgängligt kan det sparade uttaget byggas om.
Det återskapade innehållet motsvarar det sparade uttaget, inte framtida ändringar
som Skolverket har gjort efter det. Uttaget har samma dokumenterade begränsning
för äldre historik i Processautomation.

## Innehåll i uttaget den 17 november 2026

| Innehåll | Antal |
|---|---:|
| Ämneskoder Gy11 | 344 |
| Ämneskoder Gy25 | 573 |
| Ämneskoder totalt | 917 |
| Importerade ämnesversioner | 1 488 |
| Gy11-kurser i katalogvyn | 1 244 |
| Gy25-nivåer i katalogvyn | 1 120 |
| Kurser/nivåer i katalogvyn totalt | 2 364 |
| Unika kurs-/nivåkoder inklusive äldre versioner | 2 474 |
| Kurs-/nivåposter inklusive alla importerade ämnesversioner | 4 722 |

Katalogvyn innehåller en vald version per ämne vid uttagsdatum. Även utgångna
och upphävda ämnen finns kvar och har särskild status. Därför är detta inte
enbart en lista över utbildning som kan påbörjas idag.

Skolformer: gymnasieskolan (`GY`) och komvux på gymnasial nivå (`VUXGY`).
Anpassad utbildning, grundläggande nivå, SFI och Gy2000 ingår inte i ämnestabellerna.
Den generella referenskodlistan från Syllabus sparas ofiltrerad separat och
kan därför innehålla koder från andra skolformer.

För varje ämne sparas namn, kod, reform, beskrivning, syfte, skolformer,
kategorier, versioner, datum och föreskriftshänvisningar. Kurser och nivåer har
egna koder, namn, poäng, beskrivningar och centralt innehåll. Betygskriterier
och kunskapskrav sparas med uppgift om de hör till ämnet eller till kursen.
Gy25-kriterier ligger normalt på ämnet, och kopplas därifrån till nivåerna.

Originaltexter sparas som HTML och som text för sökning. Ett tomt
beskrivningsfält i källan fylls inte med påhittad text: ämnesbeskrivningen,
syftet och det centrala innehållet finns kvar i separata fält. Alla fält i
det ursprungliga JSON-underlaget bevaras, även sådana som inte fått en egen
kolumn. Kurs-/nivånivåns `raw_json` och `source_responses.response_json` gör
att det går att kontrollera innehållet mot källan.

## Öppna och använda filerna

| Fil | Användning |
|---|---|
| `data/gy11_gy25.sqlite` | Färdig SQLite-databas med fulltextindex och historik. |
| `data/kurskatalog.html` | Sökbar förhandsvisning som kan öppnas lokalt i webbläsaren. |
| `data/kurskatalog.csv` | Katalogvyn med beskrivningar och centralt innehåll; UTF-8 med BOM, semikolon som avgränsare. |
| `data/kurskatalog.jsonl.gz` | Komprimerad export av katalogvyn inklusive texter och kopplade betygskriterier. |
| `data/source-responses.jsonl.gz` | Alla sparade API-svar med källa och hämtningstid, för återuppbyggnad utan internet. |
| `data/snapshot-index.json.gz` | Ämnes- och versionsindex samt dokumenterade källfel. |
| `data/kvalitetsrapport.json` | Antal, täckningskontroll och kända begränsningar. |
| `data/kallmanifest.json` | Anropade källadresser och hämtningstider. |
| `schema.sql` | Databasens struktur och katalogvyer. |
| `examples.sql` | Exempel för uppslag, skolformsfilter, versionshistorik och sökning. |
| `src/` | Hämtnings-, bygg-, sök- och exportkod. |

SQLite kan användas direkt från bland annat Python, Node eller en separat
databastjänst. Ingen specifik appplattform behövs. Det här projektet startar
ingen publik tjänst. Källkoden och det komprimerade originalunderlaget
versionshanteras i https://github.com/Huggan1965/Databas-GY11-och-GY25 .

### Sök från terminalen

Sökning och läsning behöver bara Python 3.10 eller senare med SQLite/FTS5.
Färdiga databasfilen behöver inte importeras igen. Använd `--database output/gy11_gy25.sqlite` om den byggts med `src/rebuild.py`.

```sh
python src/query.py --code SVEN1000X
python src/query.py "laboratorium" --reform GY25 --school VUXGY
python src/query.py --code SVESVE01 --history
```

### Databasens viktigaste tabeller

| Tabell/vy | Innehåll |
|---|---|
| `subjects` | Stabil ämneskod och reform. |
| `subject_versions` | Namn, beskrivning, syfte och metadata för varje importerad version. |
| `units` | Kurser och ämnesnivåer med koppling till ämnesversion. |
| `subject_school_types` | Skolformstillhörighet per ämnesversion. |
| `subject_categories` | Källans ämneskategorier. |
| `central_contents` | Centralt innehåll med ursprunglig struktur. |
| `knowledge_requirements` | Kriterier/krav med betygssteg och ämnes- eller kurskoppling. |
| `appendix2_entries` | Källans namn och kodtillägg för särskilda ämnesplaner som omfattar flera ämnen. |
| `reference_codes` | Syllabus generella kodlista, separat från katalogens avgränsning. |
| `source_responses` | Originaldata i JSON med URL, hämtningstid och SHA-256 av normaliserad JSON. |
| `selected_subject_versions` | En vald ämnesversion per ämne vid uttagsdatum, inklusive utgångna ämnen. |
| `catalogue` | Kurser/nivåer i valda ämnesversioner med ämnesbeskrivning och källhänvisning. |
| `units_fts` | Sökindex för alla importerade kurs-/nivåversioner. |

`units.id` är en intern nyckel inom ett bygge och kan ändras efter en ny import.
Använd `(subject_code, subject_version, code)` som stabilt versionsbundet ID.
Enbart namn eller kurskod identifierar inte en historisk version.

`full_name` är ett visningsnamn. För Gy25 sätts det samman av ämnesnamn och
nivånamn, exempelvis **Svenska, nivå 1**. Källans separata originalnamn och
JSON bevaras. Poäng anges per kurs eller nivå; de är inte automatiskt ett
ämnes samlade poäng. Fältet `points_raw` bevarar källvärdet. `points` är ett
heltal när källvärdet kan tolkas entydigt som ett heltal.

`appendix2_entries` bevaras som angivet i API:et. Tillägg ska inte förväxlas
med självständiga nivåer eller genereras om till nya koder utan verifierade
regler för det aktuella ämnet.

### Datum och status

Källans datum sparas oförändrade. Katalogvyn väljer senast påbörjade
ämnesversion vid uttagsdatum, med högsta versionsnummer när startdatum är lika.
Om ett ämne enbart har framtida versioner väljs den först kommande.
Detta val är en teknisk katalogfunktion, inte en bedömning av elevens examen.

| Status i katalogen | Betydelse |
|---|---|
| `current` | Inget passerat slutdatum eller upphävandedatum finns i den valda versionen. |
| `canceled_with_transition` | Ämnesplanen är upphävd med övergångsbestämmelser enligt källans datum. |
| `expired` | Källans slutdatum har passerat vid uttaget. |
| `future` | Ämnesversionen börjar efter uttagsdatum. |

Historiska versioner kan sakna eget slutdatum i källan. Databasen hittar
inte på ett slutdatum för dem. Vill man avgöra vilka regler som gällde för
en viss elev behöver elevens studier kopplas till rätt bestämmelser separat.

## Kopplingen till examensplanering

Detta är grunddata för en examensplanerare. Följande behöver tillkomma som
egna, källbelagda och versionshanterade tabeller och regler innan systemet
kan avgöra om en examen uppfyller alla villkor:

- Examenskrav för komvux och respektive examenstyp.
- Relationslistan mellan Gy11-kurser och Gy25-nivåer, inklusive kombinationer.
- Programtillhörighet, programstrukturer och villkor för programinriktat innehåll.
- Regler för överlappning, poängräkning och blandade betyg.
- Gymnasiearbete/komvuxarbete och relevanta specialkoder.
- Gy2000/motsvarandelistan om även sådana betyg ska kunna hanteras.

Databasen gör därför ingen automatisk översättning mellan Gy11 och Gy25 och
innehåller inga antagna motsvarigheter baserade på liknande namn. Ingen
examensvalidering har implementerats i detta steg.

## Kontroll och känd begränsning

Databasen har kontrollerats mot Skolverkets separata kurslistor för båda
skolformerna och kategorierna LATEST/EXPIRED/FUTURE. Kontrollen omfattade
4 552 listposter, inklusive dubbletter mellan skolformerna, utan saknade poster.
Samtliga 2 364 poster i katalogvyn har centralt innehåll.

**Processautomation (`PROC`, Gy25):** API:ets versionslista och
versionsspecifika detaljanrop gav HTTP 503. Den gällande ämnesversionen,
version 2, gick att hämta via det vanliga ämnesanropet och har importerats med
den faktiska källadressen. Äldre historik för just detta ämne kan inte
garanteras. Felet finns kvar i kvalitetsrapporten; det har inte dolts som ett
lyckat versionslisteanrop.

För övriga 916 ämneskoder har alla versioner som API:ets versionslistor
returnerade importerats. Det är ingen garanti för att Syllabus innehåller
varje dokument som någonsin har funnits. Uttaget är en ögonblicksbild och
uppdateras först när hämtningskoden körs igen.

Testerna kontrollerar databasens integritet, främmande nycklar, att importerat
innehåll och poäng stämmer med sparade original, täckning mot de separata
kurslistorna samt att Gy11/Gy25 och kriteriernas kopplingar hålls isär.

```sh
python -m unittest discover -s tests -v
```

## Uppdatera från Skolverket

Installera Python 3.10 eller senare. SQLite ska ha FTS5. Endast
nedladdningssteget använder ett externt Python-paket:

```sh
python -m pip install -r requirements.txt
python src/download.py --date 2026-11-17 --workdir work
python src/build.py --date 2026-11-17 --workdir work
python src/preview.py
python -m unittest discover -s tests -v
```

Byt datum till önskat uttagsdatum. Utelämnas `--date` i nedladdningen används
dagens UTC-datum; ange samma datum till byggsteget. Datumet är katalogens
referensdatum, inte ett löfte om att API:et kan återskapa exakt hur innehållet
såg ut ett tidigare datum. Aktuella versionsmetadata kan ha ändrats i efterhand.

En vanlig körning hämtar om underlaget. Efter ett avbrott kan
`src/download.py` köras med samma arbetskatalog och `--resume`, vilket
återanvänder redan hämtade svar. Använd inte `--resume` när avsikten är en helt
färsk uppdatering: metadata kan ändras även inom samma ämnesversion.

Nätverksfel vid detaljhämtning gör att nedladdningen avslutas med fel. Om
versionslistan för ett ämne misslyckas försöker koden bevara den version som
ämneslistan anger, och rapporterar att historiken är ofullständig. Byggsteget
stoppar om förväntade ämnen eller versioner saknas. Rapporten ska alltid
granskas efter en ny hämtning. Automatisk schemaläggning är inte aktiverad.

### Återskapa databasen helt utan internet

De komprimerade källfilerna i paketet räcker för att bygga om samma underlag.
Välj en tom arbetskatalog:

```sh
python src/restore_snapshot.py --workdir restored-work
python src/build.py --date 2026-11-17 --workdir restored-work
python src/preview.py
```

Detta kräver inga extra Python-paket och inga anrop till Skolverket.

## GitHub

Källkoden, schemat och de komprimerade datafilerna kan läggas i ett eget repo.
Den stora SQLite-filen är undantagen i `.gitignore`; det är lämpligare att
återskapa den från källfilerna eller distribuera den som en separat releasefil.
CSV och den fristående HTML-filen är också genererade exportfiler och
undantas från Git. De komprimerade källfilerna versionshanteras tillsammans med koden.
JSONL-exporten skapas vid byggsteget och undantas från Git.

## Källa och återanvändning

- API-information: https://www.skolverket.se/om-skolverket/oppna-data/api-for-laroplaner-kurs--och-amnesplaner
- Swagger: https://api.skolverket.se/syllabus/swagger-ui/index.html
- Ämneslista: `/syllabus/v1/subjects`
- Versioner: `/syllabus/v1/subjects/{code}/versions`
- Innehåll: `/syllabus/v1/subjects/{code}/versions/{version}`
- Kurslistor för täckningskontroll: `/syllabus/v1/courses`
- Kodlistor: `/syllabus/v1/valuestore/subjectandcoursecodes`

Skolverkets API-sida hänvisar vid uttaget till **CC0 1.0** för datan:
https://creativecommons.org/publicdomain/zero/1.0/deed.sv

Källan och hämtningstiden behålls för spårbarhet. Detta är ett fristående
uttag av öppna data, inte Skolverkets egen examensplanerare.
