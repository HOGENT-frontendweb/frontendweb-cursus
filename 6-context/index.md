# Context API

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les6 6ad1816
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.
>
> Je hebt ook de bijbehorende backend nodig:
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/webservices-budget.git
> cd webservices-budget
> git checkout -b les6 3386a40
> pnpm install
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#webservices-budget) voor meer informatie.

We creëren geneste componenten om de UI te bouwen. De state plaatsen we in de root component en wordt via props doorgegeven aan de kinderen. Dit kan echter heel complex worden als je sommige props tot diep in de boom dient door te geven of als heel wat componenten dezelfde props nodig hebben.

De **Context API** laat toe om data globaal bij te houden en door te geven aan child components, zonder dat we via props de data tot in deze kinderen moeten doorgeven. Dus Context API is een alternatief voor het doorgeven van props.

**Let op**: de Context API wordt vaak ten onrechte gebruikt als oplossing. Gebruik het enkel en alleen als de state echt globaal moet zijn. Gebruik de context ook zo laag mogelijk in de component tree. Alle componenten onder de context worden nl. opnieuw gerenderd als de waarde in de context wijzigt! Een aantal use cases voor context zijn o.a. theming, taalkeuze, huidige gebruiker...

De Context API bestaat uit drie bouwstenen:

1. Een **Context Object**, aangemaakt door de factory functie `createContext`
2. Een **Context Provider**: voorziet de onderliggende componenten van data
3. Meerdere **Context Consumers** : ontvangen de data van de context (= onze eigen componenten)

## Licht of donker thema

Om de werking van de React Context API te demonstreren kan de gebruiker van onze budgetapplicatie een licht of donker thema kiezen. We gaan hiervoor een context gebruiken aangezien het thema van de website door heel veel componenten gekend moet zijn om bv. de kleuren, beelden,... aan te passen.

Het aanmaken van een context gebeurt typisch in 3 stappen:

1. Creëer een context.
2. Bied de context aan via een provider.
3. Neem data uit de context via een consumer (of dus in een child component).

In tailwindcss is er ondersteuning voor dark mode op 2 manieren `media` en `class`:

- de `media` strategie: dark mode wordt automatisch geactiveerd op basis van de voorkeuren van het besturingssysteem van de gebruiker.
- de `class` strategie: dark mode wordt geactiveerd door de class `dark` toe te voegen aan een parent element (meestal het `<html>` of `<body>` element). Alle child elementen erven dan automatisch de dark mode stijlen.

We gebruiken hier de `class` strategie omdat we de dark mode willen aan- of uitzetten via een knop in de UI.

Om de darkmode te activeren in tailwindcss volg je de stappen in de [tailwindcss documentatie](https://tailwindcss.com/docs/dark-mode).

```css
/* src/index.css */
@custom-variant dark (&:where(.dark, .dark *));
```

Eens geactiveerd kan je in je CSS klassen zoals `dark:bg-gray-800` gebruiken.

Je krijgt de ESLint/CSS-fout `unknown at rule @custom-variant(css - unknownAtRules)` omdat @custom-variant een Tailwind CSS v4-specifieke at-rule is die niet herkend wordt door de standaard CSS-validator in VS Code. Pas hiervoor de instellingen van je workspace aan in VS Code:

```json
{
 "files.associations": {
  "*.css": "tailwindcss"
}
}
```

Die configuratie vertelt VS Code om .css bestanden te behandelen als Tailwind CSS bestanden in plaats van gewone CSS. Zo wordt de standaard css validator uitgeschakeld voor deze bestanden en krijg je geen fouten meer voor Tailwind-specifieke syntax zoals @custom-variant. Meer op <https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss>, recommended VS Code settings. Merk op: je moet `CSS IntelliSense extensie` geïnstalleerd hebben.

### Stap 1: Creëer de context

In de navigatiebalk voorzien we een knop om het thema te kiezen. We maken in de `Layout` component, een context aan m.b.v. `createContext`. Deze factory-functie heeft één optioneel argument, de standaardwaarde. Exporteer `ThemeContext` zodat de consumers dit kunnen gebruiken.

```jsx
// src/pages/Layout.jsx
import { Outlet, ScrollRestoration } from 'react-router';
import Navbar from '../components/Navbar';
import { createContext } from 'react'; // 👈

export const ThemeContext = createContext(); // 👈

export default function Layout() {
  return (
    <div className='container-xl'>
      <Navbar />
      <div className='p-4'>
        <Outlet />
      </div>
      <ScrollRestoration />
    </div>
  );
}
```

### Stap 2: Bied de context aan

Voeg toe in `Layout.jsx`:

```jsx
// src/components/Layout.jsx
import { Outlet, ScrollRestoration } from 'react-router';
import Navbar from '../components/Navbar';
import { createContext } from 'react';

export const ThemeContext = createContext();

export default function Layout() {
  return (
    <ThemeContext.Provider value={{ darkmode: true }}>
      {' '}
      {/* 👈 */}
      <div className='container-xl'>
        <Navbar />
        <div className='p-4'>
          <Outlet />
        </div>
        <ScrollRestoration />
      </div>
    </ThemeContext.Provider>
  );
}
```

Elk **context object** wordt beschikbaar gemaakt met een **context provider** component waarin de data geplaatst wordt. Een context provider plaats je rond de volledige component tree of bepaalde secties ervan. Alle kinderen (de **context consumers**) binnen de context provider hebben toegang tot de data en kunnen zich abonneren op wijzigingen. De `value` property bevat de data die in de context geplaatst wordt. Geef een object door (vandaar `{{}}`). Alle kinderen onder de provider zullen opnieuw renderen wanneer de waarde van de `Provider` verandert.

> Het is niet verplicht om een context steeds in de hoogste component uit de component tree te zetten, je zet hem zo laag mogelijk zodat de nodige child componenten aan de data kunnen.

### Stap 3: Consume de context

De data hoeft niet langer doorgegeven te worden via props. Gebruik bv. de darkmode in de `TransactionTable` component. De `TransactionsTable` component zal de data consumeren, en is een context consumer.

```jsx
// src/components/transactions/TransactionTable.jsx
import { useContext } from 'react'; // 👈 1
import { ThemeContext } from '../../pages/Layout'; // 👈 1
// ...

function TransactionTable({ transactions }) {
  const { darkmode } = useContext(ThemeContext); // 👈 1

  // ...

  return (
    <div>
      {/* 👇 2 */}
      <table className={`${darkmode?'dark':''} dark:bg-black dark:text-white table-auto w-full border-collapse mb-4`}>
    {/* ... */}`
  );
}
```

1. De `useContext` hook wordt gebruikt om met de context te connecteren en heeft als parameter een context object `ThemeContext`. Het retourneert de `value` van de huidige context (zie value property van de context provider).
2. In onze context zit momenteel enkel een variable `darkmode`. Die voegen we toe aan het attribuut className van de `table` tag. Als de class `dark` aanwezig is, zal tailwindcss de dark mode toepassen op alle kindelementen.

De context provider kan data in de context plaatsen, maar het kan de data in de context niet aanpassen. Willen we ook nog functies toevoegen aan de Context om te wisselen van dark naar light mode of omgekeerd? Dan moeten we een aparte component aanmaken.

## ThemeContext en Provider

Maak een map `contexts` aan in de map `src` met daarbinnen het bestand `Theme.context.jsx`:

```jsx
// src/contexts/Theme.context.jsx
import { createContext } from 'react'; // 👈 1

export const ThemeContext = createContext(); // 👈 1

// 👇 2
export const ThemeProvider = ({ children }) => {
  // 👇 3
  return <ThemeContext.Provider>{children}</ThemeContext.Provider>;
};
```

1. Importeer `createContext` en maak de context aan
2. Maak een stateful component genaamd `ThemeProvider` die de `children` als prop ontvangt. `children` bevat de component tree waarrond de `ThemeProvider` geplaatst wordt. `ThemeProvider` beheert de data en stelt het ter beschikking van deze children.
3. De component `ThemeProvider` rendert de context provider die de consumers als children zal hebben.

De `ThemeProvider` beheert de state en functies, en stelt deze ter beschikking aan de children. Hou hier als state het thema bij en een functie om te wisselen van thema.

```jsx
// src/contexts/Theme.context.jsx
import { useState, useEffect, useMemo, useCallback } from 'react';
import { createContext } from 'react';

export const ThemeContext = createContext();

const ThemeProvider = ({ children }) => {
  // 👇 1
  const [darkmode, setDarkmode] = useState(Boolean(localStorage.getItem('darkmode')));

  // 👇 2
  useEffect(() => {
    if (darkmode) {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
    localStorage.setItem('darkmode', darkmode);
  }, [darkmode]);

  // 👇 3
  const toggleDarkmode = useCallback(() => setDarkmode((prev) => !prev), []);

  const value = useMemo(() => ({ darkmode, toggleDarkmode }), [darkmode, toggleDarkmode]);// 👈 4

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );// 👈 5
};

export default ThemeProvider;
```

1. De `ThemeProvider` houdt de state bij, hier `darkmode`. We halen de waarde uit `sessionStorage` op. getItem returnt altijd een string of undefined (als niet bestaat). Dit converteren we naar een Boolean.
2. Synchroniseert de dark mode state met de DOM en sessionStorage. De DOM update voegt/verwijdert de dark CSS class op het html-element (`document.documentElement`). sessionStorage slaat de huidige state op zodat het bewaard blijft na page refresh.
3. De `ThemeProvider` voorziet ook in een functie om de darkmode aan te passen.
4. De `ThemeProvider` bepaalt wat er gedeeld wordt met de children. Maak hiervoor de constante `value` aan. Vermits het hier om een waarde (en geen functie) gaat, gebruiken we `useMemo`.
5. De `ThemeProvider` stelt de `value` ter beschikking van de children.

!> Het gebruik van `document.documentElement` is een anti-pattern in React omdat het direct de DOM manipuleert buiten de React lifecycle om. Echter, in dit specifieke geval is het noodzakelijk om de dark mode functionaliteit van Tailwind CSS te ondersteunen, aangezien Tailwind CSS afhankelijk is van de aanwezigheid van de 'dark' class op een hoog niveau in de DOM om de juiste stijlen toe te passen.

We krijgen een eslint fout `Fast refresh only works when a file only exports components`. Fast refresh is een feature van React die ervoor zorgt dat de applicatie snel herladen wordt bij wijzigingen in de code. Dit werkt enkel als een bestand enkel componenten exporteert. Maak daarom een nieuw bestand `index.js` aan in de `contexts` folder en plaats alle exports die geen component zijn in dit bestand. Pas eventueel de verwijzingen aan.

```js
// src/contexts/index.js
import { createContext } from 'react';

export const ThemeContext = createContext();
```

En importeer `ThemeContext` in `Theme.context.jsx`:

```jsx
// src/contexts/Theme.context.jsx
import {ThemeContext} from './';
```

### Providing ThemeContext

Stel in `main.jsx` de `ThemeProvider` ter beschikking aan alle children.

```jsx
// src/main.jsx
// ... imports
import ThemeProvider from './contexts/Theme.context'; // 👈

// ...

createRoot(document.getElementById('root')).render(
  <StrictMode>
    {/* 👇 */}
    <ThemeProvider>
      <RouterProvider router={router} />
    </ThemeProvider>
    {/* 👆 */}
  </StrictMode>,
);
```

### Instellen van dark mode in Tailwind

Pas `index.css` aan zodat er met de juiste kleuren wordt gewerkt.

```css
@layer base {
  html, body {
  @apply bg-white dark:bg-gray-900 text-gray-900 dark:text-white;
  min-height: 100vh;
  }
  h1 {
    @apply text-4xl mb-4;
  }
}
```

html en body krijgen de juiste achtergrondkleur en tekstkleur in light en dark mode. `min-height` zorgt ervoor dat de body altijd minstens de volledige hoogte van het scherm heeft.

### Consuming ThemeContext

Voeg in `Navbar.jsx` een knop toe om van thema te wisselen:

```jsx
// src/components/Navbar.jsx
import { NavLink } from 'react-router';
import { useContext } from 'react'; //👈 1
import { ThemeContext } from '../contexts'; // 👈 1
import { IoMoonSharp, IoSunny } from 'react-icons/io5'; // 👈 4

const NavItem = ({ to, children, options}) => {
  return (
    <li className="mb-1">
      {/* 👇 3 */}
      <NavLink className={`text-gray-400 dark:text-white rounded  aria-[current=page]:text-blue-800 ${options}`}
        to={to}>{children}</NavLink>
    </li>
  );
};

//...

{/* 👇 2*/}
const ThemeToggle = () => {
  const { darkmode, toggleDarkmode } = useContext(ThemeContext);
  return (  <button
    type='button'
    onClick={toggleDarkmode}
    className='p-2 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors'
  >
    {darkmode ? <IoMoonSharp color='white' size={20}/> : <IoSunny size={20}/>}
  </button>);
};

export default function Navbar() {

  return (
    <>
      {/* 👇 3 */}
      <nav className="relative px-4 py-4 flex justify-between
      items-center bg-white dark:bg-gray-900 dark:text-gray-100
      border-b border-gray-200 dark:border-gray-700">
        <div className="flex items-center">
          <Logo />
        </div>

        <div className="lg:hidden">
          <button className="flex items-center text-blue-600 p-3" onClick={toggleNavbar}>
            <svg className="block h-4 w-4 fill-current" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <title>Mobile menu</title>
              <path d="M0 3h20v2H0V3zm0 6h20v2H0V9zm0 6h20v2H0v-2z"></path>
            </svg>
          </button>
        </div>

        <ul className="hidden absolute top-1/2 left-1/2
        transform -translate-y-1/2 -translate-x-1/2 lg:flex lg:mx-auto lg:items-center lg:w-auto lg:space-x-6">
          <NavItem to="/transactions">Transactions</NavItem>
          <NavItem to="/places">Places</NavItem>
          <NavItem to="/about">About</NavItem>
        </ul>

         {/* 👇 4 */}
        <div className="hidden lg:flex lg:items-center lg:space-x-4">
           <ThemeToggle />
        </div>

      </nav>
      <div className={`relative z-50 ${isNavbarOpen ? 'block' : 'hidden'} `}>
        <div className="fixed inset-0 bg-gray-800 opacity-25"></div>
        {/* 👇 3 */}
        <nav className="fixed top-0 left-0 bottom-0 flex flex-col w-5/6
        max-w-sm py-6 px-6 bg-white border-r overflow-y-auto space-between dark:bg-black">
          <div className="flex items-center mb-8">
            <Logo/>
            {/* 👇 4 */}
            <ThemeToggle />
            <button onClick={toggleNavbar} >
              {/* 👇 3 */}
              <svg className="h-6 w-6 text-gray-400 cursor-pointer hover:text-gray-500 dark:text-white"
                xmlns="http://www.w3.org/2000/svg" fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          </div>
          <div>
            <ul>
              <NavItem to="/transactions" options="block p-4 text-sm font-semibold">Transactions</NavItem>
              <NavItem to="/places" options="block p-4 text-sm font-semibold">Places</NavItem>
              <NavItem to="/about" options="block p-4 text-sm font-semibold">About</NavItem>
            </ul>
          </div>
        </nav>
      </div>
    </>
  );
}
```

1. Importeer de nodige componenten.
2. Maak een component aan met een knop om het thema te kiezen. Roep de `useContext` hook aan en gebruik destructuring om de properties, die de component nodig heeft, eruit te halen.
3. Voeg de dark klassen toe voor de achtergrondkleur en de kleur van de tekst.
4. Voeg de knop toe aan de navbar.

De toegevoegde code in de `TransactionTable` en `Layout` mag je nu verwijderen.

### Oefening 1 - ThemeContext

Voeg waar nodig de dark mode klassen toe in de andere componenten. Voorzien een grijze kleur voor de randen, lijnen van de tabel in dark mode. Pas ook de rand aan van de Place component in dark mode. De inputvelden stijlen we later in deze les.

## Custom hooks

Een custom hook is een JavaScript functie die begint met `use` en die andere hooks kan aanroepen. Custom hooks laten toe om logica te delen tussen componenten. Je kan zelf custom hooks schrijven of zoeken naar bestaande custom hooks via bv. <https://nikgraf.github.io/react-hooks/> of <https://npmjs.com>.

In elke component die gebruik maakt van de context dienen we volgende code te schrijven:

```jsx
import { ThemeContext } from '../../contexts/Theme.context';

const { darkmode, toggleDarkmode } = useContext(ThemeContext);
```

Om duplicate code te vermijden kunnen we gebruik maken van een **custom hook**. Neem hiervoor eerst [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) door.

Maak vervolgens volgende custom hook aan in `index.js`:

```jsx
// src/contexts/index.js
import { createContext, useContext } from 'react';// 👈1

export const ThemeContext = createContext();

export const useTheme = () => useContext(ThemeContext); // 👈 2

export const useThemeColor = () => {
  const { darkmode } = useContext(ThemeContext);
  return { darkmode };
}; // 👈 3
```

1. Importeer `useContext`.
2. De `useTheme` hook retourneert de waarden `darkmode`, `toggleDarkmode`.
3. De `useThemeColor` hook retourneert enkel `darkmode`.

Zo kan de code in `Navbar.jsx` als volgt aangepast worden:

```jsx
// src/components/Navbar.jsx
import { NavLink } from 'react-router';
//import { useContext } from 'react'; // 👈 1
//import { ThemeContext } from '../contexts/Theme.context'; // 👈 1
import { useTheme } from '../contexts'; // 👈 1
import { IoMoonSharp, IoSunny } from 'react-icons/io5';

const ThemeToggle = () => {
  const { darkmode, toggleDarkmode } = useTheme(); // 👈 2
  //...
};

```

1. Verwijder de import `useContext`, `ThemeContext`, en importeer `useTheme`.
2. Destructure de waarden die in deze component gebruikt worden.

### Oefening 2 - useTheme

Pas de StarRating component aan zodat deze ook van de `useThemeColor` hook gebruik maakt om i.g.v. darkmode een niet geselecteerde ster wit te kleuren.

## Anti-patterns

Er zitten een paar anti-patterns in ons formulier. Waarschijnlijk zijn deze ook aanwezig in jouw eigen project.

1. Gebruik geen constante object literals/arrays binnen de component, bv. validatieregels. Plaats deze buiten de component.
2. Definieer geen pure functies binnen de component (functies zonder afhankelijkheden van variabelen), bv. `toDateInputString`. Plaats deze buiten de component.
3. Definieer geen componenten inline in een andere component, bv. `LabelInput` (zie verder). Plaats deze buiten de component.
4. Gebruik een id als waarde voor de `key` prop in lijsten, gebruik geen index.
   - Deze fout hebben we reeds opgelost in hoofdstuk 1.

### Duplicate code

De combinatie `label` en `input` tag komen vaak voor. Kunnen we hier aparte component van maken?

Componenten mag je niet definiëren binnen een andere component. Ofwel maak je een functiecomponent `LabelInput` in het bestand van het formulier ofwel als aparte component als je deze wil hergebruiken. We kiezen voor de 2de optie. Plaats de code van het invoerveld van de gebruiker hierin en maak van de hardgecodeerde waarden props. Voeg ook de dark mode styling toe.

```jsx
// src/components/LabelInput.jsx
const LabelInput  = ({
  label,
  name,
  placeholder,
  type,
  validationRules,
  ...rest
}) => {
  const hasError = name in errors;

  return (
    <div className='mb-3'>
      <label htmlFor={name} className="block text-sm/6 font-medium text-gray-900 dark:text-white">
        {label}
      </label>
      <input
        {...register(name, validationRules)}
        id={name}
        name={name}
        type={type}
        className='rounded bg-white p-1
         text-gray-900 placeholder:text-gray-400 outline-1 outline-gray-300
          focus:outline-blue-600 w-full  dark:bg-gray-800 dark:text-white'
        placeholder={placeholder}
        {...rest}
      />
      {hasError && <p className="text-red-500 text-sm mt-1">{errors[name].message}</p> }
    </div>);
};

export default LabelInput;
```

Opdat de juiste styling zou worden toegepast op native browser controls(zoals date pickers, select dropdown, checkboxes, scrollbars) in dark mode (kalender icon wit bij input `type='date'`), passen we index.css aan. Zonder color-scheme zouden native browser controls altijd hun standaard (meestal lichte) styling behouden, zelfs in dark mode. Met deze CSS krijg je automatische native theming die perfect aansluit bij je custom Tailwind styling! Plaats dit buiten de @layer regels.

```css
@layer base {
  /*...*/
  input,
  select,
  textarea,
  button {
    color-scheme: light;
  }

  .dark input,
  .dark select,
  .dark textarea,
  .dark button {
    color-scheme: dark;
  }
}
```

We krijgen nog fouten (zie verder). Importeer eerst de `LabelInput` component in `TransactionForm` component en pas de invoervelden aan. De code voor het `userId` inputveld wordt:

```jsx
<LabelInput
  label='User Id'
  name='userId'
  placeholder='user id'
  type='number'
  validationRules={validationRules.userId}
/>;
{
  /* Herhaal dit voor de overige input fields */
}
```

We krijgen de fouten: `register and errors not defined`. Oplossing: `useFormContext` en `FormProvider`. Definitie van `useFormContext` uit de documentatie:

> This custom hook allows you to access the FormContext. useFormContext is intended to be used in deeply nested structures, where it would become inconvenient to pass the context as a prop.

Met `FormProvider` creëren we een nieuwe context provider voor een formulier. Met `useFormContext` halen we hieruit de waarde op en gebruiken dit in een (diep) geneste component tree.

In de documentatie lezen we ook het volgende over de `FormProvider`:

> React Hook Form's FormProvider is built upon React's Context API. It solves the problem where data is passed through the component tree without having to pass props down manually at every level. This also causes the component tree to trigger a rerender when React Hook Form triggers a state update.

```jsx
// src/components/transactions/TransactionForm.jsx
import { FormProvider, useForm } from 'react-hook-form'; // 👈 1
// ...

export default function TransactionForm({ places=[], transaction=EMPTY_TRANSACTION, saveTransaction }) {
  // ...
  // 👇 2
  const methods = useForm({
    mode: 'onBlur',
    defaultValues: {
      date: toDateInputString(transaction?.date),
      placeId: transaction?.place.id,
      amount: transaction?.amount,
      userId: transaction?.user.id,
    }
  });

  // 👇 3
  const {
    handleSubmit,
    formState: { isValid },
  } = methods;

  return (
    {/* 👇 4 */}
    <FormProvider {...methods}>
      <form onSubmit={handleSubmit(onSubmit)}>
        {/* ... */}
      </form>
    </FormProvider>
  );
}
```

Plaats eventjes de `select` lijst in commentaar. Verderop wordt dit ook een aparte component.

1. Importeer de `FormProvider`.
2. Verzamel alles uit de `useForm` hook in een object en deze zullen we doorgeven aan de `FormProvider` met de spread operator.
3. We moeten vervolgens enkel in deze component destructuren wat we nodig hebben. Zo kunnen we ook niets vergeten door te geven aan de `FormProvider`.
4. Plaats de `FormProvider` rond het formulier en geef alles door om de `useFormContext` correct te laten werken voor gebruik in `LabelInput` en `SelectList`.

Pas nu ook de `LabelInput` component aan:

```jsx
import { useFormContext } from 'react-hook-form'; // 👈

export default function LabelInput({
  label,
  name,
  placeholder,
  type,
  validationRules,
  ...rest
}) {
  const {
    register,
    formState: { errors },
  } = useFormContext(); // 👈

  //...
}
```

Importeer `useFormContext` en maak gebruik van `useFormContext` voor het gebruik van `register` en `errors`.

### Oefening 3 - SelectList

Maak een `SelectList` component aan. Voeg ook de dark mode styling toe.

- Oplossing +

  ```jsx
  // src/components/SelectList.jsx
  import { useFormContext } from 'react-hook-form';

  export default function SelectList({
    label, name, placeholder, items, validationRules, ...rest
  }) {
    const {
      register,
      formState: {
        errors,
      },
    } = useFormContext();

    const hasError = name in errors;

    return <div className='mb-3'>
      <label htmlFor={name} className="block text-sm/6 font-medium text-gray-900 dark:text-white mb-1.5">
        {label}
      </label>
      <select
        {...register(name, validationRules)}
        id={name}
        name={name}
        className="w-full appearance-none
            rounded-md bg-white py-1.5 pr-8 pl-3 text-base text-gray-900
            outline-1 -outline-offset-1 outline-gray-300 focus:outline-2
            focus:-outline-offset-2 focus:outline-blue-600
            sm:text-sm/6 dark:bg-gray-800 dark:text-white"
        {...rest} >
        <option value='' disabled>
          {placeholder}
        </option>
        {items.map(({ id, name }) => (
          <option key={id} value={id}>
            {name}
          </option>
        ))}
      </select>
      {hasError && <p className="text-red-500 text-sm mt-1">{errors[name].message}</p> }
    </div>;
  }
  ```

### Uitschakelen inputvelden bij submit

Je kan er ook voor zorgen dat de inputvelden en knoppen in het formulier _disabled_ worden als het formulier gesubmit wordt.
`useForm` geeft een boolean [isSubmitting](https://react-hook-form.com/api/useform/formstate) terug die `true` is als het formulier gesubmit wordt en `false` bij een reset. Voeg ook een `Cancel`knop toe

```jsx
export default function TransactionForm({
  places = [],
  transaction = EMPTY_TRANSACTION,
  saveTransaction,
}) {
  // ...
  const {
    handleSubmit,
    formState: { isSubmitting, isValid },
  } = methods; // 👆

  // ...
  return (
    <>
      <FormProvider {...methods}>
        {/* ... */}
        <div className='flex justify-end'>
              <button
              type='submit'
              disabled={isSubmitting}
              className='bg-blue-500 text-white font-medium py-2 px-4 rounded'
            >
              {transaction?.id ? 'Save transaction' : 'Add transaction'}
            </button>
            {/* 👇 */}
            <Link
              disabled={isSubmitting}
              className='py-2 px-4 rounded text-blue-500
      border border-blue-500 bg-white dark:bg-gray-900 ml-2'
              to='/transactions'
            >
              Cancel
            </Link>
          </div>
        </div>
        {/* ... */}
      </FormProvider>
    </>
  );
}
```

Disable het inputveld tijdens submit, doe hetzelfde voor de `SelectList` component.

```jsx
import { useFormContext } from 'react-hook-form';

export default function LabelInput({
  label,
  name,
  type,
  validationRules,
  ...rest
}) {
  const {
    register,
    formState: { errors, isSubmitting },
  } = useFormContext(); // 👆

  const hasError = name in errors;

  return (
    //...
      <input ...
        disabled={isSubmitting}

      /> {/* 👆 */}
    //...
  );
}
```

### Toevoegen van stijlregels voor de knoppen

- `@layer base` = voor HTML elementen (h1, p, body, a, etc.)
- `@layer components`= voor herbruikbare component classes (.btn, .card, .primary, etc.)
De cascade volgorde is: base -> components -> utilities. Dus als je een class in `@layer components` definieert, overschrijft deze de stijlen in `@layer base`.

```css
/* src/index.css */
@layer components {
  .primary {
    @apply bg-blue-500 text-white font-medium py-2 px-4 rounded;
  }
  .secondary {
    @apply py-2 px-4 rounded text-blue-500
      border border-blue-500 bg-white dark:bg-gray-900;
  }
}
```

Pas nu alle knoppen in de applicatie aan zodat ze de `primary` of `secondary` class gebruiken.

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les6-opl 4b5ec1b
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.

### Oefening 4 - Je eigen project

Controleer je eigen project op anti-patterns, duplicate code en refactor.
Denk na over global state in je project. Indien van toepassing, maak hiervoor een Context aan.
