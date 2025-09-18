# Front-end Web Development Opdracht 2025 - 2026

## 1. De opdracht

Gedurende het semester maak je een webapplicatie bestaande uit een front-end gedeelte in React en, indien je het olod Web Services ook volgt, een bijhorende back-end in NodeJS. Indien je het olod Web Services niet volgt, zie [sectie 7](#7-indien-je-het-olod-web-services-niet-volgt).

Je bent volledig vrij om te kiezen welke applicatie je ontwikkelt, maar het is wel belangrijk dat het een dynamische webapplicatie betreft. Het equivalent van iets wat met statische HTML kan bekomen worden is onvoldoende. Indien je twijfelt over jouw idee, mag je tijdens de les altijd overleggen met je lector of achteraf via een GitHub issue op jouw repository.

?> Er wordt enkel feedback gegeven op ideeën tijdens semester 1. Indien je een idee hebt dat je graag wil bespreken, doe dit dan tijdig.

Voor de examenopdracht mag je optioneel per 2 samenwerken. Zie [sectie 8](#8-groepswerk-optioneel) voor meer informatie.

Alle code moet in een GitHub classroom repository terecht komen (zie Chamilo voor een link naar de classroom). Enkel de `main` branch van deze repository zal geëvalueerd worden. Er wordt automatisch een template van de `README.md` aangemaakt als je de opdracht accepteert, vul deze correct in. Je gebruikt dezelfde repository voor zowel Web Services als Front-end Web Development.

Het is belangrijk dat de applicatie significant verschilt van de voorbeeldapplicatie die tijdens de les gemaakt wordt.

Daarnaast verwachten we dat je een dossier met uitleg over je app indient op Chamilo. Een template voor dit dossier (`dossier.md`) vind je ook in jouw repository en dien je te gebruiken. **Je dient het dossier in als pdf!**

## 2. Ontvankelijkheidscriteria

Alvorens we jouw project evalueren, controleren we of het voldoet aan een aantal ontvankelijkheidscriteria. Indien deze niet voldaan zijn, krijg je een score van 0/20. Deze criteria zijn:

- Het dossier is volledig en tijdig ingediend (zie [sectie 4](#4-dossier-vereisten) voor de vereisten)
- Er werden voldoende (kleine) commits gemaakt
  - Als je per 2 werkt, moeten we een aantal pull requests met feedback zien
- De applicatie draait online
- De applicatie start zonder problemen op gebruik makend van de instructies in de README
- De applicatie wijkt voldoende af van de voorbeeldapplicatie
- De applicatie is gemaakt in React.
- node_modules, .env, productiecredentials... werden niet gepushed op GitHub
- Er is een extra technologie gebruikt (zie [sectie 6](#6-voorbeelden-van-extras) voor voorbeelden)
- Er werden een aantal niet-triviale en werkende integratietesten gemaakt (naast de testen voor user)
- De demo duurt niet langer dan 15 minuten (incl. Web Services indien van toepassing)

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
- minstens één extra technologie (zie [sectie 6](#6-voorbeelden-van-extras) voor voorbeelden)
- node_modules, .env, productiecredentials... werden niet gepushed op GitHub
- maakt gebruik van de laatste ES-features (async/await, object destructuring, spread operator...)
- duidelijke en volledige README.md

### Demo

- de student toont een werkende React applicatie
- de student overloopt de projectstructuur
- de applicatie is responsive en heeft een degelijke stijl
- de applicatie wijkt voldoende af van de voorbeeldapplicatie
- de student toont de implementatie/werking van de extra technologie
- alle testen slagen
- de student toont een stukje code waar die fier op is

## 4. Dossier vereisten

Zorg dat de `dossier.md` van je repository aangevuld is, alle vereisten staan in het document.

In dit document staan lijnen die starten met een >, dit zijn instructies. Verwijder deze lijnen voor je het dossier indient!

Dien enkel een pdf in op Chamilo, er zijn genoeg plugins voor VS Code om Markdown naar pdf om te zetten, zoals bv. <https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf>.

!> Gebruik een degelijke opmaak in Markdown voor de README en het dossier! Zie [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) voor meer uitleg.

## 5. Demo vereisten

Naast het dossier dien je ook een demo van jouw applicatie op te nemen en te delen via Panopto met jouw lector(en). Deze demo moet voldoen aan de volgende vereisten:

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

## 6. Voorbeelden van extra's

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

## 7. Indien je het olod Web Services niet volgt

Je gebruikt een bestaande (vrij te kiezen) API op het web en bouwt daarvoor een React front-end. Hier vind je een lijst met publieke API’s: <https://github.com/public-apis/public-apis>, maar er zijn er nog veel meer natuurlijk.

Indien je reeds geslaagd bent voor Web Services, mag je die back-end hergebruiken. Voeg de code van de back-end wel toe aan jouw repository (incl. instructies in de README).

Een aantal aandachtspunten:

- Iets toevoegen, bewerken of verwijderen zal met een publieke API niet mogelijk zijn. Je kan dit wel simuleren door de data lokaal op te slaan, te bewerken of te verwijderen (bv. in [Local storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)).
- Authenticatie en autorisatie zal ook niet mogelijk zijn, of op zijn minst helemaal anders dan in de cursus. Alternatieven zijn: [Auth0](https://auth0.com/), [Firebase](https://firebase.google.com/)...

!> Bespreek dit best even met je lector als je in deze situatie zit en (g)een idee hebt.

## 8. Groepswerk (optioneel)

In het geval je voor de examenopdracht per 2 wil samenwerken, kan dit. Je maakt in dat geval één repository aan in de GitHub classroom en werkt samen aan dezelfde codebase. Volg de instructies op Chamilo voor het aanmaken van een repository.

Jullie worden samen geëvalueerd en krijgen (normaal) dezelfde score. Zorg ervoor dat jullie beiden voldoende commits maken zodat duidelijk is dat jullie beiden aan het project gewerkt hebben. Indien niet iedereen voldoende bijgedragen heeft, kan dit gevolgen hebben voor de score van de persoon die minder bijgedragen heeft.

Maak gebruik van branches en pull requests om samen te werken aan de codebase. Dit helpt om de bijdragen van elke teamgenoot duidelijk te maken en simuleert een professionele werkomgeving.

Werkt de samenwerking niet goed, dan kan je dit altijd via mail melden aan de lector. We kunnen in dat geval eventueel beslissen om jullie apart te evalueren. Het spreekt voor zich dat slechts één persoon verder kan gaan met het bestaande idee, de andere persoon moet dan een nieuw idee uitwerken.

Na de deadline bekijken we de activiteit in de repository en kunnen we zien wie wat en hoeveel heeft bijgedragen.

Voor de demo neem je best samen een video op. Jullie verdelen zelf wie wat demonstreert, maar zorg ervoor dat beide personen evenveel aan bod komen in de video.

## 9. Evaluatie

Je wordt beoordeeld op basis van een portfolio dat je samenstelt gedurende het semester. Dit portfolio bestaat uit:

- De code van je applicatie
  - Moet te vinden zijn in de GitHub classroom repository op de `main` branch
- Het ingevulde dossier, als pdf ingediend op Chamilo
- Een demo van je applicatie via een Panopto-opname

Zorg ervoor dat jouw applicatie aan alle ontvankelijkheidscriteria voldoet op het moment van de deadline. Indien dit niet het geval is, krijg je een score van 0/20.

**De deadline voor het portfolio is het einde van week 13 (vrijdag 19 december 2025, 23u59).**

Alle code zal voor de start van het volgend academiejaar verwijderd worden uit de GitHub classroom. Als je je applicatie wenst te behouden, zorg dan dat je deze tijdig naar een privé repository pusht.

Veel succes!
