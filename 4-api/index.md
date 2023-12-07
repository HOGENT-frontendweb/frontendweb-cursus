# Data ophalen uit een REST API

!> Vanaf dit hoofdstuk heb je de bijbehorende backend nodig: <https://github.com/HOGENT-Web/webservices-budget>.<br />Als je zonder MySQL-databank wil werken, check uit op commit `f6afd9b`. Op de laatste commit is een lokale MySQL-server vereist en is de data-structuur licht aangepast (sommige voorbeelden kunnen dus afwijkende code vereisen als je zonder databank werkt). Maak ook een .env aan, bekijk de README.md voor meer informatie.

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-Web/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les4 b3b27e0
> yarn install
> yarn dev
> ```

In dit hoofdstuk vervangen we de mock data door HTTP requests naar de REST API. Op ons lokaal toestel draait deze API op [http://localhost:9000/api/](http://localhost:9000/api/).

Voor de communicatie met de API, m.a.w. het versturen van HTTP requests, kan je gebruik maken van de [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) of van HTTP client libraries die je kan vinden op bv. <https://www.npmjs.com>.

Wij zullen gebruik maken van [swr](https://www.npmjs.com/package/swr), een _React Hooks library for data fetching_. Het wordt ontwikkeld door Vercel, het bedrijf achter Next.js. In de documentatie lezen we de betekenis van de naam van het package:

> The name “SWR” is derived from `stale-while-revalidate`, a cache invalidation strategy popularized by [HTTP RFC 5861](https://datatracker.ietf.org/doc/html/rfc5861). SWR first returns the data from cache (stale), then sends the request (revalidate), and finally comes with the up-to-date data again.

Naast SWR hebben we ook [axios](https://www.npmjs.com/package/axios) nodig, een HTTP client library die we gebruiken om de HTTP requests uit te voeren. SWR heeft namelijk geen ingebouwde HTTP client, je bent dus vrij om te kiezen welke HTTP client je gebruikt. In tegenstelling tot `swr` kan je `axios` ook gebruiken in een Node.js omgeving, om bv. data op te halen uit een third party API.

Installeer alvast `axios` en `swr`:

```bash
yarn add axios
yarn add swr
```

## useEffect

`swr` verbergt namelijk heel wat van de complexiteit omtrent API calls. Daarom tonen we eerst hoe je zonder externe libraries een HTTP request kan uitvoeren. Hiervoor maken we gebruik van de `useEffect` hook.

Effecten worden gebruikt om uit je React-code te stappen en te synchroniseren met een extern systeem. Dit omvat browser API's, widgets van derden, netwerken... Ze voeren een **side-effect** uit. Tegenwoordig wordt afgeraden om `useEffect` te gebruiken aangezien veel developers de hook gebruiken waarvoor hij niet gemaakt is (zie <https://www.youtube.com/watch?v=bGzanfKVFeU>).

In deze sectie werken we richting een voorbeeld van data fetching m.b.v. `useEffect`. Het uiteindelijke doel is om `useEffect` te vervangen door een library (hier dus `swr`) die specifiek ontworpen is voor data fetching, net zoals de [React docs aanbevelen](https://react.dev/reference/react/useEffect#fetching-data-with-effects).

`useEffect` is een functie die asynchroon wordt uitgevoerd na de render, en die zichzelf optioneel kan opruimen (= **cleanup**) Het opruimen gebeurt voordat het effect opnieuw wordt uitgevoerd en voor de **unmounting** (= het vernietigen van de component). React onthoudt de functie die je hebt doorgegeven ​​(we noemen dit ons "effect") en roept deze later aan, na het uitvoeren van de DOM-updates.

In onderstaand voorbeeld wordt een boodschap naar de console gelogd als de `TransactionList` gerenderd is. Deze instructie zouden we na de return kunnen plaatsen, maar die code wordt niet uitgevoerd. `useEffect` is hier de oplossing.

```jsx
import { useState, useMemo, useCallback, useContext, useEffect } from 'react'; // 👈 1
// andere imports...

export default function TransactionList() {
  // state...

  // 👇 2
  useEffect(() => {
    console.log("transactions are rendered");
  });

  // memo, callbacks...

  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={createTransaction} />
      <div className='input-group mb-3 w-50'>
        <input
          type='search'
          id='search'
          className='form-control rounded'
          placeholder='Search'
          value={text}
          onChange={(e) => setText(e.target.value)}
        />
        <button
          type='button'
          className='btn btn-outline-primary'
          onClick={() => setSearch(text)}
        >
          Search
        </button>
      </div>
      <div className='mt-4'>
        <TransactionTable transactions={filteredTransactions} />
      </div>
    </>
  );
}
```

1. We importeren `useEffect`.
2. Binnen de component roepen we de `useEffect` functie aan. We geven een **callback functie** mee als parameter. De functie die we meegeven wordt het **effect** genoemd. Wanneer React onze component rendert, onthoudt React het effect dat we hebben gedefinieerd en voert het het effect uit na het updaten van de DOM. Dit gebeurt standaard na elke render, ook na de eerste.

Start de app en bekijk de console. Voeg een transactie toe. We zien in de console dat `useEffect` na de initiële render en bij elke rerender wordt uitgevoerd.

> **Merk op:** React StrictMode (zie `main.jsx`) controleert of een component pure is door de component functie tweemaal aan te roepen. Dit gebeurt enkel in development mode, niet in productie. Dit is ook de reden waarom het loggen naar de console tweemaal gebeurt. Zie [Detecting impure calculations with StrictMode](https://react.dev/learn/keeping-components-pure) en [Why does my calculation runs twice](https://react.dev/apis/react/useMemo#my-calculation-runs-twice-on-every-rerender).

### Oefening 1 - useEffect in TransactionForm

Voeg dezelfde code toe aan de `TransactionForm` component. Wat wordt er eerst gerenderd?

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  De `TransactionForm` component wordt als eerste gerenderd en dan pas de `TransactionList`. Dit is logisch aangezien React eerst de kinderen rendert en zo omhoog beweegt tot aan de component die de render veroorzaakte.

### Effect dependencies

Stel dat we de boodschap enkel bij de initiële render wensen te loggen.

Aan de hand van een **dependency array** kan je het uitvoeren van een `useEffect` koppelen aan specifieke datawijzigingen. Zo voorkom je dat `useEffect` bij elke rerender opnieuw wordt uitgevoerd. Enkele voorbeelden:

- **[]**: een lege dependency array. `useEffect` wordt enkel bij de initiële render uitgevoerd

```jsx
useEffect(() => {
  console.log("transactions after the initial render");
}, [])
```

- **[transactions]**: `useEffect` wordt bij de initiële render en telkens de waarde van de variabele `transactions` wijzigt uitgevoerd. React zal het effect overslaan als `transactions` dezelfde waarde heeft als tijdens de laatste render.

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

Merk op: in een later hoofdstuk configureren we ESLint (een linter) die een waarschuwing (`React Hook useEffect has a missing dependency`) zal geven als de dependencies die je hebt opgegeven niet overeenkomen met wat ESLint verwacht op basis van de code in je effect. Dit helpt veel bugs in de code op te sporen. Als je effect een bepaalde waarde gebruikt, maar je het effect niet opnieuw wilt uitvoeren wanneer deze verandert moet je ervoor zorgen dat je effect geen gebruik maakt van deze dependency.

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

### Cleanup, indien nodig

Een side-effect kan een **cleanup functie** retourneren. React roept de cleanup functie elke keer aan voordat het effect opnieuw wordt uitgevoerd en een laatste keer wanneer de component wordt verwijderd (= on unmount).

Verwijder eerst de code m.b.t. de prop user en voeg dan een cleanup functie met een simpele `console.log` toe.

```jsx
useEffect(() => {
  console.log("transactions after initial render or transaction added");
  return () => console.log("unmounted..."); // 👈 de cleanup functie
}, [transactions]);
```

Start de app en bekijk de console. Je kan de cleanup functie triggeren door bv. een transactie toe te voegen.

### Opmerkingen useEffect

Er zijn een aantal opmerkingen om rekening mee te houden bij het gebruik van `useEffect`:

- Gebruik `useEffect` niet voor het aanbrengen van DOM-wijzigingen die zichtbaar zijn voor de gebruiker. Een `useEffect` wordt pas geactiveerd nadat de browser klaar is met de lay-out en het tekenen. Dit is dus te laat als je een visuele wijziging wilde aanbrengen. Voor die gevallen biedt React de hook `useLayoutEffect` die op dezelfde manier werken als `useEffect`. Ze verschillen enkel in het moment van 'afvuren'.
- Beperk het gebruik van `useEffect`, in de meeste gevallen heb je deze hook niet nodig. Je hebt het enkel nodig als je "uit de React code" wil stappen, bv. synchronisatie met een systeem in de cloud, synchronisatie met een niet-React DOM element... Probeer dus eerst je probleem op te lossen met andere hooks voor je terugvalt op `useEffect`, of gebruik een library die specifiek ontworpen is voor jouw probleem.
  - Voor extra uitleg en voorbeelden: [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- `useEffect` laat NIET toe om het keyword `async` toe te voegen in de callback function. Dit kan opgelost worden door in de effect-code een `async` functie te maken en die vervolgens aan te roepen. Dit is trouwens nog een reden waarom je best een library gebruikt voor het ophalen van data.

## GET /api/transactions (useEffect)

Maak een bestand `transactions.js` aan in de map `api`. Hierin plaatsen we alle requests naar de API [http://localhost:9000/api/transactions](http://localhost:9000/api/transactions):

```jsx
import axios from 'axios'; // 👈 1

const baseUrl = `http://localhost:9000/api/transactions`; // 👈 2

// 👇 3
export const getAll = async () => {
  const response = await axios.get(baseUrl);
  return response;
};
```

1. Importeer axios.
2. Houd het gemeenschappelijke deel van alle URLs bij in `baseUrl`.
3. De `getAll` functie maakt gebruik van `axios.get`, een asynchrone functie die een Promise retourneert (vandaar `async/await`). We retourneren voorlopig het volledige response. Later passen we dit aan.

Pas de `TransactionList` component aan. Het ophalen van de transacties is een side-effect, we maken hier voorlopig gebruik van de `useEffect` hook.

```jsx
import { useState, useMemo, useCallback, useEffect, useContext } from 'react'; // 👈 1
// andere imports...
import * as transactionsApi from '../../api/transactions'; // 👈 2

//...
export default function TransactionList() {
  // state

  useEffect(() => { // 👈 3
    // 👇 4
    const fetchTransactions = async () => {
      const data = await transactionsApi.getAll();
      console.log(data);
    };

    fetchTransactions();
  }, []); // 👈 3

  // memo, callbacks, JSX...
}
```

1. Importeer `useEffect`.
2. Importeer ook alle functies uit de Transaction API met `import * as`. Dit zal een object met alle geëxporteerde functies uit `transactions.js` maken.
3. Het ophalen van de transacties is een side-effect en dient enkel bij de eerste render te worden uitgevoerd. Daarom hebben we voor een lege dependency array.
4. Een `useEffect` mag geen async functie als argument krijgen.
   - Het probleem is nl. dat het eerste argument van `useEffect` een functie moet zijn die ofwel niets ofwel een functie retourneert (de cleanup functie). Maar een asynchrone functie retourneert een `Promise`, die niet als functie kan worden aangeroepen! Het is gewoon niet wat de `useEffect` hook verwacht als eerste argument.
   - We lossen dit op door een asynchrone functie te definiëren en deze vervolgens aan te roepen.

Start de applicatie en bekijk het response in de console.

![Response van GET /api/transactions](./images/response.png)

Een axios response bevat volgende informatie:

- `data`: de body van het HTTP response. Als dit JSON is, zal Axios dit automatisch parsen in een JavaScript object.
- `status`: de HTTP statuscode, bv. 200, 400, 404.
- `statusText`: het HTTP statusbericht, bv. OK, Bad Request, Not Found.
- `headers`: de HTTP headers uit het HTTP response.
- `config`: de configuratie die je meegaf aan de Axios API.
- `request`: het native request dat gebruikt werd. In Node.js is dit een `ClientRequest` object.

We zijn enkel geïnteresseerd in de `data.items` uit het response.

De transactions API voor `getAll` wordt dus:

```jsx
import axios from 'axios';

const baseUrl = `http://localhost:9000/api/transactions`;

export const getAll = async () => {
  const {
    data
  } = await axios.get(baseUrl); // 👈 1

  return data.items; // 👈 2
};
```

1. Gebruik destructuring om de data uit het response te halen.
2. Retourneer vervolgens `data.items`, of dus de lijst van transacties.

De `TransactionList` component wordt daardoor:

```jsx
//...
//import { TRANSACTION_DATA } from '../../api/mock_data'; // 👈 1
function TransactionTable({ transactions }) {
//...
    <tbody>
      {transactions.map((transaction) => (
        <Transaction key={transaction.id} {...transaction} /> {/* 👈 6 */}
      ))}
    </tbody>
//...
}

export default function TransactionList() {
  const [transactions, setTransactions] = useState([]); // 👈 2
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');

  useEffect(() => {
    const fetchTransactions = async () => {
      const data = await transactionsApi.getAll(); // 👈 3
      setTransactions(data); // 👈 4
    };

    fetchTransactions();
  }, []);

  const filteredTransactions = useMemo(() => transactions.filter((t) => {
    return t.place.name.toLowerCase().includes(search.toLowerCase()); // 👈 5
  }), [search, transactions])

  //...
}
```

1. We maken niet langer gebruik van mock data.
2. De initiële state is nu een lege array.
3. Haal de data asynchroon op. Omwille van performantieredenen kan je eventueel het aantal records beperken (server side). Dat laten we hier achterwege.
4. Pas de state aan nadat je de lijst terugkrijgt van de API.
5. Om te filteren op place gebruiken we nu `place.name` conform de nieuwe structuur van een transactie.
6. Ook in de `TransactionTable` kan nu de id gebruikt worden als key.

De JSON die verkregen wordt van de backend bevat een `user` en `place` object. Dus dienen we nu ook `Transaction.jsx` aan te passen:

```jsx
//...
export default memo(function Transaction({ user, amount, place, date }) {
  return (
    <tr>
      <td>
        {dateFormat.format(new Date(date))}
      </td>
      <td>{user.name}</td> {/* 👈 */}
      <td>{place.name}</td>{/* 👈 */}
      <td>
        {amountFormat.format(amount)}
      </td>
      <td>
        <div className="btn-group float-end">
        </div>
      </td>
    </tr>
  );
})
```

Start de applicatie en bekijk het resultaat.

## Laadindicator en foutafhandeling

Je merkt misschien dat je heel kort de melding krijgt dat er geen transacties zijn en dat vervolgens de lijst van transacties wordt weergegeven. Dit komt omdat de API call asynchroon is. We kunnen dit oplossen door een laadindicator toe te voegen. Daarnaast voegen we ook meteen een foutafhandeling toe, want we gaan er niet vanuit dat alles altijd goed gaat!

Omdat we de laadindicator en foutafhandeling in meerdere componenten nodig hebben, maken we hiervoor een aparte component `Loader` aan in een nieuw bestand `components/Loader.jsx`:

```jsx
export default function Loader() {
  return (
    <div className="d-flex flex-column align-items-center">
      <div className="spinner-border">
        <span className="visually-hidden">Loading...</span>
      </div>
    </div>
  );
}
```

Deze `Loader` component toont een simpele loading indicator van Bootstrap.

Ook zullen we foutafhandeling in meerdere componenten nodig hebben. Daarom maken we hiervoor een aparte component `Error` aan in een nieuw bestand `components/Error.jsx`:

```jsx
import { isAxiosError } from 'axios';

export default function Error({ error }) { // 👈 1
  if (isAxiosError(error)) { // 👈 2
    return (
      <div className="alert alert-danger">
        <h4 className="alert-heading">Oops, something went wrong</h4>
        <p>
          {/* 👇 3 */}
          {error.response?.data?.message || error.message}
          {error.response?.data?.details && (
            <>
              :
              <br />
              {JSON.stringify(error.response.data.details)}
            </>
          )}
        </p>
      </div>
    );
  }

  // 👇 4
  if (error) {
    return (
      <div className="alert alert-danger">
        <h4 className="alert-heading">An unexpected error occured</h4>
        {error.message || JSON.stringify(error)}
      </div>
    );
  }

  return null; // 👈 5
}
```

1. We geven de `error` prop mee aan de component.
2. We controleren of de `error` een `AxiosError` is. Dit is een specifiek type van error dat Axios gebruikt. We gebruiken de `isAxiosError` functie om dit te controleren.
3. In dat geval geven we de foutmelding weer. We controleren eerst of er een `message` is in de response data. Zo niet, dan geven we de `message` van de error weer. Als er een `details` object is, dan geven we dit ook weer. Dit `details` object voegen we in het hoofdstuk van validatie toe in de bijbehorende back-end, dus dit zal nu nog niet voorkomen.
4. Als er geen `AxiosError` is, dan geven we de `error` weer. We controleren eerst of er een `message` is. Zo niet, dan geven we de `error` zelf weer in JSON formaat.
5. Als er geen `error` is, dan geven we `null` terug.

Als laatste definiëren we een `AsyncData` component in een nieuw bestand `components/AsyncData.jsx`:

```jsx
import Loader from './Loader'; // 👈 1
import Error from './Error'; // 👈 1

export default function AsyncData({
  loading,  // 👈 2
  error,    // 👈 3
  children, // 👈 4
}) {
  // 👇 2
  if (loading) {
    return <Loader />;
  }

  return (
    <>
      <Error error={error} /> {/* 👈 3 */}
      {children} {/* 👈 4 */}
    </>
  );
}
```

1. Importeer de `Loader` en `Error` componenten.
2. We verwachten een `loading` prop. Als `loading` `true` is, dan geven we de `Loader` weer.
3. Daarnaast verwachten we een `error` prop. Geef een eventuele fout weer. Deze component zal `null` retourneren als er geen `error` is.
4. Als laatste verwachten we een `children` prop. Dit is de JSX die we willen weergeven als de data niet aan het laden is.

Pas dan volgende onderdelen van de `TransactionList` verder aan:

```jsx
import AsyncData from '../AsyncData'; // 👈 5
//...

export default function TransactionList() {
  const [transactions, setTransactions] = useState([]);
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');
  const [loading, setLoading] = useState(true); // 👈 1
  const [error, setError] = useState(null); // 👈 1

  useEffect(() => {
    const fetchTransactions = async () => {
      try { // 👈 2
        setLoading(true); // 👈 3
        setError(null); // 👈 3
        const data = await transactionsApi.getAll();
        setTransactions(data);
      } catch (error) { // 👈 4
        console.error(error);
        setError(error);
      } finally { // 👈 5
        setLoading(false);
      }
    }

    fetchTransactions();
  }, []);

  //...
  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={createTransaction} />
      <div className="input-group mb-3 w-50">
        <input type="search" id="search" className="form-control rounded" placeholder="Search" value={text} onChange={(e) => setText(e.target.value)} />
        <button type="button" className="btn btn-outline-primary" onClick={() => setSearch(text)}>Search</button>
      </div>
      <div className="mt-4">
        <AsyncData loading={loading} error={error}> {/* 👈 6 */}
          {!error ? <TransactionTable transactions={filteredTransactions} /> : null} {/* 👈 7 */}
        </AsyncData>
      </div>
    </>
  );
}
```

1. Definieer een `loading` en `error` state variabele om een loading indicator of een eventuele fout van de API weer te kunnen geven.
2. Voor de foutafhandeling maken we gebruik van try-catch.
3. Plaats de `loading` state op `true` en de `error` state op `null` bij een nieuw request.
4. Vang een eventuele fout op en stel de `error` state in.
5. Zet de `loading` state op `false` als het request beëindigd is, ook al was er een error.
6. We gebruiken de `AsyncData` component om de `loading` en `error` verder af te handelen.
7. Geef de transacties weer als er zich geen fout heeft voorgedaan.

Bekijk het resultaat in de applicatie. Het is normaal dat de applicatie crasht als je een transactie probeert toe te voegen. Dit komt omdat we de `createTransaction` functie nog niet hebben aangepast, die gebruikt nog een licht andere structuur voor een transactie.

## GET /api/transactions (swr)

Je merkt dat we heel wat code hebben moeten schrijven om de data op te halen. Daarom maken we gebruik van de `swr` library. Deze library is specifiek ontworpen voor data fetching. De library is gebaseerd op de [stale-while-revalidate](https://datatracker.ietf.org/doc/html/rfc5861) strategie. De data wordt eerst uit de cache gehaald (stale), dan wordt de request uitgevoerd (revalidate) en tenslotte wordt de up-to-date data opnieuw geretourneerd. We geven de hele verantwoordelijkheid voor het ophalen van de data, afhandelen van error states, caching... door aan de library.

We gooien om te beginnen volgende code uit de `TransactionList` component:

- `useEffect` om de transacties op te halen
- `useEffect` import
- `loading`, `error` en `transactions` state variabele

Om data op te halen, moeten we gebruik maken van de `useSWR` hook. Deze hook heeft drie parameters:

- `key`: een unieke key voor de data die je wil ophalen. Meestal wordt hiervoor de URL van het request gebruikt.
  - Het voordeel van de key is dat je van eender waar in de component tree kan aangeven dat bepaalde data ververst moet worden. Als we bv. een transactie toevoegen, kunnen we zeggen dat de lijst van transacties moet ververst worden. We komen hier later op terug.
- `fetcher`: een functie die de data ophaalt. Deze functie wordt asynchroon uitgevoerd en retourneert een Promise. De functie ontvangt de `key` als parameter.
- `options` (optioneel): een object met opties voor swr.

Eerst en vooral maken we een algemene functie voor een GET all request. Hernoem het bestand `transactions.js` naar `index.js`. Een GET all request zal in een RESTful API altijd hetzelfde response hebben, aanpassingen kunnen we altijd nog meegeven via de `key` of de `options`.

Pas het bestand aan als volgt:

```jsx
import axios from 'axios';

const baseUrl = `http://localhost:9000/api`; // 👈 1

export async function getAll(url) { // 👈 2
  const {
    data,
  } = await axios.get(`${baseUrl}/${url}`); // 👈 3

  return data.items;
}
```

1. We houden het gemeenschappelijke deel van alle URLs bij in `baseUrl`.
2. De parameter `url` zal van `swr` de `key` ontvangen. Zo hebben we meteen een uniek id voor elk request. We geven dus straks het laatste deel van de URL mee als `key`.
3. We voegen de `baseUrl` en de `url` samen om de volledige URL te bekomen.

Vervolgens gebruiken we de `useSWR` hook om onze transacties op te halen:

```jsx
// andere imports
import useSWR from 'swr'; // 👈 1
import { getAll } from '../../api'; // 👈 2

export default function TransactionsList() {
  // andere state variabelen
  // 👇 3
  const {
    data: transactions = [],
    isLoading,
    error
  } = useSWR('transactions', getAll);

  const createTransaction = useCallback(
    (user, place, amount, date) => {
      const newTransactions = [
        {
          user,
          place,
          amount,
          date: new Date(date),
        },
        ...transactions,
      ]; // newest first
      // 👇 5
      // setTransactions(newTransactions);
    },
    [transactions]
  );

  return (
    <>
      {/* ... */}

      <div className="mt-4">
        {/* 👇 4 */}
        <AsyncData loading={isLoading} error={error}>
          <TransactionsTable transactions={filteredTransactions} />
        </AsyncData>
      </div>
    </>
  );
}
```

1. Importeer de `useSWR` hook.
2. Importeer de `getAll` functie uit de `api/index.js`. Als je de naam van een map opgeeft, zal de `index.js` in die map geïmporteerd worden.
3. We gebruiken de `useSWR` hook met `transactions` als key en de `getAll` functie als `fetcher`. De `useSWR` hook retourneert een object met volgende properties:
   - `data`: de data die we ophalen. Dit is `undefined` als de data nog niet is opgehaald. We hernoemen deze property naar `transactions` en zetten de default waarde op een lege array.
   - `error`: de error die we ontvangen. Dit is `undefined` als er geen error is.
   - `isLoading`: een boolean die aangeeft of de data aan het ophalen is.
4. We gebruiken de `AsyncData` component om de `loading` en `error` verder af te handelen. We geven de `transactions` mee als `children`. We moeten hier enkel de naam van de variabele in de `loading` prop aanpassen naar `isLoading`.
5. Zet de `setTransactions` aanroep in `createTransaction` voorlopig in commentaar, we lossen dit later wel op.

### Oefening 2 - GET all in je eigen project

Implementeer een GET all van een willekeurige entiteit uit je eigen project:

- Installeer `swr` en `axios`.
- Voeg de componenten `Loader`, `Error` en `AsyncData` toe.
- Maak een functie aan in `api/index.js` die een GET all request uitvoert.
- Gebruik de `useSWR` hook om de data op te halen.
- Zorg ervoor dat je de data kan weergeven in jouw lijst-component.

## DELETE /api/transactions/:id

De volgende stap van de CRUD operaties is de 'D', een transactie verwijderen. Enerzijds moeten we een API call toevoegen, die het verwijderen effectief uitvoert, en deze beschikbaar maakt. Anderzijds moeten we deze op de juiste plaats gebruiken.

Voeg een `deleteById` functie toe in `index.js` in de map `api`:

```jsx
export const deleteById = async (url, { arg: id }) => { // 👈 1
  await axios.delete(`${baseUrl}/${url}/${id}`); // 👈 2
};
```

1. De parameter `url` zal van `swr` de `key` ontvangen. We krijgen ook het `id` mee als argument, we halen dit uit de `arg` optie die we van `swr` krijgen.
2. We bouwen de url (`/api/transactions/:id`) op en voeren de `DELETE` uit. Net zoals `axios.get()` kan je ook `axios.delete()` uitvoeren. Het antwoord is de HTTP status code 204. Bijgevolg is de HTTP response body ook leeg (m.a.w. `{}` in JavaScript). We negeren dat antwoord hier.

De `Transaction` component zelf is het meest geschikt om zijn eigen transactie te verwijderen. We voegen een verwijderknop toe aan deze component:

```jsx
import { memo, useCallback } from 'react'; // 👈 2
import { IoTrashOutline } from 'react-icons/io5'; // 👈 1
// ...

export default memo(function Transaction({ id, user, date, amount, place, onDelete }) { // 👈 3
  // 👇 2
  const handleDelete = useCallback(() => {
    onDelete(id);
  }, [id, onDelete]);

  return (
    <tr>
      <td>{dateFormat.format(new Date(date))}</td>
      <td>{user.name}</td>
      <td>{place.name}</td>
      <td> {amountFormat.format(amount)}</td>
      <td>
        {/* 👇 1 */}
        <button className='btn btn-primary' onClick={handleDelete}>
          <IoTrashOutline />
        </button>
      </td>
    </tr>
  );
});
```

1. Voeg een knop toe met een event handler `handleDelete`, met een icoon uit de `react-icons/io5` library.
2. Implementeer de `handleDelete` functie. Deze functie roept de `onDelete` functie aan met het `id` van de transactie.
3. De `Transaction` component ontvangt nu ook een `id` en `onDelete` prop.

We breiden de `TransactionTable` uit met een `onDelete` prop die we meteen doorgeven aan de `Transaction` component:

```jsx

function TransactionTable({ transactions, onDelete }) { // 👈
  // ...

  return (
    <div>
      <table className={`table table-hover table-responsive table-${theme}`}>
        <thead>
          <tr>
            <th>Date</th>
            <th>User</th>
            <th>Place</th>
            <th>Amount</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          {transactions.map((transaction) => (
            {/* 👇 */}
            <Transaction key={transaction.id} onDelete={onDelete} {...transaction} />
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

Nu zijn we klaar om de transactie effectief te verwijderen, we passen de `TransactionList` component aan. Het probleem van de `useSWR` hook is dat die meteen het request uitvoert als de component rendert. We willen dit pas doen als de gebruiker op de verwijderknop klikt. Daarom maken we gebruik van de `mutate` functie die we van `swr` krijgen. Deze functie zal de data in de cache aanpassen en de component opnieuw renderen.

```jsx
// imports...
import useSWRMutation from 'swr/mutation'; // 👈 1
import { getAll, deleteById } from '../../api'; // 👈 1

// TransactionTable

export default function TransactionList() {
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');
  const { data: transactions = [], isLoading, error } = useSWR('transactions', getAll);
  // 👇 2
  const { trigger: deleteTransaction, error: deleteError } = useSWRMutation('transactions', deleteById);

  // ...

  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={createTransaction} />
      <div className='input-group mb-3 w-50'>
        <input
          type='search'
          id='search'
          className='form-control rounded'
          placeholder='Search'
          value={text}
          onChange={(e) => setText(e.target.value)}
        />
        <button
          type='button'
          className='btn btn-outline-primary'
          onClick={() => setSearch(text)}
        >
          Search
        </button>
      </div>
      <div className='mt-4'>
        {/* 👇 4 */}
        <AsyncData loading={isLoading} error={error || deleteError}>
          {/* 👇 3 */}
          {!error ? <TransactionTable transactions={filteredTransactions} onDelete={deleteTransaction} /> : null}
        </AsyncData>
      </div>
    </>
  );
}
```

1. Importeer de `useSWRMutation` hook en de `deleteById` functie.
2. Gebruik de `useSWRMutation` hook. We geven als key `transactions` mee en als fetcher onze `deleteById` functie. We krijgen o.a. terug:
   - `trigger`: een functie die we kunnen aanroepen om het request effectief uit te voeren en dus de data te verwijderen. Deze functie ontvangt de `id` van de transactie als argument. We hernoemen dit naar `deleteTransaction`.
   - `error`: een eventuele fout die zich voordoet bij het verwijderen van de transactie. We hernoemen deze naar `deleteError`.
3. We geven de `deleteTransaction` functie mee als `onDelete` prop aan de `TransactionTable` component.
4. We voegen de `deleteError` toe aan `error` prop aan de `AsyncData` component. Deze zal dus de fout tonen als er een fout is bij het ophalen van de transacties of bij het verwijderen van een transactie.

Bekijk het resultaat in de applicatie, je zou een transactie moeten kunnen verwijderen. In een meer realistische applicatie zou je een bevestiging moeten vragen aan de gebruiker, dat laten we even achterwege hier.

Open de console en inspecteer het `Network` tabblad. Je zal zien dat er een `DELETE` request wordt uitgevoerd naar de API en dat meteen daarna de lijst van transacties opnieuw wordt opgehaald. Dit is de `stale-while-revalidate` strategie die `swr` gebruikt. De data wordt eerst uit de cache gehaald (stale), dan wordt de request uitgevoerd (revalidate) en tenslotte wordt de up-to-date data opnieuw geretourneerd. `swr` weet dat er iets gewijzigd is aangezien een mutation uitgevoerd is met dezelfde key als de hook die onze data ophaalt. Handig, he?

### Oefening 3 - DELETE in je eigen project

Implementeer een willekeurige DELETE uit je eigen project (liefst dezelfde entiteit als hiervoor):

- Maak een functie aan in `api/index.js` die een DELETE request uitvoert.
- Gebruik de `useSWRMutation` hook om de data te verwijderen.
- Zorg ervoor dat je de data kan verwijderen uit jouw lijst-component.

## POST /api/transactions

De volgende stap van de CRUD operaties is de 'C', een nieuwe transactie aanmaken.

Pas `index.js` in de map `api` aan:

```jsx
export const save = async (url, { arg: body }) => { // 👈 1
  await axios.post(`${baseUrl}/${url}`, body); // 👈 2
};
```

1. De parameter `url` zal van `swr` de `key` ontvangen. We krijgen ook de `transaction` mee als argument, we hernoemen de `arg` optie die we van `swr` krijgen voor de duidelijkheid.
2. We voeren een `POST` request uit naar de API. Axios zal de `transaction` automatisch omzetten naar JSON en versturen als body van het HTTP request. Het antwoord heeft als HTTP status code 200 en als response body de nieuw gecreëerde transactie. We negeren dat antwoord hier.

De creatie van een transactie gebeurt in de `TransactionForm` component. De state wordt bijgehouden in de `TransactionList` component, maar die zal automatisch geüpdatet worden als we dezelfde key doorgeven aan `swr`. We maken een nieuwe mutation in `TransactionForm`:

```jsx
// imports...
import useSWRMutation from 'swr/mutation'; // 👈 1
import { save } from '../../api'; // 👈 1
import Error from '../Error'; // 👈 5

export default function TransactionForm() { // 👈 4
  const {
    trigger: saveTransaction,
    error: saveError,
  } = useSWRMutation('transactions', save); // 👈 2

  // ...

  const onSubmit = useCallback(async (data) => { // 👈 3
    const { user, place, amount, date } = data;
    await saveTransaction({ user, place, amount: parseInt(amount), date }); // 👈 3
    reset();
  }, [reset, saveTransaction]); // 👈 3

  // ...
  return (
    <>
      <h2>Add transaction</h2>
      {/* 👇 5  */}
      <Error error={saveError} />

      {/* ... */}
    </>
  );
}
```

1. Importeer de `useSWRMutation` hook en de `save` functie.
2. Maak een trigger-functie die een transactie zal opslaan. We gebruiken dezelfde key als bij het ophalen van de transacties, dus `transactions`. We geven als fetcher onze `save` functie mee. We krijgen o.a. terug:
   - `trigger`: een functie die we kunnen aanroepen om het request effectief uit te voeren en dus de data te verwijderen. Deze functie ontvangt de `transaction` als argument. We hernoemen dit naar `saveTransaction`.
   - `error`: een eventuele fout die zich voordoet bij het opslaan van de transactie. We hernoemen deze naar `saveError`.
3. Wijzig de `onSubmit` zodat `saveTransaction` aangeroepen wordt i.p.v. `onSaveTransaction`. We geven de `user`, `place`, `amount` en `date` mee als argumenten als **object** deze keer.
   - Pas ook de dependency array aan van de `useCallback` hook.
   - **Let op:** de functie is nu `async`, dus we moeten `await` gebruiken.
4. Verwijder de `onSaveTransaction` prop, deze is niet meer nodig. Verwijder ook meteen de `createTransaction` functie en het doorgeven van deze functie aan de `TransactionForm` component uit de `TransactionList` component.
5. We tonen een eventuele fout na het opslaan d.m.v. de `Error` component. Vergeet de import niet!

De API verwacht een `id` voor de place. We retourneren `placeId` als geselecteerde waarde in de select list. In het formulier moet je vanaf nu in de select lijst werken met het `id`.

```jsx
function PlacesSelect({ name, places }) {
  //..

  return (
    {/* ... */}
    <select
      {...register(name, validationRules.place)}
      id="places"
      className="form-select"
    >
      <option defaultChecked value="">-- Select a place --</option>
      {PLACE_DATA.map(({ id, name }) => (
        <option key={id} value={id}>{name}</option>{/* 👈 */}
      ))}
    </select>
    {hasError ? (
      <div className="form-text text-danger">
        {errors[name].message}
      </div>
    ) : null}
    {/* ... */}
  );
}
```

Idem voor de `user` verwacht de API een id, we passen de validatieregels aan:

```js
const validationRules = {
  user: {
    required: 'User is required',
    min: { value: 1, message: 'min 1' }, // 👈
  },
  // ..
};
```

En ook de definitie van de `LabelInput` component voor de gebruiker, nu moeten we hier een id (= getal) invullen i.p.v. een naam:

```jsx
<LabelInput
  label='User ID' // 👈
  name='user'
  type='number' // 👈
  validationRules={validationRules.user}
/>
```

De `onSubmit` (in `TransactionForm`) wordt dan:

```js
const onSubmit = async (data) => {
  const { user, place, amount, date } = data;
  await saveTransaction({
    userId: user,  // 👈
    placeId: place, // 👈
    amount: parseInt(amount),
    date: new Date(date)
  })
  reset();
};
```

Later verwijderen we het user-veld, daarom doen we hier geen moeite om de gebruiker op te zoeken. We geven gewoon het id mee.

### Oefening 4 - PlacesSelect via API

Momenteel werken we nog met de mock data voor de places. Pas deze component aan zodat deze gebruik maakt van dezelfde id's als in de places tabel:

- Gebruik de `useSWR` hook om de places op te halen in de `TransactionForm` component.
  - Hergebruik de GET all uit `api/index.js`.
- Vervang de doorgegeven mock data door de opgehaalde data (in de props van `PlacesSelect`).
- Controleer of de places correct weergegeven worden.

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  Voeg onderstaande code toe aan `TransactionForm`:

  ```jsx
  // ...
  import { getAll, save } from '../../api'; // 👈 1
  import useSWR from 'swr'; // 👈 1

  export default function TransactionForm() {
    // 👇 2
    const {
      data: places = [],
    } = useSWR('places', getAll);

    // ...
    return (
      <>
        {/* ... */}
        <div className='mb-3'>
          <PlacesSelect name='place' places={places} /> {/* 👈 3 */}
        </div>
        {/* ... */}
      </>
    );
  }
  ```

  1. Importeer de `getAll` en `save` functies uit `api/index.js` en de `useSWR` hook.
  2. Haal de places op van de API. We gebruiken `places` als key, hier zal de `getAll` functie de `baseUrl` aan toevoegen.
  3. Geef de `places` mee als `places` prop aan de `PlacesSelect` component. Verwijder nu de import van de mock data.

  Controleer of de places correct weergegeven worden.

### Oefening 4 - POST in je eigen project

Implementeer een willekeurige POST uit je eigen project (liefst dezelfde entiteit als hiervoor):

- Maak een functie aan in `api/index.js` die een POST request uitvoert.
- Gebruik de `useSWRMutation` hook om de data toe te voegen.
- Controleer of je formulier nog steeds een item kan toevoegen.

## PUT /api/transactions/:id

Dan rest nog de 'U' van CRUD, maar die is een beetje speciaal. De API call zelf is geen probleem, dit is basically hetzelfde als 'Create' maar met een extra `id` parameter.

Maar we willen natuurlijk niet dat een gebruiker een volledig object juist moet invoeren om het aan te passen. We willen dat hij ergens op 'bewerk' kan klikken bij een bestaand element.

In ons geval willen we dat de "potlood-knop" in de lijst deze data in het formulier ingeeft. De gebruiker past vervolgens aan en doet een update. Maar om dat te doen werken moeten er een paar dingen gebeuren.

We moeten op een manier een transactie kunnen doorgeven van onze `Transaction` naar onze `TransactionForm` component. Daarnaast moeten we in `TransactionsForm` het onderscheid kunnen maken tussen een bestaande transactie updaten, of een nieuwe toevoegen.

Wat de eigenlijk moeten doen is:

- een 'huidige transactie' als state toe te voegen aan onze `TransactionList`;
- evenals een functie om dit in te stellen.

We stellen de state in als er op de potlood-knop geklikt wordt. In het formulier bekijken we via een `useEffect` bij elke render of er een huidige transactie is ingesteld en indien nodig tonen we de bijhorende gegevens in alle formuliervelden.

In de API kan je een aparte functie aanmaken om iets te updaten:

```jsx
export const updateById = async (url, { arg: body }) => {
  const { id, ...values } = body;
  await axios.put(`${baseUrl}/${url}/${id}`, values);
};
```

Of we kunnen de `save` functie aanpassen:

```jsx
export const save = async (url, { arg: body }) => {
  const { id, ...values } = body;
  await axios({
    method: id ? 'PUT' : 'POST',
    url: `${baseUrl}/${url}/${id ?? ''}`,
    data: values,
  });
};
```

Wij kiezen voor de laatste (compacte) oplossing.

Aan de `TransactionList` component voegen we de `currentTransaction` als state toe en geven de benodigde handlers door aan de child components:

```jsx
export default function TransactionList() {
  //...
  const [currentTransaction, setCurrentTransaction] = useState({}); // 👈 1

  // 👇 2
  const setTransactionToUpdate = useCallback((id) => {
    setCurrentTransaction(id === null ? {} : transactions.find((t) => t.id === id));
  }, [transactions]);
  //...

  return (
    <>
      <h1>Transactions</h1>
      {/* 👇 3 */}
      <TransactionForm
        setTransactionToUpdate={setTransactionToUpdate}
        currentTransaction={currentTransaction}
      />

      {/* ... */}
      <div className='mt-4'>
        <AsyncData loading={isLoading} error={error || deleteError}>
          {!error ? (
            <TransactionTable
              transactions={filteredTransactions}
              onDelete={deleteTransaction}
              onEdit={setTransactionToUpdate} // 👈 4
            />
          ) : null}
        </AsyncData>
      </div>
   </>
  );
}
```

1. Houd de huidige transactie bij in een state variabele.
2. Maak een functie waarmee je de huidige transactie kan instellen of verwijderen, o.b.v. een gegeven id.
3. Geef de `currentTransaction` en `setTransactionToUpdate` door aan de `TransactionForm`. De namen van deze props maken momenteel niet veel uit aangezien we dit na het hoofdstuk routing toch zullen verwijderen.
4. Voeg een `onEdit` event handler prop toe aan de `TransactionTable`.

In de `TransactionTable` component moeten we vervolgens de `onEdit` linken:

```jsx
function TransactionTable({
  transactions,
  onEdit, // 👈 1
  onDelete
}) {
  const { theme } = useContext(ThemeContext);

  if (transactions.length === 0) {
    return (
      <div className="alert alert-info">
        There are no transactions yet.
      </div>
    );
  }

  return (
    <div>
      <table className={`table table-hover table-responsive table-${theme}`}>
        <thead>
          <tr>
            <th>Date</th>
            <th>User</th>
            <th>Place</th>
            <th>Amount</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
        {transactions.map((transaction) => (
            <Transaction
              {...transaction}
              key={transaction.id}
              onDelete={onDelete}
              onEdit={onEdit} // 👈 2
            />
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

1. Deze component ontvangt nu ook een prop `onEdit`.
2. Deze event handler prop wordt doorgegeven aan de `Transaction` component via de `onEdit` prop.

Als laatste moeten we in de `Transaction` component een potlood-icoon toevoegen met een `onClick` handler:

```jsx
import { IoTrashOutline, IoPencil } from 'react-icons/io5'; // 👈 2
//...

export default memo(function Transaction({
  id,
  date,
  amount,
  user,
  place,
  onDelete,
  onEdit // 👈 1
}) {
  const handleDelete = useCallback(() => {
    onDelete(id);
  }, [id, onDelete]);

  // 👇 3
  const handleEdit = useCallback(() => {
    onEdit(id);
  }, [id, onEdit]);

  return (
    <tr>
      {/* ... */}
      <td>
        <div className="btn-group float-end">
          {/* 👇 3 */}
          <button type="button" className="btn btn-light" onClick={handleEdit}>
            <IoPencil />
          </button>
          <button type="button" className="btn btn-danger" onClick={handleDelete}>
            <IoTrashOutline />
          </button>
        </div>
      </td>
    </tr>
  );
})
```

1. Deze component krijgt ook een `onEdit` prop.
2. We importeren het potlood-icoon.
3. En voegen de knop met het potlood-icoon en een `onClick` handler toe.

In het `TransactionForm` passen we de `useEffect` aan zodat bij elke render het volgende gecontroleerd wordt: als `currentTransaction` is ingevuld dan gaat het om een update, anders om een create.

```jsx
import { useEffect } from 'react'; // 👈 2
//..

export default function TransactionForm({ currentTransaction, setTransactionToUpdate }) { // 👈 1
  const {
    register,
    handleSubmit,
    reset,
    setValue, // 👈 2
    formState: { errors },
  } = useForm();

  const onSubmit = useCallback(async (data) => {
    const { user, place, amount, date } = data;
    await saveTransaction({
      user,
      placeId: place,
      amount: parseInt(amount),
      date,
      id: currentTransaction?.id, // 👈 4
    });
    setTransactionToUpdate(null); // 👈 5
  }, [reset, saveTransaction]);

  // 👇 2
  useEffect(() => {
    if (
      // check on non-empty object
      currentTransaction &&
      (Object.keys(currentTransaction).length !== 0 ||
          currentTransaction.constructor !== Object)
    ) {
      const dateAsString = toDateInputString(new Date(currentTransaction.date));
      setValue("date", dateAsString);
      setValue("userId", currentTransaction.user.name);
      setValue("placeId", currentTransaction.place.id);
      setValue("amount", currentTransaction.amount);
    } else {
      reset();
    }
  }, [currentTransaction, setValue, reset]);

  //..
  return (
    {/* ... */}
    {/* 👇  */}
    <button type='submit' className='btn btn-primary'>
      {currentTransaction?.id
        ? "Save transaction"
        : "Add transaction"}
    </button>
    {/* ... */}
  );
}
```

1. Ontvang `currentTransaction` en `setTransactionToUpdate` door als prop.
2. Controleer bij elke render of `currentTransaction` is ingevuld.
   - Indien ingevuld, plaats de waarden in het formulier. Maak hiervoor gebruik van `setValue` uit de `useForm` hook.
   - Indien niet ingevuld, maak het formulier leeg.
3. Pas de tekst op de knop aan i.f.v. of het om een update of een create gaat.
4. Bij het aanpassen van een transactie voegen we ook het id toe.
5. In plaats van `reset()` aan te roepen, maken we de huidige transactie leeg. Ons effect zal automatisch het formulier leegmaken.

Je kan er ook voor zorgen dat de inputvelden en knoppen in het formulier _disabled_ worden als het formulier gesubmit wordt.
`useForm` geeft een boolean [isSubmitting](https://react-hook-form.com/api/useform/formstate) terug die `true` is als het formulier gesubmit wordt en `false` bij een reset.

```jsx
//..
function LabelInput({ label, name, type, ...rest }) {
  const {
    register,
    errors,
    isSubmitting // 👈 4
  } = useFormContext();

  const hasError = name in errors;

  return (
    <div className="mb-3">
      <label htmlFor={name} className="form-label">
        {label}
      </label>
      <input
        {...register(name, validationRules[name])}
        id={name}
        type={type}
        disabled={isSubmitting} // 👈 4
        className="form-control"
        {...rest}
      />
      {hasError ? (
        <div className="form-text text-danger">
          {errors[name].message}
        </div>
      ) : null}
    </div>
  );
}
//..

export default function TransactionForm({ currentTransaction, onSaveTransaction }) {
  const {
    register,
    handleSubmit,
    reset,
    setValue,
    formState: { errors, isSubmitting }, // 👈 1
    isSubmitting,
  } = useForm();
  //...
 const onSubmit = useCallback(async (data) => {
    const { user, place, amount, date } = data;
    try{ // 👈 5
    await saveTransaction({
      userId: user,
      placeId: place,
      amount: parseInt(amount),
      date,
      id: currentTransaction?.id,
    });
    setTransactionToUpdate(null);
    catch(error){
      console.log(error)
    }
  }, [saveTransaction]);
  //...
  return (
    <FormProvider
      handleSubmit={handleSubmit}
      errors={errors}
      register={register}
      isSubmitting={isSubmitting} // 👈 2
    >
      {/* ... */}
      <button
        type='submit'
        className='btn btn-primary'
        disabled={isSubmitting} // 👈 3
      >
        {currentTransaction?.id
          ? "Save transaction"
          : "Add transaction"}
      </button>
      {/* ... */}
    </FormProvider>
  );
}
```

1. Haal ook `isSubmitting` op van `useForm` en geef dit mee aan de `FormProvider`.
2. Vermits ook de componenten `LabelInput` en `PlacesSelect` hier gebruik van maken, dient de `FormProvider` hierin te voorzien.
   - Tip: je kan ook alles uit de `useForm` hook verzamelen in een object en doorgeven aan de `FormProvider` met de spread operator. Je moet vervolgens enkel in deze component destructuren wat je nodig hebt. Zo kan je ook niets vergeten door te geven.
3. Disable de knop tijdens submit.
4. Disable het inputveld tijdens submit, doe hetzelfde voor de `PlacesSelect` component.
5. Zorg voor foutafhandeling als de opslag mislukt (fout aan de api-kant), anders blijft isSubmitting true en kan de gebruiker de waarden niet meer aanpassen

### Oefening 5 - PUT in je eigen project

Implementeer een willekeurige PUT uit je eigen project (liefst dezelfde entiteit als hiervoor):

- Maak een functie aan in `api/index.js` die een PUT request uitvoert.
- Pas de code in het formulier aan zodat je nu ook een item kan aanpassen.
- Controleer of je formulier nog steeds een item kan toevoegen, en nu ook een item kan aanpassen.

### Oefening 6 - PlacesSelect via API

Doe hetzelfde voor `PlacesSelect`, disable dit veld tijdens het verzenden van het formulier. Verwijder vervolgens de mock data uit het project, dit hebben we niet meer nodig.

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  Een voorbeeldoplossing is te vinden op <https://github.com/HOGENT-Web/frontendweb-budget> in commit `1daa903`.

  ```bash
  git clone https://github.com/HOGENT-Web/frontendweb-budget.git
  cd frontendweb-budget
  git checkout -b oplossing 1daa903
  yarn install
  yarn dev
  ```

## Het .env bestand

Je kan omgevingsvariabelen definiëren in het `.env` bestand. De omgevingsvariabelen moeten beginnen met `VITE_`, alle andere variabelen behalve `NODE_ENV` worden genegeerd. Als je omgevingsvariabelen wijzigt, moet je de applicatie _niet_ opnieuw starten. Vite zal de wijzigingen automatisch detecteren en het nodige doen.

De omgevingsvariabelen zijn beschikbaar via het object `import.meta.env`. Een omgevingsvariabele met de naam `VITE_NOT_SECRET_CODE` wordt in de code `import.meta.env.VITE_NOT_SECRET_CODE`.

Er is ook een ingebouwde omgevingsvariabele met de naam `NODE_ENV`. Wanneer je `yarn dev` uitvoert, is `NODE_ENV` altijd gelijk aan `development`.

De omgevingsvariabelen worden toegevoegd aan de code _at build time_. Aangezien Vite een statische HTML/CSS/JS-bundel produceert, kan het deze onmogelijk tijdens runtime lezen. Daarom staan de waarden letterlijk in de code, plaats hier dus geen API keys en andere geheime sleutels in.

Voeg een `.env` file toe in de root folder met de environment settings. Hierin definiëren we de url naar de API:

```dotenv
VITE_API_URL='http://localhost:9000/api'
```

In de code van `api/index.js` vervang je `baseUrl` door

```js
const baseUrl = import.meta.env.VITE_API_URL;
```

Over environment variables in React & Vite vind je meer op <https://vitejs.dev/guide/env-and-mode.html>.

## Oefening 7 - PlacesList via API

Pas nu ook `PlacesList` aan zodat dit werkt met onze REST API.

## Must reads

- [SOLID principles in React](https://konstantinlebedev.com/solid-in-react/)
- [Good advice on JSX conditionals](https://blog.thoughtspile.tech/2022/01/17/jsx-conditionals/)

## Mogelijke extra's voor de examenopdracht

- [react-query](https://www.npmjs.com/package/react-query)
- [react-error-boundary](https://github.com/bvaughn/react-error-boundary)
