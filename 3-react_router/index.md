# React Router

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3 TODO:
> pnpm install
> pnpm dev
> ```

Zoals je al weet, maken we met React een Single Page Application (SPA). Daardoor bestaat onze applicatie uit slechts één `index.html`. In deze HTML-pagina worden alle door Vite gegenereerde scripts en stylesheets geïnjecteerd.

Wanneer een client een React-applicatie opent (a.k.a. naar de URL ervan surft), wordt die ene `index.html` gedownload en gebeuren alle andere acties client-side. Het probleem is dat deze `index.html` enkel gedownload wordt indien we naar de `/` navigeren. Er zijn verder geen statische of server-side gegenereerde pagina's. Bijgevolg kunnen we enkel naar `/` navigeren, de browser downloadt vervolgens standaard `index.html`.

Daarnaast is React een library en geen framework, zoals bv. Angular dat wel is. Dit zorgt ervoor dat er dus geen ingebouwde router beschikbaar is. Daarvoor biedt [React Router](https://reactrouter.com/) een oplossing.

## React Router

React Router wordt aangeboden via de npm repository en biedt routing aan voor zowel React als React Native. React Router komt in 3 modes. We kiezen voor de 'data' mode ([meer info hier](https://reactrouter.com/start/modes)). Deze mode biedt de mogelijkheid om react routes op een eenvoudige manier te definiëren.

Installeer React Router [Zie de documentatie.](https://reactrouter.com/start/data/installation)

```bash
pnpm add react-router
```

### Achter de schermen

De werking van React Router is vrij intuïtief:

1. Vang een URL-wijziging op (bv. in de adresbalk of programmatisch gewijzigd).
2. Kijk of de URL gekend is:
   1. Indien gekend: toon de juiste pagina (lees: component).
   2. Indien niet gekend: stop (toont normaal niets maar je kan ook een eigen 404-pagina maken).

Deze werking is wel overmatig versimpeld, maar het geeft toch een idee...

## BrowserRouter vs HashRouter

### BrowserRouter

De [`BrowserRouter`](https://reactrouter.com/en/main/routers/create-browser-router) is een van de mogelijkheden om te gebruiken als router. Dit type router zal functioneren zoals je verwacht dat een router functioneert: hij gebruikt het deel na de `/` om naar een pagina te navigeren. Dit is zeer gelijkaardig aan hoe server-side gerenderde pagina's werken.

Een probleem hierbij is dat browsers standaard refreshen wanneer de URL na de `/` wijzigt. In dit geval zal `react-router` dit probleem opvangen en voorkomen.

Het voordeel met dit soort routers is dat je webapplicatie werkt zoals een _old-school website_, met alle features die een URL te bieden heeft.

### HashRouter

Een tweede type router is de [`HashRouter`](https://reactrouter.com/en/main/routers/create-hash-router). Deze router gebruikt de hash (of dus de `#`) in de URL om te navigeren tussen pagina's.

Dit heeft als voordeel dat de browser by default niet zal refreshen. Het nadeel is dat je de hash niet meer kan gebruiken om te scrollen naar een element (met een bepaald id) in de pagina.

### Welke kies je?

`HashRouter` wordt typisch weinig gebruikt, we raden aan om `BrowserRouter` te gebruiken. Let hierbij op dat je de routers uit de data API kiest, dus via de functies [`createBrowserRouter`](https://reactrouter.com/en/main/routers/create-browser-router) en [`createHashRouter`](https://reactrouter.com/en/main/routers/create-hash-router) en niet via de gelijknamige componenten.

Naast deze twee types zijn er nog andere beschikbaar, [lees meer in de documentatie](https://reactrouter.com/6.26.0/routers/picking-a-router).

### In de voorbeeldapplicatie

De voorbeeldapplicatie zal gebruik maken van een `BrowserRouter`. We dienen eerst een router toe te voegen aan de app. We voegen hiervoor een [Browser Router](https://reactrouter.com/en/main/routers/create-browser-router) toe en configureren onze eerste route. We doen dit in `main.tsx`, het startpunt van de app:

```jsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.tsx';
import { RouterProvider, createBrowserRouter } from 'react-router'; // 👈

// 👇
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
]);

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <RouterProvider router={router} />
    {/* 👈 */}
  </StrictMode>,
);
```

[createBrowserRouter](https://reactrouter.com/api/data-routers/createBrowserRouter#createbrowserrouter) creëert een `DataRouter` die zal functioneren als een `BrowserRouter`. De `BrowserRouter` gebruikt de DOM History API om een URL aan te passen en beheert de history stack. We geven een array met [RouteObject](https://reactrouter.com/start/data/route-object) objecten mee. Deze koppelen een URL (`path`) aan een component (`Component`). De `router` moeten we doorgeven aan de [RouterProvider](https://reactrouter.com/api/data-routers/RouterProvider#routerprovider).

In dit voorbeeld configureren we een enkele route die de `App` component toont wanneer de URL `/` is. We zullen later meer routes toevoegen.

Alle routes van de applicatie moeten doorgegeven worden aan de `RouterProvider`, deze moet dus de root-component zijn van de applicatie.

## Routes definiëren

We voorzien volgende basis routes in de voorbeeldapplicatie

- `/`: de home page (`App.tsx`) met links naar de andere pagina's (later voegen we een navigatiebalk toe)
- `/transactions`: een lijst van transacties
- `/places`: een lijst van places
- `/about`: over ons pagina

Alvorens we routes kunnen definiëren, voeren we een kleine refactoring uit. De verschillende pagina's in onze applicatie die direct verbonden zijn aan een URL/route plaatsen we in de `pages` map. Maak een map `pages`. Verplaats de componenten `PlacesList` en `TransactionList` naar de juiste map. Pas eventueel de paden in de component aan.

Voeg ook een `About` en `NotFound` pagina toe. Omdat we te lui zijn om deze zelf te vullen met tekst, gaan we gebruik maken van `@faker-js/faker`.

Installeer dit package:

```bash
pnpm add @faker-js/faker
```

Maak de `About` page aan in de map `src/pages/about`. We maken gebruik van een submap 'about' omdat deze pagina geneste routes zal bevatten die we later zullen implementeren.

```jsx
// src/pages/about/About.tsx
import { faker } from '@faker-js/faker';

const About = () => (
  <>
    <h1 className='text-2xl font-semibold mb-6'>About</h1>
    <div>
      <p className='mb-4'>{faker.lorem.paragraph(10)}</p>
      <p>{faker.lorem.paragraph(10)}</p>
    </div>
  </>
);

export default About;
```

Maak de `NotFound` page aan. Installeer eerst de `Alert` component van ShadCN:

```bash
pnpm dlx shadcn@latest add alert
```

En voeg dan onderstaande component toe:

```jsx
// src/pages/NotFound.tsx
import { Alert, AlertDescription } from '@/components/ui/alert';

export default function NotFound() {
  return (
    <>
      <h1 className='text-3xl font-semibold mb-4'>Not found</h1>
      <Alert variant='destructive'>
        <AlertDescription>
          There is no page at this url. Try something else.
        </AlertDescription>
      </Alert>
    </>
  );
}
```

Nu we de nodige pagina's hebben, hoeven we enkel nog de routes te configureren. Hiervoor gaan we naar de `main.tsx` en voegen de extra routes toe.

```jsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.tsx';
import { RouterProvider, createBrowserRouter } from 'react-router';
import TransactionList from './pages/TransactionList.tsx'; // 👈 1
import PlacesList from './pages/PlacesList.tsx'; // 👈 1
import About from './pages/about/About.tsx'; // 👈 1
import NotFound from './pages/NotFound.tsx'; // 👈 1

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
  { path: '/transactions', element: <TransactionList /> }, // 👈 2
  { path: '/places', element: <PlacesList /> }, // 👈 2
  { path: '/about', element: <About /> }, // 👈 2
  { path: '*', element: <NotFound /> }, // 👈 3
]);

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

1. Importeer de gewenste componenten (Merk op dat we de extensie `.tsx` expliciet moeten meegeven bij de imports van de componenten in de `pages` map. Dit is een eigenaardigheid van Vite).
2. Vul de array met route objecten aan, één voor elke route. We geven telkens de component die getoond moet worden mee aan de optie `element`. Wanneer de URL in de browser wijzigt, zal de `RouterProvider` doorheen zijn routes zoeken naar een geschikte match. Een route definiëren we door gebruik te maken van het `RouteObject`. We geven een absoluut pad mee aan de `path` optie, dit pad begint altijd met een `/`.
3. Dit zorgt ervoor dat de `NotFound` component getoond wordt indien de gebruiker op een URL uitkomt die niet bestaat. **Test dit zelf eens uit!**
   - Deze route hoeft niet als laatste staan. Waarom? React Router zoekt de meest exacte match en `*` is veel te algemeen.

Uit de route voor de `NotFound` component blijkt dat je ook reguliere expressies kan meegeven aan de `path` optie.

![How to regex](./images/how-to-regex.jpg ':size=50%')

## Navigeren tussen pagina's

Om te navigeren tussen pagina's kunnen we gebruik maken van de `Link` component. Pas de `App` component als volgt aan:

```jsx
// src/App.tsx
import { Link } from 'react-router'; //👈

function App() {
  return (
    <div className='bg-white text-gray-900 m-3'>
      <h1 className='text-2xl font-bold text-center mb-4'>Mijn Budget App</h1>
      <p>Kies één van de volgende links:</p>
      <ul>
        <li>
          <Link to='/transactions' className='text-blue-600 underline'>
            Transacties
          </Link>{' '}
          {/* 👈 */}
        </li>
        <li>
          <Link to='/places' className='text-blue-600 underline'>
            Plaatsen
          </Link>{' '}
          {/* 👈 */}
        </li>
        <li>
          <Link to='/about' className='text-blue-600 underline'>
            Over ons
          </Link>{' '}
          {/* 👈 */}
        </li>
      </ul>
    </div>
  );
}

export default App;
```

Je geeft de URL waarnaar genavigeerd moet worden mee aan de `to` prop. Achter de schermen wordt een anchor-element uit HTML gebruikt. De tekst tussen de `Link` tags wordt getoond als tekst voor de link.

## Route properties

Om eigenschappen over de huidige route op te vragen bestaat de hook `useLocation`. Pas de `NotFound` component aan:

```jsx
// src/pages/NotFound.tsx
import { useLocation } from 'react-router'; // 👈
import { Alert, AlertDescription } from '@/components/ui/alert';

export default function NotFound() {
  const { pathname } = useLocation(); // 👈

  return (
    <>
      <h1 className='text-3xl font-semibold mb-4'>Not found</h1>
      <Alert variant='destructive'>
        <AlertDescription>
          There is nothing at {pathname}, try something else.
        </AlertDescription>
        {/* 👈 */}
      </Alert>
    </>
  );
}
```

Deze hook retourneert nog diverse keys, **lees hierover volgende documentatie:**

- [useLocation](https://reactrouter.com/api/hooks/useLocation#uselocation)
- [Location interface van history package](https://github.com/remix-run/history/blob/main/docs/api-reference.md#location)

![How to use docs](./images/how-to-docs.jpg ':size=50%')

## Routes nesten

Je kan [geneste routes](https://reactrouter.com/start/data/routing#nested-routes) creëren om complexe UI-structuren te ondersteunen, waarbij een component subcomponenten heeft die worden weergegeven op basis van de URL. We willen nog drie extra routes die starten met `/about`: `/about/services`, `/about/history` en `/about/location`. We passen de about page aan zodat de active tab uit de link gehaald kan worden.

```jsx
// src/pages/about/About.tsx
import { faker } from '@faker-js/faker';
import { Link } from 'react-router'; // 👈

const About = () => (
  <>
    <h1 className='text-2xl font-semibold mb-6'>About</h1>
    <div>
      <p className='mb-4'>{faker.lorem.paragraph(10)}</p>
      <p>{faker.lorem.paragraph(10)}</p>
    </div>
    <ul className='p-4 mb-4'>
      <li>
        <Link to='/about/services' className='text-blue-600 underline'>
          Services
        </Link>{' '}
        {/* 👈 */}
      </li>
      <li>
        <Link to='/about/history' className='text-blue-600 underline'>
          History
        </Link>{' '}
        {/* 👈 */}
      </li>
      <li>
        <Link to='/about/location' className='text-blue-600 underline'>
          Location
        </Link>{' '}
        {/* 👈 */}
      </li>
    </ul>
  </>
);

export default About;
```

En we voegen deze pagina's toe aan `About.tsx`.

```jsx
export const Services = () => (
  <>
    <h1 className='text-2xl font-semibold mb-6'>Services</h1>
    <p>{faker.lorem.paragraph(10)}</p>
  </>
);

export const History = () => (
  <>
    <h1 className='text-2xl font-semibold mb-6'>History</h1>
    <p>{faker.lorem.paragraph(10)}</p>
  </>
);

export const Location = () => (
  <>
    <h1 className='text-2xl font-semibold mb-6'>Location</h1>
    <p>{faker.lorem.paragraph(10)}</p>
  </>
);
```

Daarna passen we de definitie van `/about` aan, de drie nieuwe routes dienen als kind van de `/about` route te worden aangemaakt (vergeet de nodige imports niet):

```jsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.tsx';
import { RouterProvider, createBrowserRouter } from 'react-router';
import TransactionList from './pages/TransactionList.tsx';
import PlacesList from './pages/PlacesList.tsx';
import NotFound from './pages/NotFound.tsx';
import About, { Services, History, Location } from './pages/about/About.tsx'; // 👈

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
  { path: 'transactions', element: <TransactionList /> },
  { path: 'places', element: <PlacesList /> },
  {
    path: '/about',
    element: <About />,
    children: [
      {
        path: 'services',
        element: <Services />,
      },
      {
        path: 'history',
        element: <History />,
      },
      {
        path: 'location',
        element: <Location />,
      },
    ], // 👆
  },
  { path: '*', element: <NotFound /> },
]);

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

Nu willen we de subroutes `/about/services`, `/about/history` en `/about/location` tonen op de `About` component. Met andere woorden `About` moet altijd getoond worden met `Services`, `History` of `Location` onderaan deze component. Hiervoor bestaat de `Outlet` component van React Router ([bekijk de documentatie](https://reactrouter.com/en/main/components/outlet)). We werken hier met relatieve routes, dit betekent dat we geen `/` moeten toevoegen aan het begin van het pad van de subroutes. React Router zal automatisch de URL van de ouder-route (`/about`) combineren met het pad van de subroute (`services`, `history` of `location`) om zo de volledige URL te vormen.

Voeg onderaan de `About` component de `Outlet` toe:

```jsx
// src/about/About.tsx
import { Outlet, Link } from 'react-router'; // 👈

// ...

const About = () => (
  <>
    {/* ... */}
    <Outlet /> {/* 👈 */}
  </>
);

// ...
```

## Redirects

Stel we willen dat gebruikers die naar `/about` navigeren naar `/about/services` doorgestuurd worden. Daarvoor voeg je volgende route toe aan de `main.tsx`:

```jsx
import { Navigate } from 'react-router';
// ...

const router = createBrowserRouter([
  // ...
  {
    path: '/services',
    element: <Navigate to='/about/services' replace />,
  },
  { path: '*', element: NotFound },
]);

// ...
```

Deze route rendert de `Navigate` component wanneer de gebruiker naar `/services` navigeert. Deze component is onderdeel van React Router en zal naar de URL in de `to` prop navigeren. De `replace` prop zorgt ervoor dat de URL `/services` vervangen wordt en bijgevolg verwijderd wordt uit de geschiedenis. Daarom kunnen we dus niet meer terugkeren naar `/services`, gebruikmakend van de terugknop van de browser.

Als we naar de `/about` pagina navigeren, zal de `About` component getoond worden. Omdat er geen subroute opgegeven is, zal de `Outlet` component niets tonen. Wanneer we naar `/about/services` navigeren, zal de `Services` component getoond worden in de `Outlet` van de `About` component. We willen nu dat gebruikers die naar `/about` navigeren automatisch doorgestuurd worden naar `/about/services`. Hiervoor voegen we een extra route toe die `/about` matcht en de `Navigate` component rendert:

```jsx
{ path: '/about',
  element: <About />,
  children: [
    {
      index: true,
      element: <Navigate to='/about/services' replace />,
    },
    //...
  ],
}
```

Deze route heeft de `index` optie meegegeven. Dit betekent dat deze route getoond wordt wanneer er geen subroute opgegeven is, dus wanneer we naar `/about` navigeren. De `Navigate` component zal ons dan automatisch doorsturen naar `/about/services`.

## URL parameters

In sommige gevallen wil je ook stukken in de URL kunnen invullen met bv. een id van een entiteit. De URL `/places/:id` geeft de details van één place weer. Hiervoor dient elke plaatsnaam aanklikbaar te zijn zodat we naar de detail van een plaats kunnen navigeren.

Maak een component `PlaceDetail.tsx` aan in de folder `/src/pages/places`. We plaatsen deze component in de `places` map omdat deze component een child is van de `PlacesList` component.

Definieer de nieuwe route in `main.tsx`:

```jsx
import PlaceDetail from './pages/places/PlaceDetail.tsx';
//...
{
  path: '/places',
  children: [
    {
      index: true,
      element: <PlacesList />,
    },
    {
      path: ':id',
      element: <PlaceDetail />,
    },
  ],
}
```

De eerste route is een index-route die enkel getoond wordt op `/places`. Bij de eerste route valt op dat we een `index` optie meegeven. Zonder deze optie kunnen we niet navigeren naar `/`, wat de zogenaamde index-pagina is. Per definitie deelt een route met `index` hetzelfde pad als zijn ouder-route. De tweede route zal getoond worden wanneer een id (eigenlijk eender welke string) opgegeven wordt na `/places`, bv. `/places/1` maar ook `/places/foo`.

Om dit id op te halen uit de URL maken we gebruik van de `useParams` hook. Deze hook retourneert een object van key/value pairs met alle URL parameters met hun waarde uit de huidige URL. Deze hook zal een nieuw object retourneren telkens wanneer een URL parameter wijzigt.

### Voorbeeld

Stel we hebben volgende routes gedefinieerd:

```jsx
{
  path: '/places/:id',
  element: <PlaceDetail />,
},
{
  path: '/posts/:year/:month',
  element: <Posts />,
}
```

Wanneer we navigeren naar `/places/263`, dan zal de `PlaceDetail` component getoond worden. Zijn `useParams` zal volgend object retourneren:

```js
{
  id: '263',
}
```

Wanneer we navigeren naar `/posts/2021/1`, dan zal de `Posts` component getoond worden. Zijn `useParams` zal volgend object retourneren:

```js
{
  year: '2021',
  month: '1',
}
```

> Merk op dat elke value een `string` is en geen `number` in dit geval. We zullen de `string` dus zelf nog moeten parsen via de `Number`-functie.

### Details van een place

We moeten nog enkel de `PlaceDetail` component implementeren zodat we de details van een place kunnen tonen (later halen we ook de bijhorende transacties op).

```jsx
// src/pages/places/PlaceDetail.tsx
import { useParams } from 'react-router';
import { PLACE_DATA } from '../../api/mock_data';

const PlaceDetail = () => {
  const { id } = useParams<{ id: string }>();
  const idAsNumber = Number(id);

  const place = PLACE_DATA.find((p) => p.id === idAsNumber);

  if (!place) {
    return (
      <>
        <h1 className='text-2xl font-semibold mb-6'>Plaats niet gevonden</h1>
        <p>Er is geen plaats met id {id}.</p>
      </>
    );
  }

  return (
    <>
      <h1 className='text-2xl font-semibold mb-6'>Place {place.name}</h1>
      <p>Hier komen de transacties van {place.name}</p>
    </>
  );
};

export default PlaceDetail;
```

Deze component zal eerst het id uit de URL ophalen en omvormen naar een `number`. Daarna zoekt het een plaats met het opgegeven id. Indien deze plaats niet bestaat, zal een gepaste boodschap getoond worden. In het andere geval wordt de informatie van deze plaats getoond.

### Oefening 1 - Navigeren naar een place

Zorg ervoor dat je op de `Place` pagina kan navigeren naar de detailpagina van een place door te klikken op de naam van een place.

Pas hiervoor de code in de component `Place` aan.

- Oplossing +

  We moeten enkel de naam van de place omvormen naar een link. Dit doen we met de `Link` component van React Router.

  ```jsx
  // src/components/places/Place.tsx
  import { Link } from 'react-router';
  //...
  <CardTitle className='text-base'>
    <Link to={`/places/${id}`} className='hover:underline'>
      {name}
    </Link>
  </CardTitle>;
  //...
  ```

## De Layout component

Nu willen we een navigatiebalk toevoegen aan de website (we houden het heel eenvoudig). Deze navigatiebalk wordt getoond op elke pagina. Om globale layout voor de app toe te voegen maak je een `Layout` component aan in de `src/components` map. Deze bevat de navigatiebalk en de `Outlet` component voor de weergave van de onderliggende routes.

```jsx
// src/components/Layout.tsx
import { Outlet } from 'react-router';
import Navbar from './Navbar';

export default function Layout() {
  return (
    <div className='min-h-screen bg-background text-foreground'>
      <Navbar />
      <main className='container mx-auto px-4 py-6 max-w-5xl'>
        <Outlet />
      </main>
    </div>
  );
}
```

### De navbar

De `Navbar` component voorziet in het menu. We maken een responsive menu.

```jsx
// src/components/Navbar.tsx
import { Link } from 'react-router';
import { useState } from 'react';
import { PiggyBankIcon, Menu, X } from 'lucide-react';

export default function Navbar() {
  const [isOpen, setIsOpen] = useState(false); // 👈1

  return (
    <header className='sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur'>
      <div className='container mx-auto flex h-14 max-w-5xl items-center px-4'>
        <Link
          to='/transactions'
          className='flex items-center gap-2 font-semibold text-primary mr-6'
        >
          <PiggyBankIcon className='size-5' />
          Budget
        </Link>

        {/* Desktop nav */}
        <nav className='hidden md:flex items-center gap-6 flex-1'>
          <Link to='/transactions'>Transactions</Link>
          <Link to='/places'>Places</Link>
          <Link to='/about'>About</Link>
        </nav>

        {/* Mobile toggle */}
        <button
          className='ml-auto md:hidden'
          onClick={() => setIsOpen((prev) => !prev)}
          aria-label='Toggle menu'
        >
          {isOpen ? <X className='h-5 w-5' /> : <Menu className='h-5 w-5' />}
        </button>
        {/* 👈1 */}
      </div>

      {/* Mobile menu */}
      {isOpen && (
        <div className='border-t md:hidden bg-background'>
          <nav className='container mx-auto flex flex-col gap-4 px-4 py-4 max-w-5xl'>
            <Link to='/transactions'>Transactions</Link>
            <Link to='/places'>Places</Link>
            <Link to='/about'>About</Link>
          </nav>
        </div>
      )}
    </header>
  );
}
```

1. We maken een state variabele `isOpen` aan om bij te houden of de navigatiebalk open of dicht is. Het klikken op de Mobile toggle button keert deze waarde om. We gebruiken deze waarde om de navigatiebalk te tonen of te verbergen.

### Integratie van de Layout component

Pas `main.tsx` aan, alle paden zijn nu kinderen van de `Layout` component en verwijder de `App`component

```jsx
// src/main.tsx
import Layout from './components/Layout.tsx'; // 👈
//...
const router = createBrowserRouter([
  {
    element: <Layout />, // 👈
    // 👇
    children: [
      {
        path: '/',
        element: <Navigate replace to='/transactions' />,
      },
      { path: '/transactions', element: <TransactionList /> },
      {
        path: '/places',
        children: [
          {
            index: true,
            element: <PlacesList />,
          },
          {
            path: ':id',
            element: <PlaceDetail />,
          },
        ],
      },
      {
        path: '/about',
        element: <About />,
        children: [
          {
            index: true,
            element: <Navigate to='/about/services' replace />,
          },
          {
            path: 'services',
            element: <Services />,
          },
          {
            path: 'history',
            element: <History />,
          },
          {
            path: 'location',
            element: <Location />,
          },
        ],
      },
      {
        path: '/services',
        element: <Navigate to='/about/services' replace />,
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);
//...
```

In `main.tsx` kan je nu de `App` component verwijderen.

### Aanduiden van de actieve link in de navigatie

Maak hiervoor gebruik van de `NavLink` component uit `react-router`. `NavLink` zet automatisch `aria-current="page"` op de actieve link. We voegen ook extra styling toe aan de (actieve) link zodat deze beter opvalt.

```jsx
// src/components/Navbar.tsx
//...
<NavLink
  className={({ isActive }) =>
    `text-sm font-medium transition-colors hover:text-primary ${
      isActive ? 'text-foreground' : 'text-muted-foreground'
    }`
  }
  to='/transactions'
>
  Transactions
</NavLink>
//...
```

De `NavLink` component heeft een `className` prop die een functie accepteert. Deze functie krijgt een object mee met de keys `isActive` en `isPending`. De key `isActive` is `true` wanneer de link actief is, anders is deze `false`. We kunnen deze waarde gebruiken om de juiste styling toe te passen op de link.

### Refactoring NavBar

We kunnen de code van de navigatiebalk nog wat opschonen door een aparte component `NavItem` te maken voor de links. Voeg de code toe in `Navbar.tsx`:

```jsx
// src/components/NavBar.tsx
interface NavItemProps {
  to: string;
  children: React.ReactNode;
}

const NavItem = ({ to, children }: NavItemProps) => (
  <NavLink
    className={({ isActive }) =>
      `text-sm font-medium transition-colors hover:text-primary ${
        isActive ? 'text-foreground' : 'text-muted-foreground'
      }`
    }
    to={to}
  >
    {children}
  </NavLink>
);
```

Pas de `Navbar` component aan:

```jsx
// src/components/Navbar.tsx
import { Link, NavLink } from 'react-router';
import { useState } from 'react';
import { PiggyBankIcon, Menu, X } from 'lucide-react';

interface NavItemProps {
  to: string;
  children: React.ReactNode;
}

const NavItem = ({ to, children }: NavItemProps) => (
  <NavLink
    className={({ isActive }) =>
      `text-sm font-medium transition-colors hover:text-primary ${
        isActive ? 'text-foreground' : 'text-muted-foreground'
      }`
    }
    to={to}
  >
    {children}
  </NavLink>
);

export default function Navbar() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <header className='sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur'>
      <div className='container mx-auto flex h-14 max-w-5xl items-center px-4'>
        <Link
          to='/transactions'
          className='flex items-center gap-2 font-semibold text-primary mr-6'
        >
          <PiggyBankIcon className='size-5' />
          Budget
        </Link>

        {/* Desktop nav */}
        <nav className='hidden md:flex items-center gap-6 flex-1'>
          <NavItem to='/transactions'>Transactions</NavItem> {/* 👈 */}
          <NavItem to='/places'>Places</NavItem> {/* 👈 */}
          <NavItem to='/about'>About</NavItem> {/* 👈 */}
        </nav>

        {/* Mobile toggle */}
        <button
          className='ml-auto md:hidden'
          onClick={() => setIsOpen((prev) => !prev)}
          aria-label='Toggle menu'
        >
          {isOpen ? <X className='h-5 w-5' /> : <Menu className='h-5 w-5' />}
        </button>
      </div>

      {/* Mobile menu */}
      {isOpen && (
        <div className='border-t md:hidden bg-background'>
          <nav className='container mx-auto flex flex-col gap-4 px-4 py-4 max-w-5xl'>
            <NavItem to='/transactions'>Transactions</NavItem> {/* 👈 */}
            <NavItem to='/places'>Places</NavItem> {/* 👈 */}
            <NavItem to='/about'>About</NavItem> {/* 👈 */}
          </nav>
        </div>
      )}
    </header>
  );
}
```

## Scroll restoration

Bij routing in SPA's wordt de scroll-positie niet automatisch hersteld naar linksboven in de browser. Indien gewenst, moet je hier zelf voor zorgen. Maak hiervoor gebruik van de `ScrollRestoration` component. Elke keer als de URL wijzigt, vraagt deze de browser om naar boven te scrollen. Pas hiervoor de `Layout` component aan.

```jsx
// src/components/Layout.tsx
import { Outlet, ScrollRestoration } from 'react-router'; // 👈
import Navbar from './Navbar';

export default function Layout() {
  return (
    <div className='min-h-screen bg-background text-foreground'>
      <Navbar />
      <main className='container mx-auto px-4 py-6 max-w-5xl'>
        <Outlet />
      </main>
      <ScrollRestoration /> {/* 👈 */}
    </div>
  );
}
```

## Navigeren vanuit code

Soms wil je navigeren vanuit code, daarvoor bestaat de `useNavigate` hook. Deze hook geeft een functie terug met o.a. de URL waarnaar genavigeerd wordt als parameter. Meer informatie staat uiteraard in de [useNavigate documentatie](https://reactrouter.com/6.26.0/hooks/use-navigate). Je kan bijvoorbeeld ook vragen om de huidige URL te vervangen zodat deze verdwijnt uit de "terugkeer-geschiedenis" van de browser.

Als voorbeeld gaan we onderaan de NotFound pagina een knop zetten waarmee we terug naar de home-pagina kunnen. Dit doen we door volgende code toe te voegen aan `NotFound.tsx`:

```jsx
// src/pages/NotFound.tsx
import { useLocation, useNavigate } from 'react-router'; // 👈
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Button } from '@/components/ui/button'; // 👈

export default function NotFound() {
  const { pathname } = useLocation();
  const navigate = useNavigate(); // 👈

  // 👇
  const handleGoHome = () => {
    navigate('/', { replace: true });
  };

  return (
    <>
      <h1 className='text-3xl font-semibold mb-4'>Not found</h1>
      <Alert variant='destructive'>
        <AlertDescription>
          There is nothing at {pathname}, <br /> {/* 👇 */}
          <Button
            variant='link'
            onClick={handleGoHome}
            className='text-destructive hover:text-destructive'
          ></Button>
        </AlertDescription>
      </Alert>
    </>
  );
}
```

Hiermee maken we een knop met een `onClick` handler. Deze functie zal via React Router terug naar de home-pagina navigeren en de huidige URL hierdoor vervangen.

Hetzelfde kan je bekomen met de Link tag, attribuut `replace` plaats je op true.

```jsx
import { Link, useLocation } from 'react-router';
//...
<Link to='/' replace className='underline hover:no-underline'>
  go back home
</Link>;
```

?> Het is aangeraden om zoveel mogelijk gebruik te maken van de `Link` component. Dit zorgt ervoor dat de gebruiker meer controle heeft over links, zoals het openen in een nieuw tabblad.

## Tabs in shadcn: controlled vs uncontrolled

Bij het gebruik van de [Tabs component](https://ui.shadcn.com/docs/components/base/tabs) in shadcn werk je standaard met een **uncontrolled component** via `defaultValue`. Voor meer controle (bv. synchroniseren met state, routing, filters…) moet je overschakelen naar een **controlled component**. Dit betekent dat je zelf de actieve tab in state beheert.

### Stap 1: Tabs component toevoegen aan de About page

Neem de [documentatie](https://ui.shadcn.com/docs/components/base/tabs) door. Maak een nieuwe pagina `AboutTabs.tsx` aan en verwijs in `main.tsx` naar deze component.

```jsx
// src/about/AboutTabs.tsx
import { faker } from '@faker-js/faker';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Briefcase, Clock, MapPin } from 'lucide-react';

const About = () => {
  return (
    <div className='space-y-8'>
      <h1 className='text-2xl font-semibold mb-6'>About</h1>
      <div>
        <p className='text-muted-foreground leading-relaxed'>
          {faker.lorem.paragraph(10)}
        </p>
      </div>
      <Tabs defaultValue='services'>
        <TabsList variant='line'>
          <TabsTrigger value='services' className='gap-1.5'>
            <Briefcase className='h-4 w-4' />
            Our Services
          </TabsTrigger>
          <TabsTrigger value='history' className='gap-1.5'>
            <Clock className='h-4 w-4' />
            History
          </TabsTrigger>
          <TabsTrigger value='location' className='gap-1.5'>
            <MapPin className='h-4 w-4' />
            Location
          </TabsTrigger>
        </TabsList>
        <Services />
        <History />
        <Location />
      </Tabs>
    </div>
  );
};

export default About;

export const Services = () => (
  <TabsContent value='services' className='pt-4 space-y-3'>
    <p className='text-sm text-muted-foreground leading-relaxed'>
      {faker.lorem.paragraph(10)}
    </p>
  </TabsContent>
);

export const History = () => (
  <TabsContent value='history' className='pt-4 space-y-3'>
    <p className='text-sm text-muted-foreground leading-relaxed'>
      {faker.lorem.paragraph(10)}
    </p>
  </TabsContent>
);

export const Location = () => (
  <TabsContent value='location' className='pt-4 space-y-3'>
    <p className='text-sm text-muted-foreground leading-relaxed'>
      {faker.lorem.paragraph(10)}
    </p>
  </TabsContent>
);
```

De tabs beheren hier zelf hun state. Dit is **uncontrolled gedrag**. Maar wat als:

- je via URL `/about/history` binnenkomt?
- je de tab wil syncen met routing?

We moeten overschakelen naar een **controlled component** waar we zelf de state beheren.

### Stap 2: API Documentatie doornemen

`Shadcn` is geen volledige component library, maar een gestylede wrapper rond primitives. Het gedrag (logica, state, events) komt van de libraries `Base UI / Radix UI`. Shadcn voegt vooral styling en structuur toe.

De shadcn docs tonen meestal enkel een basisgebruik, voor de Tabs met de prop `defaultValue`. De `defaultValue` is de initiële actieve tab. Daarna beheert de component **zelf de state**.

```jsx
<Tabs defaultValue='tab1' />
```

Maar dat is slechts een deel van de mogelijkheden. Onderaan de shadcn documentatie vind je: "See the Basic Tabs documentation". Daar word je doorgestuurd naar:

- [Base UI](https://base-ui.com/react/components/tabs)
- of [Radix UI](https://www.radix-ui.com/primitives/docs/components/tabs#api-reference).

⚠️ Let op: kies bovenaan expliciet voor de `Base UI` tab.

Wil je begrijpen hoe iets werkt of welke props beschikbaar zijn, dan moet je naar de onderliggende API kijken. Daar vinden we extra props:

- defaultValue: startwaarde (uncontrolled)
- value: huidig actieve tab (controlled)
- onValueChange: callback bij wijziging

Controlled gedrag werkt als volgt

```jsx
const [activeTab, setActiveTab] = useState("tab1")
<Tabs value={activeTab} onValueChange={setActiveTab} />
```

### Stap 3: actieve tab afleiden uit de routing

In deze stap bepaalt de url de tab die getoond zal worden.

1. Haal het path op en extraheer het segment.
2. Definiëer een constante TABS. Door `as const` ziet TS dit als `readonly ['services', 'history', 'location']` (exact deze 3 waarden) en niet als een `sting[]`.
3. Dit creëert een type dat slechts 3 waarden kan zijn `type TabValue = 'services' | 'history' | 'location'`
4. Valideer de waarde, zorgt voor een fallback.
5. Reageer op de tabwissel. De tabs veranderen niet zelf. We veranderen de URL en de UI volgt
6. Maak de tabs controlled
7. De content wordt bepaald door React Router

```jsx
import { faker } from '@faker-js/faker';
import { Outlet, useLocation, useNavigate } from 'react-router'; // 👈1, 5, 7
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Briefcase, Clock, MapPin } from 'lucide-react';

const TABS = ['services', 'history', 'location'] as const; // 👈2
type TabValue = (typeof TABS)[number]; // 👈3

const About = () => {
  const navigate = useNavigate(); // 👈5
  const { pathname } = useLocation(); // 👈1

  const segment = pathname.split('/').pop() as TabValue; // 👈1
  const activeTab: TabValue = TABS.includes(segment) ? segment : 'services'; //👈4

  const handleTabChange = (val: TabValue) => {
    navigate(`/about/${val}`);
  }; // 👈5

  return (
    <div className='space-y-8'>
      <h1 className='text-2xl font-semibold mb-6'>About</h1>
      <div>
        <p className='text-muted-foreground leading-relaxed'>
          {faker.lorem.paragraph(10)}
        </p>
      </div>
      <Tabs value={activeTab} onValueChange={handleTabChange}>
        {/* 👈6 */}
        <TabsList variant='line'>
          <TabsTrigger value='services' className='gap-1.5'>
            <Briefcase className='h-4 w-4' />
            Our Services
          </TabsTrigger>
          <TabsTrigger value='history' className='gap-1.5'>
            <Clock className='h-4 w-4' />
            History
          </TabsTrigger>
          <TabsTrigger value='location' className='gap-1.5'>
            <MapPin className='h-4 w-4' />
            Location
          </TabsTrigger>
        </TabsList>
        <Outlet /> {/* 👈7 */}
      </Tabs>
    </div>
  );
};

export default About;
//...
```

## Custom styles

Bij elke h1-tag dienen we dezelfde styling toe te passen. Je kan custom styles definiëren in de `index.css`:

```css
@layer base {
  h1 {
    @apply text-2xl font-semibold mb-6;
  }
}
```

Hierdoor zal elke `h1` tag automatisch de juiste styling krijgen. `@apply` is een tailwind CSS directive.

Zorg ervoor dat je in `main.tsx` refereert naar de CSS:

```jsx
import './index.css';
```

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3-opl TODO:
> pnpm install
> pnpm dev
> ```

## Oefening 2 - Je eigen project

Denk voor je eigen applicatie na over de navigatie en implementeer.

## Mogelijke extra's voor de examenopdracht

- Gebruik de nieuwe [loader](https://reactrouter.com/en/main/route/loader) en [action](https://reactrouter.com/en/main/route/action) props van de `Route` component van `react-router` om de data op te halen.
  - Dit is een vrij kleine extra, dus zorg ervoor dat je nog een andere extra toevoegt.
