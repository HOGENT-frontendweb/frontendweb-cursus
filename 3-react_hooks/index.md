# Get your hooks into React

## Inleiding

`Componenten` zijn functies die de UI renderen. Het renderen gebeurt wanneer de app voor het eerst geladen wordt en wanneer state waarden wijzigen. Renderen van code moet ["puur"](https://beta.reactjs.org/learn/keeping-components-pure) zijn. Componenten mogen enkel 'hun' jsx retourneren, en mogen objecten of variabelen die bestaan voor de rendering niet wijzigen. Gegeven dezelfde input, dient het dezelfde output te retourneren. Net als een wiskundige formule zou het alleen het resultaat moeten berekenen, maar niets anders doen. 

`Events` zijn functies binnen de component die worden uitgevoerd als reactie op een actie van een gebruiker. Een event handler kan state aanpassen, een HTTP post request submitten om een transactie toe te voegen,... Event handlers bevatten 'side-effects' veroorzaakt door een interactie.

Soms is dit niet voldoende. Het ophalen van de transacties vanuit een REST API, dient sowieso te gebeuren en is niet gekoppeld aan een interactie van een gebruiker en mag ook niet gebeuren tijdens de rendering (anders is de component niet puur). Hiervoor kunnen we gebruik maken van de `useEffect` hook.

Effecten worden meestal gebruikt om uit je React-code te stappen en te synchroniseren met een extern systeem. Dit omvat browser-API's, widgets van derden, netwerken, enzovoort. Ze voeren een 'side-effect' uit.

## useEffect hook
`useEffect` is een functie die asynchroon wordt uitgevoerd na de render en die zichzelf optioneel kan opruimen(cleanup) voordat het opnieuw wordt uitgevoerd en voor de unmounting (het vernietigen van de component). React onthoudt de functie die je hebt doorgegeven ​​(we noemen dit ons "effect") en roept het later op na het uitvoeren van de DOM-updates.

In onderstaand voorbeeld wordt er een boodschap naar de console gelogd als de TransactionList gerenderd is. Deze instructie zouden we na de return kunnen plaatsen, maar die code wordt niet uitgevoerd. `useEffect`is hier de oplossing.

```jsx
import {useEffect, useState} from 'react';//👈1
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import {TRANSACTION_DATA} from '../../api/mock-data'; 

export default function TransactionList() {

const [transactions, setTransactions] = useState(TRANSACTION_DATA);

//👇2
useEffect(() => {
  console.log("transactions are rendered");
})

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
}
```
1. We maken hiervoor gebruik van `useEffect`.
2. Binnen de component roepen we de `useEffect` functie aan. We geven een `callback functie` mee als parameter. De functie die we meegeven wordt het `effect` genoemd. Wanneer React onze component rendert, onthoudt React het effect dat we hebben gedefinieerd en voert het het effect uit na het updaten van de DOM. Dit gebeurt default na elke render, ook de eerste.

[Run de app](http://localhost:3000/). Bekijk de console. Voeg een transactie toe. We zien in de console dat `useEffect` na de initiële render en bij elke rerender wordt uitgevoerd. 

Opmerking: React StrictMode (zie index.js) controleert of een component pure is door de component functie 2* aan te roepen. Dit gebeurt enkel in development mode, niet in productie. Di ook de reden waarom het loggen naar de console 2 maal gebeurt. Zie [Detecting impure calculations with StrictMode](https://beta.reactjs.org/learn/keeping-components-pure) en [Why does my calculation runs twice](https://beta.reactjs.org/apis/react/useMemo#my-calculation-runs-twice-on-every-re-render)

Oefening: voeg dezelfde code toe aan de TransactionForm component. Wat wordt er eerst gerenderd?

### Specifieer de Effect dependencies
Stel dat we de boodschap enkel bij de initïele render wensen te loggen.

Aan de hand van een `dependency array` kan je het uitvoeren van een `useEffect` koppelen aan specifieke datawijzigingen. Zo voorkom je dat `useEffect` bij elke rerender opnieuw wordt uitgevoerd.  

- **[]**: een lege dependency array. `useEffect` wordt enkel bij de initiële render  uitgevoerd
```jsx
useEffect(() => {
    console.log("transactions after the initial render");
}, [])
```
- **[transactions]**: `useEffect` wordt bij de initiele render en telkens de waarde van de variabele `transactions` wijzigt uitgevoerd. React zal het effect overslaan als `transactions`  dezelfde waarden heeft als tijdens de laatste render.
```jsx
useEffect(() => {
    console.log("transactions after initial render or transaction added");
}, [transactions])
```

- **meerdere dependencies**: React zal het opnieuw uitvoeren van het effect alleen overslaan als alle dependencies die je opgeeft exact dezelfde waarden hebben als tijdens de vorige render. 
Stel dat de user via een prop wordt doorgegeven aan de TransactionList en ook naar de console gelogt dient te worden.
```jsx
export default function TransactionList({user='Luis'}){ //👈de prop user

  const [transactions, setTransactions] = useState(TRANSACTION_DATA);

  useEffect(() => {
      console.log(`Hi  ${user}, transactions after initial render or transaction added`);//👈 maakt gebruik van user
  }, [transactions])//React Hook useEffect has a missing dependency
  //...
}
```

  Merk op: je krijgt een lintfout 'React Hook useEffect has a missing dependency' als de dependencies die je hebt opgegeven niet overeenkomen met wat React verwacht op basis van de code in je effect. Dit helpt veel bugs in de code op te sporen. Als je effect een bepaalde waarde gebruikt, maar je het effect niet opnieuw wilt uitvoeren wanneer het verandert, moet je ervoor zorgen dat je effect code geen gebruik maakt van deze dependency.
  Oplossing: 

  ```jsx
  export default function TransactionList({user='Luis'}) {

  const [transactions, setTransactions] = useState(TRANSACTION_DATA);

  useEffect(() => {
    console.log(`Hi ${user}, transactions after initial render or transaction added`);
  }, [transactions, user])//👈 meerdere dependencies

  ```
  React vergelijkt de dependency waarden met behulp van [Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). 

### Clean up, indien nodig
Een side-effect kan een `clean-up functie` retourneren. React roept de cleanup functie elke keer aan voordat het effect opnieuw wordt uitgevoerd, en een laatste keer wanneer de component wordt ontkoppeld (wordt verwijderd, on unmount).  
Verwijder eerst de code m.b.t. de prop user.

```jsx
export default function TransactionList() {

const [transactions, setTransactions] = useState(TRANSACTION_DATA);

useEffect(() => {
  console.log("transactions after initial render or transaction added");
  	return ()=>console.log("unmounted..."); //👈 de clean-up functie
}, [transactions])
```

[Run de app](http://localhost:3000/) en bekijk de console. 

### Simulatie ophalen data uit REST API

We simuleren nu het ophalen van de transactions via de REST API. We maken hiervoor gebruik van de `settimeout` functie.

```jsx
const [transactions, setTransactions] = useState([]);//👈1

useEffect(() => {//👈2
  setTimeout(()=> {
    setTransactions(TRANSACTION_DATA);
    console.log("transactions fetched");
  }, 2000);   
}, [])
```
1. Initieel is de lijst van transacties leeg
2. We halen de data op, eens ontvangen wordt de state aangepast, waardoor de pagina opnieuw wordt gerenderd. Het ophalen van de data mag enkel gebeuren bij de initiële render. Waarom?

In het volgende hoofdstuk zien we hoe we een request kunnen sturen naar een REST API, een loading indicator kunnen tonen en eventuele fouten kunnen weergeven.

### Hook rules
Hooks zijn JavaScript functions, maar je moet 2 regels volgen wanneer je er gebruik van maakt. Je kan hiervoor een linter plugin installeren (automatisch bij create-react-app)
- Gebruik hooks enkel op het top niveau. Gebruik hooks niet binnen een if, andere condities, loop en geneste functies. Reden: React valt terug op een volgorde waarin hooks worden aangeroepen om een waarde terug te geven. React houdt dit bij in een array. De volgorde moet dezelfde zijn bij elke render. [Benieuwd?](https://itnext.io/the-first-rule-of-react-hooks-in-plain-english-1e0d5ae32009)
- Roep hooks enkel aan vanuit React functies. Di function components of custom hooks.
- Gebruik `useEffect` niet voor het aanbrengen van DOM-wijzigingen die zichtbaar zijn voor de gebruiker. Een `useEffect` wordt pas geactiveerd nadat de browser klaar is met lay-out en paint. Te laat dus als je een visuele wijziging wilde aanbrengen. Voor die gevallen biedt React de hooks `useMutationEffect` en `useLayoutEffect`, die op dezelfde manier werken als useEffect, behalve wanneer ze worden afgevuurd.
- Hooks maken gebruik van closures. Let op voor stale closures. [Enkele voorbeelden](https://dmitripavlutin.com/react-hooks-stale-closures/)
- `useEffect` laat NIET toe om async toe te voegen in de callback function


Voor extra uitleg en voorbeelden: [Synchronizing with Effects](https://beta.reactjs.org/learn/synchronizing-with-effects) 


## Verbeteren van de performantie
In een React-toepassing worden componenten heel vaak gerenderd ... De performantie kan je verbeteren door het voorkomen van onnodige renders en het verminderen van de tijd die een render in beslag neemt. 

`Memoization`: "In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again." (Wikipedia)

`Memoization in React kan via
- `memo`: creatie van pure componenten
- `useMemo`: retourneert een memoized waarde
- `useCallback`: retourneert een memoized callback/functie

### useMemo hook

`useMemo` is een React Hook waarmee je het resultaat van een berekening tussen re-renders kunt cachen.  
Voorbeeld: we voegen een zoek functie toe om de transacties te filteren o.b.v. plaats in de `TransactionList`component. 

```jsx
import {useState} from 'react';
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import {TRANSACTION_DATA} from '../../api/mock-data'; 

export default function TransactionList() {

const [transactions, setTransactions] = useState(TRANSACTION_DATA);
const [text, setText] = useState('');//👈1
const [search, setSearch] = useState('');//👈1

const filteredTransactions = transactions.filter((t) =>{
  console.log("filtering...");
  return t.place.toLowerCase().includes(search.toLowerCase());
})//👈2

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
    <div className="input-group mb-3 w-50"> {/*👈3*/}
      <input type="search" id="search" className="form-control rounded" placeholder="Search" value={text} onChange={(e) => setText(e.target.value)} />
      <button type="button" className="btn btn-outline-primary" onClick={()=>setSearch(text)}>Search</button>
    </div>
		{filteredTransactions.map((trans, index) => //👈4
      <Transaction {...trans} key={index} /> )}
  </>);
}
```

1. De filtering mag enkel gebeuren als de gebruiker op search klikt, niet bij ingave van een letter in de zoekterm (vandaar de 2 state variabelen). 
2. Functie voor het filteren van de transacties
3. Het search formulier
4. Enkel de gefilterde transacties worden getoond

Bij elk karakter dat de gebruiker ingeeft in het 'search' veld, wordt de state aangepast, de component gererenderd, de filter methode uitgevoerd, hoewel de output ongewijzigd blijft. Dit kan je oplossen door gebruik te maken van `useMemo`, om de returnwaarde van een functie te onthouden, en om de functie enkel en alleen uit te voeren als de dependencies zijn veranderd. In onderstaand voorbeeld wordt de filter pas uitgevoerd bij het laden van de component en bij klikken op 'search'.

```jsx
import {useState, useMemo} from 'react';//👈
//...
const filteredTransactions = useMemo(()=> transactions.filter((t) =>{//👈
  console.log("filtering...");
  return t.place.toLowerCase().includes(search.toLowerCase());
}),[search, transactions])
//...
```
Je moet 2 zaken doorgeven aan de `useMemo` functie
1. een 'calculation functie' die het resultaat van de berekening retourneert. Het resultaat van die functie wordt bijgehouden in de cache.
2. een 'dependency array' die elke waarde bevat waarnaar verwezen wordt in de calculation function. 

Bij elke volgende render vergelijkt React de dependencies met de dependencies die je tijdens de laatste render hebt doorgegeven. Als geen van de dependencies is gewijzigd, retourneert `useMemo` de waarde die al eerder werd berekend. Anders zal React de berekening opnieuw uitvoeren en de nieuwe waarde retourneren.

Oefening: probeer de challenges op [https://beta.reactjs.org/learn/keeping-components-pure](https://beta.reactjs.org/learn/keeping-components-pure)

### React.memo en pure functions
Voeg een console.log instructie toe voor elke `return` instructie van onderstaande componenten

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

Telkens als we een letter ingeven in het 'search' veld, worden alle componenten gererenderd, alhoewel er niets wijzigt aan de output van de component. De `Transaction` component heeft als prop een transaction en dit blijft ongewijzigd als de gebruiker een letter ingeeft in het 'search' veld. Toch wordt de component gererenderd. 

Een `pure component` is een component die gegeven dezelfde props dezelfde output genereert. Transaction is een pure component. Gegeven dezelfde props, wordt dezelfde output gegenereerd. We willen een pure component niet opnieuw renderen als de properties niet gewijzigd zijn. De `memo` functie wordt gebruikt om een component te creëren die enkel zal rerenderen als de props wijzigen.   

```jsx
import {memo} from 'react';  //👈
export default memo(function Transaction(props) { //👈
  const { user, amount, place} = props; 
  console.log('Rendering transaction...');
  return <div className="text-bg-dark" style={{width:'50%'}}>{user} gaf €{amount} uit bij {place}</div>; 
})
```

[Run de app](http://localhost:3000/) en bekijk de console. Ook TransactionForm is een pure component en dient enkel gererenderd te worden als zijn state wordt aangepast.    

Oefening: Cache de component TransactionForm en [run](http://localhost:3000/) opnieuw. Als we een letter ingeven wordt de component nog steeds opnieuw gerenderd???

## useCallback hook
`TransactionForm` bevat 1 prop, nl de `onSaveTransaction` functional prop. Deze wordt doorgegeven door de parent component `TransactionList`. `createTransaction` is de event handler functie. Javascript gaat ervan uit dat de functie `createTransaction` bij elke render verschillend is.    
`useCallback` cacht een functie tussen 2 re-renders en dit totdat de dependency array wijzigt.    
Pas de code van de functie aan in de `TransactionList`component

```jsx
import {useState, useMemo, useCallback} from 'react';//👈

const createTransaction = useCallback((user, place, amount, date) => {//👈
  const newTransactions = [
    {
      user, place, amount, date: new Date(date),
    },
    ...transactions,
  ]; // newest first
  setTransactions(newTransactions);
}, [transactions]);//👈
```

[Run](http://localhost:3000/) opnieuw. De TransactionForm component wordt niet opnieuw gererenderd bij ingave van een karakter.

oefening : pas memoization toe op de Places

## useContext hook
Voorziet in een manier om data door te geven in de component tree zonder te moeten werken met props. Het wordt gebruikt voor data die gebruikt worden binnen niet gerelateerde takken van de boom (zie volgend hoofdstuk)

## useReducer hook
Is een alternatief voor useState, maar met complexe state logica

## Formulieren
In het vorige hoofdstuk hebben we een basic formulier gezien. Validatie, foutafhandeling, formArrays,... ontbreken nog. Je kan dit allemaal zelf implementeren, of je kan gebruik maken van een package, bvb [`react-hook-form`](https://react-hook-form.com/).  
Voeg de package toe aan het project:

```bash
yarn add react-hook-form
```

We maken gebruik van de [useForm](https://react-hook-form.com/api/useform) hook uit de `react-hook-form` package.
```jsx
import {memo} from 'react';
import { PLACE_DATA } from '../../api/mock-data';
import { useForm } from 'react-hook-form';//👈1

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
    // const [user, setUser] = useState(''); //👈4
    // const [date, setDate] = useState(new Date()); //👈4
    // const [place, setPlace] = useState('home'); //👈4
    // const [amount, setAmount] = useState(0); //👈4

    const { register, handleSubmit, reset } = useForm(); //👈2 en 6 en 8

    /*c const handleSubmit = (e) => {
       e.preventDefault();
       onSaveTransaction(user, place, amount, date);
       setUser('');
       setDate(new Date());
       setPlace('home');
       setAmount(0);
     };*/

//👈7
    const onSubmit = (data) => {
        console.log(JSON.stringify(data));
        const { user, place, amount, date } = data;
        onSaveTransaction(user, place, parseInt(amount), date);
        reset(); //👈8
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
                        {...register('user')}
                        defaultValue=''
                        id="user"
                        type="text"
                        className="form-control"
                        placeholder="user" required
                    />{/*👈3 en 5*/}
                </div>

                <div className="mb-3">
                    <label htmlFor="date" className="form-label">Date</label>
                    <input
                        {...register('date')}
                        id="date"
                        type="date"
                        className="form-control"
                        placeholder="date"
                    />{/*👈3*/}
                </div>

                <div className="mb-3">
                    <label htmlFor="places" className="form-label">
                        Place
                    </label>
                    <select
                        {...register('place')}
                        id="places"
                        className="form-select"
                        required
                    >
                        <option defaultChecked value="">-- Select a place --</option>
                        {PLACE_DATA.map(({ id, name }) => (
                            <option key={id} value={name}>{name}</option>
                        ))}
                    </select>{/*👈3*/}
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
                        >Add transaction</button>
                    </div>
                </div>
            </form>
        </>
    );
})
```
1.`useForm` is een `custom hook` om forms te beheren
2.[register](https://react-hook-form.com/api/useform/register) methode: registreren van  input- of selectie-elementen. De waarden van de input-elementen kunnen zo gebruikt worden voor zowel formuliervalidatie als het verzenden van het formulier
3. Registreer de input elementen in de `useForm` hook. 
4. We hoeven zelf geen state meer bij te houden.
5. Je kan ook een default waarde opgeven
6.[handleSubmit](https://react-hook-form.com/api/useform/handlesubmit): Deze functie ontvangt de formuliergegevens als de formuliervalidatie is gelukt. Als we het formulier submitten, roepen we de `handleSubmit` methode aan.
7. De `onSubmit` methode logt de verstuurde waarden naar de console. `data` bevat de gesubmitte waarden per input tag. register('user') wordt gesubmit als {user:'value'}
8. [reset](https://react-hook-form.com/api/useform/reset) Als formulier gesubmit is dan resetten we het formulier.

### validatie
Voorbeeld voor het inputveld user
```jsx
//...
const { register, handleSubmit, reset, formState: { errors } } = useForm();//👈2
//...
 <form onSubmit={handleSubmit(onSubmit)} className="w-50 mb-3"> 
    <div className="mb-3">
        <label htmlFor="date" className="form-label">Who</label>
        <input
            {...register('user',
                { required: 'user is required', minLength: { value: 2, message: 'Min length is 2' } })}
            defaultValue=''
            id="user"
            type="text"
            className="form-control"
            placeholder="user" required
        />{/*👈1*/}
        {errors.user && <p className="form-text text-danger">{errors.user.message}</p>}{/*👈2*/}
    </div>
```
1. Als 2de parameter van de `register` methode geef je de validatieregels mee (required, min, max, minLength, maxLength, pattern, validate). Je kan ook de bijhorende foutmelding opgeven. React ondersteunt ook schema validatie met Yup, Zod, Superstruct & Joi. De validatie is afgestemd op de HTML standaard voor formulier validatie. Meer op [register](https://react-hook-form.com/api/useform/register.  
Voor input 'type=number' dien je valueAsNumber in te stellen zodat je een getal ipv string terugkrijgt.
2. Voor de weergave van de fouten maken we gebruik van het `errors` object. Adhv type property kan je het type van de fout opvragen (errors.user.type==="required")

Vermits er meerdere input velden op ons formulier voorkomen en we steeds dezelfde code moeten schrijven, zouden we een aparte component moeten aanmaken. Deze component zal gebruik moeten maken van `useFormContext`. De `Context Provider` komt aan bod in het volgend hoofdstuk.

## Oefening

Als we surfen naar https://api.thecatapi.com/v1/breeds dan krijgen we json met alle cat breeds. Maak een pagina waar de gebruiker een breed kan selecteren, en waar de details van de breed getoond worden. 
- Maak voor het ophalen van de json file gebruik van `useEffect` en de fetch methode. 
- Haal alle breeds in 1 keer op en hou een breeds state variabele bij
- Hou de geselecteerde breed ook bij in state
- Maak een formulier om een nieuwe breed toe te voegen. Voeg deze toe aan de gecachte breeds. Alle velden met uitz van de imageUrl zijn verplicht in te vullen.
- Maak gebruik van bootstrap
![](./images/cats.PNG)

## Oplossing
[github](https://github.com/HOGENT-Web/frontend-ch3-exercise-solution)
