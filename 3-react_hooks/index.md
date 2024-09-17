# Formulieren & Context API

l> fe start 4331ea1 les4

## Inleiding
**Componenten** zijn functies die de UI renderen. Het renderen gebeurt wanneer de app voor het eerst geladen wordt en wanneer state waarden wijzigen. Renderen van code moet "[puur](https://react.dev/learn/keeping-components-pure)" zijn. Componenten mogen enkel 'hun' JSX retourneren en mogen objecten of variabelen die bestaan voor de rendering niet wijzigen (bv. geen nieuwe waarde toekennen aan props). Gegeven dezelfde input, dient het dezelfde output te retourneren. Net als een wiskundige formule zou het alleen het resultaat moeten berekenen, maar niets anders doen.

Een veel gemaakte denkfout is dat alle waarden (i.e. variabelen) in een component bewaard blijven, dat is **niet waar**. Een component is een functie, en functies hebben lokale variabelen. Eens de component zijn JSX geretourneerd heeft, zijn alle lokale variabelen weg. Je hebt nood aan de magie van React Hooks om waarden te bewaren tussen renders.

**Events** zijn functies binnen de component die worden uitgevoerd als reactie op een actie van een gebruiker. Een event handler kan state aanpassen, bv. een HTTP post request uitvoeren om een transactie toe te voegen. Event handlers bevatten **side-effects** veroorzaakt door een interactie. React biedt daarnaast ook de mogelijkheid voor side-effects na bv. een state-wijziging. Hierover in een volgend hoofdstuk meer.

In het vorige hoofdstuk hebben we kennis gemaakt met de `useState` en `useReducer` hooks. React heeft nog heel wat meer hooks (zie <https://react.dev/reference/react>) en er zijn reeds heel wat nuttige custom hooks te vinden op internet (zie bv. <https://nikgraf.github.io/react-hooks/>).

Hooks hebben ervoor gezorgd dat je met function components hetzelfde kan bereiken als met class components. Toch kan het zijn alsof hooks vreemd aanvoelen, alsof React de bal mis geslagen heeft in vergelijking met andere frameworks als [Solid.js](https://www.solidjs.com/) (zie <https://jakelazaroff.com/words/were-react-hooks-a-mistake/>).

## Regels voor hooks

Hooks zijn niet meer dan JavaScript functies. Echter moet je twee regels volgen wanneer je er gebruik van maakt. Je kan hiervoor een linter plugin installeren, dit gebeurt automatisch bij het gebruik van `vite`.

- Gebruik hooks enkel op het top niveau. Gebruik hooks niet binnen een if, andere condities, loops of geneste functies.
  - Reden: React valt terug op de volgorde waarin hooks worden aangeroepen om een waarde terug te geven. React houdt dit bij in een array. De volgorde moet dezelfde zijn bij elke render. Benieuwd naar meer info? Lees verder in [The First Rule of React Hooks, In Plain English](https://itnext.io/the-first-rule-of-react-hooks-in-plain-english-1e0d5ae32009)
- Roep hooks enkel aan vanuit React functies. Dit wil zeggen: enkel vanuit function components of vanuit eigen geschreven hooks.

Hooks maken gebruik van closures, let dus op voor stale closures! [Zie hier voor enkele voorbeelden](https://dmitripavlutin.com/react-hooks-stale-closures/)

In dit hoofdstuk gaan we vaak gebruik maken van hooks


## Formulieren

We maken een component voor het toevoegen en wijzigen van een transactie.

### Oefening 1

Voorzie volgende bijkomende routes in de budget-applicatie:

- `/transactions/add`: een nieuwe transactie toevoegen, via de AddOrEditTransaction component
- `/transactions/edit/:id`: een transactie bewerken, via de AddOrEditTransaction component

- Oplossing +

```jsx
//main.jsx
  {
    path: '/transactions',
    children: [
      {
        index: true,
        element: <TransactionList />,
      },
      {
        path: 'add',
        element: <AddOrEditTransaction />,
      },
      {
        path: 'edit/:id',
        element: <AddOrEditTransaction />,
      },
    ],
  }
```

### Oefening 2
Voorzie een knop "Add Transaction" naast de search bar en voor elke rij een "Edit Transaction" knop.

### Het formulier
Maak een bestand `AddOrEditTransaction.jsx` aan in de map `src\pages\transactions`. Deze pagina gebruikt de component TransactionForm die het formulier zal bevatten om een transactie te creëren en te wijzigen. We dienen alvast de places op te halen daar de gebruiker de plaats waar de transactie plaatsvindt zal moeten selecteren.


```jsx
// src/pages/transactions/AddOrEditTransaction.jsx
import useSWR from 'swr';// 👈 1
import {  getAll } from '../../api';// 👈 1
import TransactionForm from '../../components/transactions/TransactionForm';// 👈 2
import AsyncData from '../../components/AsyncData';// 👈 3

export default function AddOrEditTransaction() {
  const {
    data: places = [],
    error: placesError,
    isLoading: placesLoading,
  } = useSWR('places', getAll);// 👈 1

  return (
    <>
      <h1>
        Add transaction
      </h1>

      <AsyncData
        error={ placesError}
        loading={placesLoading}
      >{/* 👈 3 */}
        <TransactionForm
          places={places}
        />{/* 👈 2 */}
      </AsyncData>
    </>
  );
}
```

1. We maken gebruik van swr om alle plaatsen op te halen.
2. De TransactionForm component bevat het formulier voor de ingave van een transactie. We geven de plaatsen door.
3. Zorg voor foutafhandeling en loading indicator.

Maak een bestand `TransactionForm.jsx` aan in de map `src\components\transactions`. Dit bevat een formulier met drie input velden (userid, date en amount) en één select lijst (placeid)

```jsx
// src/pages/transactions/AddOrEditTransaction.jsx
export default function TransactionForm({places}) {
  return (
    <>
      <h2>Add transaction</h2>
      <form className='w-50 mb-3'>
        <div className='mb-3'>
          <label htmlFor='user' className='form-label'>
            User id
          </label>
          <input
            id='user'
            type='number'
            className='form-control'
            placeholder='user'
            required
          />
        </div>
        <div className='mb-3'>
          <label htmlFor='date' className='form-label'>
            Date
          </label>
          <input
            id='date'
            type='date'
            className='form-control'
            placeholder='date'
          />
        </div>

        <div className='mb-3'>
          <label htmlFor='places' className='form-label'>
            Place
          </label>
          <select id='places' className='form-select' required>
            <option defaultChecked>-- Select a place --</option>
            {places.map(({ id, name }) => (
              <option key={id} value={id}>
                {name}
              </option>
            ))}
          </select>
        </div>

        <div className='mb-3'>
          <label htmlFor='amount' className='form-label'>
            Amount
          </label>
          <input id='amount' type='number' className='form-control' required />
        </div>

        <div className='clearfix'>
          <div className='btn-group float-end'>
            <button type='submit' className='btn btn-primary'>
              Add transaction
            </button>
          </div>
        </div>
      </form>
    </>
  );
}
```

In HTML houden formulierelementen zoals `input`, `textarea` en `select` doorgaans hun eigen state bij. Ze werken deze bij op basis van gebruikersinvoer. Formulierelementen in React zijn read-only. Door state toe te voegen, kan de component zich aanpassen. In het vorige hoofdstuk hebben we een eenvoudig voorbeeld van een formulier behandeld maar validatie, foutafhandeling, formArrays... ontbreken nog. Je kan dit allemaal zelf implementeren of je kan gebruik maken van een package, zoals bv. [react-hook-form](https://react-hook-form.com/).

### React-hook-form
Voeg dit package toe aan het project:

```bash
yarn add react-hook-form
```

We maken gebruik van de [useForm](https://react-hook-form.com/api/useform) hook uit het `react-hook-form` package.

```jsx
// src/components/transactions/TransactionForm.jsx
// src/pages/transactions/AddOrEditTransaction.jsx
import { useForm } from 'react-hook-form'; // 👈 1
export default function TransactionForm({places}) {

  const { register, handleSubmit, reset } = useForm(); // 👈 2, 5 en 7

  // 👇 6
  const onSubmit = (data) => {
    console.log(JSON.stringify(data));
    //Nieuwe transactie moet nog worden opgeslaan
    reset(); // 👈 7
  };
  
  return (
    <>
      <h2>Add transaction</h2>
      <form onSubmit={handleSubmit(onSubmit)} className='w-50 mb-3'> {/*👈 5*/}
        <div className='mb-3'>
          <label htmlFor='user' className='form-label'>
            User Id
          </label>
          <input
            {...register('user')} 
            defaultValue='' 
            id='user'
            type='number'
            className='form-control'
            placeholder='userid'
            required
          />{/* 👈 3 en 4 */}
        </div>
        <div className='mb-3'>
          <label htmlFor='date' className='form-label'>
            Date
          </label>
          <input
            {...register('date')} 
            id='date'
            type='date'
            className='form-control'
            placeholder='date'
          />{/* 👈 3 */}
        </div>

        <div className='mb-3'>
          <label htmlFor='places' className='form-label'>
            Place
          </label>
          <select {...register('place')} id='places' className='form-select' required>
            <option defaultChecked>-- Select a place --</option>
            {places.map(({ id, name }) => (
              <option key={id} value={id}>
                {name}
              </option>
            ))}
          </select>{/* 👈 3 */}
        </div>

        <div className='mb-3'>
          <label htmlFor='amount' className='form-label'>
            Amount
          </label>
          <input {...register('amount')} id='amount' type='number' className='form-control' required />{/* 👈 3 */}
        </div>

        <div className='clearfix'>
          <div className='btn-group float-end'>
            <button type='submit' className='btn btn-primary'>
              Add transaction
            </button>
          </div>
        </div>
      </form>
    </>
  );
}
```

1. `useForm` is een **custom hook** om forms te beheren. Het geeft allerlei nuttige functies en andere info over het formulier terug. Neem maar een kijkje in de [documentatie](https://react-hook-form.com/api/useform/).
2. [register](https://react-hook-form.com/api/useform/register): met deze functie registreer je velden in het formulier en geef je een naam mee voor het veld. De waarden van de velden kunnen zo gebruikt worden voor zowel formuliervalidatie als het verzenden van het formulier. We hoeven zelf geen state bij te houden. (Achterliggend wordt [React.ref](https://react.dev/learn/referencing-values-with-refs) gebruikt)
3. Registreer de formuliervelden in de `useForm` hook. 
4. Je kan ook een standaardwaarde opgeven.
5. [handleSubmit](https://react-hook-form.com/api/useform/handlesubmit): deze functie zorgt ervoor dat de formuliergegevens verzameld worden bij het verzenden van het formulier. Je geeft aan deze functie een functie mee die opgeroepen moet worden als het formulier verzonden wordt.
6. De `onSubmit` functie logt de verstuurde waarden naar de console. `data` bevat de ingevulde waarden per formulierveld: `register('user')` wordt doorgegeven als `{ user:'value' }`.
7. [reset](https://react-hook-form.com/api/useform/reset): deze functie zet alle velden terug op de standaardwaarde (indien opgegeven) of maakt ze leeg.

### Validatie

In een applicatie kan je niet alleen werken met server-side validatie. In dat geval moet nl. de data eerst eens verzonden worden alvorens de validatie kan gebeuren. Echter kan je ook niet alleen vertrouwen op client-side validatie. Deze is eenvoudig uit te schakelen waardoor toch verkeerde data naar de server (en in de databank) kan komen.

We geven een voorbeeld voor het inputveld van de gebruiker, dit is vrij gelijkaardig voor de overige velden.

```jsx
// src/components/transactions/TransactionForm.jsx
import { useForm } from 'react-hook-form'; 

const validationRules = {
  user: {
    required: 'User is required',
    min: { value: 1, message: 'min 1' },
  },  
}; // 👈 1

export default function TransactionForm({places}) {
//...
<form onSubmit={handleSubmit(onSubmit)} className="w-50 mb-3">
  <div className="mb-3">
    <label htmlFor="user" className="form-label">User id</label>
    <input
      {...register('user', validationRules.user)}
      defaultValue=''
      id="user"
      type="number"
      className="form-control"
      placeholder="user"
      required
    />
    {/*👈 1  👇 2 */}
    {errors.user && <p className="form-text text-danger">{errors.user.message}</p> }
  </div>
```

1. Als tweede parameter van de `register` functie kan je de validatieregels meegeven (required, min, max, minLength, maxLength, pattern, validate). Je kan ook de bijhorende foutmelding opgeven. Hiervoor definiëren we een constante validationRules. Dit plaatsen we buiten de component. Waarom? 
React Hook Form ondersteunt ook schema-validatie met Yup, Zod, Superstruct & Joi. De validatie is afgestemd op de HTML-standaard voor formuliervalidatie. Meer hierover in de documentatie van [register](<https://react-hook-form.com/api/useform/register>. Voor inputveld met als type `number` dien je `valueAsNumber` in te stellen zodat je een getal i.p.v. een string terugkrijgt.
2. Voor de weergave van de fouten maken we gebruik van het `errors` object. Aan de hand van het `type` property kan je het type van de fout opvragen (bv. `errors.user.type === 'required'`). Merk op dat we hier gebruik maken van `&&`, dit wordt wel eens gezien als een anti-pattern in React. Het is eigenlijk beter om de ternary operator (`voorwaarde ? true : false`) te gebruiken. Dit wordt dus `{errors.user ? <p className="form-text text-danger">{errors.user.message}</p> : null}`.

Vermits er meerdere invoervelden op ons formulier voorkomen en we steeds dezelfde code moeten schrijven, zouden we een aparte component moeten maken. Deze component zal gebruik moeten maken van [useFormContext](https://react-hook-form.com/api/useformcontext). Dit komt op het einde van dit hoofdstuk aan bod.


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
import Error from '../Error'; // 👈 4

export default function TransactionForm({places}) { 
  const {
    trigger: saveTransaction,
    error: saveError,
  } = useSWRMutation('transactions', save); // 👈 2

  // ...

  const onSubmit = async (data) => { 
    const { user, place, amount, date } = data;
    await saveTransaction({
      userId: user, 
      placeId: place, 
      amount: parseInt(amount),
      date: new Date(date) }); 
    reset();
  } // 👈 3

  // ...
  return (
    <>
      <h2>Add transaction</h2>
      {/* 👇 4  */}
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
3. Wijzig de `onSubmit` zodat `saveTransaction` aangeroepen wordt. We geven de `user`, `place`, `amount` en `date` mee als argumenten als **object** . **Let op:** de functie is `async`, dus we moeten `await` gebruiken.
4. We tonen een eventuele fout na het opslaan d.m.v. de `Error` component. Vergeet de import niet!

De API verwacht een `id` voor de place. We retourneren `placeId` als geselecteerde waarde in de select list. In het formulier moet je vanaf nu in de select lijst werken met het `id`.

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

We geven via een param in de url mee welke transactie moet worden aangepast en dan halen we de transactie op. In het formulier bekijken we via een `useEffect` bij elke render of er een huidige transactie is ingesteld en indien nodig tonen we de bijhorende gegevens in alle formuliervelden.

In de API hebben we een functie nodig om de aan te passen transactie op te halen en up te daten.
```jsx
export const getById = async (url) => {
  const {
    data,
  } = await axios.get(`${baseUrl}/${url}`);

  return data;
};
```

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

In de `Transaction` component voegen we een potlood-icoon toe:

```jsx
import { IoTrashOutline, IoPencil } from 'react-icons/io5'; // 👈 1
//...
  <td>
    <Link to={`/transactions/edit/${id}`} className="btn btn-light">
        <IoPencilOutline />
      </Link> // 👈 2
    <button className='btn btn-danger' onClick={handleDelete}>
      <IoTrashOutline />
    </button>
  </td>
```

1. We importeren het potlood-icoon.
2. En voegen de Link toe


In `AddOrEditTransaction` kijken we of het om een add of edit gaat. In het laatste geval halen we de betreffende transactie op en geven dit door aan de `TransactionForm`.

```jsx
// src/pages/transactions/AddOrEditTransaction.jsx
import { useParams } from 'react-router-dom'; // 👈 1
import {  getAll, getById } from '../../api';// 👈 3
//andere imports

export default function AddOrEditTransaction() {
  const { id } = useParams();// 👈 2

  const {
    data: transaction,
    error: transactionError,
    isLoading: transactionLoading,
  } = useSWR(id ? `transactions/${id}` : null, getById);// 👈 3

  const {
    data: places = [],
    error: placesError,
    isLoading: placesLoading,
  } = useSWR('places', getAll);

  return (
    <>
      <h1>
        Add transaction
      </h1>

      <AsyncData
        error={transactionError || placesError}
        loading={transactionLoading || placesLoading}
      > {/* 👈 5 */}
        <TransactionForm
          places={places}
          transaction={transaction}
        />{/* 👈 4 */}
      </AsyncData>
    </>
  );
}
```
1. Importeer useParams 
2. Extraheer de id uit de url
3. Maak gebruik van useSWR om de aan te passen transactie op te halen. Indien geen id parameter werd meegegeven wordt null teruggegeven.
4. Geef de aan te passen transactie door aan `TransactionForm`
5. Zorg voor foutafhandeling en laadindicator


In het `TransactionForm` voegen we de `useEffect` toe zodat bij elke render het volgende gecontroleerd wordt: als `transaction` is ingevuld dan gaat het om een update, anders om een create.

```jsx
import { useEffect, navigate } from 'react'; // 👈 2 en 4
//..
// 👇 2
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

export default function TransactionForm({places, transaction}) {  // 👈 1
  const {
    trigger: saveTransaction,
    error: saveError,
  } = useSWRMutation('transactions', save); 

  const { register, handleSubmit, reset, formState: { errors } , setValue} = useForm(); // 👈 2

  // 👇 4
  const onSubmit = async (data) => {
    const { user, place, amount, date } = data;
    await saveTransaction({
      userId: user, 
      placeId: place, 
      amount: parseInt(amount),
      date: new Date(date),
      id: transaction?.id,
    });
    navigate('/transactions');
  };

  // 👇 2
  useEffect(() => {
    if (
      // check on non-empty object
      transaction &&
      (Object.keys(transaction).length !== 0 ||
          transaction.constructor !== Object)
    ) {
      const dateAsString = toDateInputString(new Date(transaction.date));
      setValue('date', dateAsString);
      setValue('user', transaction.user.id);
      setValue('place', transaction.place.id);
      setValue('amount', transaction.amount);
    } else {
      reset();
    }
  }, [transaction, setValue, reset]);

  //..
  return (
    {/* ... */}
    {/* 👇 3  */}
    <button type='submit' className='btn btn-primary'>
      {transaction?.id
        ? "Save transaction"
        : "Add transaction"}
    </button>
    {/* ... */}
  );
}
```

1. Ontvang `transaction` door als prop.
2. Controleer bij elke render of `transaction` is ingevuld.
   - Indien ingevuld, plaats de waarden in het formulier. Maak hiervoor gebruik van `setValue` uit de `useForm` hook. We dienen de datum te formatteren. Hiervoor voorzien we de functie `toDateInputString`. Constanten definiëren we buiten de component.
   - Indien niet ingevuld, maak het formulier leeg.
3. Pas de tekst op de knop aan i.f.v. of het om een update of een create gaat.
4. Bij het opslaan van de transactie geven we ook de id mee. En we navigeren terug naar de `TransactionList` pagina. We hoeven het formulier niet meer te resetten.

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


### Oefening 2 - Cat breeds

Als we surfen naar <https://api.thecatapi.com/v1/breeds> dan krijgen we JSON met alle kattenrassen. Maak een pagina waar de gebruiker een ras kan selecteren, en waar de details van het ras getoond worden.

- Gebruik dit bestand met mock data: [mock_data.js](https://raw.githubusercontent.com/HOGENT-Web/frontend-ch3-exercise-solution/main/src/api/mock_data.js).
- Hou alle rassen bij in een state variabele.
- Hou de geselecteerde ras ook bij in state.
- Maak een formulier om een nieuw ras toe te voegen. Voeg deze toe aan de bijgehouden rassen. Alle velden met uitzondering van de `description` en `imageUrl` zijn verplicht in te vullen.
- Maak gebruik van [bootstrap](https://getbootstrap.com/docs/).
- Verwijder voor de eenvoud ESLint en het `.eslintrc.cjs` bestand.

```bash
rm .eslintrc.cjs
yarn remove eslint eslint-plugin-react eslint-plugin-react-hooks eslint-plugin-react-refresh
```

![Voorbeeld van de kattenrassen-applicatie](./images/cats.PNG)

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  Een voorbeeldoplossing is te vinden op <https://github.com/HOGENT-Web/frontend-ch3-exercise-solution> in commit `4db7f21`

  ```bash
  git clone https://github.com/HOGENT-Web/frontend-ch3-exercise-solution.git
  cd frontend-ch3-exercise-solution
  git checkout -b oplossing 4db7f21
  yarn install
  yarn dev
  ```

## Context API

We creëren geneste componenten om de UI te bouwen. De state plaatsen we in de root component en wordt via props doorgegeven aan de kinderen. Dit kan echter heel complex worden als je sommige props tot diep in de boom dient door te geven of als heel wat componenten dezelfde props nodig hebben.

De **Context API** laat toe om data globaal bij te houden en door te geven aan child components, zonder dat we via props de data tot in deze kinderen moeten doorgeven. Dus Context API is een alternatief voor het doorgeven van props.

**Let op**, de Context API wordt vaak ten onrechte gebruikt als oplossing. Gebruik het enkel en alleen als de state echt globaal moet zijn. Gebruik de context ook zo laag mogelijk in de component tree.

Een aantal use cases voor context zijn o.a. theming, taalkeuze, huidige gebruiker...

De Context API bestaat uit drie bouwstenen:

1. Een **Context Object**, aangemaakt door de factory functie `createContext`
2. Een **Context Provider**: voorziet de onderliggende componenten van data
3. Meerdere **Context Consumers** : ontvangen de data van de context

Om de werking van de React Context API te demonstreren kan de gebruiker van onze budgetapplicatie een licht of donker thema kiezen. We gaan hiervoor een context gebruiken aangezien het thema van de website door alle componenten gekend moet zijn om bv. de kleuren aan te passen.

Het aanmaken van een context gebeurt typisch in 3 stappen:

1. Creëer een context.
2. Bied de context aan via een provider.
3. Neem data uit de context via een consumer (of dus in een child component).

### Refactor TransactionList

We refactoren de `TransactionList` component zodat die nu een tabel met transacties weergeeft.

```jsx
// src/components/transactions/TransactionList.jsx
import { useState, useMemo, useCallback } from 'react';
import Transaction from './Transaction';
import TransactionForm from './TransactionForm';
import { TRANSACTION_DATA } from '../../api/mock_data';

function TransactionTable({
  transactions
}) {
  if (transactions.length === 0) {
    return (
      <div className="alert alert-info">
        There are no transactions yet.
      </div>
    );
  }

  return (
    <div>
      <table className={`table table-hover table-responsive`}>
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
            <Transaction key={transaction.id} {...transaction} />
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default function TransactionList() {
  const [transactions, setTransactions] = useState(TRANSACTION_DATA);
  const [text, setText] = useState('');
  const [search, setSearch] = useState('');

  const filteredTransactions = useMemo(() => transactions.filter((t) => {
    return t.place.toLowerCase().includes(search.toLowerCase());
  }), [search, transactions])

  const createTransaction = useCallback((user, place, amount, date) => {
    const newTransactions = [
      {
        user, place, amount, date: new Date(date),
      },
      ...transactions,
    ]; // newest first
    setTransactions(newTransactions);
  }, [transactions]);

  return (
    <>
      <h1>Transactions</h1>
      <TransactionForm onSaveTransaction={createTransaction} />
      <div className="input-group mb-3 w-50">
        <input
          type="search"
          id="search"
          className="form-control rounded"
          placeholder="Search"
          value={text}
          onChange={(e) => setText(e.target.value)} />
        <button type="button" className="btn btn-outline-primary" onClick={() => setSearch(text)}>Search</button>
      </div>
      <div className="mt-4">
        <TransactionTable transactions={filteredTransactions} />
      </div>
    </>
  );
}
```

Pas zelf de Transaction component aan zodat de transacties als rij in de tabel worden weergegeven.

### Stap 1: Creëer de context

Voorzie voorlopig in `App.jsx` een knop om het thema te kiezen. In het hoofdstuk [routing](./../5-react_router/index.md) wordt dit een onderdeel van de navigatiebalk. We maken buiten de `App` component een context aan m.b.v. `createContext`. Deze factory-functie heeft één optioneel argument, de standaardwaarde. Exporteer `ThemeContext` zodat de consumers dit kunnen gebruiken.

```jsx
// src/App.jsx
import { createContext } from 'react'; // 👈
import TransactionList from './components/transactions/TransactionList';
import PlacesList from './components/places/PlacesList';

export const ThemeContext = createContext(); // 👈

function App() {
  return (
    <div>
      <TransactionList />
      <PlacesList />
    </div>
  );
}

export default App;
```

### Stap 2: Bied de context aan

Voeg toe in `App.jsx`:

```jsx
// src/App.jsx
import TransactionList from './components/transactions/TransactionList';
import PlacesList from './components/places/PlacesList';
import {createContext} from 'react';

export const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value={{ theme:'dark' }}> {/* 👈 */}
      <div>
        <TransactionList />
        <PlacesList />
      </div>
    </ThemeContext.Provider>
  );
}
export default App;
```

Elk **context object** wordt beschikbaar gemaakt met een **context provider** component waarin de data geplaatst wordt. Een context provider plaats je rond de volledige component tree of bepaalde secties ervan. Alle kinderen (de **context consumers**) binnen de context provider hebben toegang tot de data en kunnen zich abonneren op wijzigingen. De `value` property bevat de data die in de context geplaatst wordt. Geef een object door (vandaar `{{}}`). Alle kinderen die afstammen van de provider zullen opnieuw renderen wanneer de waarde van de Provider verandert.

> Het is niet verplicht om een context steeds in de App component te zetten, je zet hem zo laag mogelijk zodat de nodige child componenten aan de data kunnen.

### Stap 3: Consume de context

De data hoeft niet langer doorgegeven te worden via props. Gebruik bv. het thema in de `TransactionTable` component. De `TransactionsTable` component zal de data consumeren, en is een context consumer.

```jsx
// src/components/transactions/TransactionList.jsx
import { useState, useMemo, useCallback, useContext } from 'react'; // 👈 1
import { ThemeContext } from '../../App';
//...

function TransactionTable({
  transactions
}) {
  const { theme } = useContext(ThemeContext); // 👈 1

  if (transactions.length === 0) {
    return (
      <div className="alert alert-info">
        There are no transactions yet.
      </div>
    );
  }

  return (
    <div>
      <table className={`table table-hover table-responsive table-${theme}`}>{/* 👈 */}
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
          {transactions.map((transaction, index) => (
            <Transaction key={index} {...transaction} onDelete={onDelete} onEdit={onEdit} />
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

1. De `useContext` hook wordt gebruikt om met de context te connecteren en heeft als parameter een context object `ThemeContext`. Het retourneert de `value` van de huidige context (zie value property van de context provider).

De context provider kan data in de context plaatsen, maar het kan de data in de context niet aanpassen. Willen we ook nog functies toevoegen aan de Context om om te schakelen van dark naar light mode of omgekeerd, dienen we een aparte component aan te maken.

### ThemeContext en Provider

Maak een map `contexts` aan in de map `src` met daarbinnen het bestand `Theme.context.jsx`:

```jsx
// src/contexts/Theme.context.jsx
import { createContext} from 'react'; // 👈 1

export const ThemeContext = createContext(); // 👈 1

export const ThemeProvider = ({ // 👈 2
  children
}) => {
  return ( // 👈 2
    <ThemeContext.Provider>
      {children}
    </ThemeContext.Provider>
  );
};
```

1. Importeer `createContext` en maak de context aan
2. Maak een stateful component genaamd `ThemeProvider` die de `children` als prop ontvangt. `children` bevat de component tree waarrond de Provider geplaatst wordt. `ThemeProvider` beheert de data en stelt het ter beschikking van deze children.
3. De component `ThemeProvider` rendert de context provider die de consumers als children zal hebben.

De `ThemeProvider` beheert de state en functies, en stelt deze ter beschikking aan de children. Hou hier als state het thema bij en een functie om te wisselen van thema.

```jsx
// src/contexts/Theme.context.jsx
import {
  createContext,
  useState,
  useCallback,
  useMemo
} from 'react';

export const themes = { // 👈 1
  dark: "dark",
  light: "light"
}

export const ThemeContext = createContext();

export const ThemeProvider = ({
  children
}) => {
  const [theme, setTheme] = useState(sessionStorage.getItem('themeMode') || themes.dark); // 👈 2

  const toggleTheme = useCallback(() => { // 👈 3
    const newThemeValue = theme === themes.dark ? themes.light : themes.dark;
    setTheme(newThemeValue);
    sessionStorage.setItem('themeMode', newThemeValue);
  }, [theme]);

  const value = useMemo (()=> ({ theme, toggleTheme}), [theme, toggleTheme]); // 👈 4

  return (
    <ThemeContext.Provider value={value}>{/* 👈 5 */}
      {children}
    </ThemeContext.Provider>
  );
};
```

1. We definiëren eerst een constante `themes` met de mogelijke waarden.
2. De `ThemeProvider` houdt de state bij, hier het geselecteerde thema. Eventueel halen we een vorige waarde uit `sessionStorage` op.
3. De `ThemeProvider` voorziet ook in een functie om het thema aan te passen.
4. De `ThemeProvider` bepaalt wat er gedeeld wordt met de children. Maak hiervoor de constante `value` aan. Vermits het hier om een waarde (en geen functie) gaat, gebruiken we `useMemo`.
5. De `ThemeProvider` stelt de `value` ter beschikking van de children.

### Kleur van de tekst

Het thema zal gebruikt worden om de achtergrondkleur in te stellen, maar soms dient ook de kleur van de tekst of van een rand te worden ingesteld (de tegengestelde kleur). Dus we voorzien een extra berekende waarde en maken ook deze waarde beschikbaar.

```jsx
// src/contexts/Theme.context.jsx
import {
  createContext,
  useState,
  useCallback,
  useMemo
} from 'react';

export const themes = {
  dark:"dark",
  light:"light"
}

export const ThemeContext = createContext();

export const ThemeProvider = ({
  children
}) => {
  const [theme, setTheme] = useState(sessionStorage.getItem('themeMode') || themes.dark);

  const toggleTheme = useCallback(() => {
    const newThemeValue = theme === themes.dark ? themes.light : themes.dark;
    setTheme(newThemeValue);
    sessionStorage.setItem('themeMode', newThemeValue);
  }, [theme]);

  const oppositeTheme = useMemo(() => theme === themes.dark ? themes.light : themes.dark, [theme]); // 👈 1

  const value = useMemo(()=> ({ theme, oppositeTheme, toggleTheme }), [theme, oppositeTheme, toggleTheme]); // 👈 2

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

1. Voeg de berekende waarde`oppositeTheme` toe.
2. Maak `oppositeTheme` beschikbaar voor de children. Merk op: je kan ook meteen `oppositeTheme` berekenen in de `useMemo` die `value` bepaald.

### Providing ThemeContext

Stel in `main.jsx` de `ThemeProvider` ter beschikking aan alle children.

```jsx
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './index.css';
import { ThemeProvider } from './contexts/Theme.context';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <ThemeProvider>{/* 👈 */}
      <App />
    </ThemeProvider>{/* 👈 */}
  </React.StrictMode>
);
```

### Consuming ThemeContext

Voeg in `App.jsx` een knop toe om van thema te wisselen:

```jsx
// src/App.jsx
import TransactionList from './components/transactions/TransactionList';
import PlacesList from './components/places/PlacesList';
import { ThemeContext, themes } from './contexts/Theme.context'; // 👈 1
import { IoMoonSharp, IoSunny } from 'react-icons/io5'; // 👈 1
import { useContext } from 'react'; // 👈 1

function App() {
  const { theme, oppositeTheme, toggleTheme } = useContext(ThemeContext); // 👈 2

  return (
    <div className={`container-xl bg-${theme} text-${oppositeTheme}`}>{/* 👈 3 */}
      <button type="button" onClick={toggleTheme}>{/* 👈 4 */}
        {theme === themes.dark ? <IoMoonSharp /> : <IoSunny />}
      </button>
      <TransactionList />
      <PlacesList />
    </div>
  );
}
export default App;
```

1. Importeer de nodige componenten.
2. Roep de `useContext` hook aan en gebruik destructuring om de properties, die de component nodig heeft, eruit te halen.
3. Voeg de bootstrap klassen toe voor de achtergrondkleur en de kleur van de tekst.
4. Voorzie de knop om het thema te kiezen.

De `TransactionTable` component blijft behouden. De `ThemeContext` komt nu wel niet uit `App.jsx` maar `Theme.context.jsx`:

```jsx
import { ThemeContext } from '../../contexts/Theme.context';
```

### Oefening 3 - ThemeContext

Pas de andere componenten aan.

## Custom hooks

Een custom hook is een JavaScript functie die begint met `use` en die andere hooks kan aanroepen. Custom hooks laten toe om logica te delen tussen componenten. Je kan zelf custom hooks schrijven of eens zoeken naar bestaande custom hooks via bv. <https://nikgraf.github.io/react-hooks/>.

In elke component die gebruik maakt van de context dienen we volgende code te schrijven:

```jsx
import { ThemeContext } from '../../contexts/Theme.context';

const { theme, ... } = useContext(ThemeContext);
```

Om duplicate code te vermijden kunnen we gebruik maken van een **custom hook**. Neem hiervoor eerst [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) door.

Maak vervolgens twee custom hooks aan in `Theme.context.jsx`:

```jsx
// src/contexts/Theme.context.jsx
import {
  createContext,
  useState,
  useCallback,
  useMemo,
  useContext
} from 'react'; // 👈 1

export const themes = {
  dark: "dark",
  light: "light"
}

export const ThemeContext = createContext();

export const useTheme = () => useContext(ThemeContext); // 👈 1

// 👇 2
export const useThemeColors = () => {
  const { theme, oppositeTheme } = useContext(ThemeContext);
  return { theme, oppositeTheme };
};

//...
```

1. Deze hook retourneert de drie waarden `theme`, `oppositeTheme` en `toggleTheme`.
2. Deze hook retourneert enkel het `theme` en `oppositeTheme` retourneert.

Zo kan de code in `App.jsx` als volgt aangepast worden:

```jsx
// src/App.jsx
import TransactionList from './components/transactions/TransactionList';
import PlacesList from './components/places/PlacesList';
import { useTheme, themes } from './contexts/Theme.context'; // 👈 1
import { IoMoonSharp, IoSunny } from 'react-icons/io5';
//import { useContext } from 'react'; // 👈 1

function App() {
  const { theme, oppositeTheme, toggleTheme } = useTheme(); // 👈3
  //...
```

1. Verwijder de import `useContext`, `ThemeContext`, en importeer `useTheme`.
2. Destructure de waarden die in deze component gebruikt worden.

Als laatste pas je ook `Place.jsx` aan:

```jsx
// src/components/places/Place.jsx
import { memo, useCallback } from 'react'; // 👈 1
import { useThemeColors } from '../../contexts/Theme.context'; // 👈 1
import StarRating from './StarRating';

const Place = memo(({ id, name, rating, onRate, onDelete }) => {
  const { theme, oppositeTheme } = useThemeColors(); // 👈 2
  // ...
```

1. Verwijder de import `useContext`, `ThemeContext`, en importeer `useThemeColors`.
2. De hook `useThemeColors` retourneert de twee waarden die we in deze component nodig hebben.

Pas ook `StarRating` component aan.

## Form revisited - code refactoring

Er zitten een paar anti-patterns in ons formulier:

1. Gebruik geen constante object literals/arrays in de component, bv. validatieregels. Plaats deze buiten de component.
2. Definieer geen pure functies in de component (functies zonder afhankelijkheden van variabelen), bv. `toDateInputString`. Plaats deze buiten de component.
3. Definieer geen componenten inline in een andere component, bv. `LabelInput`. Plaats deze buiten de component.
4. Gebruik een id als waarde voor de `key` prop in lijsten, gebruik geen index.

### Constante objecten/arrays

```jsx
// src/components/transactions/TransactionForm.jsx
//...

const validationRules = { // 👈 1
  user: {
    required: 'User is required',
    minLength: { value: 2, message: 'Min length is 2' },
  },
  date: { required: 'Date is required' },
  place: { required: 'Place is required' },
  amount: {
    valueAsNumber: true,
    required: 'Amount is required',
    min: { value: 1, message: 'min 1' },
    max: { value: 5000, message: 'max 5000' },
  }
};

//...

export default memo(function TransactionForm({ onSaveTransaction }) {
  //...

  return (
    <>
      <h2>
        Add transaction
      </h2>
      <form onSubmit={handleSubmit(onSubmit)} className="w-50 mb-3">
        <div className="mb-3">
          <label htmlFor="user" className="form-label">Who</label>
          <input
            {...register('user', validationRules.user)}//👈2
            defaultValue=''
            id="user"
            type="text"
            className="form-control"
            placeholder="user" required
          />
          {errors.user && <p className="form-text text-danger">{errors.user.message}</p>}
        </div>

        {/* Pas zelf de andere validatieregels aan */}
      </form>
    </>
  );
})

```

1. `validationRules` bevat per formulierveld de validatieregels. We definiëren dit buiten de component, anders wordt dit bij elke render opnieuw aangemaakt.
2. Het `input` field maakt hiervan gebruik voor de validatie. Voordeel: er wordt geen nieuw object aangemaakt bij elke render.

### Duplicate code

De combinatie `label` en `input` tag komen vaak voor. Kunnen we hier aparte component van maken?

Componenten mag je niet definiëren binnen een andere component. Maak een functiecomponent `LabelInput` in het bestand van het formulier. Plaats de code van het invoerveld van de gebruiker hierin:

```jsx
// src/components/transactions/TransactionForm.jsx
function LabelInput() {
  return (
    <div className="mb-3">
      <label htmlFor="user" className="form-label">Who</label>
      <input
        {...register('user', validationRules.user)}
        defaultValue=''
        id="user"
        type="text"
        className="form-control"
        placeholder="user"
        required
      />
      {errors.user && <p className="form-text text-danger">{errors.user.message}</p>}
    </div>
  );
}
```

Definieer de props van deze component:

```jsx
// src/components/transactions/TransactionForm.jsx
function LabelInput({ label, name, type, validationRules, ...rest }) {
  const hasError = name in errors;

  return (
    <div className="mb-3">
      <label htmlFor={name} className="form-label">
        {label}
      </label>
      <input
        {...register(name, validationRules)}
        id={name}
        type={type}
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
```

We krijgen nog fouten: `register en errors not defined`. Oplossing: `useFormContext` en `FormProvider`. Definitie van `useFormContext` uit de documentatie:

> This custom hook allows you to access the FormContext. useFormContext is intended to be used in deeply nested structures, where it would become inconvenient to pass the context as a prop.

Met `FormProvider` creëren we een nieuwe context provider voor een formulier. Met `useFormContext` halen we hieruit de waarde op en gebruiken dit in een (diep) geneste component tree.

In de documentatie lezen we ook het volgende over de `FormProvider`:

> React Hook Form's FormProvider is built upon React's Context API. It solves the problem where data is passed through the component tree without having to pass props down manually at every level. This also causes the component tree to trigger a rerender when React Hook Form triggers a state update.

```jsx
// src/components/transactions/TransactionForm.jsx
import { FormProvider, useForm, useFormContext } from 'react-hook-form'; // 👈 1 en 2
// ...

function LabelInput({ label, name, type, validationRules, ...rest }) {
  const {
    register,
    errors,
  } = useFormContext(); // 👈 2

  const hasError = name in errors;

  return (
    <div className="mb-3">
      <label htmlFor={name} className="form-label">
        {label}
      </label>
      <input
        {...register(name, validationRules)}
        id={name}
        type={type}
        className="form-control"
        {...rest}
      />{/* 👈 3 */}
      {hasError ? (
        <div className="form-text text-danger">
            {errors[name].message}
        </div>
      ) : null}
    </div>
  );
}

export default function TransactionForm() {
  // ...

  return (
    <FormProvider handleSubmit={handleSubmit} errors={errors} register={register}> {/* 👈 1 */}
      <form onSubmit={handleSubmit(onSubmit)} className="m-5">
        <LabelInput
          label="User"
          name="user"
          type="user"
          validationRules={validationRules.user} /> {/* 👈 3 */}

          {/* Herhaal dit voor de overige input fields */}
      </form>
    </FormProvider>
  );
}
```

1. Importeer de `FormProvider` en plaats de `FormProvider` rond het formulier om de `useFormContext` correct te laten werken.
2. Importeer `useFormContext` en maak gebruik van `useFormContext` voor het gebruik van `register` en `errors`.
3. Maak gebruik van de component `LabelInput`.

### Oefening 4 - PlacesSelect

Maak een component `PlacesSelect` aan die de lijst van places toont. Deze component heeft geen props.

Mocht je later nog een `select` tag nodig hebben, kan je nog altijd een generieke `Select` component maken. Probeer niet te vroeg te abstraheren, dat maakt het moeilijker om de code te begrijpen.

### Oefening 5 - Cat breeds 2.0

Pas de Cat Breeds applicatie aan zodat ook hier met de twee thema's gewerkt kan worden. Refactor eventuele anti-patterns in de applicatie.

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  Een voorbeeldoplossing is te vinden op <https://github.com/HOGENT-Web/frontend-ch3-exercise-solution> in commit `9181134`

  ```bash
  git clone https://github.com/HOGENT-Web/frontend-ch3-exercise-solution.git
  cd frontend-ch3-exercise-solution
  git checkout -b oplossing 9181134
  yarn install
  yarn dev
  ```

## Mogelijke extra's voor de examenopdracht

- [Formik](https://www.npmjs.com/package/formik)
- Validatie in [react-hook-form](https://www.npmjs.com/package/react-hook-form) met [Joi](https://www.npmjs.com/package/joi), [Yup](https://npmjs.com/package/yup) of dergelijke
- Maak gebruik van custom hooks uit:
  - <https://nikgraf.github.io/react-hooks/>
  - <https://github.com/streamich/react-use>
  - Zelf custom hooks schrijven is **geen** extra

## Must reads

- [Collection of React Hooks](https://nikgraf.github.io/react-hooks/)
- [react-use: Collection of essential React Hooks.](https://github.com/streamich/react-use)
- [Are You Making This React State Mistake?](https://www.youtube.com/watch?v=NZqMVUEiDIw)
- [useState: Asynchronous or what?](https://youtu.be/RAJD4KpX8LA)
- [3 React Mistakes, 1 App Killer](https://youtu.be/QuLfCUh-iwI)
- [Were React Hooks a Mistake?](https://jakelazaroff.com/words/were-react-hooks-a-mistake/)
- [How React 18 Improves Application Performance](https://vercel.com/blog/how-react-18-improves-application-performance)
