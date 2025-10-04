# React Router

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3 f67787f
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

De voorbeeldapplicatie zal gebruik maken van een `BrowserRouter`. We dienen eerst een router toe te voegen aan de app. We voegen hiervoor een [Browser Router](https://reactrouter.com/en/main/routers/create-browser-router) toe en configureren onze eerste route. We doen dit in `main.jsx`, het startpunt van de app:

```jsx
// src/main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';
import { createBrowserRouter } from 'react-router';// 👈
import { RouterProvider } from 'react-router/dom'; // 👈

// 👇
const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
]);

createRoot(document.getElementById('root')).render(
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

- `/`: de home page (`App.jsx`) met links naar de andere pagina's (later voegen we een navigatiebalk toe)
- `/transactions`: een lijst van transacties
- `/places`: een lijst van places
- `/about`: over ons pagina

Alvorens we routes kunnen definiëren, voeren we een kleine refactoring uit. De verschillende pagina's in onze applicatie plaatsen we in de `pages` map. Maak een map `pages` aan met daarin de mappen `places` en `transactions`. Verplaats de componenten `PlacesList` en `TransactionList` naar de juiste map. Pas eventueel de paden in de component aan. Voeg een titel toe aan de PlacesList component.

Voeg ook een `About` en `NotFound` pagina toe. Omdat we te lui zijn om deze zelf te vullen met tekst, gaan we gebruik maken van `@faker-js/faker`.

Installeer dit package:

```bash
pnpm add @faker-js/faker
```

Maak de `About` page aan:

```jsx
// src/pages/about/About.jsx
import { faker } from '@faker-js/faker';

const About = () => (
  <div>
    <h1 className="text-4xl mb-4">Over ons</h1>
    <div>
      <p className="mb-4">{faker.lorem.paragraph(10)}</p>
      <p>{faker.lorem.paragraph(10)}</p>
    </div>
  </div>
);

export default About;
```

Maak de `NotFound` page aan

```jsx
// src/pages/NotFound.jsx
const NotFound = () => {
  return (
    <div>
      <h1 className="text-4xl mb-4">Pagina niet gevonden</h1>
      <p>Er is geen pagina op deze url, probeer iets anders.</p>
    </div>
  );
};

export default NotFound;
```

Nu we de nodige pagina's hebben, hoeven we enkel nog de routes te configureren. Hiervoor gaan we naar de `main.jsx` en voegen de extra routes toe.

```jsx
// src/main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import { createBrowserRouter } from 'react-router';
import { RouterProvider } from 'react-router/dom';
import TransactionList from './pages/transactions/TransactionsList.jsx'; // 👈 1
import PlacesList from './pages/places/PlacesList.jsx'; // 👈 1
import NotFound from './pages/NotFound.jsx'; // 👈 1
import About from './pages/about/About.jsx'; // 👈 1

// 👇
const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
  { path: 'transactions', Component: TransactionList }, // 👈 2
  { path: 'places', Component: PlacesList }, // 👈 2
  { path: 'about', Component: About }, // 👈 2
  { path: '*', Component: NotFound }, // 👈 3
]);

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

1. Importeer de gewenste componenten (Merk op dat we de extensie `.jsx` expliciet moeten meegeven bij de imports van de componenten in de `pages` map. Dit is een eigenaardigheid van Vite).
2. Vul de array met route objecten aan, één voor elke route. We geven telkens de component die getoond moet worden mee aan de optie `Component`. Wanneer de URL in de browser wijzigt, zal de `RouterProvider` doorheen zijn routes zoeken naar een geschikte match. Een route definiëren we door gebruik te maken van het `RouteObject`.
3. Dit zorgt ervoor dat de `NotFound` component getoond wordt indien de gebruiker op een URL uitkomt die niet bestaat. **Test dit zelf eens uit!**
   - Deze route hoeft niet als laatste staan. Waarom? React Router zoekt de meest exacte match en `*` is veel te algemeen.

Uit de route voor de `NotFound` component blijkt dat je ook reguliere expressies kan meegeven aan de `path` optie.

![How to regex](./images/how-to-regex.jpg ':size=50%')

## Navigeren tussen pagina's

Om te navigeren tussen pagina's kunnen we gebruik maken van de `Link` component. Pas de `App` component als volgt aan:

```jsx
// src/App.jsx
import { Link } from 'react-router';

function App() {
  return (
    <div className="mx-4">
      <h1 className="text-4xl mb-4">Welkom!</h1>
      <p>Kies één van de volgende links:</p>
      <ul>
        <li>
          <Link to='/transactions' className="text-blue-600 underline">Transacties</Link> {/* 👈 */}
        </li>
        <li>
          <Link to='/places' className="text-blue-600 underline">Plaatsen</Link> {/* 👈 */}
        </li>
        <li>
          <Link to='/about' className="text-blue-600 underline">Over ons</Link> {/* 👈 */}
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
// src/pages/NotFound.jsx
import { useLocation } from 'react-router'; // 👈

const NotFound = () => {
  const { pathname } = useLocation(); // 👈

  return (
    <div>
      <h1 className="text-4xl mb-4">Pagina niet gevonden</h1>
      <p>Er is geen pagina met als url {pathname}, probeer iets anders.</p> {/* 👈 */}
    </div>
  );
};
export default NotFound;
```

Deze hook retourneert nog diverse keys, **lees hierover volgende documentatie:**

- [useLocation](https://reactrouter.com/api/hooks/useLocation#uselocation)
- [Location interface van history package](https://github.com/remix-run/history/blob/main/docs/api-reference.md#location)

![How to use docs](./images/how-to-docs.jpg ':size=50%')

## Routes nesten

Je kan [geneste routes](https://reactrouter.com/start/data/routing#nested-routes) creëren om complexe UI-structuren te ondersteunen, waarbij een component subcomponenten heeft die worden weergegeven op basis van de URL. We willen nog drie extra routes die starten met `/about`: `/about/services`, `/about/history` en `/about/location`. We voegen enkele links toe aan onze `About` component:

```jsx
// src/pages/about/About.jsx
import { faker } from '@faker-js/faker';
import { Link } from 'react-router';// 👈

const About = () => (
  <div>
    <h1 className="text-4xl mb-4">Over ons</h1>
    <div>
      <p className="mb-4">{faker.lorem.paragraph(10)}</p>
      <p>{faker.lorem.paragraph(10)}</p>
    </div>
    <ul  className="p-4 mb-4">
      <li>
        <Link to='/about/services' className="text-blue-600 underline">Onze diensten</Link> {/* 👈 */}
      </li>
      <li>
        <Link to='/about/history' className="text-blue-600 underline">Geschiedenis</Link> {/* 👈 */}
      </li>
      <li>
        <Link to='/about/location' className="text-blue-600 underline">Locatie</Link> {/* 👈 */}
      </li>
    </ul>
  </div>
);

export default About;
```

En we voegen deze pagina's toe aan `About.jsx`.

```jsx
export const Services = () => (
  <div>
    <h1 className="text-4xl mb-4">Onze diensten</h1>
    <p>{faker.lorem.paragraph(10)}</p>
  </div>
);

export const History = () => (
  <div>
    <h1 className="text-4xl mb-4">Geschiedenis</h1>
    <p>{faker.lorem.paragraph(10)}</p>
  </div>
);

export const Location = () => (
  <div>
    <h1 className="text-4xl mb-4">Locatie</h1>
    <p>{faker.lorem.paragraph(10)}</p>
  </div>
);
```

Daarna passen we de definitie van `/about` aan, de drie nieuwe routes dienen als kind van de `/about` route te worden aangemaakt (vergeet de nodige imports niet):

```jsx
// src/main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import { createBrowserRouter } from 'react-router';
import { RouterProvider } from 'react-router/dom';
import TransactionList from './pages/transactions/TransactionList';
import PlacesList from './pages/places/PlacesList';
import NotFound from './pages/NotFound';
import About, { Services, History, Location } from './pages/about/About.jsx'; // 👈

const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
  { path: 'transactions', Component: TransactionList },
  { path: 'places', Component: PlacesList },
  {
    path: 'about',
    Component: About,
    children: [
      {
        path: 'services',
        Component: Services,
      },
      {
        path: 'history',
        Component: History,
      },
      {
        path: 'location',
        Component: Location,
      },
    ], // 👆
  },
  { path: '*', Component: NotFound },
]);

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

Nu willen we de subroutes `/about/services`, `/about/history` en `/about/location` tonen op de `About` component. Met andere woorden `About` moet altijd getoond worden met `Onze diensten`, `Geschiedenis` of `Locatie` onderaan deze component. Hiervoor bestaat de `Outlet` component van React Router ([bekijk de documentatie](https://reactrouter.com/en/main/components/outlet)).

Voeg onderaan de `About` component de `Outlet` toe:

```jsx
// src/about/About.jsx
import { Outlet, Link } from 'react-router'; // 👈

// ...

const About = () => (
  <div>
    {/* ... */}
    <Outlet /> {/* 👈 */}
  </div>
);

// ...
```

## Redirects

Stel we willen dat gebruikers die naar `/services` navigeren naar `/about/services` doorgestuurd worden. Daarvoor voeg je volgende route toe aan de `main.jsx`:

```jsx
import { Navigate } from 'react-router';
// ...

const router = createBrowserRouter([
  // ...
  {
    path: 'services',
    element: <Navigate to='/about/services' replace />,
  },
  { path: '*', element: <NotFound /> },
]);

// ...
```

Deze route rendert de `Navigate` component wanneer de gebruiker naar `/services` navigeert. Deze component is onderdeel van React Router en zal naar de URL in de `to` prop navigeren. De `replace` prop zorgt ervoor dat de URL `/services` vervangen wordt en bijgevolg verwijderd wordt uit de geschiedenis. Daarom kunnen we dus niet meer terugkeren naar `/services`, gebruikmakend van de terugknop van de browser.

## URL parameters

In sommige gevallen wil je ook stukken in de URL kunnen invullen met bv. een id van een entiteit. De URL `/places/:id` geeft de details van één place weer. Hiervoor dient elke plaatsnaam aanklikbaar te zijn zodat we naar de detail van een plaats kunnen navigeren.

Maak een component `PlaceDetail.jsx` aan in de folder `/src/pages/places`.

Definieer de nieuwe route in `main.jsx`:

```jsx
import PlaceDetail from './pages/places/PlaceDetail.jsx';
//...
{
  path: '/places',
  children: [
    {
      index: true,
      Component: PlacesList,
    },
    {
      path: ':id',
      Component: PlaceDetail,
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
  Component: PlaceDetail
},
{
  path: '/posts/:year/:month',
  Component: Posts
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
// src/pages/places/PlaceDetail.jsx
import { useParams } from 'react-router';
import { PLACE_DATA } from '../../api/mock_data';

const PlaceDetail = () => {
  const { id } = useParams();
  const idAsNumber = Number(id);

  const place = PLACE_DATA.find((p) => p.id === idAsNumber);

  if (!place) {
    return (
      <div>
        <h1 className="text-4xl mb-4">Plaats niet gevonden</h1>
        <p>Er is geen plaats met id {id}.</p>
      </div>
    );
  }

  return (
    <div>
      <h1 className="text-4xl mb-4">{place.name}</h1>
      <p>Hier komen de transacties van {place.name}</p>
    </div>
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
  // src/components/places/Place.jsx
  import { Link } from 'react-router';
  //...
  <h5 className="text-xl font-medium mb-2">
    <Link className="text-blue-600 underline" to={`/places/${id}`}>{name}</Link>
  </h5>
  //...
  ```

## De Layout component

Nu willen we een navigatiebalk toevoegen aan de website (we houden het heel eenvoudig). Deze navigatiebalk wordt getoond op elke pagina. Om globale layout voor de app toe te voegen maak je een `Layout` component aan in de `src/pages` map. Deze bevat de navigatiebalk en de `Outlet` component voor de weergave van de onderliggende routes.

```jsx
// src/pages/Layout.jsx
import { Outlet } from 'react-router';
import Navbar from '../components/Navbar';

export default function Layout() {
  return (
    <div className='container-xl'>
      <Navbar />
      <div className='p-4'>
        <Outlet />
      </div>
    </div>
  );
}
```

### De navbar

De `Navbar` component voorziet in het menu. We maken een responsive menu.

```jsx
// src/components/Navbar.jsx
// src/components/Navbar.jsx
import { Link } from 'react-router';
import { useState } from 'react';
import { BsFillPiggyBankFill } from 'react-icons/bs';

export default function Navbar() {

  const [isNavbarOpen, setIsNavbarOpen] = useState(false);// 👈1

  const toggleNavbar = () => {
    setIsNavbarOpen(!isNavbarOpen);
  };// 👈1

  return (
    <>
      <nav className="relative px-4 py-4 flex justify-between items-center bg-gray-200">

        <div className="flex items-center">
          <Link to="/" className="flex items-center text-blue-600 hover:text-blue-800">
            <BsFillPiggyBankFill size={28} className="text-blue-600" />
            <span className="font-semibold text-lg pl-2">Budget</span>
          </Link>
        </div>
        <div className="lg:hidden">
          <button className="navbar-burger flex items-center text-blue-600 p-3" onClick={toggleNavbar}>{/* 👈1 */}
            <svg className="block h-4 w-4 fill-current" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <title>Mobile menu</title>
              <path d="M0 3h20v2H0V3zm0 6h20v2H0V9zm0 6h20v2H0v-2z"></path>
            </svg>
          </button>
        </div>
        <ul className="hidden absolute top-1/2 left-1/2
        transform -translate-y-1/2 -translate-x-1/2 lg:flex lg:mx-auto lg:items-center lg:w-auto lg:space-x-6">
          <li><Link className='text-gray-400' to='/transactions'>
            Transactions
          </Link></li>
          <li><Link className='text-gray-400' to='/places'>
            Places
          </Link></li>
          <li><Link className='text-gray-400' to='/about'>
            About us
          </Link></li>
        </ul>
      </nav>
      <div className={`navbar-menu relative z-50 ${isNavbarOpen ? 'block' : 'hidden'}`}>{/* 👈 */}
        <div className="navbar-backdrop fixed inset-0 bg-gray-800 opacity-25"></div>
        <nav className="fixed top-0 left-0 bottom-0 flex flex-col w-5/6
        max-w-sm py-6 px-6 bg-white border-r overflow-y-auto space-between">
          <div className="flex items-center mb-8">
            <Link to="/" className="mr-auto flex items-center space-x-2 text-blue-600 hover:text-blue-800">
              <BsFillPiggyBankFill size={28} className="text-blue-600" />
              <span className="font-semibold text-lg">Budget</span>
            </Link>
            <button onClick={toggleNavbar} className="navbar-close" >
              <svg className="h-6 w-6 text-gray-400 cursor-pointer hover:text-gray-500"
                xmlns="http://www.w3.org/2000/svg" fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          </div>
          <div>
            <ul>
              <li className="mb-1">
                <Link className="block p-4 text-sm font-semibold
                text-gray-400 rounded" to="/transactions">Transactions</Link>
              </li>
              <li className="mb-1">
                <Link className="block p-4 text-sm font-semibold
                text-gray-400 rounded" to="/places">Places</Link>
              </li>
              <li className="mb-1">
                <Link className="block p-4 text-sm font-semibold
                text-gray-400 rounded" to="/about">About us</Link>
              </li>
            </ul>
          </div>
        </nav>
      </div>
    </>
  );
}
```

1. We maken een state variabele `isNavbarOpen` aan om bij te houden of de navigatiebalk open of dicht is. De `toggleNavbar` functie keert deze waarde om. We gebruiken deze waarde om de navigatiebalk te tonen of te verbergen.

### Integratie van de Layout component

Pas `main.jsx` aan, alle paden zijn nu kinderen van de `Layout` component en verwijder de `App`component

```jsx
// src/main.jsx
import Layout from './pages/Layout.jsx';// 👈
//...
const router = createBrowserRouter([
  {
    Component: Layout, // 👈
    // 👇
    children: [
      {
        path: '/',
        element: <Navigate replace to='/transactions' />,
      },
      { path: 'transactions', Component: TransactionList },
      {
        path: '/places',
        children: [
          {
            index: true,
            Component: PlacesList,
          },
          {
            path: ':id',
            Component: PlaceDetail,
          },
        ],
      },
      {
        path: 'about',
        Component: About,
        children: [
          {
            path: 'services',
            Component: Services,
          },
          {
            path: 'history',
            Component: History,
          },
          {
            path: 'location',
            Component: Location,
          },
        ], // 👆
      },
      {
        path: 'services',
        element: <Navigate to='/about/services' replace />,
      },
      { path: '*', Component: NotFound },
    ],
  }]);
//...
```

In `main.jsx` kan je nu de `App` component verwijderen.

### Aanduiden van de actieve link in de navigatie

Maak hiervoor gebruik van de `NavLink` component uit `react-router`. `NavLink` zet automatisch `aria-current="page"` op de actieve link. Tailwind's `aria-[current=page]:text-blue-800` selector pakt deze status op.

```jsx
// src/components/Navbar.jsx
//...
<NavLink className="text-gray-400 aria-[current=page]:text-blue-800"
  to="/transactions">Transactions</NavLink>
//...
```

### Refactoring NavBar

We kunnen de code van de navigatiebalk nog wat opschonen door een aparte component `NavItem` te maken voor de links:

```jsx
// src/components/NavBar.jsx
const NavItem = ({ to, children, options}) => {
  return (
    <li className="mb-1">
      <NavLink className={`text-gray-400 rounded  aria-[current=page]:text-blue-800 ${options}`}
        to={to}>{children}</NavLink>
    </li>
  );
};
```

Voor het logo maken we ook een aparte component `Logo` aan:

```jsx
// src/components/Navbar.jsx
const Logo = () => {
  return (
    <Link to="/" className="mr-auto flex items-center space-x-2 text-blue-600 hover:text-blue-800">
      <BsFillPiggyBankFill size={28} className="text-blue-600" />
      <span className="font-semibold text-lg">Budget</span>
    </Link>
  );
};
```

Pas  de `Navbar` component aan:

```jsx
// src/components/Navbar.jsx
//...
export default function Navbar() {

  const [isNavbarOpen, setIsNavbarOpen] = useState(false);

  const toggleNavbar = () => {
    setIsNavbarOpen(!isNavbarOpen);
  };

  return (
    <>
      <nav className="relative px-4 py-4 flex justify-between items-center bg-gray-200">

        <div className="flex items-center">
          <Logo />{/* 👈 */}
        </div>

        <div className="lg:hidden">
          <button className="navbar-burger flex items-center text-blue-600 p-3" onClick={toggleNavbar}>
            <svg className="block h-4 w-4 fill-current" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <title>Mobile menu</title>
              <path d="M0 3h20v2H0V3zm0 6h20v2H0V9zm0 6h20v2H0v-2z"></path>
            </svg>
          </button>
        </div>
        <ul className="hidden absolute top-1/2 left-1/2
        transform -translate-y-1/2 -translate-x-1/2 lg:flex lg:mx-auto lg:items-center lg:w-auto lg:space-x-6">
          <NavItem to="/transactions">Transactions</NavItem>{/* 👈 */}
          <NavItem to="/places">Places</NavItem>{/* 👈 */}
          <NavItem to="/about">About</NavItem>{/* 👈 */}
        </ul>
      </nav>
      <div className={`navbar-menu relative z-50 ${isNavbarOpen ? 'block' : 'hidden'}`}>
        <div className="navbar-backdrop fixed inset-0 bg-gray-800 opacity-25"></div>
        <nav className="fixed top-0 left-0 bottom-0 flex flex-col w-5/6
        max-w-sm py-6 px-6 bg-white border-r overflow-y-auto space-between">
          <div className="flex items-center mb-8">
            <Logo/>{/* 👈 */}
            <button onClick={toggleNavbar} className="navbar-close" >
              <svg className="h-6 w-6 text-gray-400 cursor-pointer hover:text-gray-500"
                xmlns="http://www.w3.org/2000/svg" fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          </div>
          <div>
            <ul>
              <NavItem to="/transactions" options="block p-4 text-sm font-semibold">Transactions</NavItem>{/* 👈 */}
              <NavItem to="/places" options="block p-4 text-sm font-semibold">Places</NavItem>{/* 👈 */}
              <NavItem to="/about" options="block p-4 text-sm font-semibold">About</NavItem>{/* 👈 */}
            </ul>
          </div>
        </nav>
      </div>
    </>
  );
}
//...
```

## Scroll restoration

Bij routing in SPA's wordt de scroll-positie niet automatisch hersteld naar linksboven in de browser. Indien gewenst, moet je hier zelf voor zorgen. Maak hiervoor gebruik van de `ScrollRestoration` component. Elke keer als de URL wijzigt, vraagt deze de browser om naar boven te scrollen. Pas hiervoor de `Layout` component aan.

```jsx
// src/pages/Layout.jsx
import { Outlet, ScrollRestoration } from 'react-router'; // 👈
import Navbar from '../components/Navbar';

export default function Layout() {
  return (
    <div className='container-xl'>
      <Navbar />
      <div className='p-4'>
        <Outlet />
      </div>
      <ScrollRestoration /> {/* 👈 */}
    </div>
  );
}
```

## Navigeren vanuit code

Soms wil je navigeren vanuit code, daarvoor bestaat de `useNavigate` hook. Deze hook geeft een functie terug met o.a. de URL waarnaar genavigeerd wordt als parameter. Meer informatie staat uiteraard in de [useNavigate documentatie](https://reactrouter.com/6.26.0/hooks/use-navigate). Je kan bijvoorbeeld ook vragen om de huidige URL te vervangen zodat deze verdwijnt uit de "terugkeer-geschiedenis" van de browser.

Als voorbeeld gaan we onderaan de NotFound pagina een knop zetten waarmee we terug naar de home-pagina kunnen. Dit doen we door volgende code toe te voegen aan `NotFound.jsx`:

```jsx
// src/pages/NotFound.jsx
import { useLocation, useNavigate } from 'react-router'; // 👈

const NotFound = () => {
  const navigate = useNavigate(); // 👈
  const { pathname } = useLocation();

  // 👇
  const handleGoHome = () => {
    navigate('/', { replace: true });
  };

  return (
    <div>
      <h1>Pagina niet gevonden</h1>
      <p>Er is geen pagina met als url {pathname}, probeer iets anders.</p>
      {/* 👇 */}
      <button className='py-2 px-2.5 rounded-md text-blue-600
      border border-blue-600 mt-4' onClick={handleGoHome}>Go home!</button>
    </div>
  );
};

export default NotFound;
```

Hiermee maken we een knop met een `onClick` handler. Deze functie zal via React Router terug naar de home-pagina navigeren en de huidige URL hierdoor vervangen.

Hetzelfde kan je bekomen met de Link tag, attribuut `replace` plaats je op true.

```jsx
<Link to='/' replace className='py-2 px-2.5 rounded-md text-blue-600 border border-blue-600 mt-4'>
  Go home!
</Link>
```

?> Het is aangeraden om zoveel mogelijk gebruik te maken van de `Link` component. Dit zorgt ervoor dat de gebruiker meer controle heeft over links, zoals het openen in een nieuw tabblad.

## Custom styles

Bij elke h1-tag dienen we dezelfde styling toe te passen. Je kan custom styles definiëren in de `index.css`:

```css
@layer base {
  h1 {
    font-size: var(--text-4xl);
    margin-bottom: 4px;
  }
}
```

Hierdoor zal elke `h1` tag automatisch de juiste styling krijgen.

Zorg ervoor dat je in `main.jsx` refereert naar de CSS:

```jsx
import './index.css';
```

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3-opl 5a31e56
> pnpm install
> pnpm dev
> ```

## Oefening 2 - Je eigen project

Denk voor je eigen applicatie na over de navigatie en implementeer.

## Mogelijke extra's voor de examenopdracht

- Gebruik de nieuwe [loader](https://reactrouter.com/en/main/route/loader) en [action](https://reactrouter.com/en/main/route/action) props van de `Route` component van `react-router` om de data op te halen.
  - Dit is een vrij kleine extra, dus zorg ervoor dat je nog een andere extra toevoegt.
