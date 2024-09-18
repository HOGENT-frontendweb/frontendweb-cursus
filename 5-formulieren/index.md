# Formulieren & hooks

In dit hoofdstuk maken we een component aan voor het toevoegen en wijzigen van een transactie. We bekijken ook hoe we de performantie verder kunnen verbeteren.

l> fe start 4331ea1 les5

## Routing

We maken een component voor het toevoegen en wijzigen van een transactie.

### Oefening 1

Maak een bestand `AddOrEditTransaction.jsx` aan in de map `src\pages\transactions`. Voeg hieraan een placeholder toe zodat je weet dat de pagina correct wordt weergegeven.

Voorzie volgende bijkomende routes in de budget-applicatie:

- `/transactions/add`: een nieuwe transactie toevoegen, via de `AddOrEditTransaction` component
- `/transactions/edit/:id`: een transactie bewerken, via de `AddOrEditTransaction` component

<br />

- Oplossing +

  De `AddOrEditTransaction` component:

  ```jsx
  // src/pages/transactions/AddOrEditTransaction.jsx
  export default function AddOrEditTransaction() {
    return <h1>Add transaction</h1>;
  }
  ```

  De routes worden toegevoegd in `main.jsx`:

  ```jsx
  // src/main.jsx
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

Voorzie een knop "Add Transaction" naast de search bar en een potloodknop in de lijst voor elke transactie.

- Oplossing +

  In `TransactionList.jsx` voeg je onderstaande code toe:

  ```jsx
  <div className='clearfix'>
    <Link to='/transactions/add' className='btn btn-primary float-end'>
      Add transaction
    </Link>
  </div>
  ```

  In `Transaction.jsx` voeg je onderstaande code toe:

  ```jsx
  <Link to={`/transactions/edit/${id}`} className='btn btn-light'>
    <IoPencilOutline />
  </Link>
  ```

  Vergeet de import van de `IoPencilOutline` component niet!

## Het formulier

Maak een bestand `AddOrEditTransaction.jsx` aan in de map `src\pages\transactions`. Deze pagina gebruikt de component `TransactionForm` die het formulier zal bevatten om een transactie te creëren en te wijzigen. We dienen alvast de places op te halen aangezien de gebruiker de plaats waar de transactie plaatsvindt zal moeten selecteren.

```jsx
// src/pages/transactions/AddOrEditTransaction.jsx
import useSWR from 'swr'; // 👈 1
import { getAll } from '../../api'; // 👈 1
import TransactionForm from '../../components/transactions/TransactionForm'; // 👈 2
import AsyncData from '../../components/AsyncData'; // 👈 3

export default function AddOrEditTransaction() {
  const {
    data: places = [],
    error: placesError,
    isLoading: placesLoading,
  } = useSWR('places', getAll); // 👈 1

  return (
    <>
      <h1>Add transaction</h1>

      {/* 👇 3 */}
      <AsyncData error={placesError} loading={placesLoading}>
        {/* 👇 2 */}
        <TransactionForm places={places} />
      </AsyncData>
    </>
  );
}
```

1. We maken gebruik van swr om alle plaatsen op te halen.
2. De `TransactionForm` component bevat het formulier voor de ingave van een transactie. We geven de plaatsen door.
   - Deze component voegen we hieronder toe.
3. Zorg voor foutafhandeling en loading indicator.

Maak een bestand `TransactionForm.jsx` aan in de map `src\components\transactions`. Dit bevat een formulier met drie input velden (userId, date en amount) en één select lijst (placeId). Het userId zal later geschrapt worden en vervangen worden door het id van de aangemelde gebruiker.

```jsx
// src/components/transactions/TransactionForm.jsx
export default function TransactionForm({ places }) {
  return (
    <>
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

?> Voor simpele formulieren zoals bv. een zoekbalk kan je gebruik maken van controlled components. Voor complexere formulieren is het aangeraden om een library te gebruiken zoals `react-hook-form`.

### React-hook-form

Voeg dit package toe aan het project:

```terminal
yarn add react-hook-form
```

We maken gebruik van de [useForm](https://react-hook-form.com/api/useform) hook uit het `react-hook-form` package.

```jsx
// src/components/transactions/TransactionForm.jsx
import { useForm } from 'react-hook-form'; // 👈 1
export default function TransactionForm({ places }) {
  const { register, handleSubmit, reset } = useForm(); // 👈 2, 5 en 7

  // 👇 6
  const onSubmit = (data) => {
    console.log(JSON.stringify(data));
    //Nieuwe transactie moet nog worden opgeslaan
    reset(); // 👈 7
  };

  return (
    <>
      {/* 👇 5*/}
      <form onSubmit={handleSubmit(onSubmit)} className='w-50 mb-3'>
        <div className='mb-3'>
          <label htmlFor='user' className='form-label'>
            User Id
          </label>
          {/* 👇 3 en 4 */}
          <input
            {...register('user')}
            defaultValue=''
            id='user'
            type='number'
            className='form-control'
            placeholder='userid'
            required
          />
        </div>
        <div className='mb-3'>
          <label htmlFor='date' className='form-label'>
            Date
          </label>
          {/* 👇 3 */}
          <input
            {...register('date')}
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
          {/* 👇 3 */}
          <select
            {...register('place')}
            id='places'
            className='form-select'
            required
          >
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
          {/* 👇 3 */}
          <input
            {...register('amount')}
            id='amount'
            type='number'
            className='form-control'
            required
          />
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
2. [register](https://react-hook-form.com/api/useform/register): met deze functie registreer je velden in het formulier en geef je een naam mee voor het veld. De waarden van de velden kunnen zo gebruikt worden voor zowel formuliervalidatie als het verzenden van het formulier. We hoeven zelf geen state bij te houden. Achterliggend wordt [React.ref](https://react.dev/learn/referencing-values-with-refs) gebruikt.
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

// 👇 1
const validationRules = {
  user: {
    required: 'User is required',
    min: { value: 1, message: 'min 1' },
  },
};

export default function TransactionForm({places}) {
  // ...

  return (
    <>
      <form onSubmit={handleSubmit(onSubmit)} className="w-50 mb-3">
        <div className="mb-3">
          <label htmlFor="user" className="form-label">User id</label>
          {/* 👇 1 */}
          <input
            {...register('user', validationRules.user)}
            defaultValue=''
            id="user"
            type="number"
            className="form-control"
            placeholder="user"
            required
          />
          {/* 👇 2 */}
          {errors.user && <p className="form-text text-danger">{errors.user.message}</p> }
        </div>
        {/** ... */}
      </form>
    </>
  );
```

1. Als tweede parameter van de `register` functie kan je de validatieregels meegeven (required, min, max, minLength, maxLength, pattern, validate). Je kan ook de bijhorende foutmelding opgeven. Hiervoor definiëren we een constante validationRules. Gebruik geen constante object literals/arrays in de component, bv. validatieregels. Plaats deze buiten de component. Anders wordt dit bij elke render opnieuw aangemaakt. Bovendien maakt het `input` field hiervan gebruik en wordt er geen nieuw object aangemaakt bij elke render.
   React Hook Form ondersteunt ook schema-validatie met Yup, Zod, Superstruct & Joi. De validatie is afgestemd op de HTML-standaard voor formuliervalidatie. Meer hierover in de documentatie van [register](https://react-hook-form.com/api/useform/register). Voor inputveld met als type `number` dien je `valueAsNumber` in te stellen zodat je een getal i.p.v. een string terugkrijgt.
2. Voor de weergave van de fouten maken we gebruik van het `errors` object. Aan de hand van het `type` property kan je het type van de fout opvragen (bv. `errors.user.type === 'required'`). Merk op dat we hier gebruik maken van `&&`, dit wordt wel eens gezien als een anti-pattern in React. Het is eigenlijk beter om de ternary operator (`voorwaarde ? true : false`) te gebruiken. Dit wordt dus: `{errors.user ? <p className="form-text text-danger">{errors.user.message}</p> : null}`.

Vermits er meerdere invoervelden op ons formulier voorkomen en we steeds dezelfde code moeten schrijven, zouden we een aparte component moeten maken. Deze component zal gebruik moeten maken van [useFormContext](https://react-hook-form.com/api/useformcontext). Dit komt op het einde van dit hoofdstuk aan bod.

## POST /api/transactions

De volgende stap van de CRUD operaties is de 'C', een nieuwe transactie aanmaken.

Pas `index.js` in de map `api` aan:

```jsx
// 👇 1
export const save = async (url, { arg: body }) => {
  await axios.post(`${baseUrl}/${url}`, body); // 👈 2
};
```

1. De parameter `url` zal van `swr` de `key` ontvangen. We krijgen ook de `transaction` mee als argument, we hernoemen de `arg` optie voor de duidelijkheid.
2. We voeren een `POST` request uit naar de API. Axios zal de `transaction` automatisch omzetten naar JSON en versturen als body van het HTTP request. Het antwoord heeft als HTTP status code 201 en als response body de nieuw gecreëerde transactie. We negeren dat antwoord hier.

De creatie van een transactie gebeurt in de `AddOrEditTransaction` component. De callback functie wordt doorgegeven aan de `TransactionForm`. De state (de transacties) wordt bijgehouden in de `TransactionList` component, maar die zal automatisch geüpdatet worden als we dezelfde key doorgeven aan `swr`. We maken een nieuwe mutation in `TransactionForm`:

```jsx
// src/pages/transactions/AddOrEditTransaction.jsx
// imports...
import useSWRMutation from 'swr/mutation'; // 👈 1
import { save } from '../../api'; // 👈 1

export default function AddOrEditTransaction() {
  const {
    data: transaction,
    error: transactionError,
    isLoading: transactionLoading,
  } = useSWR(id ? `transactions/${id}` : null, getById); // 👈 2

  // ...
  return (
    <>
      <h1>Add transaction</h1>
      {/* 👇 4 */}
      <AsyncData error={saveError || placesError} loading={placesLoading}>
        <TransactionForm
          places={places}
          transaction={transaction}
          onSave={saveTransaction}
        />
        {/* 👆 3 */}
      </AsyncData>
      {/* ... */}
    </>
  );
}
```

1. Importeer de `useSWRMutation` hook en de `save` functie.
2. Maak een trigger-functie die een transactie zal opslaan. We gebruiken dezelfde key als bij het ophalen van de transacties, dus `transactions`. We geven als fetcher onze `save` functie mee. We krijgen o.a. terug:
   - `trigger`: een functie die we kunnen aanroepen om het request effectief uit te voeren en dus de transactie toe te voegen. Deze functie ontvangt de `transaction` als argument. We hernoemen dit naar `saveTransaction`.
   - `error`: een eventuele fout die zich voordoet bij het opslaan van de transactie. We hernoemen deze naar `saveError`.
3. Geef de callback functie door aan `TransactionForm`component
4. We tonen een eventuele fout na het opslaan

```jsx
// src/components/transactions/TransactionForm.jsx
// ...

export default function TransactionForm({ places, onSave }) {
  // 👆 1
  // ...

  const onSubmit = async (data) => {
    const { user, place, amount, date } = data;
    // 👇 2
    await onSave({
      userId: user,
      placeId: place,
      amount: parseInt(amount),
      date: new Date(date),
    });
    reset();
  };
  // ...
}
```

1. Voeg `onSave` toe aan de props
2. Wijzig de `onSubmit` zodat `onSave` aangeroepen wordt. We geven de `userId`, `placeId`, `amount` en `date` mee als argumenten als **object** . **Let op:** de functie is `async`, dus we moeten `await` gebruiken.

Later verwijderen we het user-veld, daarom doen we hier geen moeite om de gebruiker op te zoeken. We geven gewoon het id mee.

### Oefening 3 - POST in je eigen project

Implementeer een willekeurige POST uit je eigen project (liefst dezelfde entiteit als hiervoor):

- Maak een functie die een POST request uitvoert aan in `api/index.js`.
- Gebruik de `useSWRMutation` hook om de data toe te voegen.
- Controleer of je formulier nog steeds een item kan toevoegen.

## PUT /api/transactions/:id

Dan rest nog de 'U' van CRUD, maar die is een beetje speciaal. De API call zelf is geen probleem, dit is basically hetzelfde als 'Create' maar met een extra `id` parameter.

Maar we willen natuurlijk niet dat een gebruiker een volledig object juist moet invoeren om het aan te passen. We willen dat hij ergens op 'bewerk' kan klikken bij een bestaand element.

In ons geval willen we dat de "potlood-knop" in de lijst deze data in het formulier ingeeft. De gebruiker past vervolgens aan en doet een update. Maar om dat te doen werken moeten er een paar dingen gebeuren.

We moeten op een manier een transactie kunnen doorgeven van onze `Transaction` naar onze `TransactionForm` component. Daarnaast moeten we in `TransactionsForm` het onderscheid kunnen maken tussen een bestaande transactie updaten, of een nieuwe toevoegen.

Wat de eigenlijk moeten doen is:

We geven via een parameter in de url mee welke transactie moet worden aangepast en dan halen we de transactie op. In het formulier bekijken we via een `useEffect` bij elke render of er een huidige transactie is ingesteld en indien nodig tonen we de bijhorende gegevens in alle formuliervelden.

In de API hebben we een functie nodig om de aan te passen transactie op te halen en up te daten.

```jsx
export const getById = async (url) => {
  const { data } = await axios.get(`${baseUrl}/${url}`);

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

Als we in de `Transaction` component klikken op de potlood-knop, navigeren we naar `/transactions/edit/${id}`. In `AddOrEditTransaction` kijken we of het om een add of edit gaat. In het laatste geval halen we de betreffende transactie op en geven dit door aan de `TransactionForm`.

```jsx
// src/pages/transactions/AddOrEditTransaction.jsx
import { useParams } from 'react-router-dom'; // 👈 1
import { getAll, getById } from '../../api'; // 👈 3
//andere imports

export default function AddOrEditTransaction() {
  const { id } = useParams(); // 👈 2

  const {
    data: transaction,
    error: transactionError,
    isLoading: transactionLoading,
  } = useSWR(id ? `transactions/${id}` : null, getById); // 👈 3

  //...

  return (
    <>
      <h1>Add transaction</h1>

      {/* 👇 5 */}
      <AsyncData
        error={transactionError || placesError || saveError}
        loading={transactionLoading || placesLoading}
      >
        {/* 👇 4 */}
        <TransactionForm
          places={places}
          transaction={transaction}
          onSave={saveTransaction}
        />
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
import { useNavigate } from 'react-router-dom';// 👈 4
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

export default function TransactionForm({places, transaction, onSave}) {  // 👈 1
  const navigate = useNavigate(); // 👈 4

  // 👇 2
  const getDefaultValues = () => {
    if (
      // check on non-empty object
      transaction &&
      (Object.keys(transaction).length !== 0 ||
        transaction.constructor !== Object)
    ) {
      return ({
        user: transaction.user.id,
        place: transaction.place.id,
        amount: transaction.amount,
        date: toDateInputString(new Date(transaction.date)),
      });
    } else {
      return ({ user: '', place: '', amount: '', date: toDateInputString(new Date()) });
    }
  };

  const { register, handleSubmit, formState: { errors } } = useForm(); // 👈 2

  // 👇 4
  const onSubmit = async (data) => {
    const { user, place, amount, date } = data;
    await onSave({
      userId: user,
      placeId: place,
      amount: parseInt(amount),
      date: new Date(date),
      id: transaction?.id,
    });
    navigate('/transactions'); 
  };

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
2. De `defaultValues` prop van `useForm` is een object dat wordt gebruikt om de initiële waarden in te stellen voor de formuliervelden. We zoepen hiervoor de functie getDefaultValues aan
   - Indien `transaction` is ingevuld, plaats de waarden in het formulier. We dienen de datum te formatteren. Hiervoor voorzien we de functie `toDateInputString`. Definieer geen pure functies in de component (functies zonder afhankelijkheden van variabelen). Plaats deze buiten de component.
   - Indien `transaction` null is, voorzie defaultwaarden.
3. Pas de tekst op de knop aan i.f.v. of het om een update of een create gaat.
4. Bij het opslaan van de transactie geven we ook de id mee. En we navigeren terug naar de `TransactionList` pagina. We hoeven het formulier niet meer te resetten.

## Verbeteren van de performantie

In een React applicatie worden componenten heel vaak gerenderd. De performantie kan je verbeteren door het voorkomen van onnodige renders en het verminderen van de tijd die een render in beslag neemt.

Een oplossing voor dit probleem is **memoization**.

React biedt een paar vormen van memoization:

- `memo`: creatie van pure componenten (let op: dit is **geen** hook)
- `useMemo`: retourneert een memoized **waarde**
- `useCallback`: retourneert een memoized **functie**

We hebben reeds `useMemo` gebruikt. Nu komen de andere vormen aan bod.

### Hooks

**Componenten** zijn functies die de UI renderen. Het renderen gebeurt wanneer de app voor het eerst geladen wordt en wanneer state waarden wijzigen. Renderen van code moet "[puur](https://react.dev/learn/keeping-components-pure)" zijn. Componenten mogen enkel 'hun' JSX retourneren en mogen objecten of variabelen die bestaan voor de rendering niet wijzigen (bv. geen nieuwe waarde toekennen aan props). Gegeven dezelfde input, dient het dezelfde output te retourneren. Net als een wiskundige formule zou het alleen het resultaat moeten berekenen, maar niets anders doen.

Een veel gemaakte denkfout is dat alle waarden (i.e. variabelen) in een component bewaard blijven, dat is **niet waar**. Een component is een functie, en functies hebben lokale variabelen. Eens de component zijn JSX geretourneerd heeft, zijn alle lokale variabelen weg. Je hebt nood aan de magie van React Hooks om waarden te bewaren tussen renders.

**Events** zijn functies binnen de component die worden uitgevoerd als reactie op een actie van een gebruiker. Een event handler kan state aanpassen, bv. een HTTP POST request uitvoeren om een transactie toe te voegen. Event handlers bevatten **side-effects** veroorzaakt door een interactie. React biedt daarnaast ook de mogelijkheid voor side-effects na bv. een state-wijziging. Hierover in een volgend hoofdstuk meer.

In het vorige hoofdstuk hebben we kennis gemaakt met de `useState` en `useReducer` en `useMemo` hooks. React heeft nog heel wat meer hooks (zie <https://react.dev/reference/react>) en er zijn reeds heel wat nuttige custom hooks te vinden op internet (zie bv. <https://nikgraf.github.io/react-hooks/>).

Hooks hebben ervoor gezorgd dat je met function components hetzelfde kan bereiken als met class components. Toch kan het zijn alsof hooks vreemd aanvoelen, alsof React de bal mis geslagen heeft in vergelijking met andere frameworks als [Solid.js](https://www.solidjs.com/) (zie <https://jakelazaroff.com/words/were-react-hooks-a-mistake/>).

### Regels voor hooks

Hooks zijn niet meer dan JavaScript functies. Echter moet je twee regels volgen wanneer je er gebruik van maakt. Je kan hiervoor een linter plugin installeren, dit gebeurt automatisch bij het gebruik van `vite`.

- Gebruik hooks enkel op het top niveau. Gebruik hooks niet binnen een if, andere condities, loops of geneste functies.
  - Reden: React valt terug op de volgorde waarin hooks worden aangeroepen om een waarde terug te geven. React houdt dit bij in een array. De volgorde moet dezelfde zijn bij elke render. Benieuwd naar meer info? Lees verder in [The First Rule of React Hooks, In Plain English](https://itnext.io/the-first-rule-of-react-hooks-in-plain-english-1e0d5ae32009)
- Roep hooks enkel aan vanuit React functies. Dit wil zeggen: enkel vanuit function components of vanuit eigen geschreven hooks.

Hooks maken gebruik van closures, let dus op voor stale closures! [Zie hier voor enkele voorbeelden](https://dmitripavlutin.com/react-hooks-stale-closures/)

### React.memo en pure functions

Voeg een `console.log` instructie toe voor elke `return` in onderstaande componenten:

```jsx
// src/pages/transactions/TransactionList.jsx
export default function TransactionList() {
  // ...
  console.log('Rendering transactionlist...');
  return (...);
}

// src/components/transactions/Transaction.jsx
export default function Transaction(props) {
  // ...
  console.log('Rendering transaction...');
  return (...);
}
```

Telkens als we een letter ingeven in het zoekveld worden alle componenten gererenderd, hoewel er niets wijzigt aan de output van de component. De `Transaction` component heeft als prop een transaction en deze blijft ongewijzigd als de gebruiker een letter ingeeft in het zoekveld. Toch wordt de component gererenderd.

Een **pure component** is een component die gegeven dezelfde props dezelfde output genereert. `Transaction` is een pure component. Gegeven dezelfde props, wordt dezelfde output gegenereerd. We willen een pure component niet opnieuw renderen als de properties niet gewijzigd zijn. De `memo` functie wordt gebruikt om een component te creëren die enkel zal rerenderen als de props wijzigen.

```jsx
// src/components/transactions/Transaction.jsx
import { memo } from 'react'; // 👈

const TransactionMemoized = memo(function Transaction({
  id,
  user,
  date,
  amount,
  place,
  onDelete,
}) {
  // 👈
  //...
});
export default TransactionMemoized;
```

Start de app en bekijk de console. Verwijder nadien de toegevoegd `console.log` statements.

### useCallback hook

Pas de `TransactionList` aan en voeg onderstaande functie toe:

```js
const handleDelete = async (id) => {
  await deleteTransaction(id);
  alert('Transaction is removed');
};
```

Geef deze door als property `onDelete` aan de `TransactionTable` component:

```jsx
<TransactionsTable
  transactions={filteredTransactions}
  onDelete={handleDelete}
/>
```

Van zodra we een letter ingeven in de search bar worden alle transacties toch opnieuw gerenderd.

`TransactionTable` bevat een prop `onDelete`. Deze wordt doorgegeven door de parent component `TransactionList`. `handleDelete` is de event handler functie. Javascript gaat er vanuit dat de functie `handleDelete` bij elke render verschillend is. Echter is dit niet het geval. `useCallback` cachet een functie tussen twee renders en dit totdat de dependency array wijzigt.

Pas de code van de functie in de `TransactionList` component aan:

```jsx
// src/components/transactions/TransactionList.jsx
import { useState, useMemo, useCallback } from 'react'; // 👈

// ...

// 👇
const handleDelete = useCallback(
  async (id) => {
    await deleteTransaction(id);
    alert('Transaction is removed');
  },
  [deleteTransaction],
);
```

Start de app en bekijk de console. De functie wordt nu gecachet. Merk op dat swr dit ook doet.

### Oefening 3 - Memoization

Ook `Place` is een pure componenten en dient enkel gererenderd te worden als zijn state wordt aangepast.

Voeg `useCallback`toe waar nodig.

### Oefening 4 - Memoization in je eigen project

Pas memoization toe in je eigen project. Let wel op het volgende:

> Premature optimization is the root of all evil - Donald Knuth

Het is dus niet de bedoeling om elke component te wrappen in `memo`. Gebruik de React DevTools om te achterhalen welke component (te) vaak renderen en pas daar memoization toe.

<!-- TODO: eindpunt toevoegen -->

## Mogelijke extra's voor de examenopdracht

- [Formik](https://www.npmjs.com/package/formik)
- Validatie in [react-hook-form](https://www.npmjs.com/package/react-hook-form) met [Joi](https://www.npmjs.com/package/joi), [Yup](https://npmjs.com/package/yup) of dergelijke
- Maak gebruik van **meerdere** custom hooks uit:
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
