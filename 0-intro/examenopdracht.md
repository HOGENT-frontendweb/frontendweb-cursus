# Front-end Web Development Opdracht 2024 - 2025

## 1. De opdracht

Gedurende het semester maak je een webapplicatie bestaande uit een front-end gedeelte in React en, indien je het olod Web Services ook volgt, een bijhorende back-end in NodeJS. Indien je het olod Web Services niet volgt, zie [sectie 5](#5-indien-je-het-vak-web-services-niet-volgt).

Je bent volledig vrij om te kiezen welke applicatie je ontwikkelt, maar het is wel belangrijk dat het een dynamische webapplicatie betreft. Het equivalent van iets wat met statische HTML kan bekomen worden is onvoldoende. Indien je twijfelt over jouw idee, mag je altijd eerst met je lector overleggen of een GitHub issue aanmaken.

Alle code moet in een GitHub classroom repository terecht komen (zie Chamilo voor een link naar de classroom). Enkel de `main` branch van deze repository zal geëvalueerd worden. Er wordt automatisch een template van de `README.md` aangemaakt als je de opdracht accepteert, vul deze correct in. Je gebruikt dezelfde repository voor zowel Web Services als Front-end Web Development.

Het is belangrijk dat de applicatie significant verschilt van de voorbeeldapplicatie die tijdens de les gemaakt wordt.

Daarnaast verwachten we dat je een dossier met uitleg over je app indient op Chamilo. Een template voor dit dossier (`dossier.md`) wordt ook aangeleverd als je de opdracht accepteert en dien je te gebruiken. **Je dient het dossier in als PDF!**

## 2. Minimumvereisten

- Een werkende webapplicatie
- React front-end
- Login systeem
- Meerdere API-calls (naast de login & register)
- Best practices toepassen
  - degelijke foutmeldingen indien API-call faalt
  - splitsen van components (dom en slim)
  - gebruik van de juiste hooks
  - ...
- Meerdere componenten (naast login & register)
- Minstens één formulier met validatie (naast login & register)
- Routing met minstens 2 pagina’s (naast login & register)
- Gebruik de laatste ES-features (async/await, object destructuring, spread operator...) - dus geen callbacks, then/catch...
- Responsive en een degelijke stijl
- Regelmatige commits in git (één of een paar commits helemaal op het einde wordt niet aanvaard)
- Een aantal niet-triviale én werkende e2e testen
- Correct ingevulde README met een degelijke opmaak in Markdown (zie [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet))
- Een volledig en tijdig ingediend dossier (zie [sectie 3](#3-dossier-vereisten) voor de vereisten)
- De applicatie draait online
- Minstens één extra technologie die niet aan bod kwam in de cursus (zie [sectie 4](#4-voorbeelden-van-extras) voor voorbeelden)

## 3. Dossier vereisten

- Zorg dat de `dossier.md` van je repository aangevuld is, alle vereisten staan in het document.
- In dit document staan lijnen die starten met een >, dit zijn instructies. Verwijder deze lijnen voor je het dossier indient!
- Dien enkel een PDF in op Chamilo, er zijn genoeg plugins voor VS Code om Markdown naar PDF om te zetten, zoals bv. <https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf>.

## 4. Voorbeelden van extra's

- Production build automatiseren en online plaatsen (front- én back-end)
- Real time toepassing (sockets...)
- PWA
- State management met andere tools (Redux Toolkit, elf, react-query...)
- UI Component library (antd, Chakra UI…)
- Front-end in TypeScript i.p.v. JavaScript
- Ander op React gebaseerd framework (Next.js, preact...)
  - Let op: nog steeds met het toepassen van de nodige best practices, aangepast aan het framework
- ... (eigen inbreng, verras ons)

## 5. Indien je het olod Web Services niet volgt

Je gebruikt een bestaande (vrij te kiezen) API op het web en bouwt daarvoor een React front-end. Hier vind je een lijst met publieke API’s: <https://github.com/public-apis/public-apis>, maar er zijn er nog veel meer natuurlijk.

Aangezien authenticatie en autorisatie niet mogelijk zal zijn, of op zijn minst helemaal anders dan in de cursus, bespreek je dit best even met je lector als je in deze situatie zit en (g)een idee hebt.

## 6. Vragen

Als je vragen of hulp nodig hebt: maak altijd een GitHub issue aan en tag je lector. Gebruik het voorziene template voor het GitHub issue. Uiteraard kan je dit ook gewoon na de les vragen.

Mails worden niet beantwoord.

## 7. Evaluatie

Je wordt beoordeeld op basis van een portfolio dat je samenstelt gedurende het semester. Dit portfolio bestaat uit:

- De code van je applicatie
  - Moet te vinden zijn in de GitHub classroom repository op de `main` branch
- Het ingevulde dossier, als PDF ingediend op Chamilo
- Een ingevulde rubrics (evaluatiekaart) die je kan vinden op Chamilo, en ook ingediend op Chamilo
- Een demo van je applicatie via een Panopto-opname
  - De demo mag maximaal 15 minuten duren (incl. demo Web Services)
  - De webcam moet aanstaan tijdens de demo
  - Je deelt de demo via Panopto met jouw lector(en)
  - Dit is geen commerciële presentatie, maar een technische demo
  - De demo moet minstens de volgende zaken bevatten/tonen:
    - Context van de applicatie (= wat doet de applicatie? wat is het doel?)
    - Projectstructuur overlopen (mappenstructuur, speciale ontwerpkeuzes...)
    - Demo van de applicatie (gebruik de online versie)
    - Demo van de extra technologie + werking/implementatie
    - Testen laten lopen
    - Toon een stukje code waar je fier op bent en leg uit

**De deadline voor het portfolio is het einde van week 13 (vrijdag 20 december 2024, 23u59).**

Alle code zal een jaar later (januari 2026) verwijderd worden uit de GitHub classroom. Als je je applicatie wenst te behouden, zorg dan dat je ook naar een privé repository pusht.

Veel succes!
