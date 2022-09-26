# Webpack your bags, time to React

## Inleiding

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

Deze libraries importeer je simpelweg in de HTML:

```html
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
```

Vervolgens maak je één root voor de applicatie met een uniek `id`. Onder deze tag zal één React-applicatie alle DOM-manipulaties en magie uitvoeren. Het is dus perfect mogelijk om meerdere afzonderlijke React-applicaties in eenzelfde HTML-pagina te hebben, elk met een aparte root.

```html
<div id="greeting"></div>
```

Vervolgens kan je met `React.createElement` bepaalde elementen van de pagina aanmaken. `createElement` verwacht als eerste argument de naam van een HTML-tag, als tweede een object met properties om door te geven aan de HTML en als derde optioneel de inhoud van deze tag (m.a.w. eventuele kind-tags of simpelweg tekst).

Aangezien elke React-applicatie start vanaf één bepaalde root, moeten we eerst met `ReactDOM.createRoot` een root aanmaken. Deze functie verwacht het DOM-element waaronder de applicatie draait als enige argument. Vervolgens kan je op deze root de `render` functie aanroepen om een element te renderen onder deze root.

```html
<script>
  const root = ReactDOM.createRoot(document.getElementById('greeting'));
  root.render(
    React.createElement('div', null, 'hello world'),
  );
</script>
```

Dit geeft volgend resultaat:

<!-- tabs:start -->

### **Voorbeeld**

[example4.html](./examples/example4.html ':include height=100px')

### **Code**

[example4.html](./examples/example4.html ':include :type=code')

<!-- tabs:end -->

Wat is nu het nut van het tweede argument? Bekijk onderstaande code. Hierbij isoleren we de code van de begroeting in een functie genaamd `GreetingElement`. Vervolgens geven we deze functie door aan `createElement`. Dit geeft opnieuw hetzelfde resultaat als de vorige demo (test zelf maar uit).

```js
function GreetingElement() {
  return React.createElement(
    'div',
    null,
    'hello world'
  );
}

const root = ReactDOM.createRoot(
  document.getElementById('greeting')
);
root.render(
  React.createElement(GreetingElement),
);
```

Inspecteer het resultaat van dit voorbeeld via de DevTools van je browser.

<!-- tabs:start -->

### **Voorbeeld**

[example5.html](./examples/example5.html ':include height=100px')

### **Code**

[example5.html](./examples/example5.html ':include :type=code')

<!-- tabs:end -->

Het wordt pas echt spannend wanneer we de functie `GreetingElement` een argument meegeven, dit argument is **altijd een object**. Het bevat alle properties die van bovenaf doorgegeven worden. In de React-wereld worden dit de **props** van een component genoemd. In onderstaand voorbeeld krijgt de functie `GreetingElement` een prop met als naam `name` mee. Het zal deze `name` tonen in een span met id gelijk aan `name`.

> Merk op dat we hier meteen object destructuring toepassen op de props, m.a.w. we pakken meteen de name uit dit object

```js
function GreetingElement({ name }) {
  return React.createElement(
    'div',
    null,
    'hello ',
    React.createElement('span', { id: 'name' }, name)
  );
}

const root = ReactDOM.createRoot(document.getElementById('greeting'));
root.render(
  React.createElement(GreetingElement, { name: 'world' }),
);
```

Inspecteer het resultaat van dit voorbeeld via de DevTools van je browser.

<!-- tabs:start -->

### **Voorbeeld**

[example6.html](./examples/example6.html ':include height=100px')

### **Code**

[example6.html](./examples/example6.html ':include :type=code')

<!-- tabs:end -->

Bijgevolg is het ook eenvoudig om twee afzonderlijke React-applicaties te hebben op één pagina. Je maakt hiervoor twee verschillende roots en rendert een (verschillende) component in deze roots.

Inspecteer de werking hiervan a.d.h.v. het volgend voorbeeld.

<!-- tabs:start -->

### **Voorbeeld**

[example7.html](./examples/example7.html ':include height=100px')

### **Code**

[example7.html](./examples/example7.html ':include :type=code')

<!-- tabs:end -->

Je vraagt je nu waarschijnlijk af: "Is dit echt beter dan vanilla JavaScript?". Het antwoord is dat niemand React op deze manier gebruikt. Maar het is wel belangrijk om te beseffen dat deze acties weldegelijk onderliggend gebeuren. In de volgende sectie gaan we een stap verder richting wat React eigenlijk wel is.

## JSX

Als we enkel React zouden kunnen schrijven door immens, nauwelijks leesbare, boomstructuren van `createElement` te creëren hadden we waarschijnlijk nooit van React gehoord. Samen met React heeft Facebook ook JSX geïntroduceerd (een samentrekking van JavaScipt en XML). Hiermee is het mogelijk om veel efficiënter (en leesbaarder) zulke componenten uit te schrijven.

![JSX example](./images/jsx.png)

Hier heb je een simpel JavaScript-bestand, maar waar je normaal ingewikkelde `React.createElement` structuren hebt om HTML te manipuleren, schrijf je gewoon iets wat erg op HTML lijkt om dat te doen.

Het 'HTML' stuk van JSX voelt echt vertrouwd als je HTML kent (en dat is natuurlijk de bedoeling), maar er zijn een aantal dingen waar je moet op letten:

- `class` is een reserved keyword in JavaScript, en kan dus niet gebruikt worden om een CSS class mee te geven aan een element, het wordt vervangen door `className`
- `for` is ook een reserved keyword in JavaScript, het wordt vervangen door `htmlFor`
- Als je JavaScript code wilt mixen met een stuk 'HTML', dien je het tussen `{ }` te zetten, bijvoorbeeld `<h1>{title}</h1>`.

Wanneer we het stuk code van het `GreetingElement` omzetten naar JSX, krijgen we het volgende resultaat:

```jsx
function GreetingElement({ name }) {
  return <div>hello <span id="name">{name}</span></div>;
}
```

`{name}` zorgt ervoor dat de waarde van de variable `name` gerenderd wordt. Met `{ }` kan je eender welke expressie in JavaScript uitvoeren in de HTML, je kan hier geen statements gebruiken. De uitvoer van deze code zal gerenderd worden in de HTML.

> Geen idee wat het verschil is tussen een statement of expression? Check dat eens de [Must read](#must-read) onderaan deze pagina.

Browsers kunnen natuurlijk geen JSX renderen, de code moet eerst omgezet worden door een compiler (net zoals Java code niet rechtstreeks door een processor kan uitgevoerd worden). Babel is een compiler die als een stuk JavaScript in de browser kan geladen worden en vervolgens JSX vertaalt. Babel werd oorspronkelijk gecreëerd om moderne JavaScript te kunnen draaien in oudere browsers. Om babel te gebruiken, voeg je onderstaand script toe aan de HTML en voeg je `type="text/babel"` toe aan de `script` tag met de JSX-code.

```html
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
```

Dit geeft het volgende resultaat:

<!-- tabs:start -->

### **Voorbeeld**

[example8.html](./examples/example8.html ':include height=100px')

### **Code**

[example8.html](./examples/example8.html ':include :type=code')

<!-- tabs:end -->

## webpack

Onze React-applicaties gaan natuurlijk liefst niet bestaan uit een paar grote HTML-bestanden doorspekt met `script` blokken met daarin JSX. Aangezien er toch een compilatiestap is, om de JSX om te zetten naar HTML + JavaScript, kunnen we even goed gebruik maken van deze stap om ook 'andere dingen' te doen. Deze 'andere dingen' zijn bijvoorbeeld bestanden samen nemen, afbeeldingen en CSS optimaliseren, dependencies beheren, etc.

Er bestaan tegenwoordig heel wat tools die deze taken op zich nemen, één van de populairdere is [webpack](https://webpack.js.org/). Door `webpack` te gebruiken kunnen we code in verschillende bestanden en mappen structureren, dependencies beheren, minification toepassen, verschillende debug en production builds maken, etc. Allemaal op een manier zoals je gewoon bent als je Java code schrijft.

Net zoals we React zelf opgezet hebben kan je voor `webpack` alles downloaden en de juiste configuratiebestanden aanmaken en zo alles weer manueel opzetten en werkend krijgen. Maar `webpack` heeft ontzettend véél [configuratie](https://webpack.js.org/configuration/) opties (en wordt trouwens ook voor veel andere libraries en frameworks gebruikt). Het volledig leren gebruiken is waarschijnlijk een cursus op zich.

Gelukkig bestaat er een handige CLI tool die dit allemaal voor ons doet en degelijke defaults configureert voor React: [create-react-app (CRA)](https://reactjs.org/docs/create-a-new-react-app.html).

## create-react-app

Het is eenvoudig om een nieuwe React-applicatie te maken m.b.v. [create-react-app](https://reactjs.org/docs/create-a-new-react-app.html). Hiervoor maken we gebruik van `npx`, de `npm package runner`. `npx` zorgt ervoor dat je niet langer CLI packages lokaal moet installeren om ze te gebruiken. Dit heeft als voordeel dat we onze global npm scope niet vervuilen met CLI tools die we mogelijks maar één keer gebruiken.

Een nieuwe React-applicatie maken is zo simpel als:

```bash
npx create-react-app budget
```

> Verwijder de npx cache met `npx clear-cpx-cache` indien je volgende warning krijgt: `You are running 'create-react-app' 4.x.x, which is behind the latest release (5.x.x).`

Dit commando maakt een map `budget` met alle bestanden voor deze React-applicatie. We gaan doorheen deze cursus een budgetapplicatie ontwikkelen, we bouwen steeds verder op deze startapplicatie.

Deze map bevat onder andere volgende bestanden/mappen:

- `node_modules`: deze map bevat alle dependencies van de applicatie, m.a.w. de React libraries en alle libraries waar die dan weer op steunen. Dit is typisch een map met immens veel heel kleine bestanden (bij het maken van deze cursus: 40.020 (!) bestanden die 148 MB innemen)
- `package.json`: bestand dat beschrijft welke dependencies we nodig hebben, hoe de applicatie moet gestart, getest... worden, etc
- `public`: map die alles bevat wat publiek beschikbaar zal zijn voor onze webapplicaties (bv. afbeeldingen...)
- `src`: map die alle broncode bevat waarmee onze applicaties gebouwd gaat worden, dus allemaal JSX- en CSS-bestanden, etc.
- er werd ook automatisch een git repository toegevoegd, met een relevante `.gitignore`

> Merk op: de React-community raadt het gebruik van CRA (en react-scripts) meer en meer af wegens o.a. het onbreken van de mogelijkheid om eenvoudig configuratie van webpack aan te passen. Er zijn reeds meer hedendaagse compilers zoals bv. [Vite](https://vitejs.dev/). Durf jij de switch aan?

### yarn

[yarn](https://yarnpkg.com/) is het programma dat alle dependencies zal installeren, een andere misschien iets gekendere is [npm](https://www.npmjs.com/package/npm). Ze doen beide hetzelfde en zijn inwisselbaar maar de ene keer `yarn` gebruiken en de andere keer `npm` is dan weer geen goed idee. Ze kunnen andere versies van packages cachen e.d. en dan kan je rare fouten tegenkomen.

### package.json

De [package.json](https://docs.npmjs.com/cli/v8/configuring-npm/package-json) bevat alle metadata van ons project, meer in het bijzonder alle dependencies en commando's om onze app te starten. Een voorbeeld van een `package.json` is:

[example8.html](./examples/package.json ':include :type=code')

De `package.json` bevat enkele properties:

- `dependencies`: de packages waarvan deze applicatie gebruik maakt
- `devDependencies`: packages enkel nodig in development (en dus niet in productie)
- `scripts`: laten toe om een soort van shortcuts te maken voor scripts (bv. de applicatie starten, testen, builden voor productie, etc.)

Met een simpele `yarn install` installeren we meteen een identieke omgeving (met zowel `dependencies` als `devDependencies`) en dat maakt het handiger om in een team te werken (`yarn install --prod` installeert enkel de `dependencies`).

Het verschil tussen `dependencies` en `devDependencies` is het moment wanneer ze gebruikt worden. De `dependencies` zijn nodig in productie, m.a.w. de applicatie kan niet werken zonder deze packages. De `devDependencies` zijn enkel nodig om bv. het leven van de developer makkelijker te maken (types in TypeScript, linting, etc.) of bevatten packages die enkel gebruikt worden *at build time*, of dus wanneer de applicatie (door webpack) omgevormd wordt tot iets wat browsers begrijpen.

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

Bij een `create-react-app` wordt er steeds automatisch een `yarn install` uitgevoerd, dus die hoeven we niet meer te doen. Met een `yarn start` zien we dan onze (default, lege) React-applicatie. Standaard wordt deze op poort 3000 gestart: <http://localhost:3000>.

### src

De src map bevat een aantal JSX-bestanden (`index.js`, `App.js`...) en wat CSS, e.d. `webpack` zet dit om naar HTML en (door de browser begrijpbare) JavaScript. Dit gebeurt automatisch als een van de bronbestanden wijzigt.

Probeer maar iets aan te passen in de `App.js`. Je zal zien dat de brower automatisch herlaadt met de nieuwe inhoud (uiteraard als je geen compilatiefouten veroorzaakt).

> **Best practice**: het is beter om bestanden met JSX de extensie `.jsx` te geven, dit brengt o.a. betere IntelliSense met zich mee (in bv. VS Code).

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

Om deze component te kunnen zien moet hij ergens in de `ReactDOM` gerenderd worden. De (enige) call naar `ReactDOM.render` gebeurt in de `index.js`:

```jsx
// src/index.js
ReactDOM.render(
 <React.StrictMode>
  <App />
 </React.StrictMode>,
 document.getElementById('root')
);
```

Standaard rendert deze de `App` component. Het `index.js` bestand ga je zelden zelf aanpassen. Je past normaal de `App` component aan.

[StrictMode](https://reactjs.org/docs/strict-mode.html) doet een aantal checks op alle (onderliggende) componenten. Zeker voor een onervaren React programmeur het een goed idee om altijd `StrictMode` aan te zetten.

### App.js

```jsx
// src/App.js
import logo from './logo.svg';
import './App.css';

function App() {
 return (
  <div className="App">
   <header className="App-header">
    <img src={logo} className="App-logo" alt="logo" />
    <p>
     Edit <code>src/App.js</code> and save to reload.
    </p>
    <a
     className="App-link"
     href="https://reactjs.org"
     target="_blank"
     rel="noopener noreferrer"
    >
     Learn React
    </a>
   </header>
  </div>
 );
}

export default App;
```

In `App.js` staat de code voor de standaard startpagina: het React-logo, een link naar de documentatie, enz. Zoals eerder gezegd kan je ook CSS-klassen toevoegen aan HTML-elementen, maar hiervoor moet je het attribuut `className` gebruiken i.p.v. `class`. De `App` component heeft zijn eigen stijl in het bestand `App.css`, deze is enkel voor deze component. Globale stijlen definieer je in het bestand `index.css`.

JSX is strenger dan HTML. Je moet tags zoals `<img />` sluiten. Uw component kan ook niet meerdere JSX-tags retourneren. Je moet ze in een gedeelde bovenliggende parent plaatsen, zoals een `<div>...</div>` of een lege `<>...</>` wrapper.

Verwijder alle code uit deze component en vervang dit door de `Transaction` component. We maken ook geen gebruik meer van het CSS bestand. Verder in dit hoofdstuk voegen we voor de opmaak `Bootstrap` toe.

```jsx
// src/App.js
import Transaction from './components/transactions/Transaction'; 

function App() {
 return (
  <div className="App">
   <Transaction />
  </div>
 );
}

export default App;
```

Als we nu naar [onze site](http://localhost:3000/) gaan, zouden we de hard gecodeerde string moeten zien.

### JSON

Een hard gecodeerde string als component tonen is niet erg nuttig, daar is React complete overkill voor. In dat geval schrijf je beter een HTML pagina met wat CSS zoals je het in Webapplications I geleerd hebt.

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

> Dit zou allemaal herhaling moeten zijn, als 't wat ver zit: [Working with json](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)

De data zal meestal ergens in een databank leven. Via één of andere API kunnen we deze data aanspreken en eventueel wijzigen. Dit leer je allemaal uitgebreid in het vak Web Services.

Uiteraard kunnen we niet alles tegelijk maken. Daarom gaan we eerste met *mock object* werken. We steken wat JSON data hard gecodeerd in een bestand. Vervolgens importeren en gebruiken we die data om onze componenten op te bouwen.

Later, als we een backend hebben, kunnen we dan makkelijk 'echte' data ophalen en tonen. Daarbij dienen we enkel die import te vervangen door een echte API call en hoeven we niet onze volledige component te herschrijven.

### Mock data

Maak een map `api` met een bestand `mock-data.js` aan in de `src` map. Later vervangen we deze mock data door API calls.

```js
// src/api/mock-data.js
const TRANSACTION_DATA = [{
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
}];
 
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
 const user = "Benjamin"; // 👈 1
 const amount = 200; // 👈 1
 const place = "Dranken Geers"; // 👈 1
 return <div>{user} gaf €{amount} uit bij {place}</div>; // 👈 2
}
```

1. Definieer een variabele voor `user`, `amount` en `place`
2. Vervang de hardcoded info door de variabelen.  
Ter herinnering: als je JavaScript code wilt uitvoeren binnen een `jsx` stuk: tussen `{}` plaatsen.

De data zal natuurlijk van een andere component moeten komen, nu hebben we nog steeds hard gecodeerde informatie. We passen dus aan:

```jsx
// src/components/transaction/Transaction.jsx
export default function Transaction(props) { // 👈 2
 // const user = "Benjamin"; 👈 1
 // const amount = 200; 👈 1
 // const place = "Dranken Geers"; 👈 1
 const { user, amount, place} = props; // 👈 3
 return <div>{user} gaf €{amount} uit bij {place}</div>;
}
```

1. Verwijder de constanten met de hard gecodeerde data.
2. We geven de data door via de `props` parameter, de React properties. Op die manier kunnen we informatie doorgeven van de ene component aan de andere.
3. Om niet telkens `props.user`, `props.amount`... te moeten schrijven, maken we gebruik van **object destructuring**.

Hoe krijg je nu de juiste data in de `props` van een component? In `App.js` willen we:

1. drie variabelen `user`, `amount`, `place`.
2. die we doorgeven aan de `Transaction` component. Dit doet je op dezelfde manier als bij HTML: je voegt simpelweg attributen toe op een bepaalde tag.

```jsx
// src/App.js
import Transaction from './components/transactions/Transaction';

function App() {
 const user = "Benjamin"; // 👈 1
 const amount = 200; // 👈 1
 const place = "Dranken Geers"; // 👈 1
 return (
  <div className="App">
   <Transaction user={user} place={place} amount={amount}/> { /* 👈 2 */}
  </div>
 );
}

export default App;
```

We willen natuurlijk dat hier de data van ons mock object komt.

1. Importeer de constante `TRANSACTION_DATA`.
2. Laat ons beginnen met gewoon het eerste element van de array eens te tonen.

```jsx
// src/App.js
import Transaction from './components/transactions/Transaction';
import TRANSACTION_DATA from './mock-data'; // 👈 1

function App() {
 const trans = TRANSACTION_DATA[0]; // 👈 2
 return (
  <div className="App">
   <Transaction user={trans.user} place={trans.place} amount={trans.amount}/> { /* 👈 2 */}
  </div>
 );
}

export default App;
```

Eigenlijk willen we voor elk van de elementen in de array `TRANSACTION_DATA` een `Transaction` component tonen.

1. Verwijder de lijn die de eerste transactie ophaalt.
2. We hier kunnen in de JSX-code ook onze array **mapppen** waarbij we elk element omzetten naar een `Transaction` component.

```jsx
// src/App.js
import Transaction from './components/transactions/Transaction';
import TRANSACTION_DATA from './api/mock-data'; 

function App() {
 //c onst trans = TRANSACTION_DATA[0]; 👈 1
 return (
  <div className="App">
  {TRANSACTION_DATA.map(trans => 
   <Transaction user={trans.user} place={trans.place} amount={trans.amount}/> )}{ /* 👈 2 */}
  </div>
 );
}

export default App;
```

Je kan ook gebruik maken van object destructuring om attributen te genereren. Elke key wordt dan een attribuut met de bijbehorende waarde. Het is niet altijd een goed idee maar het bespaart soms wel wat typwerk.

```jsx
import Transaction from './components/transactions/Transaction';
import TRANSACTION_DATA from './api/mock-data'; 

function App() {
 return (
  <div className="App">
  {TRANSACTION_DATA.map(trans => 
   <Transaction {...trans}/>)}{ /* 👈 */}
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

Om Bootstrap op deze manier toe te voegen, voeg je een paar links toe aan de entry file van je applicatie. In een typische React applicatie gecreëerd met `create-react-app` is dit het `public/index.html` bestand.

1. We linken naar de [huidige stabiele versie](https://getbootstrap.com/docs/versions/) van Bootstrap CSS
2. Als je project ook gebruikt maakt van de JavaScript-componenten uit Bootstrap, zoals een modal venster, vervolgkeuzemenu of navigatiebalk moeten we het bestand `bootstrap.bundle.min.js` koppelen. Dat vooraf is gecompileerd met `Popper.js`. Voorzie hiervoor een script-tag die linkt naar de gebundelde Javascript CDN's vlak voor het sluiten van de body-tag. Meer info op [https://getbootstrap.com/docs/5.2/getting-started/introduction/](https://getbootstrap.com/docs/5.2/getting-started/introduction/)
3. Pas ook de title van de app aan

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <meta name="theme-color" content="#000000" />
  <meta name="description" content="Web site created using create-react-app" />
  <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
  <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
  <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
  <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
  <!--
      Notice the use of %PUBLIC_URL% in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
  <link
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/css/bootstrap.min.css"
  rel="stylesheet"
  integrity="sha384-iYQeCzEYFbKjA/T2uDLTpkwGzCiq6soy8tYaI1GyVh/UjpbCx/TYkiZhlZB6+fzT"
  crossorigin="anonymous"> <!-- 👈 1 -->
  <title>BudgetApp</title><!-- 👈 3 -->
</head>

<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root"></div>
  <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
    <script
   src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/js/bootstrap.bundle.min.js"
   integrity="sha384-u1OknCvxWvY5kfmNBILK2hRnQC3Pr17a+RTT6rIHI7NnikvbZlHgTPOOmMi466C8"
   crossorigin="anonymous"></script> <!-- 👈 2 -->
</body>

</html>
```

Pas nu de `Transaction` component aan en maak gebruik van de Bootstrap class `text_bg_dark` voor de `div` tag. Ook het `style` attribuut kan je binnen een JSX-bestand gebruiken. Hiervoor gebruik je een inline Javascript object. Vandaar de `{{}}` in onderstaand voorbeeld:

```jsx
// src/components/transaction/Transaction.jsx
export default function Transaction(props) { 
  const { user, amount, place} = props; 
  return <div className="text-bg-dark" style={{width:'50%'}}>{user} gaf €{amount} uit bij {place}</div>; // 👈
}
```

## Oefening

Creëer zelf een simpele to do app zoals in onderstaande screenshot. Gebruik een `TodoItem` component en render deze meerdere malen op basis van de data uit een lijst.

![To do app](/./images/todo-app-screenshot.png ':size=50%')

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

### Oplossing

Een voorbeeldoplossing (maar er zijn er uiteraard heel veel mogelijk) is te vinden op <https://github.com/HOGENT-Web/frontendweb-ch1-solution>.

> Uiteraard zijn heel veel oplossingen mogelijk.

## Mogelijke extra's examenopdracht

- [Switch van CRA (create-react-app) naar Vite](https://medium.com/codex/you-should-choose-vite-over-cra-for-react-apps-heres-why-47e2e7381d13)

## Must read

- [Statements vs expressions](https://www.joshwcomeau.com/javascript/statements-vs-expressions/)
