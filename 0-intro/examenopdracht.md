# Front-end Web Development Opdracht 2026 - 2027

## 1. De opdracht

Gedurende het semester maak je samen met een medestudent een webapplicatie bestaande uit een front-end gedeelte in React en, indien je het olod Web Services ook volgt, een bijhorende back-end in NodeJS. Indien je het olod Web Services niet volgt, zie [sectie 9](#9-indien-je-het-olod-web-services-niet-volgt).

Je bent volledig vrij om te kiezen welke applicatie je ontwikkelt, maar het is wel belangrijk dat het een dynamische webapplicatie betreft. Het equivalent van iets wat met statische HTML kan bekomen worden is onvoldoende. Indien je twijfelt over jouw idee, mag je tijdens de les altijd overleggen met je lector of achteraf via een GitHub issue op jouw repository.

?> Er wordt enkel feedback gegeven op ideeën tijdens semester 1. Indien je een idee hebt dat je graag wil bespreken, doe dit dan tijdig.

De examenopdracht wordt uitgevoerd in teams van twee. Beide teamleden dragen bij aan alle onderdelen van de applicatie en zijn verantwoordelijk voor het volledige eindresultaat. Zie [sectie 4](#4-groepswerk) voor de deadlines van de registratie  van je team en meer informatie.

Alle code moet in een private GitHub repository terecht komen die je deelt met alle lectoren (zie [Sectie 11](#11-github)). Enkel de `main` branch van deze repository zal geëvalueerd worden. Voor de aanmaak van de repository vertrek je van een template. De template bevat een `README.md` bestand, vul deze correct in. Je gebruikt dezelfde repository voor zowel Web Services als Front-end Web Development. Check zeker onze [appendix over Git & GitHub](https://hogent-frontendweb.github.io/webservices-cursus/#/appendices/2-github/index.md) als je hiermee nog niet vertrouwd bent.

Het is belangrijk dat de applicatie significant verschilt van de voorbeeldapplicatie die tijdens de les gemaakt wordt.

Daarnaast verwachten we dat je een dossier met uitleg over je app indient op Orion. Een template voor dit dossier (`dossier.md`) vind je ook in jouw repository en dien je te gebruiken. **Je dient het dossier in als pdf!**

## 2. Ontvankelijkheidscriteria

Alvorens we jouw project evalueren, controleren we of het voldoet aan een aantal ontvankelijkheidscriteria.

!> Als niet voldaan is aan de ontvankelijkheidscriteria, krijg je de score "Afwezig" (conform het DOER).

Deze criteria zijn:

- Je bent ingeschreven in een groep op Orion (deadline: einde week 2, vrijdag 2 oktober 2026, 23u59)(zie [sectie 4](#4-groepswerk) voor de vereisten)
- Het dossier is volledig en tijdig ingediend  (deadline: einde week 13, vrijdag 18 december 2026, 23u59)(zie [sectie 5](#5-dossier-vereisten) voor de vereisten)
- Elke student heeft minstens twee feature branches waarin een volledige feature werd uitgewerkt. Voor elke feature branch werd minstens één pull request aangemaakt. De student voert minstens 10 kleine, betekenisvolle commits uit tijdens de ontwikkeling van de feature. Op elke pull request werd inhoudelijke feedback gegeven door de medestudent(zie [sectie 4](#4-groepswerk) en [sectie Feature Branch workflow](https://hogent-frontendweb.github.io/webservices-cursus/#/appendices/2-github/index?id=feature-branch-workflow)).
- De applicatie is gemaakt in React
- De applicatie draait online
- De applicatie start zonder problemen op a.d.h.v. de instructies in de README en gebruikt hiervoor Docker.
- De applicatie wijkt voldoende af van de voorbeeldapplicatie
- node_modules, .env, productiecredentials... werden niet gepushed op GitHub
- Er is een extra technologie gebruikt (zie [sectie 8](#8-voorbeelden-van-extras) voor voorbeelden)
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

- de student kan de vragen van de lector gedetailleerd beantwoorden a.d.h.v. de geschreven code, en toont zo aan dat hij/zij de behandelde theoretische concepten begrijpt en kan toepassen
- de student kan de gemaakte design keuzes verdedigen

## 4. Groepswerk

Voor de examenopdracht werk je in groepen van twee.

### groep inschrijving

Schrijf je in Orion in het vak Frontendweb development in voor een groep van twee studenten. Indien je nog geen partner hebt gevonden, schrijf je dan in bij de groep 'Op zoek naar medestudent'. De lector zal na afloop van de inschrijvingsperiode de studenten uit deze groep samenbrengen tot projectgroepen van twee.

De deadline voor het vormen van groepen is **vrijdag 2 oktober 2026 om 23.59 uur (einde van week 2)**.

!> Studenten die na deze deadline niet zijn ingeschreven in een groep, krijgen voor de score AFWEZIG voor het examen.

### ANS-opdracht

Zodra je bent ingeschreven in een groep, dienen beide groepsleden de ANS-opdracht in te vullen en in te dienen. Ga hiervoor naar [ANS](https://ans.app/), log in met je studenten account en vul de opdracht 'Inschrijven voor examenopdracht'in.

In deze opdracht vermeld je:

- het groepsnummer (zoals vermeld in Orion);
- je naam en voornaam
- de URL van de GitHub-repository die je hebt aangemaakt op basis van de aangeboden template (zie [sectie 11](#11-repository-aanmaken));
- je publieke SSH-sleutel (niet de private sleutel) (zie [sectie 12](#12-ssh-sleutel-aanmaken)).

Beide studenten moeten de ANS-opdracht afzonderlijk indienen.

De deadline voor deze opdracht is **vrijdag 9 oktober 2026 om 23.59 uur (einde van week 3)**. Deze gegevens hebben we nodig voor het aanmaken van de VPS op het VIC en je toegang tot deze server te geven. Indien je de ANS-opdracht niet indient, dien je zelf [contact op te nemen met het VIC](https://vichogent.be/nl/hosting) en sta je zelf in voor de correcte aanmaak van de VPS.

### Repository en individuele bijdrage

Je maakt per groep één repository aan(zie [sectie 11](#11-repository-aanmaken)) en werkt samen aan dezelfde codebase. Volg de instructies in [Sectie 2](#2-ontvankelijkheidscriteria) om de repository correct aan te maken. De repository wordt gedeeld met beide studenten en de lectoren. De repository is privé, maar de lectoren hebben toegang tot de code.

Om de individuele bijdrage te kunnen beoordelen, wordt gewerkt volgens de Feature Branch Workflow. Elke feature wordt door één student ontwikkeld in een afzonderlijke feature branch en via een Pull Request (PR) geïntegreerd in de main-branch. De student die de feature ontwikkelt, maakt minstens 10 kleine commits en is verantwoordelijk voor het maken van de PR en het beantwoorden van eventuele vragen van de medestudent. De medestudent is verantwoordelijk voor het reviewen van de PR en het geven van inhoudelijke feedback. De medestudent mag ook suggesties doen voor verbeteringen, maar mag geen code toevoegen aan de feature branch van de andere student.

De Pull Request moet voldoende informatie bevatten om de uitgevoerde werkzaamheden te kunnen beoordelen. Hieruit moet duidelijk blijven:

- welke functionaliteit werd ontwikkeld;
- welke onderdelen door de student werden geïmplementeerd;
- welke technische keuzes werden gemaakt;
- hoe de functionaliteit werd getest.

Hoewel het project in groep wordt uitgevoerd, wordt van elke student verwacht dat die een substantiële en aantoonbare individuele bijdrage levert aan het eindresultaat. Deze bijdrage moet zichtbaar zijn in de commits, feature branches, Pull Requests en gegeven feedback op Pull Requests van medestudenten.

Na de deadline wordt de activiteit in de repository geanalyseerd. Daarbij wordt onder meer gekeken naar het aantal en de kwaliteit van de commits, het gebruik van feature branches, de inhoud van de Pull Requests en de bijdrage aan het reviewproces. Wanneer onvoldoende kan worden aangetoond welke bijdrage een student heeft geleverd, kan dit leiden tot een lagere individuele score.

## 5. Dossier vereisten

Zorg dat de `dossier.md` van je repository aangevuld is, alle vereisten staan in het document.

In dit document staan lijnen die starten met een >, dit zijn instructies. Verwijder deze lijnen voor je het dossier indient!

Dien enkel een pdf in op Orion, er zijn genoeg plugins voor VS Code om Markdown naar pdf om te zetten, zoals bv. <https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf>.

!> Gebruik een degelijke opmaak in Markdown voor de README en het dossier! Zie [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) voor meer uitleg.

## 6. Demo vereisten

Tijdens het mondeling examen dien je te starten met een demo van jouw applicatie. Jullie verdelen zelf wie wat demonstreert, maar zorg ervoor dat beide personen evenveel aan bod komen. Deze demo moet voldoen aan de volgende vereisten:

- De demo mag maximaal 15 minuten duren (inclusief Web Services, indien van toepassing)
- De webcam moet aanstaan tijdens de demo zodat je gezicht zichtbaar is
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
    - Toon dat ze succesvol slagen en toon het coverage report
  - **Code showcase**:
    - Toon een stukje code waar je bijzonder fier op bent
    - Leg uit waarom je dit goed vindt en wat het doet
    - Dit geldt voor beide olods (Front-end Web Development en Web Services indien van toepassing)

## 7. Mondelinge verdediging

Tijdens de mondelinge verdediging (15 minuten per applicatie) wordt per student nagegaan in welke mate de student inzicht heeft in de gerealiseerde applicatie en de behandelde leerinhouden. De student moet a.d.h.v. de geschreven code de gebruikte technieken, ontwerpkeuzes en implementaties kunnen verklaren en verantwoorden, de onderliggende theoretische concepten correct kunnen toelichten en kunnen aantonen hoe deze in de applicatie werden toegepast. Daarnaast moet de student kunnen aangeven hoe gevraagde wijzigingen of uitbreidingen aan de applicatie gerealiseerd kunnen worden.

De mondelinge verdediging dient tevens als validatie van de beoordeling op basis van de code-rubrics. De student moet kunnen aantonen, toelichten en verantwoorden hoe de criteria uit deze rubrics werden toegepast binnen de gerealiseerde applicatie en moet de gemaakte keuzes kunnen motiveren aan de hand van de ingediende code.

De vragen kunnen betrekking hebben op de code, de architectuur, de gebruikte technologieën, de behandelde theoretische concepten, de toegepaste best practices, de criteria uit de rubrics en de algemene werking van de applicatie..

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
- Het ingevulde dossier, als pdf ingediend op Orion
- Het mondeling examen met de demo van je applicatie en de mondelinge verdediging

Zorg ervoor dat jouw applicatie aan alle ontvankelijkheidscriteria voldoet op het moment van de deadline. Indien dit niet het geval is, krijg je een score 'AFWEZIG'.

**De deadline voor het portfolio is het einde van week 13 (vrijdag 18 december 2026, 23u59).**

In de **2e zit** mag je gewoon verder werken aan je huidig project, de opdracht wijzigt niet. De deadline is 22 augustus 2027, 23u59.

Veel succes!

## 11. Aanmaken van de repository

- Via de GitHub website, navigeer naar [de template-repository op GitHub](https://github.com/HOGENT-frontendweb/frontendweb-webservices-project-template-2627)
- Klik rechtsboven op de groene knop Use this template en kies Create a new repository.
- Selecteer de Owner (jouw account).
- Geef je nieuwe repository een Repository name : groepsnummer-frontendweb-webservices-2627.
- Kies voor een Private repository.
- Klik op Create repository from template. GitHub maakt nu een exacte kopie voor je aan zonder de commit-geschiedenis van de template.
- Voeg je medestudent toe als Collaborator aan de repository. Ga hiervoor naar Settings > Collaborators > Add people. Je medestudent krijgt een uitnodiging via e-mail en moet deze accepteren.
- Voeg ook de lectoren toe als Collaborators aan de repository met read rechten. Ga hiervoor naar Settings > Collaborators > Add people. De lectoren krijgen een uitnodiging via e-mail en moeten deze accepteren. Voeg de volgende lectoren toe:
  - Andreas De Smet: @dreeki
  - Karine Samyn: @ksa607
  - Pieter Vander Vennet: @pietervdvn (enkel indien je ook het olod Web Services volgt)

## 12. Aanmaken van de SSH-sleutel

Elke student maakt een SSH-sleutel aan.

- Open je terminal of command line.
- Voer het commando `ssh-keygen -t ed25519` in.
- Volg de instructies om de sleutel te genereren. Je zal een naam voor de sleutel moeten opgeven en een passphrase (wachtwoord) instellen. Je kan de standaardnaam gebruiken (`id_ed25519`) en een passphrase instellen of leeg laten.
- De sleutel wordt gegenereerd in de map `~/.ssh/` met de naam `id_ed25519` (private) en `id_ed25519.pub` (public).

De publieke sleutel deel je mee in de ANS opdracht. De lectoren gebruiken deze sleutel om je toegang te geven toteen VPS (Virtual Private Server) op het VIC. De private sleutel blijft privé en mag je niet delen. Je gebruikt deze private sleutel om in te loggen op het VIC.

Een voorbeeld van een publieke sleutel is als volgt:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHAA6JxFA0rcaXAEFJKf8ifSCvjzTnn2qsYuLEI0GVqN karine.samyn@hogent.be
```

<!-- TODO: Andreas, sectie ook toevoegen aan webservices-->
