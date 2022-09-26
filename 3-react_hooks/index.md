# Get your hooks into React

## Inleiding

**Componenten** zijn functies die de UI renderen. Het renderen gebeurt wanneer de app voor het eerst geladen wordt en wanneer state waarden wijzigen. Renderen van code moet "[puur](https://beta.reactjs.org/learn/keeping-components-pure)" zijn. Componenten mogen enkel 'hun' JSX retourneren en mogen objecten of variabelen die bestaan voor de rendering niet wijzigen (bv. geen nieuwe waarde toekennen aan props). Gegeven dezelfde input, dient het dezelfde output te retourneren. Net als een wiskundige formule zou het alleen het resultaat moeten berekenen, maar niets anders doen.

**Events** zijn functies binnen de component die worden uitgevoerd als reactie op een actie van een gebruiker. Een event handler kan state aanpassen, een HTTP post request uitvoeren om een transactie toe te voegen... Event handlers bevatten **side-effects** veroorzaakt door een interactie.

Soms is dit niet voldoende. Het ophalen van de transacties vanuit een REST API, dient sowieso te gebeuren en is niet gekoppeld aan een interactie van een gebruiker en mag ook niet gebeuren tijdens de rendering (anders is de component niet puur). Hiervoor kunnen we gebruik maken van de `useEffect` hook.

Effecten worden meestal gebruikt om uit je React-code te stappen en te synchroniseren met een extern systeem. Dit omvat browser API's, widgets van derden, netwerken... Ze voeren een **side-effect** uit.

## useEffect hook

`useEffect` is een functie die asynchroon wordt uitgevoerd na de render en die zichzelf optioneel kan opruimen (= **cleanup**) Het opruimen gebeurt voordat het effect opnieuw wordt uitgevoerd en voor de **unmounting** (= het vernietigen van de component). React onthoudt de functie die je hebt doorgegeven ​​(we noemen dit ons "effect") en roept het later op na het uitvoeren van de DOM-updates.

In onderstaand voorbeeld wordt er een boodschap naar de console gelogd als de `TransactionList` gerenderd is. Deze instructie zouden we na de return kunnen plaatsen, maar die code wordt niet uitgevoerd. `useEffect` is hier de oplossing.

```jsx
import {useEffect, useState} from 'react'; // 👈 1
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import {TRANSACTION_DATA} from '../../api/mock-data'; 

export default function TransactionList() {
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);

  // 👇 2
  useEffect(() => {
    console.log("transactions are rendered");
  });

  const createTransaction = (user, place, amount, date) => {
    const newTransactions = [
      {
        user, place, amount, date: new Date(date),
      },
      ...transactions,
    ]; // newest first
    setTransactions(newTransactions);
    console.log("transactions", JSON.stringify(transactions));
    console.log("newtransactions", JSON.stringify(newTransactions));
  };

  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={createTransaction} />
      {transactions.map((trans, index) => 
        <Transaction {...trans} key={index} /> )}
    </>);
  };
```

1. We maken hiervoor gebruik van `useEffect`.
2. Binnen de component roepen we de `useEffect` functie aan. We geven een **callback functie** mee als parameter. De functie die we meegeven wordt het `effect` genoemd. Wanneer React onze component rendert, onthoudt React het effect dat we hebben gedefinieerd en voert het het effect uit na het updaten van de DOM. Dit gebeurt default na elke render, ook na de eerste.

Start de app en bekijk de console. Voeg een transactie toe. We zien in de console dat `useEffect` na de initiële render en bij elke rerender wordt uitgevoerd.

> **Merk op:** React StrictMode (zie `index.js`) controleert of een component pure is door de component functie tweemaal aan te roepen. Dit gebeurt enkel in development mode, niet in productie. Dit is ook de reden waarom het loggen naar de console tweemaal gebeurt. Zie [Detecting impure calculations with StrictMode](https://beta.reactjs.org/learn/keeping-components-pure) en [Why does my calculation runs twice](https://beta.reactjs.org/apis/react/useMemo#my-calculation-runs-twice-on-every-re-render).

### Oefening

Voeg dezelfde code toe aan de TransactionForm component. Wat wordt er eerst gerenderd?

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  De `TransactionForm` component wordt als eerste gerenderd en dan pas de `TransactionList`. Dit is logisch aangezien React eerst de kinderen rendert en zo omhoog beweegt tot aan de component die de render veroorzaakte.

### Effect dependencies

Stel dat we de boodschap enkel bij de initïele render wensen te loggen.

Aan de hand van een **dependency array** kan je het uitvoeren van een `useEffect` koppelen aan specifieke datawijzigingen. Zo voorkom je dat `useEffect` bij elke rerender opnieuw wordt uitgevoerd. Enkele voorbeelden:

- **[]**: een lege dependency array. `useEffect` wordt enkel bij de initiële render uitgevoerd

```jsx
useEffect(() => {
  console.log("transactions after the initial render");
}, [])
```

- **[transactions]**: `useEffect` wordt bij de initiele render en telkens de waarde van de variabele `transactions` wijzigt uitgevoerd. React zal het effect overslaan als `transactions` dezelfde waarde heeft als tijdens de laatste render.

```jsx
useEffect(() => {
  console.log("transactions after initial render or transaction added");
}, [transactions])
```

- **meerdere dependencies**: React zal het opnieuw uitvoeren van het effect alleen overslaan als alle dependencies die je opgeeft exact dezelfde waarden hebben als tijdens de vorige render.

Stel dat de user via een prop wordt doorgegeven aan de `TransactionList` en ook naar de console gelogd dient te worden.

```jsx
export default function TransactionList({ user = 'Louis' }){ // 👈 de prop user
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);

  useEffect(() => {
      console.log(`Hi  ${user}, transactions after initial render or transaction added`); // 👈 maakt gebruik van user
  }, [transactions])// Warning: React Hook useEffect has a missing dependency

  //...
}
```

Merk op: je krijgt een waarschuwing van de linter (`React Hook useEffect has a missing dependency`) als de dependencies die je hebt opgegeven niet overeenkomen met wat ESLint (= een linter, zie een later hoofdstuk) verwacht op basis van de code in je effect. Dit helpt veel bugs in de code op te sporen. Als je effect een bepaalde waarde gebruikt, maar je het effect niet opnieuw wilt uitvoeren wanneer deze verandert moet je ervoor zorgen dat je effect geen gebruik maakt van deze dependency.

Oplossing:

```jsx
export default function TransactionList({ user = 'Louis' }) {
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);

  useEffect(() => {
    console.log(`Hi ${user}, transactions after initial render or transaction added`);
  }, [transactions, user]) // 👈 meerdere dependencies

  // ..
```

React vergelijkt de dependency waarden met behulp van [Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Voor arrays en objecten wordt hier bijgevolg gekeken naar de referentie!

### Clean up, indien nodig

Een side-effect kan een **clean-up functie** retourneren. React roept de clean-up functie elke keer aan voordat het effect opnieuw wordt uitgevoerd en een laatste keer wanneer de component wordt verwijderd (= on unmount).

Verwijder eerst de code m.b.t. de prop user en voeg dan een clean-up functie met een simpele `console.log` toe.

```jsx
export default function TransactionList() {
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);

  useEffect(() => {
    console.log("transactions after initial render or transaction added");
    return () => console.log("unmounted..."); // 👈 de clean-up functie
  }, [transactions]);
```

Start de app en bekijk de console.

### Simulatie ophalen data uit REST API

We simuleren nu het ophalen van de transacties via de REST API. We maken hiervoor gebruik van de `setTimeout` functie.

```jsx
const [transactions, setTransactions] = useState([]); // 👈 1

useEffect(() => { // 👈 2
  setTimeout(() => {
    setTransactions(TRANSACTION_DATA);
    console.log("transactions fetched");
  }, 2000);
}, [])
```

1. Initieel is de lijst van transacties leeg
2. We halen de data op, eens ontvangen wordt de state aangepast waardoor de pagina opnieuw wordt gerenderd. Het ophalen van de data mag enkel gebeuren bij de initiële render. Waarom?

<!-- markdownlint-disable-next-line -->
+ Antwoord +
  
  We hoeven de data niet telkens opnieuw op te halen als de component rendert, de data is dan niet noodzakelijk gewijzigd. Zonder de lege dependency array worden alle transacties opnieuw opgehaald bij elke render.

In het volgende hoofdstuk zien we hoe we een request kunnen sturen naar een REST API, een loading indicator kunnen tonen en eventuele fouten kunnen weergeven.

### Regels voor hooks

Hooks zijn niet meer dan JavaScript functies. Echter moet je twee regels volgen wanneer je er gebruik van maakt. Je kan hiervoor een linter plugin installeren, dit gebeurt automatisch bij het gebruik van `create-react-app`.

- Gebruik hooks enkel op het top niveau. Gebruik hooks niet binnen een if, andere condities, loops of geneste functies.
  - Reden: React valt terug op de volgorde waarin hooks worden aangeroepen om een waarde terug te geven. React houdt dit bij in een array. De volgorde moet dezelfde zijn bij elke render. Benieuwd naar meer info? Lees verder in [The First Rule of React Hooks, In Plain English](https://itnext.io/the-first-rule-of-react-hooks-in-plain-english-1e0d5ae32009)
- Roep hooks enkel aan vanuit React functies. Dit wil zeggen: enkel vanuit function components of vanuit custom hooks.

Daarnaast zijn er nog enkele opmerkingen waarmee je rekening moet houden:

- Gebruik `useEffect` niet voor het aanbrengen van DOM-wijzigingen die zichtbaar zijn voor de gebruiker. Een `useEffect` wordt pas geactiveerd nadat de browser klaar is met de lay-out en het tekenen. Dit is dus te laat als je een visuele wijziging wilde aanbrengen. Voor die gevallen biedt React de hooks `useMutationEffect` en `useLayoutEffect` die op dezelfde manier werken als `useEffect`. Ze verschillen enkel in het moment van 'afvuren'.
- Hooks maken gebruik van closures. Let dus op voor stale closures. [Zie hier voor enkele voorbeelden](https://dmitripavlutin.com/react-hooks-stale-closures/)
- `useEffect` laat NIET toe om het keyword `async` toe te voegen in de callback function, zie een volgend hoofdstuk voor een oplossing

Voor extra uitleg en voorbeelden: [Synchronizing with Effects](https://beta.reactjs.org/learn/synchronizing-with-effects)

## Verbeteren van de performantie

In een React-toepassing worden componenten heel vaak gerenderd. De performantie kan je verbeteren door het voorkomen van onnodige renders en het verminderen van de tijd die een render in beslag neemt.

Een oplossing voor dit probleem is **memoization**. Wikipedia geeft hiervoor volgende definitie:

> In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again.

React biedt een paar vormen van memoization:

- `memo`: creatie van pure componenten (let op: dit is **geen** hook)
- `useMemo`: retourneert een memoized **waarde**
- `useCallback`: retourneert een memoized **functie**

### useMemo hook

`useMemo` is een React Hook waarmee je het resultaat van een berekening tussen re-renders kan cachen.

In onderstaand voorbeeld voegen we een zoekfunctie toe om de transacties te filteren o.b.v. de plaats. Voeg deze code toe aan de `TransactionList` component:

```jsx
import {useState} from 'react';
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import {TRANSACTION_DATA} from '../../api/mock-data'; 

export default function TransactionList() {
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);
  const [text, setText] = useState(''); // 👈 1
  const [search, setSearch] = useState(''); // 👈 1

  // 👇 2
  const filteredTransactions = transactions.filter((t) => {
    console.log("filtering...");
    return t.place.toLowerCase().includes(search.toLowerCase());
  });

  const createTransaction = (user, place, amount, date) => {
    const newTransactions = [
      {
        user, place, amount, date: new Date(date),
      },
      ...transactions,
    ]; // newest first
    setTransactions(newTransactions);
  };

  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={createTransaction} />

      <div className="input-group mb-3 w-50"> {/* 👈 3 */}
        <input
          type="search"
          id="search"
          className="form-control rounded"
          placeholder="Search"
          value={text}
          onChange={(e) => setText(e.target.value)} />
        <button
          type="button"
          className="btn btn-outline-primary"
          onClick={() => setSearch(text)}
        >
          Search
        </button>
      </div>

      {filteredTransactions.map((trans, index) => // 👈 4
        <Transaction {...trans} key={index} /> )}
    </>);
  };
```

1. De filtering mag enkel gebeuren als de gebruiker op search klikt, niet bij ingave van een letter in het zoekveld (vandaar de twee state variabelen).
2. We maken een functie voor het filteren van de transacties.
3. We voegen een formulier met zoekveld en -knop toe.
4. We tonen enkel de gefilterde transacties.

Bij elk karakter dat de gebruiker ingeeft in het zoekveld wordt de state aangepast, de component opnieuw gerenderd, de filter-functie uitgevoerd. Hoewel de output ongewijzigd blijft tot we op de knop klikken en effectief zoeken. Dit kan je oplossen door gebruik te maken van `useMemo`. Hiermee kan React de returnwaarde van de zoekfunctie onthouden en zal het deze functie enkel en alleen uitvoeren als de dependencies gewijzigd zijn. In onderstaand voorbeeld wordt de filter pas uitgevoerd bij het laden van de component en bij het klikken op `Search`.

```jsx
import {useState, useMemo} from 'react'; // 👈

//...

const filteredTransactions = useMemo(() => transactions.filter((t) =>{ // 👈
  console.log("filtering...");
  return t.place.toLowerCase().includes(search.toLowerCase());
}),[search, transactions]);

//...
```

Je moet twee parameters doorgeven aan de `useMemo` functie:

1. een **calculation function** die het resultaat van de berekening retourneert. Het resultaat van die functie wordt bijgehouden in de cache, **niet** de functie zelf.
2. een **dependency array** die elke waarde bevat waarnaar verwezen wordt in de calculation function.

Bij elke volgende render vergelijkt React de dependencies met de dependencies die je tijdens de laatste render hebt doorgegeven. Als geen van de dependencies is gewijzigd, retourneert `useMemo` de waarde die al eerder werd berekend. Anders zal React de berekening opnieuw uitvoeren en de nieuwe waarde retourneren.

#### Oefening

Probeer de challenges op [https://beta.reactjs.org/learn/keeping-components-pure](https://beta.reactjs.org/learn/keeping-components-pure).

### React.memo en pure functions

Voeg een `console.log` instructie toe voor elke `return` in onderstaande componenten:

```jsx
export default function TransactionList() {
  ...
  console.log('Rendering transactionlist...');
  return (...);
}

export default function Transaction(props) {
  ...
  console.log('Rendering transaction...');
  return (...);
}

export default function TransactionForm({places, onRate}) {
  ...
  console.log('Rendering TransactionForm ...');
  return (...);
} 
```

Telkens als we een letter ingeven in het zoekveld worden alle componenten gererenderd, hoewel er niets wijzigt aan de output van de component. De `Transaction` component heeft als prop een transaction en deze blijft ongewijzigd als de gebruiker een letter ingeeft in het zoekveld. Toch wordt de component gererenderd.

Een **pure component** is een component die gegeven dezelfde props dezelfde output genereert. `Transaction` is een pure component. Gegeven dezelfde props, wordt dezelfde output gegenereerd. We willen een pure component niet opnieuw renderen als de properties niet gewijzigd zijn. De `memo` functie wordt gebruikt om een component te creëren die enkel zal rerenderen als de props wijzigen.

```jsx
import {memo} from 'react'; // 👈
export default memo(function Transaction(props) { // 👈
  const { user, amount, place} = props; 
  console.log('Rendering transaction...');
  return <div className="text-bg-dark" style={{ width: '50%' }}>{user} gaf €{amount} uit bij {place}</div>;
});
```

Start de app en bekijk de console. Ook `TransactionForm` is een pure component en dient enkel gererenderd te worden als zijn state wordt aangepast.

#### Oefening

Cache de `TransactionForm` component en bekijk de app opnieuw. Als we een letter ingeven wordt de component nog steeds opnieuw gerenderd. Hoe komt dit?

## useCallback hook

`TransactionForm` bevat één prop, nl. de `onSaveTransaction` functie. Deze wordt doorgegeven door de parent component `TransactionList`. `createTransaction` is de event handler functie. Javascript gaat er vanuit dat de functie `createTransaction` bij elke render verschillend is. Echter is dit niet altijd het geval. `useCallback` cacht een functie tussen twee renders en dit totdat de dependency array wijzigt.

Pas de code van de functie in de `TransactionList` component aan:

```jsx
import {useState, useMemo, useCallback} from 'react'; // 👈

const createTransaction = useCallback((user, place, amount, date) => { // 👈
  const newTransactions = [
    {
      user, place, amount, date: new Date(date),
    },
    ...transactions,
  ]; // newest first
  setTransactions(newTransactions);
}, [transactions]); // 👈
```

Start de app opnieuw. De `TransactionForm` component wordt niet opnieuw gererenderd bij ingave van een karakter.

### Oefening

Pas memoization toe op de `Place` component.

## useContext hook

De `useContext` hook voorziet een manier om data door te geven in de component tree zonder te moeten werken met props. Het wordt gebruikt voor data die gebruikt worden binnen niet gerelateerde takken van de component tree (zie volgend hoofdstuk).

## useReducer hook

Dit is een alternatief voor `useState` maar met complexe state logica. Lees hierover meer in de [documentatie van de hook](https://reactjs.org/docs/hooks-reference.html#usereducer).

## Formulieren

In het vorige hoofdstuk hebben we een eenvoudig formulier gezien. Validatie, foutafhandeling, formArrays... ontbreken nog. Je kan dit allemaal zelf implementeren of je kan gebruik maken van een package, bv. [react-hook-form](https://react-hook-form.com/).

Voeg dit package toe aan het project:

```bash
yarn add react-hook-form
```

We maken gebruik van de [useForm](https://react-hook-form.com/api/useform) hook uit het `react-hook-form` package.

```jsx
import {memo} from 'react';
import { PLACE_DATA } from '../../api/mock-data';
import { useForm } from 'react-hook-form'; // 👈 1

const toDateInputString = (date) => {
    // ISO String without the trailing 'Z' is fine 🙄
    // (toISOString returns something like 2020-12-05T14:15:74Z,
    // datetime-local HTML5 input elements expect 2020-12-05T14:15:74, without the (timezone) Z)
    //
    // the best thing about standards is that we have so many to chose from!
    if (!date) return null;
    if (typeof date !== Object) {
        date = new Date(date);
    }
    let asString = date.toISOString();
    return asString.substring(0, asString.indexOf('T'));
};

export default memo(function TransactionForm({ onSaveTransaction }) {
  // const [user, setUser] = useState(''); // 👈 4
  // const [date, setDate] = useState(new Date()); // 👈 4
  // const [place, setPlace] = useState('home'); // 👈 4
  // const [amount, setAmount] = useState(0); // 👈 4

  const { register, handleSubmit, reset } = useForm(); // 👈 2, 6 en 8

  /*c const handleSubmit = (e) => {
      e.preventDefault();
      onSaveTransaction(user, place, amount, date);
      setUser('');
      setDate(new Date());
      setPlace('home');
      setAmount(0);
    };*/

  // 👇 7
  const onSubmit = (data) => {
      console.log(JSON.stringify(data));
      const { user, place, amount, date } = data;
      onSaveTransaction(user, place, parseInt(amount), date);
      reset(); // 👈 8
  };

  return (
    <>
      <h2>
          Add transaction
      </h2>

      <form onSubmit={handleSubmit(onSubmit)} className="w-50 mb-3"> {/*👈6*/}
        <div className="mb-3">
          <label htmlFor="date" className="form-label">Who</label>
          <input
            {...register('user')} {/* 👈 3 */}
            defaultValue='' {/* 👈 5 */}
            id="user"
            type="text"
            className="form-control"
            placeholder="user"
            required
          />
        </div>

        <div className="mb-3">
          <label htmlFor="date" className="form-label">Date</label>
          <input
            {...register('date')} {/* 👈 3 */}
            id="date"
            type="date"
            className="form-control"
            placeholder="date"
          />
        </div>

        <div className="mb-3">
          <label htmlFor="places" className="form-label">
            Place
          </label>
          <select
            {...register('place')} {/* 👈 3 */}
            id="places"
            className="form-select"
            required
          >
            <option defaultChecked value="">-- Select a place --</option>
            {PLACE_DATA.map(({ id, name }) => (
              <option key={id} value={name}>{name}</option>
            ))}
          </select>
        </div>

        <div className="mb-3">
          <label htmlFor="amount" className="form-label">
            Amount
          </label>
          <input
            {...register('amount')}
            id="amount"
            type="number"
            className="form-control"
            required
          />
        </div>

        <div className="clearfix">
          <div className="btn-group float-end">
            <button
              type="submit"
              className="btn btn-primary"
            >
              Add transactio
            </button>
          </div>
        </div>
      </form>
    </>
  );
});
```

1. `useForm` is een **custom hook** om forms te beheren, het geeft allerlei nuttige functies en andere info terug. Neem maar een kijkje in de **documentatie**.
2. [register](https://react-hook-form.com/api/useform/register) methode: registreren van de velden in het formulier. De waarden van de velden kunnen zo gebruikt worden voor zowel formuliervalidatie als het verzenden van het formulier.
3. Registreer de formuliervelden in de `useForm` hook.
4. We hoeven zelf geen state meer bij te houden.
5. Je kan ook een standaardwaarde opgeven.
6. [handleSubmit](https://react-hook-form.com/api/useform/handlesubmit): deze functie zorgt ervoor dat de formuliergegevens verzameld worden bij het submitten van het formulier. Je geeft aan deze functie een functie mee die opgeroepen moet worden als het formulier verzonden wordt.
7. De `onSubmit` methode logt de verstuurde waarden naar de console. `data` bevat de ingevulde waarden per formulierveld: `register('user')` wordt gesubmit als `{ user:'value' }`.
8. [reset](https://react-hook-form.com/api/useform/reset): deze functie zet alle velden terug op de standaardwaarde (indien opgegeven) of maakt ze leeg.

### Validatie

We geven een voorbeeld voor het inputveld van de gebruiker, dit is vrij gelijkaardig voor de overige velden.

```jsx
//...

const { register, handleSubmit, reset, formState: { errors } } = useForm(); // 👈 2

//...
<form onSubmit={handleSubmit(onSubmit)} className="w-50 mb-3"> 
  <div className="mb-3">
    <label htmlFor="date" className="form-label">Who</label>
    <input
      {/* 👇 1 */}
      {...register('user', {
        required: 'user is required',
        minLength: { value: 2, message: 'Min length is 2' }
      })}
      defaultValue=''
      id="user"
      type="text"
      className="form-control"
      placeholder="user"
      required
    />
    {/* 👇 2 */}
    {errors.user && <p className="form-text text-danger">{errors.user.message}</p> }
  </div>
```

1. Als tweede parameter van de `register` methode kan je de validatieregels meegeven (required, min, max, minLength, maxLength, pattern, validate). Je kan ook de bijhorende foutmelding opgeven. React Hook Form ondersteunt ook schema-validatie met Yup, Zod, Superstruct & Joi. De validatie is afgestemd op de HTML standaard voor formuliervalidatie. Meer hierover in de documentatie van [register](<https://react-hook-form.com/api/useform/register>.
Voor inputveld met als type `number` dien je `valueAsNumber` in te stellen zodat je een getal i.p.v. een string terugkrijgt.
2. Voor de weergave van de fouten maken we gebruik van het `errors` object. A.d.h.v. `type` property kan je het type van de fout opvragen (bv. `errors.user.type === "required"`). Merk op dat we hier gebruik maken van `&&`, dit wordt wel eens gezien als een anti-pattern in React. Het is eigenlijk beter om de ternary operator (voorwaarde ? true : false) te gebruiken. Dit wordt dus `{errors.user ? <p className="form-text text-danger">{errors.user.message}</p> : null}`.

Vermits er meerdere invoervelden op ons formulier voorkomen en we steeds dezelfde code moeten schrijven, zouden we een aparte component moeten aanmaken. Deze component zal gebruik moeten maken van [useFormContext](https://react-hook-form.com/api/useformcontext). De **Context Provider** komt echter pas aan bod in het volgend hoofdstuk, we laten die even zo.

## Oefening

Als we surfen naar <https://api.thecatapi.com/v1/breeds> dan krijgen we JSON met alle kattenrassen. Maak een pagina waar de gebruiker een ras kan selecteren, en waar de details van het ras getoond worden.

- Maak voor het ophalen van de JSON data gebruik van `useEffect` en de `fetch` functie.
- Haal alle breeds in één keer op en hou een breeds state variabele bij.
- Hou de geselecteerde breed ook bij in state.
- Maak een formulier om een nieuwe breed toe te voegen. Voeg deze toe aan de gecachte breeds. Alle velden met uitzondering van de `imageUrl` zijn verplicht in te vullen.
- Maak gebruik van [bootstrap](https://getbootstrap.com/docs/).

![Voorbeeld van de kattenrassen-applicatie](./images/cats.PNG)
