# Data ophalen uit een REST API

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les4 5a31e56
> pnpm install
> pnpm dev
> ```
>
> Vanaf dit hoofdstuk heb je de bijbehorende backend nodig. Maak een database `budget` aan. Zie instructies in de [cursus Webservices](https://hogent-frontendweb.github.io/webservices-cursus/#/4-datalaag/deel1?id=mysql-databank) :
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/webservices-budget.git
> cd webservices-budget
> git checkout -b les5 d486627
> pnpm install
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#webservices-budget) voor meer informatie.

In dit hoofdstuk vervangen we de mock data door HTTP requests naar de REST API. Op ons lokaal toestel draait deze API op [http://localhost:9000/api/](http://localhost:3000/api/).

Voor de communicatie met de API, m.a.w. het versturen van HTTP requests, kan je gebruik maken van de [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) of van HTTP client libraries die je kan vinden op bv. <https://www.npmjs.com>.

Wij zullen gebruik maken van [swr](https://www.npmjs.com/package/swr), een _React Hooks library for data fetching_. Het wordt ontwikkeld door Vercel, het bedrijf achter Next.js. In de documentatie lezen we de betekenis van de naam van het package:

> The name “SWR” is derived from `stale-while-revalidate`, a cache invalidation strategy popularized by [HTTP RFC 5861](https://datatracker.ietf.org/doc/html/rfc5861). SWR first returns the data from cache (stale), then sends the request (revalidate), and finally comes with the up-to-date data again.

Naast SWR hebben we ook [axios](https://www.npmjs.com/package/axios) nodig, een HTTP client library die we gebruiken om de HTTP requests uit te voeren. SWR heeft namelijk geen ingebouwde HTTP client, je bent dus vrij om te kiezen welke HTTP client je gebruikt. In tegenstelling tot `swr` kan je `axios` ook gebruiken in een Node.js omgeving, om bv. data op te halen uit een third party API.

Installeer alvast `axios` en `swr`:

```bash
pnpm add axios swr
```

## useEffect

`swr` verbergt namelijk heel wat van de complexiteit omtrent API calls. Daarom tonen we eerst hoe je zonder externe libraries een HTTP request kan uitvoeren. Hiervoor maken we gebruik van de `useEffect` hook.

Effecten worden gebruikt om uit je React-code te stappen en te synchroniseren met een extern systeem. Ze voeren een **side-effect** uit. Neveneffecten zijn acties die buiten het renderproces van React vallen, zoals het ophalen van gegevens, het aanroepen van browser API's, widgets van derden, netwerken... Tegenwoordig wordt afgeraden om `useEffect` te gebruiken aangezien veel developers de hook gebruiken waarvoor die niet gemaakt is (zie <https://www.youtube.com/watch?v=bGzanfKVFeU>).

In deze sectie werken we richting een voorbeeld van data fetching m.b.v. `useEffect`. Het uiteindelijke doel is om `useEffect` te vervangen door een library (hier dus `swr`) die specifiek ontworpen is voor data fetching, net zoals de [React docs aanbevelen](https://react.dev/reference/react/useEffect#fetching-data-with-effects).

`useEffect` is een functie die asynchroon wordt uitgevoerd na de render, en die zichzelf optioneel kan opruimen (= **cleanup**). Het opruimen gebeurt voordat het effect opnieuw wordt uitgevoerd en voor de **unmounting** (= het vernietigen van de component). React onthoudt de functie die je hebt doorgegeven ​​(we noemen dit ons "effect") en roept deze later aan, na het uitvoeren van de DOM-updates.

In onderstaand voorbeeld wordt een boodschap naar de console gelogd nadat de `TransactionList` gerenderd is. Deze instructie zouden we na de return kunnen plaatsen, maar die code wordt niet uitgevoerd. `useEffect` is hier de oplossing.

```tsx
// src/pages/TransactionList.tsx
import { TRANSACTION_DATA } from '../api/mock_data';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { useState, useMemo, useEffect } from 'react'; // 👈 1
import TransactionsTable from '../components/transactions/TransactionsTable'; //

export default function TransactionList() {
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');

  // 👇 2
  useEffect(() => {
    console.log('transactions are rendered');
  });

  //...
}
```

1. We importeren `useEffect`.
2. Binnen de component roepen we de `useEffect` functie aan. We geven een **callback functie** mee als parameter. De functie die we meegeven wordt het **effect** genoemd. Wanneer React onze component rendert, onthoudt React het effect dat we hebben gedefinieerd en voert het het effect uit na het updaten van de DOM. Dit gebeurt standaard na elke render, ook na de eerste.

Start de app en bekijk de console. Geef een zoekterm in. We zien in de console dat `useEffect` na de initiële render en bij elke rerender wordt uitgevoerd.

> **Merk op:** React StrictMode (zie `main.tsx`) controleert of een component pure is door de component functie tweemaal aan te roepen. Dit gebeurt enkel in development mode, niet in productie. Dit is ook de reden waarom het loggen naar de console tweemaal gebeurt. Zie [Detecting impure calculations with StrictMode](https://react.dev/learn/keeping-components-pure) en [Why does my calculation runs twice](https://react.dev/reference/react/useMemo#my-calculation-runs-twice-on-every-re-render).

### Effect dependencies

Stel dat we de boodschap enkel bij de initiële render wensen te loggen.

Aan de hand van een **dependency array** kan je het uitvoeren van een `useEffect` koppelen aan specifieke datawijzigingen. Zo voorkom je dat `useEffect` bij elke rerender opnieuw wordt uitgevoerd. Enkele voorbeelden:

- **[]**: een lege dependency array. `useEffect` wordt enkel bij de initiële render uitgevoerd

```tsx
useEffect(() => {
  console.log('transactions after the initial render');
}, []);
```

- **[search]**: `useEffect` wordt bij de initiële render en telkens de waarde van de variabele `search` wijzigt uitgevoerd. React zal het effect overslaan als `search` dezelfde waarde heeft als tijdens de laatste render.

```tsx
useEffect(() => {
  console.log('transactions after initial render or transaction added');
}, [search]);
```

- **meerdere dependencies**: React zal het opnieuw uitvoeren van het effect alleen overslaan als alle dependencies die je opgeeft exact dezelfde waarden hebben als tijdens de vorige render.

Stel dat de user via een prop wordt doorgegeven aan de `TransactionList` en ook naar de console gelogd dient te worden.

```tsx
export default function TransactionList({ user = 'Louis' }: { user: string }) {
  // 👆 de prop user

  useEffect(() => {
    console.log(
      `Hi  ${user}, transactions after initial render or search changed`,
    ); // 👆 maakt gebruik van user
  }, [search]); // Warning: React Hook useEffect has a missing dependency

  //...
}
```

Merk op: We hebben ESLint geconfigureerd zodat die een waarschuwing (`React Hook useEffect has a missing dependency`) zal geven als de dependencies die je hebt opgegeven niet overeenkomen met wat ESLint verwacht op basis van de code in je effect. Dit helpt veel bugs in de code op te sporen. Als je effect een bepaalde waarde gebruikt, maar je het effect niet opnieuw wilt uitvoeren wanneer deze verandert moet je ervoor zorgen dat je effect geen gebruik maakt van deze dependency.

Oplossing:

```tsx
export default function TransactionList({ user = 'Louis' }: { user: string }) {

  useEffect(() => {
    console.log(`Hi ${user}, transactions after initial render or transaction added`);
  }, [search, user]) // 👈 meerdere dependencies

  //  JSX...
```

React vergelijkt de dependency waarden met behulp van [Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Voor arrays en objecten wordt hier bijgevolg gekeken naar de referentie, en niet naar de exacte waarde!

### Cleanup, indien nodig

Een side-effect kan een **cleanup functie** retourneren. React roept de cleanup functie elke keer aan voordat het effect opnieuw wordt uitgevoerd en een laatste keer wanneer de component wordt verwijderd (= on unmount).

Verwijder eerst de code m.b.t. de prop user en voeg dan een cleanup functie met een simpele `console.log` toe.

```tsx
useEffect(() => {
  console.log('transactions after initial render or search changed');
  return () => console.log('unmounted...'); // 👈 de cleanup functie
}, [search]);
```

Start de app en bekijk de console. Je kan de cleanup functie triggeren door te zoeken.

### Opmerkingen useEffect

Er zijn een aantal opmerkingen om rekening mee te houden bij het gebruik van `useEffect`:

- Gebruik `useEffect` niet voor het aanbrengen van DOM-wijzigingen die zichtbaar zijn voor de gebruiker. Een `useEffect` wordt pas geactiveerd nadat de browser klaar is met de lay-out en het tekenen. Dit is dus te laat als je een visuele wijziging wilde aanbrengen. Voor die gevallen biedt React de hook `useLayoutEffect` die op dezelfde manier werken als `useEffect`. Ze verschillen enkel in het moment van 'afvuren'.
- Beperk het gebruik van `useEffect`, in de meeste gevallen heb je deze hook niet nodig. Je hebt het enkel nodig als je "uit de React code" wil stappen, bv. synchronisatie met een systeem in de cloud, synchronisatie met een niet-React DOM element... Probeer dus eerst je probleem op te lossen met andere hooks voor je terugvalt op `useEffect`, of gebruik een library die specifiek ontworpen is voor jouw probleem.
  - Voor extra uitleg en voorbeelden: [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- `useEffect` laat **niet** toe om het keyword `async` toe te voegen in de callback function. Dit kan opgelost worden door in de effect-code een `async` functie te maken en die vervolgens aan te roepen. Dit is trouwens nog een reden waarom je best een library gebruikt voor het ophalen van data.

## GET /api/transactions (useEffect)

Maak een bestand `transactions.ts` aan in de map `api`. Hierin plaatsen we alle requests naar de API [http://localhost:9000/api/transactions](http://localhost:9000/api/transactions):

```ts
import axios, { type AxiosResponse } from 'axios'; // 👈 1

const baseUrl = 'http://localhost:9000/api/transactions'; // 👈 2

// 👇 3
export const getAll = async (): Promise<AxiosResponse> => {
  const response = await axios.get(baseUrl);
  return response;
};
```

1. Importeer axios.
2. Houd het gemeenschappelijke deel van alle URLs bij in `baseUrl`.
3. De `getAll` functie maakt gebruik van `axios.get`, een asynchrone functie die een Promise retourneert (vandaar `async/await`). We retourneren voorlopig het volledige response. Later passen we dit aan.

Pas de `TransactionList` component aan. Het ophalen van de transacties is een side-effect, we maken hier voorlopig gebruik van de `useEffect` hook.

```tsx
import { useState, useMemo, useEffect } from 'react'; // 👈 1
// andere imports...
import * as transactionsApi from '../api/transactions'; // 👈 2

//...
export default function TransactionList() {
  // state

  // 👇 3
  useEffect(() => {
    // 👇 4
    const fetchTransactions = async () => {
      const data = await transactionsApi.getAll();
      console.log(data);
    };

    fetchTransactions();
  }, []); // 👈 3

  //  JSX...
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

```tsx
import axios from 'axios';

const baseUrl = 'http://localhost:9000/api/transactions';

export const getAll = async (): Promise<Transaction[]> => {
  const { data } = await axios.get(baseUrl); // 👈 1
  return data.items; // 👈 2
};
```

1. Gebruik destructuring om de data uit het response te halen.
2. Retourneer vervolgens `data.items`, of dus de lijst van transacties.

De `TransactionList` component wordt daardoor:

```tsx
// src/pages/transactions/TransactionList.tsx
//...

export default function TransactionList() {
  const [transactions, setTransactions] = useState([]); // 👈 1
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');

  // 👇 3
  useEffect(() => {
    // 👇 4
    const fetchTransactions = async (): Promise<void> => {
      const data = await transactionsApi.getAll(); // 👈 2
      setTransactions(data); // 👈 3
    };

    fetchTransactions();
  }, []);

  const filteredTransactions = useMemo(
    () =>
      // 👇 4
      transactions.filter((t) => {
        return t.place.name.toLowerCase().includes(search.toLowerCase());
      }),
    [search, transactions], // 👈 4
  );
  //...
}
```

1. We maken niet langer gebruik van mock data, we bewaren onze transacties in state. De initiële state is een lege array. Als de data asynchroon is opgehaald dient de lijst te worden getoond (rerender)
2. Haal de data asynchroon op. Omwille van performantieredenen kan je eventueel het aantal records beperken (server side). Dat laten we hier achterwege.
3. Pas de state aan nadat je de lijst terugkrijgt van de API.
   - Een effect met een `setState` is vaak de trigger van een oneindige lus. Dit is niet het geval hier, omdat de `useEffect` enkel bij de initiële render wordt uitgevoerd. De state wordt enkel aangepast bij de initiële render.
   - `setState` gebruiken in een `useEffect` zou een belletje moeten doen rinkelen om te zoeken naar een betere oplossing!
4. Om te filteren gebruiken we nu de opgehaalde transacties en dit wordt ook een dependency van de `useMemo`.

Start de applicatie en bekijk het resultaat.

## Laadindicator en foutafhandeling

Je merkt misschien dat je heel kort de melding krijgt dat er geen transacties zijn en dat vervolgens de lijst van transacties wordt weergegeven. Dit komt omdat de API call asynchroon is en omdat het effect na de render uitgevoerd wordt. We kunnen dit oplossen door een laadindicator toe te voegen. Daarnaast voegen we ook meteen foutafhandeling toe, want we gaan er niet vanuit dat alles altijd goed gaat!

Omdat we de laadindicator in meerdere componenten nodig hebben, maken we hiervoor een aparte component `Loader` aan in een nieuw bestand `components/Loader.tsx`:

```tsx
// src/components/Loader.tsx
export default function Loader() {
  return (
    <div
      className='flex flex-col items-center justify-center py-8'
      data-testid='loader'
    >
      <div className='h-8 w-8 animate-spin rounded-full border-4 border-muted border-t-primary' />
    </div>
  );
}
```

Deze `Loader` component toont een simpele loading indicator van tailwindcss.

Ook zullen we foutafhandeling in meerdere componenten nodig hebben. Daarom maken we hiervoor een aparte component `Error` aan in een nieuw bestand `components/Error.tsx`:

```tsx
// src/components/Error.tsx
import { isAxiosError } from 'axios';
import { Alert, AlertDescription, AlertTitle } from './ui/alert';

interface ErrorProps {
  error?: unknown;
} // 👈 1

export default function Error({ error }: ErrorProps) {
  // 👆 1 👇 2
  if (isAxiosError(error)) {
    return (
      <Alert variant='destructive' className='mb-4'>
        <AlertTitle>Oops, something went wrong</AlertTitle>
        <AlertDescription>
          {/* 👇 3 */}
          {error?.response?.data?.message || error.message}
          {error?.response?.data?.details &&
            Object.keys(error?.response?.data?.details).length > 0 && (
              <>
                <br />
                {JSON.stringify(error?.response?.data?.details)}
              </>
            )}
        </AlertDescription>
      </Alert>
    );
  }
  // 👇 4
  if (error) {
    const msg =
      error instanceof globalThis.Error ? error.message : JSON.stringify(error);
    return (
      <Alert variant='destructive' className='mb-4'>
        <AlertTitle>An unexpected error occurred</AlertTitle>
        <AlertDescription>{msg}</AlertDescription>
      </Alert>
    );
  }

  return null; // 👈 5
}
```

1. We geven de `error` prop mee aan de component.
2. We controleren of de `error` een `AxiosError` is. Dit is een specifiek type error dat Axios gebruikt. We gebruiken de `isAxiosError` functie om dit te controleren.
3. In dat geval geven we de foutmelding weer. We controleren eerst of er een `message` is in de response data. Zo niet, dan geven we de `message` van de error weer. Als er een `details` object is, dan geven we dit ook weer. Dit `details` object voegen we in het hoofdstuk van validatie toe in de bijbehorende back-end, dus dit zal nu nog niet voorkomen.
4. Als er geen `AxiosError` is, dan geven we de `error` weer. We controleren eerst of er een `message` is. Zo niet, dan geven we de `error` zelf weer in JSON formaat.
5. Als er geen `error` is, dan geven we `null` terug. Zo ziet de gebruiker niets.

Als laatste definiëren we een `AsyncData` component in een nieuw bestand `components/AsyncData.tsx`:

```tsx
// src/components/AsyncData.tsx
import Loader from './Loader'; // 👈 1
import Error from './Error'; // 👈 1

interface AsyncDataProps {
  loading: boolean; // 👈 2
  error?: unknown; // 👈 3
  children: React.ReactNode; // 👈 4
}

export default function AsyncData({
  loading,
  error,
  children,
}: AsyncDataProps) {
  // 👇 2
  if (loading) {
    return <Loader />;
  }

  if (error) {
    return <Error error={error} />;
  } //👈 3

  return <>{children}</>; //👈 4
}
```

1. Importeer de `Loader` en `Error` componenten.
2. We verwachten een `loading` prop. Als `loading` gelijk is aan `true` geven we de `Loader` weer.
3. Daarnaast verwachten we een `error` prop. Die geven we mee aan de `Error` component. Als er een `error` is, dan geven we de `Error` weer.
4. Als laatste verwachten we een `children` prop. Dit is de TSX die we willen weergeven als de data niet aan het laden is en er geen error is.

Pas dan volgende onderdelen van de `TransactionList` verder aan:

```tsx
// src/pages/TransactionList.tsx
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { useState, useMemo, useEffect } from 'react';
import TransactionsTable from '../components/transactions/TransactionsTable';
import * as transactionsApi from '../api/transactions';
import type { Transaction } from '@/types';
import AsyncData from '../components/AsyncData'; // 👈 5

export default function TransactionList() {
  const [transactions, setTransactions] = useState<Transaction[]>([]); // 👈 1
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');
  const [loading, setLoading] = useState(true); // 👈 1
  const [error, setError] = useState<unknown>(null); // 👈 1

  useEffect(() => {
    const fetchTransactions = async () => {
      // 👇 2
      try {
        setLoading(true); // 👈 3
        setError(null); // 👈 3
        const data = await transactionsApi.getAll();
        setTransactions(data);
      } catch (error) {
        // 👇 4
        console.error(error);
        setError(error);
      } finally {
        // 👇 5
        setLoading(false);
      }
    };

    fetchTransactions();
  }, []);

  const filteredTransactions = useMemo(
    () =>
      transactions.filter((t) => {
        return t.place.name.toLowerCase().includes(search.toLowerCase());
      }),
    [search, transactions],
  );

  return (
    <>
      <h1 className='text-2xl font-semibold mb-6'>Transactions</h1>

      <div className='flex justify-between mb-4 gap-2'>
        <div className='flex gap-2 w-1/2'>
          <Input
            type='search'
            placeholder='Search by place…'
            value={text}
            onChange={(e) => setText(e.target.value)}
          />
          <Button variant='outline' onClick={() => setSearch(text)}>
            Search
          </Button>
        </div>
      </div>
      {/* 👇 6 */}
      <AsyncData loading={loading} error={error}>
        {/* 👇 7 */}
        <TransactionsTable transactions={filteredTransactions} />
      </AsyncData>
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
7. De transacties worden enkel weergegeven als er zich geen fout heeft voorgedaan.

Bekijk het resultaat in de applicatie.

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

```tsx
// src/api/index.js
import axios from 'axios';

const baseUrl = 'http://localhost:9000/api'; // 👈 1

// 👇 2 en 3
export async function getAll<T>(url: string): Promise<T> {
  const { data } = await axios.get(`${baseUrl}/${url}`); // 👈 4

  return data.items;
}
```

1. We houden het gemeenschappelijke deel van alle URLs bij in `baseUrl`.
2. We maken een generieke functie. `T` is het type dat je later kiest. Bvb `getAll<Transaction[]>("transactions")`
3. De parameter `url` zal van `swr` de `key` ontvangen. Zo hebben we meteen een uniek id voor elk request. We geven dus straks het laatste deel van de URL mee als `key`.
4. We voegen de `baseUrl` en de `url` samen om de volledige URL te bekomen.

Vervolgens gebruiken we de `useSWR` hook om onze transacties op te halen:

```tsx
// src/pages/TransactionList.tsx
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { useState, useMemo } from 'react';
import TransactionsTable from '../components/transactions/TransactionsTable';
import type { Transaction } from '../types';
import AsyncData from '../components/AsyncData';
import useSWR from 'swr'; // 👈 1
import { getAll } from '../api'; // 👈 2

export default function TransactionList() {
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');
  // const [transactions, setTransactions] = useState([]);// 👈 3
  // const [loading, setLoading] = useState(true);// 👈 3
  // const [error, setError] = useState<unkown>(null);// 👈 3

  const { data, isLoading, error } = useSWR<Transaction[]>(
    'transactions',
    getAll,
  ); // 👈 3

  const filteredTransactions = useMemo(
    () =>
      data?.filter((t) =>
        t.place.name.toLowerCase().includes(search.toLowerCase()),
      ) ?? [],
    [search, data],
  ); // 👈 4

  return (
    <>
      <h1 className='text-2xl font-semibold mb-6'>Transactions</h1>

      <div className='flex justify-between mb-4 gap-2'>
        <div className='flex gap-2 w-1/2'>
          <Input
            type='search'
            placeholder='Search by place…'
            value={text}
            onChange={(e) => setText(e.target.value)}
          />
          <Button variant='outline' onClick={() => setSearch(text)}>
            Search
          </Button>
        </div>
      </div>
      {/* 👇 5 */}
      <AsyncData loading={isLoading} error={error}>
        <TransactionsTable transactions={filteredTransactions} />
      </AsyncData>
    </>
  );
}
```

1. Importeer de `useSWR` hook en verwijder de import van `useEffect`
2. Importeer de `getAll` functie uit de `api/index.js`. Als je de naam van een map opgeeft, zal de `index.js` in die map geïmporteerd worden.
3. We gebruiken de `useSWR` hook met `transactions` als key en de `getAll` functie als `fetcher`. De `useSWR` hook retourneert een object met volgende properties:
   - `data`: de data die we ophalen. Dit is `undefined` als de data nog niet is opgehaald en van het type `Transaction[]` als de data is opgehaald. Als de data geladen maar er is geen data beschikbaar dan hebben we een lege array
   - `error`: de error die we ontvangen. Dit is `undefined` als er geen error is.
   - `isLoading`: een boolean die aangeeft of de data aan het ophalen is.
     We hoeven niet langer zelf state bij te houden voor de transactions, error en loading. Deze lijnen code mag je schrappen
4. `filteredTransactions` is `undefined` tijdens het laden. Als het resultaat undefined of null is gebruik dan de []. Zo retourneert `filteredTransactions` steeds een array ook tijdens het laden.
5. We gebruiken de `AsyncData` component om de `loading` en `error` verder af te handelen. We geven de `TransactionsTable` mee als `children`. We moeten hier de naam van de variabele in de `loading` prop aanpassen naar `isLoading`.

### Oefening 1 - GET all in je eigen project

Implementeer een GET all van een willekeurige entiteit uit je eigen project:

- Installeer `swr` en `axios`.
- Voeg de componenten `Loader`, `Error` en `AsyncData` toe.
- Maak een functie aan in `api/index.js` die een GET all request uitvoert.
- Gebruik de `useSWR` hook om de data op te halen.
- Zorg ervoor dat je de data kan weergeven in jouw lijst-component.

## DELETE /api/transactions/:id

De volgende stap van de CRUD operaties is de 'D', een transactie verwijderen. Enerzijds moeten we een API call toevoegen, die het verwijderen effectief uitvoert, en deze beschikbaar maakt. Anderzijds moeten we deze op de juiste plaats gebruiken.

Voeg een `deleteById` functie toe in `index.js` in de map `api`:

```tsx
// src/api/index.js
// 👇 1
export const deleteById = async (
  url: string,
  { arg: id }: { arg: number },
): Promise<void> => {
  await axios.delete(`${baseUrl}/${url}/${id}`); // 👈 2
};
```

1. De parameter `url` zal van `swr` de `key` ontvangen. We krijgen ook het `id` mee als argument, we halen dit uit de `arg` optie die we van `swr` krijgen.
2. We bouwen de url (`/api/transactions/:id`) op en voeren de `DELETE` uit. Net zoals `axios.get()` kan je ook `axios.delete()` uitvoeren. Het antwoord is de HTTP status code 204. Bijgevolg is de HTTP response body ook leeg (m.a.w. `{}` in JavaScript). We negeren dat antwoord hier.

De `Transaction` component zelf is het meest geschikt om zijn eigen transactie te verwijderen. Echter is het niet zijn verantwoordelijkheid om de effectieve API call uit te voeren, die is voor de `TransactionList`. We voegen een verwijderknop toe aan deze component:

```tsx
// src/components/transactions/Transaction.tsx
import { Button } from '@/components/ui/button'; // 👈 1
import { Trash2 } from 'lucide-react'; // 👈 1
// ...

interface TransactionProps extends TransactionType {
  onDelete?: (id: number) => void;
} // 👈 3

export default function Transaction({
  id,
  user,
  place,
  amount,
  date,
  onDelete,
}: TransactionProps) {
  // 👈 3
  // 👇 2
  const handleDelete = () => {
    onDelete?.(id);
  };

  return (
    <TableRow>
      <TableCell>
        <LocalizedDate date={date} />
      </TableCell>
      <TableCell>{user.name}</TableCell>
      <TableCell>{place.name}</TableCell>
      <TableCell>{amountFormat.format(amount)}</TableCell>
      <TableCell className='text-right'>
        <div className='flex justify-end gap-1'>
          <Button
            variant='ghost'
            size='icon'
            aria-label='Delete transaction'
            className='text-destructive hover:text-destructive'
            onClick={handleDelete}
          >
            <Trash2 className='h-4 w-4' />
          </Button>
        </div>
      </TableCell>{' '}
      {/*  👈 1 */}
    </TableRow>
  );
}
```

1. Voeg een knop toe met een event handler `handleDelete`, met een icoon uit de `lucide-react` library.
2. Implementeer de `handleDelete` functie. Deze functie roept de `onDelete` functie aan met het `id` van de transactie.
3. De `Transaction` component ontvangt nu ook een `id` en `onDelete` prop.

We breiden de `TransactionTable` uit met een `onDelete` prop die we meteen doorgeven aan de `Transaction` component:

```tsx
// src/components/transactions/TransactionsTable.tsx
//...
interface TransactionsTableProps {
  transactions?: TransactionType[];
  onDelete?: (id: number) => void; //👈
}

function TransactionsTable({ transactions, onDelete }: TransactionsTableProps) {
  //👈
  //...

  return (
    <div className='rounded-md border'>
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Date</TableHead>
            <TableHead>User</TableHead>
            <TableHead>Place</TableHead>
            <TableHead>Amount</TableHead>
            {/* 👇 */}
            <TableHead />
          </TableRow>
        </TableHeader>
        <TableBody>
          {transactions.map((transaction) => (
            <Transaction
              key={transaction.id}
              {...transaction}
              onDelete={onDelete}
            />
          ))}
          {/* 👈 */}
        </TableBody>
      </Table>
    </div>
  );
}

export default TransactionsTable;
```

Nu zijn we klaar om de transactie effectief te verwijderen, we passen de `TransactionList` component aan. Het probleem van de `useSWR` hook is dat die meteen het request uitvoert als de component rendert. We willen dit pas doen als de gebruiker op de verwijderknop klikt. Daarom maken we gebruik van de `mutate` functie die we van `swr` krijgen. Deze functie zal de data in de cache aanpassen en de component opnieuw renderen.

```tsx
// src/pages/TransactionList.tsx
// imports...
import useSWRMutation from 'swr/mutation'; // 👈 1
import { getAll, deleteById } from '../../api'; // 👈 1
import { toast } from 'sonner'; // 👈 3

export default function TransactionList() {
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');

  const { data, isLoading, error } = useSWR<Transaction[]>(
    'transactions',
    getAll,
  );

  // 👇 2
  const { trigger: deleteTransaction, error: deleteError } = useSWRMutation(
    'transactions',
    deleteById,
  );

  const filteredTransactions = useMemo(
    () =>
      data?.filter((t) =>
        t.place.name.toLowerCase().includes(search.toLowerCase()),
      ) ?? [],
    [search, data],
  );

  // 👇3
  const handleDeleteTransaction = async (id: number) => {
    await deleteTransaction(id);
    toast.success('Transaction removed');
  };

  return (
    <>
      <h1 className='text-2xl font-semibold mb-6'>Transactions</h1>

      <div className='flex justify-between mb-4 gap-2'>
        <div className='flex gap-2 w-1/2'>
          <Input
            type='search'
            placeholder='Search by place…'
            value={text}
            onChange={(e) => setText(e.target.value)}
          />
          <Button variant='outline' onClick={() => setSearch(text)}>
            Search
          </Button>
        </div>
      </div>
      {/* 👇 4 */}
      <AsyncData loading={isLoading} error={error || deleteError}>
        {/* 👇 3 */}
        <TransactionsTable
          transactions={filteredTransactions}
          onDelete={handleDeleteTransaction}
        />
      </AsyncData>
    </>
  );
}
```

1. Importeer de `useSWRMutation` hook en de `deleteById` functie.
2. Gebruik de `useSWRMutation` hook. We geven als key `transactions` mee en als fetcher onze `deleteById` functie. We krijgen o.a. terug:
   - `trigger`: een functie die we kunnen aanroepen om het request effectief uit te voeren en dus de data te verwijderen. Deze functie ontvangt de `id` van de transactie als argument. We hernoemen dit naar `deleteTransaction`.
   - `error`: een eventuele fout die zich voordoet bij het verwijderen van de transactie. We hernoemen deze naar `deleteError`.
3. De `handleDeleteTransaction` functie zal de `deleteTransaction` trigger aanroepen en toont een toast.
4. We geven de `handleDeleteTransaction` functie mee als `onDelete` prop aan de `TransactionsTable` component.
5. We voegen de `deleteError` toe aan `error` prop aan de `AsyncData` component. Deze zal dus de fout tonen als er een fout is bij het ophalen van de transacties of bij het verwijderen van een transactie.

Bekijk het resultaat in de applicatie, je zou een transactie moeten kunnen verwijderen. In een meer realistische applicatie zou je een bevestiging moeten vragen aan de gebruiker, dat laten we even achterwege hier.

Open de console en inspecteer het `Network` tabblad. Je zal zien dat er een `DELETE` request wordt uitgevoerd naar de API en dat meteen daarna de lijst van transacties opnieuw wordt opgehaald. Dit is de `stale-while-revalidate` strategie die `swr` gebruikt. De data wordt eerst uit de cache gehaald (stale), dan wordt de request uitgevoerd (revalidate) en tenslotte wordt de up-to-date data opnieuw geretourneerd. `swr` weet dat er iets gewijzigd is aangezien een mutation uitgevoerd is met dezelfde key als de hook die onze data ophaalt. Handig, he?

TODO : hier de hasData toevoegen!!Wil je dat de tabel zichtbaar blijft terwijl je transacties verwijdert, houd je hasData. Wil je alles vervangen door een Loader, verwijder je het. Zonder hasData (of altijd false), zou je de tabel vervangen door een Loader tijdens het verwijderen van een transactie, wat slechter UX is. Zo blijft de tabel weergegeven en gebeurt het laden op de achtergrond

### Oefening 2 - DELETE in je eigen project

Implementeer een willekeurige DELETE uit je eigen project (liefst dezelfde entiteit als hiervoor):

- Maak een functie aan in `api/index.js` die een DELETE request uitvoert.
- Gebruik de `useSWRMutation` hook om de data te verwijderen.
- Zorg ervoor dat je de data kan verwijderen uit jouw lijst-component.

## Search refactored

We willen dat er ook gefiltered wordt als de gebruiker op Enter klikt of het zoekveld leegmaakt

```tsx
// src/pages/TransactionList.tsx
//...
const handleSearchChange = useCallback(
  (e: React.ChangeEvent<HTMLInputElement>) => {
    setText(e.target.value);

    if (e.target.value === '') {
      setSearch('');
    }
  },
  [],
);

const handleKeyDown = useCallback(
  (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      setSearch(text);
    }
  },
  [text],
);
//...
<Input
  type='search'
  placeholder='Search by place…'
  value={text}
  onChange={handleSearchChange}
  onKeyDown={handleKeyDown}
/>;
//...
```

1. `handleSearchChange` wordt geactiveerd als de gebruiker iets in het zoekveld typt. Werkt de text state bij met wat de gebruiker typt. Als het veld leeggemaakt wordt, wordt de search state ook geleegd. Dit triggert de filter om alle transacties weer te geven
2. `handleKeyDown` wordt geactiveerd als de gebruiker een toets inslaat (maar de tekst verandert niet). Als de gebruiker op Enter drukt, wordt de text waarde doorgegeven aan search. Dit triggert de filter om de transacties te filteren op de zoekterm.

TODO: Pagineren door de transacties
TODO : hasData in async

## Het .env bestand

Je kan omgevingsvariabelen definiëren in het `.env` bestand. De omgevingsvariabelen moeten beginnen met `VITE_`, alle andere variabelen behalve `NODE_ENV` worden genegeerd. Als je omgevingsvariabelen wijzigt, moet je de applicatie _niet_ opnieuw starten. Vite zal de wijzigingen automatisch detecteren en het nodige doen.

De omgevingsvariabelen zijn beschikbaar via het object `import.meta.env`. Een omgevingsvariabele met de naam `VITE_NOT_SECRET_CODE` wordt in de code `import.meta.env.VITE_NOT_SECRET_CODE`.

Er is ook een ingebouwde omgevingsvariabele met de naam `NODE_ENV`. Wanneer je `pnpm dev` uitvoert, is `NODE_ENV` altijd gelijk aan `development`.

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

## Oefening 3 - README

Pas `README.md` aan zodat de gebruiker weet dat er een `.env` bestand aangemaakt moet worden alvorens de applicatie gestart kan worden. Voeg ook een voorbeeld voor het `.env` bestand toe.

## Oefening 4 - PlacesList via API

Pas nu ook `PlacesList` aan zodat dit werkt met onze REST API voor het ophalen, verwijderen van de places en het aanpassen van de rating. Voorzie in de `src/components/places` folder de component `PlacesCards.tsx` die de lijst van `Places` weergeeft. `PlacesList.tsx` communiceert met de API en geeft de data door via props aan `PlacesCards.tsx`.

Pas ook `PlaceDetail` aan. Geef de transacties van de betreffende plaats weer. Maak hiervoor gebruik van de `TransactionTable` component.

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les4-opl 6d60fc7
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.

## Must reads

- [JavaScript Visualized: Promises & Async/Await](https://medium.com/@lydiahallie/javascript-visualized-promises-async-await-a3f1aad8a943)
- [SOLID principles in React](https://konstantinlebedev.com/solid-in-react/)
- [Good advice on JSX conditionals](https://blog.thoughtspile.tech/2022/01/17/jsx-conditionals/)
- [Component Composition is great btw](https://tkdodo.eu/blog/component-composition-is-great-btw)

## Mogelijke extra's voor de examenopdracht

- [react-query](https://www.npmjs.com/package/react-query)
- [react-error-boundary](https://github.com/bvaughn/react-error-boundary)
