# Context API

<!-- TODO: startpunt toevoegen -->

We creëren geneste componenten om de UI te bouwen. De state plaatsen we in de root component en wordt via props doorgegeven aan de kinderen. Dit kan echter heel complex worden als je sommige props tot diep in de boom dient door te geven of als heel wat componenten dezelfde props nodig hebben.

De **Context API** laat toe om data globaal bij te houden en door te geven aan child components, zonder dat we via props de data tot in deze kinderen moeten doorgeven. Dus Context API is een alternatief voor het doorgeven van props.

**Let op**, de Context API wordt vaak ten onrechte gebruikt als oplossing. Gebruik het enkel en alleen als de state echt globaal moet zijn. Gebruik de context ook zo laag mogelijk in de component tree. Alle componenten onder de context worden nl. opnieuw gerenderd als de waarde in de context wijzigt!

Een aantal use cases voor context zijn o.a. theming, taalkeuze, huidige gebruiker...

De Context API bestaat uit drie bouwstenen:

1. Een **Context Object**, aangemaakt door de factory functie `createContext`
2. Een **Context Provider**: voorziet de onderliggende componenten van data
3. Meerdere **Context Consumers** : ontvangen de data van de context (= onze eigen componenten)

## Licht of donker thema

Om de werking van de React Context API te demonstreren kan de gebruiker van onze budgetapplicatie een licht of donker thema kiezen. We gaan hiervoor een context gebruiken aangezien het thema van de website door alle componenten gekend moet zijn om bv. de kleuren aan te passen.

Het aanmaken van een context gebeurt typisch in 3 stappen:

1. Creëer een context.
2. Bied de context aan via een provider.
3. Neem data uit de context via een consumer (of dus in een child component).

### Stap 1: Creëer de context

In de navigatiebalk voorzien we een knop om het thema te kiezen. We maken in de `main` component, de root component, een context aan m.b.v. `createContext`. Deze factory-functie heeft één optioneel argument, de standaardwaarde. Exporteer `ThemeContext` zodat de consumers dit kunnen gebruiken.

```jsx
// src/main.jsx
import { Outlet, ScrollRestoration } from 'react-router-dom';
import Navbar from './Navbar';
import { createContext } from 'react'; // 👈

export const ThemeContext = createContext(); // 👈

export default function Layout() {
  return (
    <div className='container-xl'>
      <Navbar />
      <Outlet />
      <ScrollRestoration />
    </div>
  );
}
```

### Stap 2: Bied de context aan

Voeg toe in `Layout.jsx`:

```jsx
// src/components/Layout.jsx
import { Outlet, ScrollRestoration } from 'react-router-dom';
import Navbar from './Navbar';
import { createContext } from 'react';

export const ThemeContext = createContext();

export default function Layout() {
  return (
    <ThemeContext.Provider value={{ theme: 'dark' }}> {/* 👈 */}
      <div className='container-xl'>
        <Navbar />
        <Outlet />
        <ScrollRestoration />
      </div>
    </ThemeContext.Provider> {/* 👈 */}
  );
}
```

Elk **context object** wordt beschikbaar gemaakt met een **context provider** component waarin de data geplaatst wordt. Een context provider plaats je rond de volledige component tree of bepaalde secties ervan. Alle kinderen (de **context consumers**) binnen de context provider hebben toegang tot de data en kunnen zich abonneren op wijzigingen. De `value` property bevat de data die in de context geplaatst wordt. Geef een object door (vandaar `{{}}`). Alle kinderen die afstammen van de provider zullen opnieuw renderen wanneer de waarde van de `Provider` verandert.

> Het is niet verplicht om een context steeds in de App component te zetten, je zet hem zo laag mogelijk zodat de nodige child componenten aan de data kunnen.

### Stap 3: Consume de context

De data hoeft niet langer doorgegeven te worden via props. Gebruik bv. het thema in de `TransactionTable` component. De `TransactionsTable` component zal de data consumeren, en is een context consumer.

```jsx
// src/components/transactions/TransactionList.jsx
import { useContext } from 'react'; // 👈 1
import { ThemeContext } from '../../App'; // 👈 1
// ...

function TransactionTable({ transactions }) {
  const { theme } = useContext(ThemeContext); // 👈 1

  // ...

  return (
    <div>
      {/* 👇 2 */}
      <table className={`table table-hover table-responsive table-${theme}`}>
    {/* ... */}
  );
}
```

1. De `useContext` hook wordt gebruikt om met de context te connecteren en heeft als parameter een context object `ThemeContext`. Het retourneert de `value` van de huidige context (zie value property van de context provider).
2. In onze context zit momenteel enkel een variable `theme`. Die voegen we toe aan het attribuut className van de `table` tag. Bootstrap heeft twee klassen voor een lichte en donkere table: `table-light` en `table-dark`.

De context provider kan data in de context plaatsen, maar het kan de data in de context niet aanpassen. Willen we ook nog functies toevoegen aan de Context om te wisselen van dark naar light mode of omgekeerd? Dan moeten we een aparte component aan te maken.

<!-- TODO: hier verder nalezen -->

## ThemeContext en Provider

Maak een map `contexts` aan in de map `src` met daarbinnen het bestand `Theme.context.jsx`:

```jsx
// src/contexts/Theme.context.jsx
import { createContext } from 'react'; // 👈 1

export const ThemeContext = createContext(); // 👈 1

export const ThemeProvider = ({
  // 👈 2
  children,
}) => {
  return (
    // 👈 2
    <ThemeContext.Provider>{children}</ThemeContext.Provider>
  );
};
```

1. Importeer `createContext` en maak de context aan
2. Maak een stateful component genaamd `ThemeProvider` die de `children` als prop ontvangt. `children` bevat de component tree waarrond de Provider geplaatst wordt. `ThemeProvider` beheert de data en stelt het ter beschikking van deze children.
3. De component `ThemeProvider` rendert de context provider die de consumers als children zal hebben.

De `ThemeProvider` beheert de state en functies, en stelt deze ter beschikking aan de children. Hou hier als state het thema bij en een functie om te wisselen van thema.

```jsx
// src/contexts/Theme.context.jsx
import { createContext, useState, useCallback, useMemo } from 'react';

export const themes = {
  // 👈 1
  dark: 'dark',
  light: 'light',
};

export const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState(
    sessionStorage.getItem('themeMode') || themes.dark,
  ); // 👈 2

  const toggleTheme = useCallback(() => {
    // 👈 3
    const newThemeValue = theme === themes.dark ? themes.light : themes.dark;
    setTheme(newThemeValue);
    sessionStorage.setItem('themeMode', newThemeValue);
  }, [theme]);

  const value = useMemo(() => ({ theme, toggleTheme }), [theme, toggleTheme]); // 👈 4

  return (
    <ThemeContext.Provider value={value}>
      {/* 👈 5 */}
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
// src/contexts/
import { createContext, useState, useCallback, useMemo } from 'react';

export const themes = {
  dark: 'dark',
  light: 'light',
};

export const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState(
    sessionStorage.getItem('themeMode') || themes.dark,
  );

  const toggleTheme = useCallback(() => {
    const newThemeValue = theme === themes.dark ? themes.light : themes.dark;
    setTheme(newThemeValue);
    sessionStorage.setItem('themeMode', newThemeValue);
  }, [theme]);

  const oppositeTheme = useMemo(
    () => (theme === themes.dark ? themes.light : themes.dark),
    [theme],
  ); // 👈 1

  const value = useMemo(
    () => ({ theme, oppositeTheme, toggleTheme }),
    [theme, oppositeTheme, toggleTheme],
  ); // 👈 2

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
};
```

1. Voeg de berekende waarde`oppositeTheme` toe.
2. Maak `oppositeTheme` beschikbaar voor de children. Merk op: je kan ook meteen `oppositeTheme` berekenen in de `useMemo` die `value` bepaald.

### Providing ThemeContext

Stel in `main.jsx` de `ThemeProvider` ter beschikking aan alle children.

```jsx
// src/main.jsx
// ... imports
import { ThemeProvider } from './contexts/Theme.context'; // 👈

//...

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <ThemeProvider>
      {' '}
      {/* 👈 */}
      <RouterProvider router={router} />
    </ThemeProvider>{' '}
    {/* 👈 */}
  </StrictMode>,
);
```

### Consuming ThemeContext

Voeg in `Navbar.jsx` een knop toe om van thema te wisselen:

```jsx
// src/components/Navbar.jsx
import { NavLink } from 'react-router-dom';
import { useContext } from 'react'; //👈 1
import { ThemeContext } from '../contexts/Theme.context'; // 👈 1
import { IoMoonSharp, IoSunny } from 'react-icons/io5'; // 👈 4

export default function Navbar() {
  const { theme, toggleTheme } = useContext(ThemeContext); // 👈 2

  return (
    <nav className={`navbar sticky-top bg-${theme} text-bg-${theme} mb-4`}>
      {' '}
      {/* 👈 3 */}
      <div className='container-fluid flex-column flex-sm-row align-items-start align-items-sm-center'>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <NavLink className='nav-link' to='/'>
            Transactions
          </NavLink>
        </div>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <NavLink className='nav-link' to='/places'>
            Places
          </NavLink>
        </div>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <NavLink className='nav-link' to='/about'>
            Over ons
          </NavLink>
        </div>
        <div className='flex-grow-1'></div>
        <button
          className='btn btn-secondary'
          type='button'
          onClick={toggleTheme}
        >
          {theme === 'dark' ? <IoMoonSharp /> : <IoSunny />}
        </button> {/* 👈 4 */}
      </div>
    </nav>
  );
}
```

1. Importeer de nodige componenten.
2. Roep de `useContext` hook aan en gebruik destructuring om de properties, die de component nodig heeft, eruit te halen.
3. Voeg de bootstrap klassen toe voor de achtergrondkleur en de kleur van de tekst.
4. Voorzie de knop om het thema te kiezen.

De `TransactionTable` component blijft behouden. De `ThemeContext` komt nu wel niet uit `main.jsx` maar `Theme.context.jsx`:

```jsx
import { ThemeContext } from '../../contexts/Theme.context';
```

Verwijder de context uit `Layout.jsx`

### Oefening 1 - ThemeContext

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
  useContext,
} from 'react'; // 👈 1

export const themes = {
  dark: 'dark',
  light: 'light',
};

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

Zo kan de code in `Navbar.jsx` als volgt aangepast worden:

```jsx
// src/components/Navbar.jsx
import { NavLink } from 'react-router-dom';
//import { useContext } from 'react'; // 👈 1
//import { ThemeContext } from '../contexts/Theme.context'; // 👈 1
import { useTheme } from '../contexts/Theme.context'; // 👈 1
import { IoMoonSharp, IoSunny } from 'react-icons/io5';

export default function Navbar() {
  const { theme, toggleTheme } = useTheme(); // 👈 2

  return (

    <nav className={`navbar sticky-top bg-${theme} text-bg-${theme} mb-4`}> {/* 👈 2 */}
  //...
```

1. Verwijder de import `useContext`, `ThemeContext`, en importeer `useTheme`.
2. Destructure de waarden die in deze component gebruikt worden.

### Oefening 2 - custom hooks

Pas de andere componenten aan.

## Anti-patterns

Er zitten een paar anti-patterns in ons formulier. Maak hier ook gebruik van in je eigen project!

1. Gebruik geen constante object literals/arrays in de component, bv. validatieregels. Plaats deze buiten de component.
2. Definieer geen pure functies in de component (functies zonder afhankelijkheden van variabelen), bv. `toDateInputString`. Plaats deze buiten de component.
3. Definieer geen componenten inline in een andere component, bv. `LabelInput`(zie verder) Plaats deze buiten de component.
4. Gebruik een id als waarde voor de `key` prop in lijsten, gebruik geen index.

### Duplicate code

De combinatie `label` en `input` tag komen vaak voor. Kunnen we hier aparte component van maken?

Componenten mag je niet definiëren binnen een andere component. Maak een functiecomponent `LabelInput` in het bestand van het formulier (of als aparte component i.g.v. hergebruik). Plaats de code van het invoerveld van de gebruiker hierin:

```jsx
// src/components/transactions/TransactionForm.jsx
function LabelInput() {
  return (
    <div className='mb-3'>
      <label htmlFor='user' className='form-label'>
        Who
      </label>
      <input
        {...register('user', validationRules.user)}
        defaultValue=''
        id='user'
        type='text'
        className='form-control'
        placeholder='user'
        required
      />
      {errors.user && (
        <p className='form-text text-danger'>{errors.user.message}</p>
      )}
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
    <div className='mb-3'>
      <label htmlFor={name} className='form-label'>
        {label}
      </label>
      <input
        {...register(name, validationRules)}
        id={name}
        type={type}
        className='form-control'
        {...rest}
      />
      {hasError ? (
        <div className='form-text text-danger'>{errors[name].message}</div>
      ) : null}
    </div>
  );
}
```

We krijgen nog fouten: `register and errors not defined`. Oplossing: `useFormContext` en `FormProvider`. Definitie van `useFormContext` uit de documentatie:

> This custom hook allows you to access the FormContext. useFormContext is intended to be used in deeply nested structures, where it would become inconvenient to pass the context as a prop.

Met `FormProvider` creëren we een nieuwe context provider voor een formulier. Met `useFormContext` halen we hieruit de waarde op en gebruiken dit in een (diep) geneste component tree.

In de documentatie lezen we ook het volgende over de `FormProvider`:

> React Hook Form's FormProvider is built upon React's Context API. It solves the problem where data is passed through the component tree without having to pass props down manually at every level. This also causes the component tree to trigger a rerender when React Hook Form triggers a state update.

```jsx
// src/components/transactions/TransactionForm.jsx
import { FormProvider, useForm, useFormContext } from 'react-hook-form'; // 👈 1 en 2
// ...

function LabelInput({ label, name, type, validationRules, ...rest }) {
  const { register, errors } = useFormContext(); // 👈 2

  const hasError = name in errors;

  return (
    <div className='mb-3'>
      <label htmlFor={name} className='form-label'>
        {label}
      </label>
      <input
        {...register(name, validationRules)}
        id={name}
        type={type}
        className='form-control'
        {...rest}
      />
      {/* 👈 3 */}
      {hasError ? (
        <div className='form-text text-danger'>{errors[name].message}</div>
      ) : null}
    </div>
  );
}

export default function TransactionForm({ places, transaction, onSave }) {
  // ...

  return (
    <FormProvider
      handleSubmit={handleSubmit}
      errors={errors}
      register={register}
    >
      {/* 👈 1 */}
      <form onSubmit={handleSubmit(onSubmit)} className='m-5'>
        <LabelInput
          label='User'
          name='user'
          type='user'
          validationRules={validationRules.user}
        />
        {/* 👈 3 */}
        {/* Herhaal dit voor de overige input fields */}
      </form>
    </FormProvider>
  );
}
```

1. Importeer de `FormProvider` en plaats de `FormProvider` rond het formulier om de `useFormContext` correct te laten werken.
2. Importeer `useFormContext` en maak gebruik van `useFormContext` voor het gebruik van `register` en `errors`.
3. Maak gebruik van de component `LabelInput`.

Je kan er ook voor zorgen dat de inputvelden en knoppen in het formulier _disabled_ worden als het formulier gesubmit wordt.
`useForm` geeft een boolean [isSubmitting](https://react-hook-form.com/api/useform/formstate) terug die `true` is als het formulier gesubmit wordt en `false` bij een reset.

```jsx
//..
function LabelInput({ label, name, type, ...rest }) {
  const {
    register,
    errors,
    isSubmitting, // 👈 4
  } = useFormContext();

  const hasError = name in errors;

  return (
    <div className='mb-3'>
      <label htmlFor={name} className='form-label'>
        {label}
      </label>
      <input
        {...register(name, validationRules[name])}
        id={name}
        type={type}
        disabled={isSubmitting} // 👈 4
        className='form-control'
        {...rest}
      />
      {hasError ? (
        <div className='form-text text-danger'>{errors[name].message}</div>
      ) : null}
    </div>
  );
}
//..

export default function TransactionForm({ places, transaction, onSave }) {
  const {
    register,
    handleSubmit,
    reset,
    setValue,
    formState: { errors, isSubmitting }, // 👈 1
    isSubmitting,
  } = useForm();
  //...
  const onSubmit = useCallback(
    async (data) => {
      const { user, place, amount, date } = data;
      try {
        // 👈 5
        await onSave({
          userId: user,
          placeId: place,
          amount: parseInt(amount),
          date: new Date(date),
          id: transaction?.id,
        });
        navigate('/transactions/');
      } catch (error) {
        console.log(error);
      }
    },
    [onSave],
  );
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
        {currentTransaction?.id ? 'Save transaction' : 'Add transaction'}
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

### Oefening 3 je eigen project

Controleer je project op anti-patterns, duplicate code en refactor.
Denk na over global state in je project. Indien van toepassing, maak hiervoor een Context aan.
