# Algemene info

## Situering

Dit olod behoort tot het keuzepakket `Development`, zoals je hieronder kan zien:

![Dit olod in de keuzepakketten](./images/MT_olods.png ':size=70%')

## Wat gaan we doen?

Concreet maken we een Single Page Application (SPA), met JavaScript. Zoals je misschien al weet komen er bijna dagelijks nieuwe JavaScript-frameworks en -libraries voor SPA's bij.

Wij hebben gekozen voor React. Waarom? Het wordt veel gebruikt (zie <https://2023.stateofjs.com/en-US/libraries/front-end-frameworks> en ook <https://emnudge.dev/blog/react-hostage>), het is van Meta... maar eigenlijk doen alle frameworks en libraries min of meer hetzelfde. Gelijkaardige frameworks zijn bijvoorbeeld [Angular](https://angular.io/), [VueJS](https://vuejs.org/), [Svelte](https://svelte.dev/) en [SolidJS](https://www.solidjs.com/).

## Wat gaan jullie doen?

Programmeren leer je enkel door het te doen, niet door onze cursus te lezen. Je zal bijgevolg merken dat in het cursusmateriaal enkel het absolute minimum staat.

Voor dit olod is er een [examenopdracht](0-intro/examenopdracht.md). Kort gezegd moet je een React-applicatie maken tegen week 13. De voorwaarden van deze app en de examenvorm staan duidelijk in de opdracht. De bijbehorende back-end maak je, indien van toepassing, in het olod Web Services.

De Chamilo-cursus vind je [hier](https://chamilo.hogent.be/index.php?go=CourseViewer&application=Chamilo%5CApplication%5CWeblcms&course=64499). Hierin komen alle belangrijke aankondigingen, een link naar de cursus en een uploadmodule voor de examenopdracht. Op de cursus zal je ook een link naar de GitHub-classroom zien verschijnen. Zonder repository in deze classroom kunnen wij niet aan je code en kan je hierop dus niet geëvalueerd worden.

### Deadline

> Week 13: vrijdag 20 december 2024 om 23u59

Je weet de deadline, plan je werk goed in! Wacht niet tot de laatste paar weken om te starten, dan zal je gegarandeerd in tijdsnood komen. Tijdens de lessen is ook voldoende tijd om aan de applicatie te werken, maak hier gebruik van!

### Voorbeelden

Naar goeie traditie schrijven we hier enkele voorbeelden van jullie voorgangers. Imponeer ons en mogelijks komt jouw idee hier te staan:

- Auto-verhuur
- Stockbeheer voor het IT-lab
- Chat-applicatie (met WebSockets)
- Beheer van verzamelingen (zeldzame strips, antiek...)
- Websites om te zoeken/luisteren naar podcasts
- Quiz-applicatie
- Website voor een vereniging of het bedrijf van een vriend(in), familielid...

## Cursus?

Het cursusmateriaal wordt op GitHub gehost: <https://HOGENT-frontendweb.github.io/frontendweb-cursus>.

Er is een voorbeeldapplicatie (stap per stap opgebouwd, zoals in de cursus): <https://github.com/HOGENT-frontendweb/frontendweb-budget>.

De bijhorende back-end is te vinden op: <https://github.com/HOGENT-frontendweb/webservices-budget>.

> Suggesties voor verbeteringen of aanpassingen van schrijffouten zijn altijd welkom! Maak hiervoor een issue of pull request op de GitHub-repository van de cursus: <https://github.com/HOGENT-frontendweb/frontendweb-cursus>.

## Planning

> TODO: planning updaten

Deze planning is een richtlijn en kan nog wijzigen in functie van verlofdagen.

| Week    | Inhoud                                        |
| ------- | --------------------------------------------- |
| week 1  | Inleiding, React Basics                       |
| week 2  | React Basics, useState                        |
| week 3  | useState, useContext                          |
| week 4  | useContext, formulieren                       |
| week 5  | Data ophalen van een back-end, useEffect      |
| week 6  | React Router                                  |
| week 7  | Testen en linting                             |
| week 8  | (geen nieuwe theorie, aan de opdracht werken) |
| week 9  | authenticatie / autorisatie                   |
| week 10 | testen met authenticatie / autorisatie        |
| week 11 | CI/CD, online zetten                          |
| week 12 | (geen nieuwe theorie, aan de opdracht werken) |

## Help, ik zit vast?

Lees de foutboodschappen, copy-paste ze in Google. Vaak 'helpen' we studenten door de fout te copy-pasten en de eerste link in Google te kopiëren.

### Het werkt niet maar geen error te zien?

- eerst en vooral stappen vinden die het probleem reproduceren
- dan het probleem proberen isoleren (databank? back-end? front-end?)
- gebruik een debugger, log statements; denk even na

### Nog altijd vast?

- maak een GitHub issue op jouw repository
- vul een van de gegeven templates in
  - **let op:** dit is NIET een bestand in de map `.github/ISSUE_TEMPLATE` aanpassen, deze laat je gewoon staan!
  - lees dit: <https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue>
- link jouw lector aan dit issue (als assignee en/of getagd)
  - anders krijgen we geen melding van jouw issue en kunnen we je niet helpen
