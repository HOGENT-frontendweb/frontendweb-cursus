# You can call me API

!> Vanaf dit hoofdstuk heb je de bijbehorende backend nodig: <https://github.com/HOGENT-Web/webservices-budget>.<br />Als je zonder MySQL-databank wil werken, check uit op commit `37a0083`. Op de laatste commit is een lokale MySQL-server vereist!

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-Web/frontendweb-budget/
> git checkout -b les4 5d5a53f
> yarn install
> yarn start
> 
> ```

In dit hoofdstuk vervangen we de mock data door HTTP-requests naar de REST API. Op ons lokaal toestel draait deze API op [http://localhost:9000/api/](http://localhost:9000/api/).

Voor de communicatie met de API, m.a.w. het versturen van HTTP-requests, kan je gebruik maken van de [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) of van HTTP client libraries die je kan vinden op bv. <https://www.npmjs.com>.

Wij gaan gebruik maken van [Axios](https://www.npmjs.com/package/axios), een _Promise based HTTP client library_ voor de browser en NodeJS.

Installeer eerst Axios:

```bash
yarn add axios
```

## GET /api/transactions

Maak een bestand `transactions.js` aan in de map `api`. Hierin plaatsen we alle requests naar de API [http://localhost:9000/api/transactions](http://localhost:9000/api/transactions):

```jsx
import axios from 'axios'; // 👈 1

const baseUrl = `http://localhost:9000/api/transactions`; // 👈 2

// 👇 3
export const getAll = async () => {
  const response = await axios.get(baseUrl); 
  console.log(response)
};
```

1. Importeer axios.
2. Houd het gemeenschappelijke deel van alle URLs bij in `baseUrl`.
3. De `getAll` functie maakt gebruik van `axios.get`, een asynchrone methode die een Promise retourneert (vandaar `async/await`). We loggen voorlopig het antwoord en bekijken in de volgende stap in de console wat geretourneerd zal worden.

Pas de `TransactionList` component aan. Het ophalen van de transacties is een side-effect, we maken dus gebruik van de `useEffect` hook.

```jsx
import { useState, useMemo, useCallback, useEffect, useContext } from 'react'; // 👈 1
import { ThemeContext } from '../../contexts/Theme.context';
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import { TRANSACTION_DATA } from '../../api/mock-data';
import * as transactionsApi from '../../api/transactions'; // 👈 2

//...
export default function TransactionList() {
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');

  useEffect(() => { // 👈 3
    // 👇 4
    const fetchTransactions = async () => {
      const data = await transactionsApi.getAll();
      console.log(data);
    };

    fetchTransactions();
  }, []); // 👈 3
  //...
```

1. Importeer `useEffect`.
2. Importeer ook alle functies uit de transaction api met `import * as`.
3. Het ophalen van de transacties is een side-effect en dient enkel bij de eerste render te worden uitgevoerd.
4. Een `useEffect` mag geen async functie als argument krijgen.

  ```jsx
    useEffect(() => { async () => {
      const data = await transactionsApi.getAll();
      console.log(data);
    } , []);
  ```

  Het probleem hier is dat het eerste argument van `useEffect` een functie zou moeten zijn die ofwel niets ofwel een functie retourneert (de cleanup functie). Maar een asynchrone functie retourneert een promise, die niet als functie kan worden aangeroepen! Het is gewoon niet wat de `useEffect` hook verwacht als eerste argument. We lossen dit op door een asynchrone functie te definiëren en dan aan te roepen.

Start de applicatie en bekijk het response in de console.

![Response van GET all transactions](./images/response.png)

Een axios response bevat volgende informatie:

- `data` - de body van het HTTP response. Als dit JSON is, zal Axios dit automatisch parsen in een JavaScript object.
- `status` - de HTTP statuscode, bv. 200, 400, 404.
- `statusText` - het HTTP statusbericht, bv. OK, Bad Request, Not Found.
- `headers` - de HTTP headers uit het HTTP response.
- `config` - de configuratie die je meegaf aan de Axios API.
- `request` - het native request dat gebruikt werd. In NodeJS is dit een `ClientRequest` object.

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
2. Retourneer vervolgens `data.items`.

De `TransactionList` component wordt daardoor:

```jsx
//...
//import { TRANSACTION_DATA } from '../../api/mock-data'; // 👈 1
function TransactionTable({ transactions}) {
//...
        <tbody>
          {transactions.map((transaction) => (
            <Transaction key={transaction.id} {...transaction} />
          ))}{/*👈 5*/}
        </tbody>
//...

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
3. Haal de data asynchroon op. Omwille van performantieredenen kan je eventueel het aantal records beperken (server side).
4. Pas de state aan nadat je de lijst terugkrijgt van de API.
5. Om te filteren op place gebruiken we nu `place.name` conform de nieuwe structuur van een transactie. Ook in de TransactionTable kan nu de id gebruikt worden als key.

De json die verkregen wordt van de backend bevat een `user` en `place` object. Dus dienen we nu ook `Transaction.jsx` aan te passen:

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

Ook dienen we foutafhandeling toe te voegen. Hiervoor maken we eerst een `Error` component aan in de map `components`:

```jsx
export default function Error({ error }) {
  if (error) {
    return (
      <div className="alert alert-danger">
        <h4 className="alert-heading">An error occured</h4>
        {error.message || JSON.stringify(error)}
      </div>
    );
  }

  return null;
}
```

Pas dan volgende onderdelen van de `TransactionList` verder aan:

```jsx
import Error from '../Error'; // 👈 5
//...

export default function TransactionList() {
  const [transactions, setTransactions] = useState([]);
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');
  const [error, setError] = useState(null); // 👈 1

  useEffect(() => {
    const fetchTransactions = async () => {
      try { //👈2
        setError(null); //👈3
        const data = await transactionsApi.getAll();
        setTransactions(data);
      } catch (error) {//👈4
        console.error(error);
        setError(error);
      };
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
        <Error error={error} /> {/* 👈 5 */}
        {!error ? <TransactionTable transactions={filteredTransactions} /> : null}{/* 👈 6 */}
      </div>
    </>
  );
}
```

1. Definieer een `error` state variabele om een eventuele fout van de API weer te kunnen geven.
2. Voor de foutafhandeling maken we gebruik van try-catch.
3. Plaats de `error` state op `null` bij een nieuw request.
4. Vang een eventuele fout op en stel de `error` state in.
5. Geef de fout weer. Dit zal enkel gebeuren als `error` niet `null` is (zie `Error` component).
6. Geef de transacties weer als er zich geen fout heeft voorgedaan

Tijdens het ophalen van de transacties geven we een loading indicator weer. Hiervoor definiëren we een aparte component `Loader` in de map `components`:

```jsx
export default function Loader({ loading }) {
  if (loading) {
    return (
      <>
        <div className="spinner-border">
          <span className="visually-hidden">Loading...</span>
        </div>
        <p>Loading...</p>
      </>
    );
  }

  return null;
}
```

Pas dan `TransactionList` verder aan:

```jsx
import Loader from '../Loader'; // 👈 4
//...

export default function TransactionList() {
  const [transactions, setTransactions] = useState([]);
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true); // 👈 1

  useEffect(() => {
    const fetchTransactions = async () => {
      try {
        setLoading(true); // 👈 2
        setError(null);
        const data = await transactionsApi.getAll();
        setTransactions(data);
      } catch (error) {
        console.error(error);
        setError(error);
      } finally {// 👈 3
        setLoading(false);
      }
    };

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
        <Loader loading={loading} /> {/* 👈 4 */}
        <Error error={error} />

        {!loading && !error ? <TransactionTable transactions={filteredTransactions} /> : null}{/* 👈 5 */}
      </div>
    </>
  );
}
```

1. Definieer een `loading` state variabele voor de weergave van de loading indicator.
2. Stel de `loading` state in op `true` wanneer het ophalen van de transactions wordt gestart.
3. Als het ophalen beëindigd is, plaats dan de `loading` state op `false`.
4. Geef de loading indicator weer. Dit zal enkel gebeuren als `loading` `true` is (zie `Loader` component).
5. De transacties worden niet weergegeven zolang de request wordt uitgevoerd

Bekijk het resultaat in de applicatie.

## DELETE /api/transactions/:id

De volgende stap van de CRUD operaties is de 'D', een transactie verwijderen. Enerzijds moeten we een API call toevoegen, die het verwijderen effectief uitvoert, en deze beschikbaar maakt. Anderzijds moeten we deze op de juiste plaats gebruiken.

Voeg een `deleteById` functie toe in `transaction.js` in de map `api`:

```jsx
export const deleteById = async (id) => {
  await axios.delete(
    `${baseUrl}/${id}`
  );
};
```

Om een element te verwijderen moeten we een HTTP `DELETE` uitvoeren naar `/api/transactions/:id`. Onze methode gaat dus zeker een `id` als parameter nodig hebben. Net zoals `axios.get()` kan je ook `axios.delete()` uitvoeren. Het antwoord is de HTTP status code 204. Bijgevolg is de HTTP response body ook leeg (m.a.w. `{}` in JavaScript).

De `Transaction` component zelf is het meest geschikt om zijn eigen transactie te deleten (en daarom hadden we er ook al een delete knop aan toegevoegd in vorige lessen, niet toevallig). Maar de `TransactionList` component bevat de state `transactions`. Dus deze component zal de state moeten aanpassen via een delete-functie. Vervolgens moet deze functie als event handler prop doorgegeven worden aan de kinderen: eerst de `TransactionTable` component, dan de `Transaction` component.

```jsx
//..

  // 👇 1
  const handleDelete = useCallback(async (idToDelete) => {
    try {
      setError(null);
      await transactionsApi.deleteById(idToDelete);
      // of gewoon opnieuw ophalen
      setTransactions((transactions) => transactions.filter(({ id }) => id !== idToDelete)); // 👈 2
    } catch (error) {
      console.error(error);
      setError(error);
    }
  }, []);
  //...

  return (
    <>
      {/* ... */}
      <div className="mt-4">
        <Loader loading={loading} />
        <Error error={error} />
        {!loading && !error ? <TransactionTable transactions={filteredTransactions} onDelete={handleDelete} /> : null} {/* 👈 3 */}
      </div>
   </>
  );
}
```

1. `handleDelete` is een event handler voor het verwijderen van een transactie. Het `id` van de te verwijderen transactie wordt als parameter meegegeven. Zorg ook voor de error en loading state, ook een delete kan mislopen.
2. Hoe zorg je dat de tabel nadien aangepast wordt? Pas de state aan:
    1. of je haalt de transacties opnieuw op via de get request;
    2. of je filtert de transactie uit de lijst (dit doen wij).
3. Voeg een `onDelete` event handler prop toe aan de `TransactionTable`.

In de `TransactionTable` component moeten we enkel de nieuw prop ontvangen en doorgeven aan elke transactie:

```jsx
function TransactionTable({
  transactions,
  onDelete // 👈 1
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
            <Transaction key={transaction.id} {...transaction} onDelete={onDelete} />
          ))}{/* 👈 2 */}
        </tbody>
      </table>
    </div>
  );
}
```

1. Ontvangt nu ook een prop `ìd` en `onDelete`.
2. Geeft deze event handler door aan de `Transaction` component via de `onDelete` prop.

Aan de `Transaction` component voegen we een vuilbakicoon toe met een `onClick` handler. Let op: je wil een functie meegeven die pas de transactie verwijdert als ze aangeroepen wordt. Je mag hier niet rechtstreeks de `onDelete` functie aanroepen, anders wordt je element verwijderd bij een render.

```jsx
import { memo, useCallback } from 'react'; // 👈 3
import { IoTrashOutline } from 'react-icons/io5'; // 👈 2
//...

export default memo(function Transaction({ id, user, amount, place, date, onDelete }) { // 👈 1
  // 👇 3
  const handleDelete = useCallback((event) => {
    event.preventDefault();
    onDelete(id);
  }, [id, onDelete]);

  return (
    <tr>
      <td>
        {dateFormat.format(new Date(date))}
      </td>
      <td>{user.name}</td>
      <td>{place.name}</td>
      <td>
        {amountFormat.format(amount)}
      </td>
      <td>
        <div className="btn-group float-end">
        <button type="button" className="btn btn-danger" onClick={handleDelete}>{/* 👈 2 */}
          <IoTrashOutline />
        </button>
        </div>
      </td>
    </tr>
  );
})
```

1. Bevat nu ook een `onDelete` prop.
2. Importeer het vuilbakicoon en voeg een knop toe met dit icoon.
3. Implementeer de `onClick` event handler

Bekijk het resultaat in de applicatie.

## POST /api/transactions

De volgende stap van de CRUD operaties is de 'C', een nieuwe transactie aanmaken.

Pas `transaction.js` in de map `api` aan:

```jsx
export const save = async (transaction) => {
  await axios.post(baseUrl, transaction);
};
```

De functie krijgt de te creëren transaction als parameter. Om een `POST` request te sturen geef je

- als eerste parameter de url mee;
- en als 2de parameter de nieuwe transactie, die in JSON formaat naar de API zal verstuurd worden.

Het antwoord heeft als HTTP status code 200 en als response body de nieuw gecreëerde transactie.

De creatie van een transactie gebeurt in de `TransactionForm` component. De state wordt bijgehouden in de `TransactionList` component. Daar passen we de `createTransaction` methode aan:

```jsx
const createTransaction = useCallback(async (transaction) => {
  try {
    setError(null);
    await transactionsApi.save({
      ...transaction,
    });
  } catch (error) {
    console.error(error);
    setError(error);
  }
}, []);
```

Maar nu wordt de gecreëerde transactie nog niet getoond. Daarvoor dienen we de transacties opnieuw op te vragen. De functie om dit te doen bestaat reeds, we refactoren de code. Plaats de code die uitgevoerd wordt bij de initiële render in een aparte functie:

```jsx
const refreshTransactions = useCallback(async () => {
  try {
    setLoading(true);
    setError(null);
    const data = await transactionsApi.getAll();
    setTransactions(data);
  } catch (error) {
    console.error(error);
    setError(error);
  } finally {
    setLoading(false);
  }
}, []);
```

De functie `refreshTransactions` haalt de transacties op en past de state, error en loading indicator aan. Vervang de code in het effect van de initiële render van de component door:

```jsx
useEffect(() => {
  refreshTransactions();
}, [refreshTransactions]);
```

Bij het toevoegen van de nieuwe transactie, halen we de transacties terug op. We doen dit natuurlijk enkel als het toevoegen succesvol was!

```jsx
const createTransaction = useCallback(async ( transaction ) => {
  try {
    setError(null);
    await transactionsApi.save({
      ...transaction,
    });
    await refreshTransactions(); // 👈
  } catch (error) {
    console.error(error);
    setError(error);
  } finally{
    setLoading(false);
  }
}, [refreshTransactions]); // 👈
```

De api verwacht een `id` voor de place. We retourneren `placeId` als geselecteerde waarde in de select list. In het formulier moet je vanaf nu in de select lijst werken met het `id`.

```jsx
function PlacesSelect() {
  //..

  return (
    {/* ... */}
    <select
      {...register('place', validationRules.place)}
      id="places"
      className="form-select"
    >
      <option defaultChecked value="">-- Select a place --</option>
      {PLACE_DATA.map(({ id, name }) => (
        <option key={id} value={id}>{name}</option>{/*👈*/}
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

De `onSubmit` wordt dan:

```jsx
const onSubmit = (data) => {
  const { user, place, amount, date } = data;
  onSaveTransaction({
    user,
    placeId: place, // 👈
    amount: parseInt(amount),
    date: new Date(date)
  })
  reset();
};
```

Merk op: momenteel werken we nog met de mock-data voor de places. Pas dit aan zodat het gebruik maakt van dezelfde id's als in de places tabel.

## PUT /api/transactions/:id

Dan rest nog de 'U' van CRUD, maar die is een beetje speciaal. De API call zelf is geen probleem. Dit is basically hetzelfde als 'Create' maar met een extra `id` parameter.

Maar we willen natuurlijk niet dat een gebruiker een volledig object juist moet invoeren om het aan te passen. We willen dat hij ergens op 'edit' kan klikken bij een bestaand element.

In ons geval willen we dat de "potlood-knop" in de lijst deze data in het formulier ingeeft. De gebruiker past vervolgens aan en doet een update. Maar om dat te doen werken moeten er een paar dingen gebeuren.

We moeten op een manier een transactie kunnen doorgeven van onze `Transaction` naar onze `TransactionForm` component. Daarnaast moeten we in `TransactionsForm` het onderscheid kunnen maken tussen een bestaande transactie updaten, of een nieuwe toevoegen.

De truc is om

- een 'huidige transactie' als state toe te voegen aan onze `TransactionList`;
- evenals een functie om dit in te stellen.

We stellen de state in als er op het potlood geklikt wordt. In het formulier bekijken we via een `useEffect` bij elke render of er een huidige transactie is ingesteld en indien nodig tonen we de bijhorende gegevens in alle formuliervelden.

In de API kan je hiervoor een aparte methode aanmaken:

```jsx
export const update = async (transaction) => {
  const { id, ...values } = transaction;
  await axios.put(`${baseUrl}/${id}`, values);
};
```

Of de `save` functie aanpassen:

```jsx
export const save = async (transaction) => {
  const { id, ...values } = transaction;
  await axios({
    method: id ? 'PUT' : 'POST',
    url: `${baseUrl}/${id ?? ''}`,
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

  const handleSaveTransaction = useCallback(async (transaction) => { // 👈 3
    try {
      setError(null);
      await transactionsApi.save({
        ...transaction,
      });
      setCurrentTransaction({}); // 👈 3
      await refreshTransactions(); 
    } catch (error) {
      console.error(error);
      setError(error);
    } finally {
      setLoading(false);
    }
  }, [refreshTransactions]);

  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={handleSaveTransaction} currentTransaction={currentTransaction}/>{/* 👈 3 en 4 */}
      <div className="input-group mb-3 w-50">
        <input type="search" id="search" className="form-control rounded" placeholder="Search" value={text} onChange={(e) => setText(e.target.value)} />
        <button type="button" className="btn btn-outline-primary" onClick={() => setSearch(text)}>Search</button>
      </div>
      <div className="mt-4">
        <Loader loading={loading} />
        <Error error={error} />
        {!loading && !error ? <TransactionTable transactions={filteredTransactions} onDelete={handleDelete} onEdit={setTransactionToUpdate} /> : null} {/* 👈 5 */}
    </div>
   </>
  );
}
```

1. De state om de huidige transactie bij te houden.
2. `setTransactionToUpdate` past de state aan o.b.v. een opgegeven `id`.
3. Hernoem de functie `createTransaction` naar `handleSaveTransaction` en stel `currentTransaction` terug in op een leeg object na het opslaan.
4. Geef de `currentTransaction` door aan de `TransactionForm` component en vervang `createTransaction` door `handleSaveTransaction`.
5. Voeg een `onEdit` event handler prop toe aan de `TransactionTable`.

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
            <Transaction key={transaction.id} {...transaction} onDelete={onDelete} onEdit={onEdit} />
          ))}{/* 👈 2 */}
        </tbody>
      </table>
    </div>
  );
}
```

1. Ontvangt nu ook een prop `onEdit`.
2. Geeft deze event handler prop door aan de `Transaction` component via de `onEdit` prop.

Als laatste moeten we in de `Transaction` component een potloodicoon toevoegen met een `onClick` handler:

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
  const handleDelete = useCallback((event) => {
    event.preventDefault();
    onDelete(id);
  }, [id, onDelete]);

  // 👇 3
  const handleUpdate = useCallback((event) => {
    event.preventDefault();
    onEdit(id);
  }, [id, onEdit]); 

  return (
    <tr>
      <td>
        {dateFormat.format(new Date(date))}
      </td>
      <td>{user.name}</td>
      <td>{place.name}</td>
      <td>
        {amountFormat.format(amount)}
      </td>
      <td>
        <div className="btn-group float-end">
            <button type="button" className="btn btn-light" onClick={handleUpdate}>{/* 👈 3 */}
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

1. Krijgt nu ook een `onEdit` prop.
2. Importeer het potloodicoon.
3. Voeg de knop met het potloodicoon en een `onClick` handler toe.

In het `TransactionForm` passen we de `useEffect` aan zodat bij elke render het volgende gecontroleerd wordt: als `currentTransaction` is ingevuld dan gaat het om een update, anders om een create.

```jsx
import { memo, useEffect } from 'react'; // 👈 2
//..

export default memo(function TransactionForm({ currentTransaction, onSaveTransaction }) { // 👈 1
  const { setValue, register, handleSubmit, reset, formState: { errors } } = useForm(); // 👈 2

  const onSubmit = (data) => {
    const { user, place, amount, date } = data;
    let transaction = {
      user,
      placeId: place,
      amount: parseInt(amount),
      date: new Date(date)
    };
    onSaveTransaction({ ...transaction, id: currentTransaction?.id }) // 👈 4
    reset();
  };

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
      setValue("user", currentTransaction.user.name);
      setValue("place", currentTransaction.place.id);
      setValue("amount", currentTransaction.amount);
    } else {
      reset();
    }
  }, [currentTransaction, setValue, reset]);
  
  //..
  <div className="clearfix">
    <div className="btn-group float-end">
      <button
        type="submit"
        className="btn btn-primary"
      > {currentTransaction?.id
          ? "Save transaction"
          : "Add transaction"}</button> {/* 👈 3 */}
    </div>
  </div>
```

1. Geef de `currentTransaction` door.
2. Controleer bij elke render of `currentTransaction` is ingevuld. Indien ingevuld, plaats de waarden in het formulier. Maak hiervoor gebruik van `setValue` uit de `useForm` hook.
3. Pas de tekst op de knop aan.
4. Bij het aanpassen van een transactie voegen we ook het id toe.

Je kan er ook voor zorgen dat de inputvelden en knoppen in het formulier _disabled_ worden als het formulier gesubmit wordt.
`useForm` geeft een boolean [isSubmitting](https://react-hook-form.com/api/useform/formstate) terug die `true` is als het formulier gesubmit wordt en `false` bij reset.

```jsx
//..
function LabelInput({ label, name, type, ...rest }) {
  const {
    register,
    errors,
    isSubmitting
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
        disabled={isSubmitting}
        className="form-control"
        {...rest}
      />{/* 👈 4 */}
      {hasError ? (
        <div className="form-text text-danger">
          {errors[name].message}
        </div>
      ) : null}
    </div>
  );
}
//..

export default memo(function TransactionForm({ currentTransaction, onSaveTransaction }) {
  const { register, handleSubmit, reset, formState: { errors, isSubmitting }, setValue } = useForm(); // 👈 1
  //...

  <FormProvider handleSubmit={handleSubmit} errors={errors} register={register} isSubmitting={isSubmitting}>{/* 👈 2 */}
  {/* ... */}
    <div className="btn-group float-end">
      <button
        type="submit"
        disabled={isSubmitting}
        className="btn btn-primary"
    > {currentTransaction?.id
        ? "Save transaction"
        : "Add transaction"}</button>{/* 👈 3 */}
    </div>
  {/* ... */}
```

1. Haal ook `isSubmitting` op van `useForm`.
2. Vermits ook de componenten `LabelInput` en `PlacesSelect` hier gebruik van maken dient de `FormProvider` hierin te voorzien.
3. Disable de knop tijdens submit.
4. Disable het inputveld tijdens submit.

### Oefening

Doe hetzelfde voor `PlacesSelect`: disable dit veld tijdens de submit.

## Het .env bestand

Je kan omgevingsvariabelen definiëren in het `.env` bestand. De omgevingsvariabelen moeten beginnen met `REACT_APP_`, alle andere variabelen behalve `NODE_ENV` worden genegeerd. Als je omgevingsvariabelen wijzigt, moet je de applicatie opnieuw starten.

De omgevingsvariabelen zijn beschikbaar via het object `process.env`. Een omgevingsvariabele met de naam `REACT_APP_NOT_SECRET_CODE` wordt in de code `process.env.REACT_APP_NOT_SECRET_CODE`.

Er is ook een ingebouwde omgevingsvariabele met de naam `NODE_ENV`. Wanneer je `yarn start` uitvoert, is `NODE_ENV` altijd gelijk aan `development`, bij `yarn test` (= testen uitvoeren) aan `test`, en bij `yarn build` (= production build maken) aan `production`.

De omgevingsvariabelen worden toegevoegd aan de code _at build time_. Aangezien `create-react-app` een statische HTML/CSS/JS-bundel produceert, kan het deze onmogelijk tijdens runtime lezen. Daarom staan de waarden letterlijk in de code, plaats hier dus geen API keys en andere geheime sleutels in.

Voeg een `.env` file toe in de root folder met de environment settings. Hierin definiëren we de url naar de API:

```dotenv
REACT_APP_API_URL='http://localhost:9000/api'
```

In de code van `api/transactions.js` vervang je `baseUrl` door

```js
const baseUrl = `${process.env.REACT_APP_API_URL}/transactions`;
```

Over environment variables in React vind je meer op <https://create-react-app.dev/docs/adding-custom-environment-variables/>.

## Oefening

Pas nu ook `PlacesList` aan zodat dit werkt met onze REST API.

## Must reads

- [SOLID principles in React](https://konstantinlebedev.com/solid-in-react/)
- [Good advice on JSX conditionals](https://blog.thoughtspile.tech/2022/01/17/jsx-conditionals/)

## Mogelijke extra's voor de examenopdracht

- [react-query](https://www.npmjs.com/package/react-query)
- [swr](https://www.npmjs.com/package/swr)
