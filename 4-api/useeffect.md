Soms is dit niet voldoende. Het ophalen van de transacties vanuit een REST API, dient sowieso te gebeuren en is niet gekoppeld aan een interactie van een gebruiker en mag ook niet gebeuren tijdens de rendering (anders is de component niet puur). Hiervoor kunnen we gebruik maken van de `useEffect` hook.

Effecten worden meestal gebruikt om uit je React-code te stappen en te synchroniseren met een extern systeem. Dit omvat browser API's, widgets van derden, netwerken... Ze voeren een **side-effect** uit. Tegenwoordig wordt afgeraden om `useEffect` te gebruiken aangezien veel developers de hook gebruiken waarvoor hij niet gemaakt is (zie <https://www.youtube.com/watch?v=bGzanfKVFeU>).

In dit hoofdstuk werken we richting een voorbeeld van data fetching m.b.v. `useEffect`. Het uiteindelijke doel is om `useEffect` te vervangen door een library die specifiek ontworpen is voor data fetching, net zoals de [React docs aanbevelen](https://react.dev/reference/react/useEffect#fetching-data-with-effects).

## useEffect hook

`useEffect` is een functie die asynchroon wordt uitgevoerd na de render, en die zichzelf optioneel kan opruimen (= **cleanup**) Het opruimen gebeurt voordat het effect opnieuw wordt uitgevoerd en voor de **unmounting** (= het vernietigen van de component). React onthoudt de functie die je hebt doorgegeven ​​(we noemen dit ons "effect") en roept het later op na het uitvoeren van de DOM-updates.

In onderstaand voorbeeld wordt er een boodschap naar de console gelogd als de `TransactionList` gerenderd is. Deze instructie zouden we na de return kunnen plaatsen, maar die code wordt niet uitgevoerd. `useEffect` is hier de oplossing.

```jsx
import {useEffect, useState} from 'react'; // 👈 1
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import {TRANSACTION_DATA} from '../../api/mock_data'; 

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
2. Binnen de component roepen we de `useEffect` functie aan. We geven een **callback functie** mee als parameter. De functie die we meegeven wordt het **effect** genoemd. Wanneer React onze component rendert, onthoudt React het effect dat we hebben gedefinieerd en voert het het effect uit na het updaten van de DOM. Dit gebeurt default na elke render, ook na de eerste.

Start de app en bekijk de console. Voeg een transactie toe. We zien in de console dat `useEffect` na de initiële render en bij elke rerender wordt uitgevoerd.

> **Merk op:** React StrictMode (zie `index.js`) controleert of een component pure is door de component functie tweemaal aan te roepen. Dit gebeurt enkel in development mode, niet in productie. Dit is ook de reden waarom het loggen naar de console tweemaal gebeurt. Zie [Detecting impure calculations with StrictMode](https://beta.reactjs.org/learn/keeping-components-pure) en [Why does my calculation runs twice](https://beta.reactjs.org/apis/react/useMemo#my-calculation-runs-twice-on-every-rerender).

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

React vergelijkt de dependency waarden met behulp van [Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Voor arrays en objecten wordt hier bijgevolg gekeken naar de referentie, en niet naar de exacte waarde!

### Clean up, indien nodig

Een side-effect kan een **clean-up functie** retourneren. React roept de clean-up functie elke keer aan voordat het effect opnieuw wordt uitgevoerd en een laatste keer wanneer de component wordt verwijderd (= on unmount).

Verwijder eerst de code m.b.t. de prop user en voeg dan een clean-up functie met een simpele `console.log` toe.

```jsx
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

## Opmerkingen hooks

- Gebruik `useEffect` niet voor het aanbrengen van DOM-wijzigingen die zichtbaar zijn voor de gebruiker. Een `useEffect` wordt pas geactiveerd nadat de browser klaar is met de lay-out en het tekenen. Dit is dus te laat als je een visuele wijziging wilde aanbrengen. Voor die gevallen biedt React de hooks `useInsertionEffect` en `useLayoutEffect` die op dezelfde manier werken als `useEffect`. Ze verschillen enkel in het moment van 'afvuren'.
- Beperk het gebruik van `useEffect`, in de meeste gevallen heb je deze hook niet nodig. Je hebt het enkel nodig als je "uit de React code" wil stappen, bv. synchronisatie met een cloudsysteem, synchronisatie met een niet-React DOM element... Probeer dus eerst je probleem op te lossen met andere hooks voor je terugvalt op `useEffect`.
  - Voor extra uitleg en voorbeelden: [Synchronizing with Effects](https://beta.reactjs.org/learn/synchronizing-with-effects)
- `useEffect` laat NIET toe om het keyword `async` toe te voegen in de callback function, zie een volgend hoofdstuk voor een oplossing. Dit is trouwens nog een reden waarom je best een library gebruikt voor het ophalen van data.