# React basics

Neem volgende HTML-code als voorbeeld:

[example1.html](./examples/example1.html ':include :type=code')

Wanneer een browser een HTML-pagina rendert, zal hij o.a. een [Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) genereren. Hierbij wordt een `div` bijvoorbeeld een `HTMLDivElement` en een `p` een `HTMLParagraphElement`.

Met JavaScript is het mogelijk om de DOM te manipuleren, zoals bijvoorbeeld:

[example2.html](./examples/example2.html ':include :type=code')

Wat als resultaat heeft:

[example2.html](./examples/example2.html ':include height=100px')

Hiermee kan je ook eenvoudig een pagina interactief maken zonder de pagina volledig opnieuw te moeten renderen:

[example3.html](./examples/example3.html ':include :type=code')

Wat als resultaat heeft:

[example3.html](./examples/example3.html ':include height=100px')

## React

[React](https://reactjs.org/) is in principe niet meer dan een library om hetzelfde te bereiken, maar dan handiger in gebruik. Om React te gebruiken, hebben we twee libraries nodig

- `React`: laat toe om "views" aan te maken.
- `ReactDOM`: rendert deze "views" in de DOM, m.a.w. dit is React voor de browser. Er is bijvoorbeeld ook nog [React Native](https://reactnative.dev/) voor mobile applicaties.

Een React applicatie bestaat uit een heleboel componenten die samen een webpagina vormen. Een component is een stukje code dat een bepaald deel van de webpagina voorstelt. Een component kan andere componenten bevatten. Dit is een van de redenen waarom React zo populair is: je kan je code opdelen in kleine, herbruikbare stukken. Deze componenten samen vormen één boomstructuur, de component tree.

Binnen de context van React zal je vaak het begrip **renderen** horen. Renderen betekent letterlijk "het omzetten van een datastructuur naar een visuele voorstelling". Binnen React betekent dit dus het omzetten van React-componenten naar HTML. Tijdens het renderen zal React de component tree doorlopen en voor elk component de bijbehorende functie uitvoeren. Deze functie zal JSX teruggeven, wat React dan omzet naar HTML.

### JSX

Samen met React heeft Meta ook JSX geïntroduceerd (een samentrekking van JavaScript en XML). Hiermee is het mogelijk om een soort van HTML te schrijven in JavaScript. Let welop: JSX is geen standaard JavaScript en wordt niet begrepeen door de browser. JSX wordt door de compiler omgezet naar JavaScript.

Het 'HTML' stuk van JSX voelt echt vertrouwd als je HTML kent (en dat is natuurlijk de bedoeling), maar er zijn een aantal dingen waarop je moet letten:

- `class` is een reserved keyword in JavaScript, en kan dus niet gebruikt worden om een CSS klasse mee te geven aan een element, het wordt vervangen door `className`
- `for` is ook een reserved keyword in JavaScript, het wordt vervangen door `htmlFor`
- Als je JavaScript code wilt mixen met een stuk 'HTML', dien je het tussen `{ }` te zetten, bijvoorbeeld `<h1>{title}</h1>`.

<!-- TODO: onderstaande ergens anders zetten -->

`{name}` zorgt ervoor dat de waarde van de variable `name` gerenderd wordt. Met `{ }` kan je eender welke expressie in JavaScript uitvoeren in de HTML, je kan hier geen statements gebruiken. De uitvoer van deze code zal gerenderd worden in de HTML.

> Geen idee wat het verschil is tussen een statement of expression? Check dan eens de [Must read/watch](#must-readwatch) onderaan deze pagina.

## Vite

Bij een React applicatie is er steeds een compilatiestap nodig die de JSX omzet naar JavaScript. Deze compilatiestap zorgt ervoor dat de browser de code kan begrijpen, maar is niet beperkt tot enkel het omzetten van JSX naar JavaScript. Tijdens deze stap worden ook afbeeldingen goed gezet, CSS geoptimaliseerd, dependencies gebundeld, etc. Een tool die dergelijke taken uitvoert noemen we een buildtool of **bundler**.

Vroeger werd [`create-react-app`](https://create-react-app.dev/) gebruikt om een nieuwe React-applicatie te maken. Dit is een command line tool die een nieuwe React-applicatie opzet met een aantal standaardinstellingen. Het gebruikt achter de schermen [Webpack](https://webpack.js.org/) als bundler.

De laatste jaren is [Vite](https://vitejs.dev/) echter populairder geworden. [**Vite**](https://vitejs.dev/) (afgeleid van het Franse woord voor "snel") is een **buildtool** en **ontwikkelingsserver** die voornamelijk wordt gebruikt voor het bouwen van moderne webtoepassingen, zoals Single Page applications (SPA's) en Progressive Web Apps (PWA's). Het is ontwikkeld door Evan You, de maker van het populaire JavaScript-framework Vue.js, maar Vite kan ook worden gebruikt voor het bouwen van toepassingen met andere JavaScript-frameworks, zoals React en Svelte.

Hier zijn enkele belangrijke kenmerken en concepten met betrekking tot Vite:

- **Native ES modules**: Vite maakt gebruik van native ES modserviceules (ESM) voor het laden van modules in moderne browsers. Dit betekent dat bestanden afzonderlijk kunnen worden geladen zonder de noodzaak van een bundel-stap tijdens de ontwikkeling.

  ![Vite - native ES Modules](./images/vite_ESModules.webp ':size=80%')

- Wanneer de development build wordt gestart, verdeelt Vite de JavaScript-modules in twee categorieën: dependency modules en applicatie modules.service

  - De **dependency modules** zijn JavaScript-modules die je hebt geïmporteerd uit de map `node_modules`. Deze modules worden verwerkt en gebundeld met behulp van [esbuild](https://esbuild.github.io/), een JavaScript-bundler geschreven in Go die 10-100x sneller presteert dan [Webpack](https://webpack.js.org/).
  - De **applicatie modules** zijn modules die je voor je applicatie schrijft, zoals .jsx-bestanden.

- Vite verwerkt de dependency modules alleen vóór een enkel browserverzoek. De applicatie modules worden door Vite getransformeerd en aangereikt wanneer ze nodig zijn voor je applicatie.

- **HMR (Hot Module Replacement)**: In Vite worden enkel de gewijzigde modules vervangen. Dit zorgt voor snelle code-updates in de browser tijdens ontwikkeling.

- **Ontwikkelingsserver**: Vite bevat een ingebouwde ontwikkelingsserver die je gebruikt tijdens development. Deze server ondersteunt functies zoals snel laden (fast refresh), waardoor codeveranderingen direct worden weergegeven in de browser zonder de hele toepassing opnieuw te moeten compileren.

- **Builds**: Vite kan ook worden gebruikt om productiebuilds te maken. Tijdens de productiebuild past Vite verschillende optimalisaties toe, zoals minificatie, tree-shaking (waarbij ongebruikte code wordt verwijderd), en bundling (het samenvoegen van bestanden) om de laadtijden te minimaliseren en de prestaties van je applicatie te verbeteren. Vite bevat een vooraf geconfigureerde `build`-opdracht die de applicatie bundelt met behulp van [Rollup](https://rollupjs.org/). Vite biedt ook een standaard Rollup-configuratie die je kan aanpassen wanneer dat nodig is. De output bevindt zich in de `dist` folder en bevat statische assets die je plaatst op je productie server.

  ![Vite en bundling](./images/vite_bundling.png)

Lees hierover meer in de [Vite documentatie](https://vitejs.dev/guide/why.html).

## create-vite

Het is eenvoudig om een nieuwe React-applicatie te maken m.b.v. [create-vite](https://vitejs.dev/guide/). Een nieuwe React-applicatie maken is zo simpel als:

```bash
yarn create vite budget --template react-swc
```

Dit commando maakt een map `budget` met alle bestanden voor deze React-applicatie. We gaan doorheen deze cursus een budgetapplicatie ontwikkelen. In deze applicatie kan je transacties op bepaalde plaatsen bijhouden om zo je budget te beheren. We bouwen steeds verder op deze startapplicatie.

Deze map bevat onder andere volgende bestanden/mappen:

- `node_modules`: deze map bevat alle dependencies van de applicatie, m.a.w. de React libraries en alle libraries waar die dan weer op steunen. Dit is typisch een map met immens veel heel kleine bestanden (bij het maken van deze cursus: 40.020 (!) bestanden die 148 MB innemen).
- `package.json`: dit bestand beschrijft welke dependencies we nodig hebben, hoe de applicatie moet gestart, getest... worden, etc.
- `yarn.lock`: dit bestand bevat de exacte versies van de dependencies die effectief geïnstalleerd zijn.
- `public`: map die alles bevat wat publiek beschikbaar zal zijn voor onze webapplicaties (bv. afbeeldingen...).
- `src`: map die alle broncode bevat waarmee onze applicaties gebouwd gaat worden, dus allemaal JSX- en CSS-bestanden, etc.
- er werd ook automatisch een `.gitignore` voorzien.
- `eslint.config.js`: configuratiebestand voor [eslint](https://eslint.org/), een tool die je code analyseert en je waarschuwt voor mogelijke fouten, slechte praktijken, etc.
- `vite.config.js`: configuratiebestand voor Vite.
- `index.html`: de enige HTML-pagina van de applicatie. De inhoud van deze pagina zal steeds aangepast worden door React.

### yarn

[yarn](https://yarnpkg.com/) is het programma dat alle dependencies zal installeren, een andere misschien iets bekendere is [npm](https://www.npmjs.com/package/npm). Ze doen beide hetzelfde en zijn inwisselbaar maar de ene keer `yarn` gebruiken en de andere keer `npm` is dan weer geen goed idee. Ze kunnen andere versies van packages cachen e.d. en dan kan je rare fouten tegenkomen.

Het project gebruikt standaard Yarn v1. Dit passen we aan naar Yarn v2, naar analogie bij Web Services. Dit doen we door de volgende commando's uit te voeren:

```bash
corepack enable
yarn set version berry
```

Maak vervolgens een `.yarnrc.yml` bestand aan in de root van je project met volgende inhoud:

```yml
nodeLinker: node-modules
```

Dit zorgt ervoor dat Yarn v2 de dependencies installeert in de `node_modules` map. Voer hierna een `yarn install` uit om de dependencies te installeren.

### package.json

De [package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json) bevat alle metadata van ons project, meer in het bijzonder alle dependencies en commando's om onze app te starten. Het `yarn init` commando zou een `package.json` gemaakt moeten hebben in de root van je project. Open deze, en je zou iets als volgt moeten zien:

[package.json](examples/package.json ':include :type=code')

De `package.json` kan enkele properties bevatten:

- `name`: de naam van het project
- `version`: de versie van het project
- `description`: een korte beschrijving van het project
- `main`: het entry point van de applicatie
- `repository`: de URL van de git repository
- `author`: de auteur van de applicatie
- `license`: de licentie van de applicatie
- `private`: of de applicatie publiek is of niet, npm zal bv. niet toelaten om een private package te publiceren
- `dependencies`: de packages waarvan deze applicatie gebruik maakt
- `devDependencies`: packages enkel nodig in development (en dus niet in productie)
- `scripts`: laten toe om een soort van shortcuts te maken voor scripts (bv. de applicatie starten, testen, builden voor productie, etc.)
- `packageManager`: de package manager die gebruikt wordt (in dit geval Yarn)

Met een simpele `yarn install` installeren we meteen een identieke omgeving (met zowel `dependencies` als `devDependencies`) en dat maakt het handiger om in een team te werken (`yarn install --prod` installeert enkel de `dependencies`).

Het verschil tussen `dependencies` en `devDependencies` is het moment wanneer ze gebruikt worden. De `dependencies` zijn nodig in productie, m.a.w. de applicatie kan niet werken zonder deze packages. De `devDependencies` zijn enkel nodig om bv. het leven van de developer makkelijker te maken (types in TypeScript, linting, etc.) of bevatten packages die enkel gebruikt worden *at build time*, of dus wanneer de applicatie (bv. door webpack) omgevormd wordt tot iets wat browsers begrijpen.

Dependencies maken gebruik van [semantic versioning](https://semver.org/) (lees gerust eens door de specificatie). Kort gezegd houdt dit in dat elk versienummer bestaat uit drie delen: `MAJOR.MINOR.PATCH`, elke deel wordt met één verhoogd in volgende gevallen:

- `MAJOR`: wijzigingen die ***niet*** compatibel zijn met oudere versies
- `MINOR`: wijzigen die ***wel*** compatibel zijn met oudere versies
- `PATCH`: kleine bugfixes (compatibel met oudere versies)

In een `package.json` zie je ook vaak versies zonder prefix of met een tilde (~) of hoedje (^) als prefix, dit heeft volgende betekenis:

- geen prefix: exact deze versie
- tilde (~): ongeveer deze versie (zie <https://docs.npmjs.com/cli/v6/using-npm/semver#tilde-ranges-123-12-1>)
- hoedje (^): compatibel met deze versie (<https://docs.npmjs.com/cli/v6/using-npm/semver#caret-ranges-123-025-004>)

Kortom, een tilde is strenger dan een hoedje.

Het lijkt misschien een beetje raar, maar zo'n `package.json` wordt voor vele toepassingen en frameworks gebruikt. JavaScript programmeurs zijn gewoon van een `git pull`, `yarn install` en `yarn start` te doen, zonder per se te moeten weten hoe een specifiek framework opgestart wordt.

We hebben in bovenstaand voorbeeld een start-script toegevoegd dat je kan gebruiken om de server te starten. Kopieer de `scripts` naar jouw `package.json`. Dit script kan je nu uitvoeren met `yarn start`.

Er zijn nog heel wat andere opties voor de `package.json`. Je vindt alles op <https://docs.npmjs.com/cli/v10/configuring-npm/package-json>.

### yarn.lock en .yarn map

Wanneer je een package installeert, zal yarn een `yarn.lock` bestand aanmaken. Dit bestand bevat de exacte versies van de packages die geïnstalleerd zijn. Dit bestand moet je zeker mee opnemen in je git repository. Dit zorgt ervoor dat iedereen exact dezelfde versies van de packages gebruikt.

Dit bestand vermijdt versieconflicten aangezien in de `package.json` niet altijd de exacte versie staat maar een bepaalde syntax die aangeeft welke versies toegelaten zijn (zie vorige sectie).

Yarn zal ook een `.yarn` map aanmaken. Deze map bevat de geïnstalleerde versie van Yarn en wordt door [Corepack](https://nodejs.org/api/corepack.html) gebruikt om de packages te installeren. Deze map moet je niet opnemen in je git repository.

### .gitignore

Voor we verder gaan, maken we nog een `.gitignore` bestand aan. Dit bestand zorgt ervoor dat bepaalde bestanden/mappen niet naar GitHub gepusht worden. Dit is bv. handig voor de `node_modules` map, die we niet willen pushen omdat deze heel groot is en we deze niet nodig hebben om de applicatie te laten werken. Je kan nl. de dependencies eenvoudig opnieuw installeren d.m.v. `yarn install`.

Download de `.gitignore` van <https://github.com/github/gitignore/blob/main/Node.gitignore> en plaats deze in de root van je project. Het is belangrijk dat je het bestand exact de naam `.gitignore` heeft.

Kijk gerust eens welke bestanden er allemaal genegeerd worden. Je kan dit bestand ook aanpassen naar eigen wens, maar dit is een vrij complete voor een Node.js project.

### src

Start de applicatie met het commando `yarn dev`.

De `src` map bevat een aantal JSX-bestanden (`main.jsx`, `App.jsx`...) en wat CSS, e.d. `vite` zet dit om naar (door de browser begrijpbare) JavaScript. Dit gebeurt automatisch als een van de bronbestanden wijzigt.

Probeer maar iets aan te passen in de `App.jsx`. Je zal zien dat de browser automatisch herlaadt met de nieuwe inhoud (uiteraard als je geen compilatiefouten veroorzaakt).

?> **Best practice**: het is beter om bestanden met JSX de extensie `.jsx` te geven, dit brengt o.a. betere IntelliSense met zich mee (in bv. VS Code).

### eslint.config.js

Het bestand `eslint.config.js` bevat de configuratie voor [ESLint](https://eslint.org/), een linting tool. Linting is statische analyse van code om problemen zoals verkeerde syntax en twijfelachtig gebruik van code te detecteren. Waarom zou je gebruiken maken van linting en formatting? Het kan vroegtijdig fouten, typo's en syntax errors vinden. Het verplicht developers dezelfde codeerstijl te gebruiken, best practices te volgen en vermijdt het committen van slechte code.

De configuratie is gebaseerd op de aanbevolen regels van volgende plugins:

- [eslint-plugin-react](https://npmjs.com/package/eslint-plugin-react): regels voor React
- [eslint-plugin-react-hooks](https://npmjs.com/package/eslint-plugin-react-hooks): regels voor React hooks (bv. checken of ze correct gebruikt worden)
- [eslint-plugin-react-refresh](https://npmjs.com/package/eslint-plugin-react-refresh): regels om te controleren of componenten kunnen vervest worden met fast refresh

ESLint gebruikt sinds v9 [flat configuration files](https://eslint.org/blog/2022/08/new-config-system-part-2/). Deze syntax zorgt ervoor dat je geen `extends` meer moet gebruiken, dat je plugins simpelweg kan importeren via `import` i.p.v. een vaste string te gebruiken en dat je de regels rechtstreeks en eenvoudiger kan instellen.

Als je meer wil weten over de configuratie, gebruik dan de [ESLint documentatie](https://eslint.org/docs/user-guide/configuring).

Je kan de linting starten met het commando `yarn lint`. Deze print vervolgens alle fouten en waarschuwingen in de console. Als je aan dit commando `--fix` toevoegt, zal ESLint proberen om de fouten automatisch op te lossen.

## Transaction

In onze budget applicatie willen we uitgaven en inkomsten beheren via transacties. We maken een eerste component aan voor de weergave van één transactie.

Maak in de map `src` een map `components` aan met daarin een map `transactions`. In deze map maken we een nieuw bestand `Transaction.jsx` met volgende inhoud:

```jsx
// src/components/transactions/Transaction.jsx
export default function Transaction() {
  return <div>Benjamin gaf €200 uit bij Dranken Geers.</div>;
}
```

Components zijn niets meer dan functies die html terug geven, die moet getoond worden voor deze component.
Hier voegen we hard gecodeerde tekst toe want weten of een lege component correct gerenderd wordt is nogal lastig.

Om deze component te kunnen zien moet hij ergens in de `ReactDOM` gerenderd worden. De (enige) call naar `ReactDOM.render` gebeurt in de `main.jsx`:

```jsx
// src/main.jsx
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Standaard rendert deze de `App` component. Het `main.jsx` bestand ga je zelden zelf aanpassen. Je past normaal de `App` component aan.

[StrictMode](https://reactjs.org/docs/strict-mode.html) doet een aantal checks op alle (onderliggende) componenten. Zeker voor een onervaren React programmeur het een goed idee om altijd `StrictMode` aan te zetten.

### App.jsx

```jsx
// src/App.jsx
import { useState } from 'react';
import reactLogo from './assets/react.svg';
import viteLogo from '/vite.svg';
import './App.css';

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <div>
        <a href='https://vitejs.dev' target='_blank' rel='noreferrer'>
          <img src={viteLogo} className='logo' alt='Vite logo' />
        </a>
        <a href='https://react.dev' target='_blank' rel='noreferrer'>
          <img src={reactLogo} className='logo react' alt='React logo' />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className='card'>
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.jsx</code> and save to test HMR
        </p>
      </div>
      <p className='read-the-docs'>
        Click on the Vite and React logos to learn more
      </p>
    </>
  );
}

export default App;
```

In `App.jsx` staat de code voor de standaard startpagina: het Vite en React-logo, een link naar de documentatie, enz. Zoals eerder gezegd kan je ook CSS-klassen toevoegen aan HTML-elementen, maar hiervoor moet je het attribuut `className` gebruiken i.p.v. `class`. De `App` component heeft zijn eigen stijl in het bestand `App.css`, deze is enkel voor deze component. Globale stijlen definieer je in het bestand `index.css`.

JSX is strenger dan HTML. Je moet tags zoals `<img />` sluiten. Uw component kan ook niet meerdere JSX-tags retourneren. Je moet ze in een gedeelde bovenliggende parent plaatsen, zoals een `<div>...</div>` of een lege `<>...</>` wrapper.

Verwijder alle code uit deze component en vervang dit door de `Transaction` component. We maken ook geen gebruik meer van het CSS bestand. Verder in dit hoofdstuk voegen we voor de opmaak `Bootstrap` toe.

```jsx
// src/App.jsx
import Transaction from './components/transactions/Transaction';

function App() {
  return (
    <div className='App'>
      <Transaction />
    </div>
  );
}

export default App;
```

Als we nu naar [onze site](http://localhost:5173/) gaan, zouden we de hard gecodeerde string moeten zien.

### JSON

Een hard gecodeerde string als component tonen is niet erg nuttig, daar is React complete overkill voor. In dat geval schrijf je beter een HTML pagina met wat CSS zoals je het in Web Development I geleerd hebt.

Dit soort frameworks worden pas de moeite als we componenten gaan schrijven die data op een bepaalde manier tonen en manipuleerbaar maken. Met andere woorden: als componenten hergebruikt kunnen worden.

Het is ooit anders geweest, maar tegenwoordig is die data zo goed als altijd in JSON-formaat (JavaScript Object Notation). Heel kort gezegd is dit de voorstelling van een JavaScript-object. (of alleszins, lijkt het er heel sterk op)

```js
{
 "key": "some string",
 "otherKey": 15,
 "key3": true,
 "listOfData": ["a", 15],
 "otherObject": {
  "innerkey" : 42
 }
}
```

We zien dat JSON een comma-separated key-value lijst is. De keys zijn hierbij altijd strings. De values zijn strings, numbers, booleans, arrays of opnieuw objecten.

> Dit zou allemaal herhaling moeten zijn, als 't wat ver zit: [Working with JSON](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)

De data zal meestal ergens in een databank leven. Via één of andere API kunnen we deze data aanspreken en eventueel wijzigen. Dit leer je allemaal uitgebreid in het olod Web Services.

Uiteraard kunnen we niet alles tegelijk maken. Daarom gaan we eerst met *mock object* werken. We steken wat JSON data hard gecodeerd in een bestand. Vervolgens importeren en gebruiken we die data om onze componenten op te bouwen.

Later, als we een backend hebben, kunnen we dan makkelijk 'echte' data ophalen en tonen. Daarbij dienen we enkel die import te vervangen door een echte API call en hoeven we niet onze volledige component te herschrijven.

### Mock data

Maak een map `api` met een bestand `mock_data.js` aan in de `src` map. Later vervangen we deze mock data door API calls.

```js
// src/api/mock_data.js
const TRANSACTION_DATA = [
  {
    user: 'Benjamin',
    amount: -200,
    date: '2021-07-01T12:32:04.534Z',
    place: 'Dranken Geers',
  },
  {
    user: 'Benjamin',
    amount: 1500,
    date: '2021-06-30T10:09:22.534Z',
    place: 'Loon',
  },
];

export default TRANSACTION_DATA;
```

`TRANSACTION_DATA` is een array met twee objecten die een transactie voorstellen. We exporteren deze array zodat we deze in andere bestanden kunnen importeren en gebruiken.

### React props

We gaan de `Transaction` component aanpassen zodat hij data van verschillende transacties kan weergeven:

1. Definieer een variabele voor `user`, `amount` en `place`.
2. Vervang de hard gecodeerde info door de variabelen.

> Ter herinnering: als je JavaScript-code wil uitvoeren binnen JSX-code moet je deze tussen `{}` plaatsen.

```jsx
// src/components/transaction/Transaction.jsx
export default function Transaction() {
  const user = 'Benjamin'; // 👈 1
  const amount = 200; // 👈 1
  const place = 'Dranken Geers'; // 👈 1
  // 👇 2
  return (
    <div>
      {user} gaf €{amount} uit bij {place}
    </div>
  );
}
```

1. Definieer een variabele voor `user`, `amount` en `place`
2. Vervang de hardcoded info door de variabelen.

De data zal natuurlijk van een andere component moeten komen, nu hebben we nog steeds hard gecodeerde informatie. We passen dus aan:

```jsx
// src/components/transaction/Transaction.jsx
export default function Transaction(props) {
  // 👈 2
  // const user = "Benjamin"; 👈 1
  // const amount = 200; 👈 1
  // const place = "Dranken Geers"; 👈 1
  const { user, amount, place } = props; // 👈 3
  return (
    <div>
      {user} gaf €{amount} uit bij {place}
    </div>
  );
}
```

1. Verwijder de constanten met de hard gecodeerde data.
2. We geven de data door via de `props` parameter, de React properties. Op die manier kunnen we informatie doorgeven van de ene component aan de andere.
3. Om niet telkens `props.user`, `props.amount`... te moeten schrijven, maken we gebruik van **object destructuring**.

Hoe krijg je nu de juiste data in de `props` van een component? In `App.js` willen we:

1. drie variabelen `user`, `amount`, `place`.
2. die we doorgeven aan de `Transaction` component. Dit doet je op dezelfde manier als bij HTML: je voegt simpelweg attributen toe op een bepaalde tag.

```jsx
// src/App.jsx
import Transaction from './components/transactions/Transaction';

function App() {
  const user = 'Benjamin'; // 👈 1
  const amount = 200; // 👈 1
  const place = 'Dranken Geers'; // 👈 1
  return (
    <div className='App'>
      <Transaction user={user} place={place} amount={amount} /> {/* 👈 2 */}
    </div>
  );
}

export default App;
```

We willen natuurlijk dat hier de data van ons mock object komt.

1. Importeer de constante `TRANSACTION_DATA`.
2. Laat ons beginnen met gewoon het eerste element van de array eens te tonen.

```jsx
// src/App.jsx
import Transaction from './components/transactions/Transaction';
import TRANSACTION_DATA from './api/mock_data'; // 👈 1

function App() {
  const trans = TRANSACTION_DATA[0]; // 👈 2
  return (
    <div className='App'>
      {/* 👇 2 */}
      <Transaction
        user={trans.user}
        place={trans.place}
        amount={trans.amount}
      />
    </div>
  );
}

export default App;
```

Eigenlijk willen we voor elk van de elementen in de array `TRANSACTION_DATA` een `Transaction` component tonen.

1. Verwijder de lijn die de eerste transactie ophaalt.
2. We hier kunnen in de JSX-code ook onze array **mappen** waarbij we elk element omzetten naar een `Transaction` component.

```jsx
// src/App.jsx
import Transaction from './components/transactions/Transaction';
import TRANSACTION_DATA from './api/mock_data';

function App() {
  // const trans = TRANSACTION_DATA[0]; 👈 1
  return (
    <div className='App'>
      {/* 👇 2 */}
      {TRANSACTION_DATA.map((trans) => (
        <Transaction
          user={trans.user}
          place={trans.place}
          amount={trans.amount}
        />
      ))}
    </div>
  );
}

export default App;
```

Je kan ook gebruik maken van object destructuring om attributen te genereren. Elke key wordt dan een attribuut met de bijbehorende waarde. Het is niet altijd een goed idee maar het bespaart soms wel wat typwerk.

```jsx
import Transaction from './components/transactions/Transaction';
import TRANSACTION_DATA from './api/mock_data';

function App() {
  return (
    <div className='App'>
      {TRANSACTION_DATA.map((trans) => (
        <Transaction {...trans} />
      ))}
      {/* 👆 */}
    </div>
  );
}

export default App;
```

### Property 'key'

Alles lijkt te werken, maar als je de console opent zal je een error zien staan: `Each child in a list should have a unique "key" prop.`

Als je een lijst van elementen maakt, moet je altijd een `key` property voorzien (van het type `string`). Op basis van deze keys kan React weten welke elementen gewijzigd, toegevoegd of verwijderd zijn. Keys moeten uniek zijn binnen de array waarin de componenten gecreëerd worden, dus enkel t.o.v. de broer en zus elementen, niet over de hele applicatie.

Hoewel je altijd de index van een array kan gebruiken is dat zelden een goed idee. De key is het enige dat React gebruikt om DOM-elementen te identificeren. Dus stel dat je een element toevoegt op het einde van de lijst en halfweg een element verwijdert, dan zou bij een index-key React denken dat alle tussenliggende elementen dezelfde zijn!

Als je met 'echte' data werkt, die via een backend komt, heeft elk element heel vaak een unieke id (uit de databank). Dit id gebruik je dan ook om het element te identificeren in API calls. Dit vormt meteen een uitstekende key voor React-lijsten. In ons eenvoudig voorbeeld hebben we nog geen unieke id's, dus kunnen we voorlopig niet anders dan de index expliciet toevoegen als key om de error te zien verdwijnen. Als je niets meegeeft, gebruikt React onderliggend sowieso een index als key. Functioneel maakt het geen verschil of we het expliciet maken of niet. We komen hier later zeker op terug.

## CSS in React

In dit project maken we gebruik van [Bootstrap](https://getbootstrap.com/), een populair JavaScript- en CSS-framework. Bootstrap kan op verschillende manieren worden toegevoegd aan React. Laten we gebruik maken van Bootstrap CDN, de eenvoudigste manier om Bootstrap toe te voegen aan de React. Geen extra installatie of download is vereist. Een alternatief is om gebruik te maken van [react-bootstrap](https://www.npmjs.com/package/react-bootstrap).

Om Bootstrap op deze manier toe te voegen, voeg je een paar links toe aan de entry file van je applicatie. In een typische React applicatie gecreëerd met `create-vite` is dit het `index.html` bestand.

1. We linken naar de [huidige stabiele versie](https://getbootstrap.com/docs/versions/) van Bootstrap CSS.
2. Als je project ook gebruikt maakt van de JavaScript-componenten uit Bootstrap, zoals een modal venster, vervolgkeuzemenu of navigatiebalk moeten we het bestand `bootstrap.bundle.min.js` koppelen. Dat vooraf is gecompileerd met `Popper.js`. Voorzie hiervoor een script-tag die linkt naar de gebundelde Javascript CDN's vlak voor het sluiten van de body-tag. Meer info op [https://getbootstrap.com/docs/5.2/getting-started/introduction/](https://getbootstrap.com/docs/5.2/getting-started/introduction/).
3. Pas ook de `title` van de app aan.

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN"
      crossorigin="anonymous"
    />
    <!-- 👆 1 -->
    <title>BudgetApp</title>
    <!-- 👆 3 -->
  </head>
  <body>
    <div id="root"></div>
    <script
      src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"
      integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL"
      crossorigin="anonymous"
    ></script>
    <!-- 👆 2 -->
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

Gooi het bestand `App.css` weg en maak `index.css` leeg. We gaan de styling van Bootstrap gebruiken. Als je zelf toch iets wil aanpassen, kan je dit in het bestand `index.css` doen.

Pas nu de `Transaction` component aan en maak gebruik van de Bootstrap class `text_bg_dark` voor de `div` tag. Ook het `style` attribuut kan je binnen een JSX-bestand gebruiken. Hiervoor gebruik je een inline Javascript object. Vandaar de `{{}}` in onderstaand voorbeeld. Je kan ook de props meteen destructuren in de parameters van de functie.

```jsx
// src/components/transaction/Transaction.jsx
export default function Transaction({ user, amount, place }) {
  // 👈
  // 👇
  return (
    <div className='text-bg-dark' style={{ textAlign: 'center' }}>
      {user} gaf €{amount} uit bij {place}
    </div>
  );
}
```

## Oefening 1 - Je eigen project

Maak een nieuwe Vite React-applicatie aan met de naam van je project.

```bash
yarn create vite <PROJECTNAAM> --template react-swc
```

Maak een nieuwe GitHub repository aan via de GitHub classroom link in de introductie van de Chamilo-cursus. Clone jouw Git repository uit de GitHub classroom:

```bash
git clone <JOUW_GIT_REPOSITORY_URL>
```

Kopieer ***alle*** bestanden, ***behalve*** de `.eslintrc.cjs`, `.gitignore` en `README.md`, uit de Vite React-applicatie naar je eigen project. Maak ***geen*** submap voor de React-applicatie!

Vul alvast de `README.md` en `dossier.md` aan voor zover mogelijk:

- `README.md`: vul de titel en je naam, studentennummer en e-mailadres in.
- `dossier.md`: vul de titel in, duid aan welk(e) vak(ken) je volgt, en vul de link(s) naar de GitHub repository/repositories in.

Commit vervolgens de lege React-applicatie:

```bash
git add .
git commit -m "✨ Initial commit ✨"
git push
```

Experimenteer al een beetje met componenten, props, etc. en commit je wijzigingen. Je kan al een paar basiscomponent maken voor je eigen project.

> 💡 Tip: begin ***niet*** met het maken van een login- of registratiecomponent. Dat is niet belangrijk en kopieert iedereen toch van onze voorbeeldoplossing, steek er in het begin dus je tijd niet in. Het is niet zoveel werk om deze componenten in een latere fase toe te voegen.

## Oefening 2 - To do app

Creëer zelf een simpele to do app zoals in onderstaande screenshot. Gebruik een `TodoItem` component en render deze meerdere malen op basis van de data uit een lijst.

![To do app](/./images/todo-app-screenshot.png ':size=80%')

Een `TodoItem` heeft volgende props:

```plantuml
@startuml
interface TodoItemProps {
  text: string
  description: string
  done: boolean
}
note right of TodoItemProps::text
  De titel van het TodoItem
end note
note right of TodoItemProps::description
  Een uitgebreidere beschrijving van het TodoItem
end note
note left of TodoItemProps::done
  Geeft aan of het item al dan niet compleet is
end note
@enduml
```

<!-- markdownlint-disable-next-line -->
  + Oplossing +

  Een voorbeeldoplossing (maar er zijn er uiteraard heel veel mogelijk) is te vinden op <https://github.com/HOGENT-Web/frontendweb-ch1-solution>.

## Must read/watch

- [Don't Use JS for That: Moving Features to CSS and HTML by Kilian Valkhof](https://www.youtube.com/watch?v=IP_rtWEMR0o)
- [Practice React by fixing tests - Check your JSX knowledge!](https://reactpractice.dev/exercise/practice-react-by-fixing-tests-check-your-jsx-knowledge/)
- [Statements vs expressions](https://www.joshwcomeau.com/javascript/statements-vs-expressions/)
- [React.js: The Documentary](https://www.youtube.com/watch?v=8pDqJVdNa44)
- [JavaScript Visualized: Promise Execution](https://lydiahallie.framer.website/blog/promise-execution)
