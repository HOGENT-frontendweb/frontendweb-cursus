# Front-end Web Development Opdracht 2025 - 2026

## 1. De opdracht

Gedurende het semester maak je samen met een medestudent een webapplicatie bestaande uit een front-end gedeelte in React en, indien je het olod Web Services ook volgt, een bijhorende back-end in NodeJS. Indien je het olod Web Services niet volgt, zie [sectie 7](#7-indien-je-het-olod-web-services-niet-volgt).

Je bent volledig vrij om te kiezen welke applicatie je ontwikkelt, maar het is wel belangrijk dat het een dynamische webapplicatie betreft. Het equivalent van iets wat met statische HTML kan bekomen worden is onvoldoende. Indien je twijfelt over jouw idee, mag je tijdens de les altijd overleggen met je lector of achteraf via een GitHub issue op jouw repository.

?> Er wordt enkel feedback gegeven op ideeën tijdens semester 1. Indien je een idee hebt dat je graag wil bespreken, doe dit dan tijdig.

De examenopdrachtwordt uitgevoerd in teams van twee. Beide teamleden dragen bij aan alle onderdelen van de applicatie en zijn verantwoordelijk voor het volledige eindresultaat. Zie [sectie 8](#8-groepswerk) voor meer informatie.

Alle code moet in een GitHub classroom repository terecht komen (zie Chamilo voor een link naar de classroom). Enkel de `main` branch van deze repository zal geëvalueerd worden. Er wordt automatisch een template van de `README.md` aangemaakt als je de opdracht accepteert, vul deze correct in. Je gebruikt dezelfde repository voor zowel Web Services als Front-end Web Development.

Het is belangrijk dat de applicatie significant verschilt van de voorbeeldapplicatie die tijdens de les gemaakt wordt.

Daarnaast verwachten we dat je een dossier met uitleg over je app indient op Chamilo. Een template voor dit dossier (`dossier.md`) vind je ook in jouw repository en dien je te gebruiken. **Je dient het dossier in als pdf!**

## 2. Ontvankelijkheidscriteria

Alvorens we jouw project evalueren, controleren we of het voldoet aan een aantal ontvankelijkheidscriteria.

!> Als niet voldaan is aan de ontvankelijkheidscriteria, krijg je de score "Afwezig" (conform het DOER).

Deze criteria zijn:

- Het dossier is volledig en tijdig ingediend (zie [sectie 4](#4-dossier-vereisten) voor de vereisten)
- Er werden voldoende (kleine) commits gemaakt.
- Elke student heeft PR's aangemaakt en feedback gegeven op de PR's van de andere student.
- Elke student heeft minstens 2 volledige features branch gemaakt waarin een volledige feature (van begin tot einde) werd uitgewerkt. De PR's zijn goedgekeurd door de andere student.
- De demo duurt niet langer dan 15 minuten (incl. Front-end Web Development indien van toepassing)
- De applicatie is gemaakt in React
- De applicatie draait online
- De applicatie start zonder problemen op a.d.h.v. de instructies in de README en gebruikt hiervoor Docker.
- De applicatie wijkt voldoende af van de voorbeeldapplicatie
- node_modules, .env, productiecredentials... werden niet gepushed op GitHub
- Er is een extra technologie gebruikt (zie [sectie 6](#6-voorbeelden-van-extras) voor voorbeelden)
- Er werden een aantal niet-triviale en werkende e2e-testen gemaakt (naast de testen voor de user).
- De applicatie is voldoende complex

## 3. Evaluatiecriteria

### Componenten

- heeft meerdere componenten - dom & slim (naast login/register)
- applicatie is voldoende complex
- definieert constanten (variabelen, functies en componenten) buiten de component
- minstens één form met meerdere velden met validatie (naast login/register)
- login systeem

### Routing

- heeft minstens 2 pagina's (naast login/register)
- routes worden afgeschermd met authenticatie en autorisatie

### State management

- meerdere API calls (naast login/register)
- degelijke foutmeldingen indien API-call faalt
- gebruikt useState enkel voor lokale state
- gebruikt gepast state management voor globale state - indien van toepassing

### Hooks

- gebruikt de hooks op de juiste manier

### Algemeen

- een aantal niet-triviale én werkende e2e testen
- minstens één extra technologie
- node_modules, .env, productiecredentials... werden niet gepushed op GitHub
- maakt gebruik van de laatste ES-features (async/await, object destructuring, spread operator...)
- de applicatie start zonder problemen op a.d.h.v. de instructies in de README en gebruikt hiervoor Docker.
- de applicatie draait online
- duidelijke en volledige README.md
- duidelijk en volledig dossier

### Demo

- de student toont een werkende en responsive React applicatie
- de student kan het project probleemloos starten
- de student toont aan dat alle testen slagen

### Mondeling examen

- de student kan de vragen van de lector gedetailleerd beantwoorden a.d.h.v. de geschreven code
- de student kan de gemaakte design keuzes verdedigen

## 4. Groepswerk

Voor de examenopdracht werk je in groepen van twee. Je maakt één repository aan in de GitHub classroom en werkt samen aan dezelfde codebase. Volg de instructies op Orion voor het aanmaken van een repository.

Jullie worden samen geëvalueerd en krijgen (normaal) dezelfde score. Zorg ervoor dat jullie beiden voldoende commits maken zodat duidelijk is dat jullie beiden aan het project gewerkt hebben. Indien niet iedereen voldoende bijgedragen heeft, kan dit gevolgen hebben voor de score van de persoon die minder bijgedragen heeft.

Hoewel het project in groep wordt uitgevoerd, wordt van elke student verwacht dat hij een significante en aantoonbare bijdrage levert aan het eindresultaat.

Elke student implementeert minstens één volledige feature van begin tot einde. Dit betekent dat de student verantwoordelijk is voor de volledige implementatie van deze functionaliteit, inclusief:

- de gebruikersinterface;
- de integratie met de back-end of REST API;
- authenticatie en autorisatie indien van toepassing;
- validatie en foutafhandeling;
- de bijhorende end-to-end tests.

Om de individuele bijdrage te kunnen beoordelen, wordt gewerkt volgens de **Feature Branch Workflow**. Elke feature wordt ontwikkeld in een afzonderlijke feature branch en via een duidelijke Pull Request (PR) geïntegreerd in de `main`-branch.

De Pull Request moet voldoende informatie bevatten om de uitgevoerde werkzaamheden te kunnen beoordelen. Het moet duidelijk zijn:

- welke functionaliteit werd ontwikkeld;
- welke onderdelen door de student werden geïmplementeerd;
- welke technische keuzes werden gemaakt;
- hoe de functionaliteit getest werd.

De individuele bijdrage van elke student moet aantoonbaar zijn aan de hand van de commits, feature branches en Pull Requests. Wanneer onvoldoende kan worden aangetoond welke bijdrage een student heeft geleverd, kan dit een negatieve invloed hebben op de evaluatie.

Werkt de samenwerking niet goed, dan kan je dit altijd via mail melden aan de lector. We kunnen in dat geval eventueel beslissen om jullie apart te evalueren. Het spreekt voor zich dat slechts één persoon verder kan gaan met het bestaande idee, de andere persoon moet dan een nieuw idee uitwerken.

Na de deadline bekijken we de activiteit in de repository en kunnen we zien wie wat en hoeveel heeft bijgedragen.

## 5. Dossier vereisten

Zorg dat de `dossier.md` van je repository aangevuld is, alle vereisten staan in het document.

In dit document staan lijnen die starten met een >, dit zijn instructies. Verwijder deze lijnen voor je het dossier indient!

Dien enkel een pdf in op Chamilo, er zijn genoeg plugins voor VS Code om Markdown naar pdf om te zetten, zoals bv. <https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf>.

!> Gebruik een degelijke opmaak in Markdown voor de README en het dossier! Zie [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) voor meer uitleg.

## 6. Demo vereisten

Naast het dossier dien je ook een demo van jouw applicatie op te nemen en te delen via Panopto met jouw lector(en).Je neemt samen een video op. Jullie verdelen zelf wie wat demonstreert, maar zorg ervoor dat beide personen evenveel aan bod komen in de video. Deze demo moet voldoen aan de volgende vereisten:

- De demo mag maximaal 15 minuten duren (inclusief Web Services, indien van toepassing)
- De webcam moet aanstaan tijdens de demo zodat je gezicht zichtbaar is
- Je deelt de demo via Panopto met jouw lector(en) - zorg ervoor dat de toegangsrechten correct ingesteld zijn
- Dit is geen commerciële presentatie, maar een technische demo gericht op de implementatie
- De demo moet opgenomen zijn vóór de deadline
- De demo moet minstens de volgende onderdelen bevatten/tonen:
  - **Context van de applicatie**: leg uit wat de applicatie doet, wat het doel is en waarom je dit onderwerp gekozen hebt
  - **Projectstructuur overlopen (optioneel)**:
    - Leg eventuele speciale ontwerpkeuzes uit (waarom bepaalde mappen/bestanden georganiseerd zijn zoals ze zijn)
    - Dit hoef je niet te doen als de projectstructuur identiek is aan de voorbeeldapplicatie
  - **Demo van de applicatie**:
    - Gebruik uitsluitend de online versie van je applicatie (geen localhost)
    - Indien je zowel Web Services als Front-end Web Development volgt:
      - Demonstreer de webservice door verschillende API endpoints uit te testen in Postman (GET, POST, PUT, DELETE operaties)
      - Toon je front-end applicatie en demonstreer dat deze responsive is door het scherm te verkleinen/vergroten of verschillende apparaatgroottes te simuleren
      - Demonstreer de werking van je front-end applicatie
    - Indien je enkel Web Services volgt:
      - Focus op het demonstreren van je API endpoints in Postman
      - Toon verschillende CRUD operaties voor je entiteiten
  - **Demo van de extra technologie**:
    - Toon de werking van de extra technologie in actie
    - Laat de code zien waar je de extra technologie geïmplementeerd hebt
    - Leg uit waarom je voor deze technologie gekozen hebt
    - Doe dit voor beide olods (Front-end Web Development en Web Services indien van toepassing)
  - **Testen demonstreren**:
    - Laat alle testen lopen via de command line
    - Toon dat ze succesvol slagen
  - **Code showcase**:
    - Toon een stukje code waar je bijzonder fier op bent
    - Leg uit waarom je dit goed vindt en wat het doet
    - Dit geldt voor beide olods (Front-end Web Development en Web Services indien van toepassing)

## 7. Mondeling examen

Tijdens het mondeling examen beantwoordt elke student twee vragen over de gerealiseerde applicatie. Daarbij wordt geëvalueerd in welke mate de student de behandelde theoretische concepten begrijpt en kan toepassen. De student toont aan dat hij de gebruikte technieken, ontwerpkeuzes en implementaties in de code kan verklaren en verantwoorden.

## 8. Voorbeelden van extra's

Je vindt misschien wel een interessante extra technologie in de [Node.js Toolbox](https://nodejstoolbox.com/). Een aantal veelgebruikte extra's zijn:

- Real time toepassing (sockets...)
- PWA
- State management met andere tools (Redux Toolkit, elf, react-query...)
- UI Component library (antd, Chakra UI...)
- Front-end in TypeScript i.p.v. JavaScript
- Ander op React gebaseerd framework (Next.js, preact...)
  - Let op: nog steeds met het toepassen van de nodige best practices, aangepast aan het framework! We merken vaak dat studenten die afwijken op dit punt, ook de best practices vergeten (door online tutorials) waardoor het project niet voldoet aan meerdere minimumvereisten.
  - Check bij jouw lector of het framework dat je wil gebruiken toegelaten is.
- ... (eigen inbreng, verras ons)

Bij het toevoegen van een extra technologie is het belangrijk dat deze ook echt gebruikt wordt in de applicatie. Een package toevoegen die je niet gebruikt, is ook niet-ontvankelijk. Houd ook rekening met de best practices die we in de les gezien hebben bij het implementeren van de extra technologie.

## 9. Indien je het olod Web Services niet volgt

Je gebruikt een bestaande (vrij te kiezen) API op het web en bouwt daarvoor een React front-end. Hier vind je een lijst met publieke API’s: <https://github.com/public-apis/public-apis>, maar er zijn er nog veel meer natuurlijk.

Indien je reeds geslaagd bent voor Web Services, mag je die back-end hergebruiken. Voeg de code van de back-end wel toe aan jouw repository (incl. instructies in de README).

Een aantal aandachtspunten:

- Iets toevoegen, bewerken of verwijderen zal met een publieke API niet mogelijk zijn. Je kan dit wel simuleren door de data lokaal op te slaan, te bewerken of te verwijderen (bv. in [Local storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)).
- Authenticatie en autorisatie zal ook niet mogelijk zijn, of op zijn minst helemaal anders dan in de cursus. Alternatieven zijn: [Auth0](https://auth0.com/), [Firebase](https://firebase.google.com/)...

!> Bespreek dit best even met je lector als je in deze situatie zit en (g)een idee hebt.

## 10. Evaluatie

Je wordt beoordeeld op basis van een portfolio dat je samenstelt gedurende het semester. Dit portfolio bestaat uit:

- De code van je applicatie
  - Moet te vinden zijn in de GitHub classroom repository op de `main` branch
- Het ingevulde dossier, als pdf ingediend op Chamilo
- Een demo van je applicatie via een Panopto-opname
- Het mondeling examen

Zorg ervoor dat jouw applicatie aan alle ontvankelijkheidscriteria voldoet op het moment van de deadline. Indien dit niet het geval is, krijg je een score 'AFWEZIG'.

**De deadline voor het portfolio is het einde van week 13 (vrijdag 18 december 2026, 23u59).**

Alle code zal voor de start van het volgend academiejaar verwijderd worden uit de GitHub classroom. Als je je applicatie wenst te behouden, zorg dan dat je deze tijdig naar een privé repository pusht.

Veel succes!

TODO: Andreas, sectie ook toevoegen aan webservices

## 10. Feature Branch Workflow

Om professioneel samen te werken aan het project gebruiken jullie de **Feature Branch Workflow**. Hierbij wordt elke nieuwe functionaliteit ontwikkeld in een aparte branch. Zo blijft de `main`-branch steeds een stabiele en werkende versie van de applicatie bevatten.

### Waarom gebruiken we deze workflow?

De Feature Branch Workflow biedt verschillende voordelen:

- Teamleden kunnen gelijktijdig aan hetzelfde project werken.
- Nieuwe functionaliteiten worden geïsoleerd ontwikkeld en getest.
- Conflicten worden sneller opgespoord en opgelost.
- De geschiedenis van het project blijft overzichtelijk.
- De workflow sluit aan bij de werkwijze die in professionele softwareteams wordt gebruikt.

---

### Stappenplan

### 1. Maak een feature branch

Voor elke nieuwe functionaliteit maak je een aparte branch vanuit de `main`-branch.

```bash
git checkout main
git pull origin main
git checkout -b feature/login
```

Gebruik duidelijke namen die beschrijven aan welke functionaliteit je werkt.

Voorbeelden:

```text
feature/login
feature/product-overview
feature/favorites
feature/profile
```

---

### 2. Ontwikkel en commit regelmatig

Werk de functionaliteit uit op je eigen branch en maak regelmatig commits.

```bash
git add .
git commit -m "Voegt loginformulier toe"
```

Geef je commits een duidelijke beschrijving van de aangebrachte wijziging.

---

### 3. Push de branch naar GitHub

Publiceer je branch op GitHub.

```bash
git push -u origin feature/login
```

Zo kan je teamlid je werk bekijken en opvolgen.

---

### 4. Maak een Pull Request

Wanneer de functionaliteit klaar is, maak je op GitHub een **Pull Request (PR)** van je feature branch naar `main`.

Beschrijf hierbij:

- welke functionaliteit werd toegevoegd;
- welke bestanden aangepast werden;
- hoe de functionaliteit getest kan worden;
- eventuele gekende beperkingen of aandachtspunten.

---

### 5. Voer een code review uit

Voordat de wijzigingen worden samengevoegd, bekijkt het andere teamlid de code.

Tijdens een code review controleer je onder andere:

- of de functionaliteit correct werkt;
- of de code leesbaar en onderhoudbaar is;
- of de afgesproken coding standards gevolgd worden;
- of er geen onnodige duplicatie aanwezig is.

Indien nodig worden eerst verbeteringen aangebracht voordat de Pull Request wordt goedgekeurd.

---

### 6. Merge naar main

Na goedkeuring kan de Pull Request worden gemerged.

De nieuwe functionaliteit maakt nu deel uit van de stabiele versie van de applicatie.

Controleer daarna steeds of je lokale versie van `main` up-to-date is:

```bash
git checkout main
git pull origin main
```

---

### 7. Verwijder de feature branches niet!

Na het mergen mag je de feature branches niet verwijderen zodat de lectoren de code kunnen nakijken. Indien je de feature branch lokaal wil verwijderen, kan dat met:

```bash
git branch -d feature/login
```

### Goede afspraken

- Werk **nooit rechtstreeks op de `main`-branch**.
- Maak voor elke nieuwe functionaliteit een aparte feature branch.
- Commit regelmatig met duidelijke commitberichten.
- Maak voor elke feature een Pull Request.
- Laat je code nakijken door je teamlid voordat deze wordt gemerged.
- Synchroniseer regelmatig met de laatste versie van `main`.
- Zorg ervoor dat de applicatie blijft werken voordat een Pull Request wordt goedgekeurd.

---

### Overzicht

```text
main
 │
 ├── feature/login
 │         │
 │         └── Pull Request ──► main
 │
 ├── feature/favorites
 │         │
 │         └── Pull Request ──► main
 │
 └── feature/profile
           │
           └── Pull Request ──► main
```

Met deze workflow blijft de `main`-branch steeds stabiel en kunnen beide teamleden onafhankelijk van elkaar aan nieuwe functionaliteiten werken zonder elkaars werk te verstoren.

## 11. Code Review

Een code review is een gestructureerde controle van code die door een teamlid werd geschreven. Het doel is om fouten vroegtijdig op te sporen, de kwaliteit van de code te verbeteren en kennis binnen het team te delen.

Bij elke Pull Request voert je medestudent een code review uit voordat de wijzigingen naar `main` worden gemerged.

### Hoe voer je een code review uit?

#### 1. Open de Pull Request

Bekijk eerst de beschrijving van de Pull Request:

- Wat werd ontwikkeld?
- Welke functionaliteit werd toegevoegd of gewijzigd?
- Zijn er specifieke aandachtspunten?

Controleer of de wijzigingen overeenkomen met het doel van de feature.

#### 2. Test de functionaliteit

Controleer of de nieuwe functionaliteit correct werkt:

- Werkt alles zoals verwacht?
- Zijn er foutmeldingen?
- Werken bestaande functionaliteiten nog steeds?

#### 3. Bekijk de code

Lees de gewijzigde bestanden aandachtig en stel jezelf de volgende vragen:

#### Correctheid

- Werkt de code correct?
- Zijn alle randgevallen behandeld?
- Kan de code onverwachte fouten veroorzaken?

#### Leesbaarheid

- Zijn variabelen en functies duidelijk benoemd?
- Is de code overzichtelijk gestructureerd?
- Zijn complexe stukken voldoende gedocumenteerd?

#### Onderhoudbaarheid

- Kan een andere ontwikkelaar de code gemakkelijk begrijpen?
- Is er onnodige duplicatie aanwezig?
- Zijn componenten niet groter of complexer dan nodig?

#### React Best Practices

- Zijn componenten logisch opgebouwd?
- Wordt state correct gebruikt?
- Is de verantwoordelijkheid van componenten duidelijk afgebakend?
- Worden props correct gebruikt?

### 4. Geef constructieve feedback

Wanneer je een probleem opmerkt, formuleer je feedback op een respectvolle en constructieve manier.

**Goed voorbeeld:**

> Deze logica komt ook voor in component X. Misschien kunnen we hiervan een herbruikbare functie maken.

**Minder goed voorbeeld:**

> Dit is fout.

Leg bij voorkeur uit:

- wat het probleem is;
- waarom het een probleem is;
- hoe het eventueel verbeterd kan worden.

### 5. Keur de Pull Request goed of vraag wijzigingen

Na de review heb je drie mogelijkheden:

- **Approve**: de code voldoet aan de verwachtingen.
- **Comment**: je geeft opmerkingen zonder wijzigingen te eisen.
- **Request changes**: er moeten eerst nog aanpassingen gebeuren.

Pas wanneer alle gevraagde wijzigingen zijn uitgevoerd, mag de Pull Request worden gemerged.

### Checklist voor een code review

Voor je een Pull Request goedkeurt, controleer je of:

- [ ] De applicatie correct werkt.
- [ ] Er geen console-errors aanwezig zijn.
- [ ] De code leesbaar en onderhoudbaar is.
- [ ] Variabelen en functies duidelijke namen hebben.
- [ ] Er geen onnodige duplicatie aanwezig is.
- [ ] De React best practices gevolgd worden.
- [ ] De code voldoet aan de afgesproken conventies.
- [ ] De Pull Request beperkt blijft tot één feature.

### Waarom is code review belangrijk?

Door elkaars code te reviewen:

- verhoog je de kwaliteit van het project;
- leer je van elkaar;
- ontdek je fouten sneller;
- zorg je ervoor dat beide teamleden de volledige applicatie begrijpen.

Code review is daarom een essentieel onderdeel van professionele softwareontwikkeling.
