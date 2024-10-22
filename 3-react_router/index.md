# React Router

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3 2e1eb31
> yarn install
> yarn dev
> ```

Zoals je al weet, maken we met React een Single Page Application (SPA). Daardoor bestaat onze applicatie uit slechts één `index.html`. In deze HTML-pagina worden alle door Vite gegenereerde scripts en stylesheets geïnjecteerd.

Wanneer een client een React-applicatie opent (a.k.a. naar de URL ervan surft), wordt die ene `index.html` gedownload en gebeuren alle andere acties client-side. Het probleem is dat deze `index.html` enkel gedownload wordt indien we naar de `/` navigeren. Er zijn verder geen statische of server-side gegenereerde pagina's. Bijgevolg kunnen we enkel naar `/` navigeren, de browser downloadt vervolgens standaard `index.html`.

Daarnaast is React een library en geen framework, zoals bv. Angular dat wel is. Dit zorgt ervoor dat er dus geen ingebouwde router beschikbaar is. Daarvoor biedt [React Router](https://reactrouter.com/) een oplossing.

!> Merk op: wij gebruiken hier [React Router versie 6.26](https://reactrouter.com/en/main/start/overview). Andere versies hebben mogelijks andere implementaties/APIs.

## React Router

React Router wordt aangeboden via de npm repository en biedt routing aan voor zowel React als React Native.

We hebben volgende dependencies nodig om met React Router aan de slag te kunnen (in React):

- [react-router](https://www.npmjs.com/package/react-router): de core van React Router (gedeeld met React Native)
- [react-router-dom](https://www.npmjs.com/package/react-router-dom): implementaties specifiek voor routing in webapplicaties

> Let op: `react-router` wordt niet automatisch geïnstalleerd als je `react-router-dom` installeert!

`react-router` is nodig voor de werking, maar tijdens implementatie zal je enkel componenten en hooks importeren uit `react-router-dom`.

Installeer beide dependencies

```bash
yarn add react-router@~6.26.0 react-router-dom@~6.26.0
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

De [`BrowserRouter`](https://reactrouter.com/en/main/routers/create-browser-router) is een van de mogelijkheden in `react-router-dom` om te gebruiken als router. Dit type router zal functioneren zoals je verwacht dat een router functioneert: hij gebruikt het deel na de `/` om naar een pagina te navigeren. Dit is zeer gelijkaardig aan hoe server-side gerenderde pagina's werken.

Een probleem hierbij is dat browsers standaard refreshen wanneer de URL na de `/` wijzigt. In dit geval zal `react-router-dom` dit probleem opvangen en voorkomen.

Het voordeel met dit soort routers is dat je webapplicatie werkt zoals een _old-school website_, met alle features die een URL te bieden heeft.

### HashRouter

Een tweede type router is de [`HashRouter`](https://reactrouter.com/en/main/routers/create-hash-router). Deze router gebruikt de hash (of dus de `#`) in de URL om te navigeren tussen pagina's.

Dit heeft als voordeel dat de browser by default niet zal refreshen. Het nadeel is dat je de hash niet meer kan gebruiken om te scrollen naar een element (met een bepaald id) in de pagina.

### Welke kies je?

`HashRouter` wordt typisch weinig gebruikt, we raden aan om `BrowserRouter` te gebruiken. Let hierbij op dat je de routers uit de data API kiest, dus via de functies [`createBrowserRouter`](https://reactrouter.com/en/main/routers/create-browser-router) en [`createHashRouter`](https://reactrouter.com/en/main/routers/create-hash-router) en niet via de gelijknamige componenten.

Naast deze twee types zijn er nog andere beschikbaar, [lees meer in de documentatie](https://reactrouter.com/en/main/routers/picking-a-router).

### In de voorbeeldapplicatie

De voorbeeldapplicatie zal gebruik maken van een `BrowserRouter`. We dienen eerst een router toe te voegen aan de app. We voegen hiervoor een [Browser Router](https://reactrouter.com/en/main/routers/create-browser-router) toe en configureren onze eerste route. We doen dit in `main.jsx`, het startpunt van de app:

```jsx
// src/main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';
import { createBrowserRouter, RouterProvider } from 'react-router-dom'; // 👈

// 👇
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
]);

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
    {/* 👈 */}
  </StrictMode>,
);
```

[createBrowserRouter](https://reactrouter.com/en/main/routers/create-browser-router) creëert een `RemixRouter` die zal functioneren als een `BrowserRouter`. De `BrowserRouter` gebruikt de DOM History API om een URL aan te passen en beheert de history stack. We geven een array met [Route](https://reactrouter.com/en/main/route/route) objecten mee. Deze koppelen een URL (`path`) aan een component (`element`). De `router` moeten we doorgeven aan de [RouterProvider](https://reactrouter.com/en/main/routers/router-provider).

In dit voorbeeld configureren we een enkele route die de `App` component toont wanneer de URL `/` is. We zullen later meer routes toevoegen.

Alle routes van de applicatie moeten doorgegeven worden aan de `RouterProvider`, deze moet dus de root-component zijn van de applicatie.

## Routes definiëren

We voorzien volgende basis routes in de voorbeeldapplicatie

- `/`: de home page (`App.jsx`) met links naar de andere pagina's (later voegen we een navigatiebalk toe)
- `/transactions`: een lijst van transacties
- `/places`: een lijst van places
- `/about`: over ons pagina

Alvorens we routes kunnen definiëren, voeren we een kleine refactoring uit. De verschillende pagina's in onze applicatie plaatsen we in de `pages` map. Maak een map `pages` aan met daarin de mappen `places` en `transactions`. Verplaats de componenten `PlacesList` en `TransactionList` naar de juiste map. Pas eventueel de paden in de component aan.

Voeg ook een `About` en `NotFound` pagina toe. Omdat we te lui zijn om deze zelf te vullen met tekst, gaan we gebruik maken van `react-lorem-ipsum`.

Installeer dit package:

```bash
yarn add react-lorem-ipsum
```

Maak de `About` page aan:

```jsx
// src/pages/about/About.jsx
import { LoremIpsum } from 'react-lorem-ipsum';

const About = () => (
  <div>
    <h1>Over ons</h1>
    <div>
      <LoremIpsum p={2} />
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
      <h1>Pagina niet gevonden</h1>
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
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import TransactionList from './pages/transactions/TransactionList'; // 👈 1
import PlacesList from './pages/places/PlacesList'; // 👈 1
import NotFound from './pages/NotFound'; // 👈 1
import About from './pages/about/About.jsx'; // 👈 1

// 👇
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
  { path: 'transactions', element: <TransactionList /> }, // 👈 2
  { path: 'places', element: <PlacesList /> }, // 👈 2
  { path: 'about', element: <About /> }, // 👈 2
  { path: '*', element: <NotFound /> }, // 👈 3
]);

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

1. Importeer de gewenste componenten.
2. Vul de array met route objecten aan, één voor elke route. We geven telkens de component die getoond moet worden mee aan de optie `element`. Wanneer de URL in de browser wijzigt, zal de `RouterProvider` doorheen zijn routes zoeken naar een geschikte match. Een route definiëren we door gebruik te maken van het `Route` object.
3. Dit zorgt ervoor dat de `NotFound` component getoond wordt indien de gebruiker op een URL uitkomt die niet bestaat. **Test dit zelf eens uit!**
   - Deze route hoeft niet als laatste staan. Waarom? React Router zoekt de meest exacte match en `*` is veel te algemeen.

Uit de route voor de `NotFound` component blijkt dat je ook reguliere expressies kan meegeven aan de `path` optie.

![How to regex](./images/how-to-regex.jpg ':size=50%')

## Navigeren tussen pagina's

Om te navigeren tussen pagina's kunnen we gebruik maken van de `Link` component. We voegen enkele links toe aan de `App` component:

```jsx
// src/App.jsx
import { Link } from 'react-router-dom'; // 👈

function App() {
  return (
    <div>
      <h1>Welkom!</h1>
      <p>Kies één van de volgende links:</p>
      <ul>
        <li>
          <Link to='/transactions'>Transacties</Link> {/* 👈 */}
        </li>
        <li>
          <Link to='/places'>Plaatsen</Link> {/* 👈 */}
        </li>
        <li>
          <Link to='/about'>Over ons</Link> {/* 👈 */}
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
import { useLocation } from 'react-router-dom'; // 👈

const NotFound = () => {
  const { pathname } = useLocation(); // 👈

  return (
    <div>
      <h1>Pagina niet gevonden</h1>
      <p>Er is geen pagina met als url {pathname}, probeer iets anders.</p> {/* 👈 */}
    </div>
  );
};
export default NotFound;
```

Deze hook retourneert nog diverse keys, **lees hierover volgende documentatie:**

- [useLocation](https://reactrouter.com/en/main/hooks/use-location)
- [Location interface van history package](https://github.com/remix-run/history/blob/main/docs/api-reference.md#location)

![How to use docs](./images/how-to-docs.jpg ':size=50%')

## Routes nesten

Je kan [geneste routes](https://reactrouter.com/en/main/start/overview#nested-routes) creëren om complexe UI-structuren te ondersteunen, waarbij een component subcomponenten heeft die worden weergegeven op basis van de URL. We willen nog drie extra routes die starten met `/about`: `/about/services`, `/about/history` en `/about/location`. We voegen enkele links toe aan onze `About` component:

```jsx
// src/pages/about/About.jsx
import { LoremIpsum } from 'react-lorem-ipsum';
import { Link } from 'react-router-dom'; // 👈

const About = () => (
  <div>
    <h1>Over ons</h1>
    <div>
      <LoremIpsum p={2} />

      <ul>
        <li>
          <Link to='/about/services'>Onze diensten</Link> {/* 👈 */}
        </li>
        <li>
          <Link to='/about/history'>Geschiedenis</Link> {/* 👈 */}
        </li>
        <li>
          <Link to='/about/location'>Locatie</Link> {/* 👈 */}
        </li>
      </ul>
    </div>
  </div>
);

export default About;
```

En we voegen deze pagina's toe aan `About.jsx`.

```jsx
export const Services = () => (
  <div>
    <h1>Onze diensten</h1>
    <LoremIpsum p={2} />
  </div>
);

export const History = () => (
  <div>
    <h1>Geschiedenis</h1>
    <LoremIpsum p={2} />
  </div>
);

export const Location = () => (
  <div>
    <h1>Locatie</h1>
    <LoremIpsum p={2} />
  </div>
);
```

Daarna passen we de definitie van `/about` aan, de drie nieuwe routes dienen als kind van de `/about` route te worden aangemaakt (vergeet de nodige imports niet):

```jsx
// src/main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import TransactionList from './pages/transactions/TransactionList';
import PlacesList from './pages/places/PlacesList';
import NotFound from './pages/NotFound';
import About, { Services, History, Location } from './pages/about/About.jsx'; // 👈

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
  },
  { path: 'transactions', element: <TransactionList /> },
  { path: 'places', element: <PlacesList /> },
  {
    path: 'about',
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
import { Outlet, Link } from 'react-router-dom'; // 👈

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
import { Navigate } from 'react-router-dom';
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
  element: <PlaceDetail />
},
{
  path: '/posts/:year/:month',
  element: <Posts />
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
import { useParams } from 'react-router-dom';
import { PLACE_DATA } from '../../api/mock_data';

const PlaceDetail = () => {
  const { id } = useParams();
  const idAsNumber = Number(id);

  const place = PLACE_DATA.find((p) => p.id === idAsNumber);

  if (!place) {
    return (
      <div>
        <h1>Plaats niet gevonden</h1>
        <p>Er is geen plaats met id {id}.</p>
      </div>
    );
  }

  return (
    <div>
      <h1>{place.name}</h1>
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
  <h5 className='card-title'>
    <Link to={`/places/${id}`}>{name}</Link>
  </h5>
  ```

## De Layout component

Nu willen we een navigatiebalk toevoegen aan de website (we houden het heel eenvoudig). Deze navigatiebalk wordt getoond op elke pagina. Om globale layout voor de app toe te voegen maak je een `Layout` component aan in de `src/pages` map. Deze bevat de navigatiebalk en de `Outlet` component voor de weergave van de onderliggende routes.

```jsx
// src/pages/Layout.jsx
import { Outlet } from 'react-router-dom';
import Navbar from '../components/Navbar';

export default function Layout() {
  return (
    <div className='container-xl'>
      <Navbar />
      <Outlet />
    </div>
  );
}
```

De `Navbar` component voorziet in het menu.

```jsx
// src/components/Navbar.jsx
import { Link } from 'react-router-dom';

export default function Navbar() {
  return (
    <nav className='navbar sticky-top mb-4 navbar-light bg-light'>
      <div className='container-fluid flex-column flex-sm-row align-items-start align-items-sm-center'>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <Link className='nav-link' to='/'>
            Transactions
          </Link>
        </div>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <Link className='nav-link' to='/places'>
            Places
          </Link>
        </div>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <Link className='nav-link' to='/about'>
            About us
          </Link>
        </div>
        <div className='flex-grow-1'></div>
      </div>
    </nav>
  );
}
```

Pas `main.jsx` aan, alle paden zijn nu kinderen van de `Layout` component en verwijder de `App`component

```jsx
const router = createBrowserRouter([
  {
    element: <Layout />, // 👈
    // 👇
    children: [
      {
        path: '/',
        element: <Navigate replace to='/transactions' />,
      },
      {
        path: '/transactions',
        element: <TransactionList />,
      },
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
        path: 'about',
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
        ],
      },
      {
        path: 'services',
        element: <Navigate to='/about/services' replace />,
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

In `main.jsx` kan je nu de `App` component verwijderen.

## Scroll restoration

Bij routing in SPA's wordt de scroll-positie niet automatisch hersteld naar linksboven in de browser. Indien gewenst, moet je hier zelf voor zorgen. Maak hiervoor gebruik van de `ScrollRestoration` component. Elke keer als de URL wijzigt, vraagt deze de browser om naar boven te scrollen. Pas hiervoor de `Layout` component aan.

```jsx
// src/pages/Layout.jsx
import { Outlet, ScrollRestoration } from 'react-router-dom'; // 👈
import Navbar from '../components/Navbar';

export default function Layout() {
  return (
    <div className='container-xl'>
      <Navbar />
      <Outlet />
      <ScrollRestoration /> {/* 👈 */}
    </div>
  );
}
```

## Navigeren vanuit code

Soms wil je navigeren vanuit code, daarvoor bestaat de `useNavigate` hook. Deze hook geeft een functie terug met o.a. de URL waarnaar genavigeerd wordt als parameter. Meer informatie staat uiteraard in de [useNavigate documentatie](https://reactrouter.com/en/6.26.0/hooks/use-navigate). Je kan bijvoorbeeld ook vragen om de huidige URL te vervangen zodat deze verdwijnt uit de "terugkeer-geschiedenis" van de browser.

Als voorbeeld gaan we onderaan de NotFound pagina een knop zetten waarmee we terug naar de home-pagina kunnen. Dit doen we door volgende code toe te voegen aan `NotFound.jsx`:

```jsx
// src/pages/NotFound.jsx
import { useLocation, useNavigate } from 'react-router-dom'; // 👈

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
      <button onClick={handleGoHome}>Go home!</button>
    </div>
  );
};

export default NotFound;
```

Hiermee maken we een knop met een `onClick` handler. Deze functie zal via React Router terug naar de home-pagina navigeren en de huidige URL hierdoor vervangen.

Hetzelfde kan je bekomen met de Link tag, attribuut `replace` plaats je op true.

```jsx
<Link to='/' replace className='alert-link'>
  Go home!
</Link>
```

?> Het is aangeraden om zoveel mogelijk gebruik te maken van de `Link` component. Dit zorgt ervoor dat de gebruiker meer controle heeft over links, zoals het openen in een nieuw tabblad.

## Aanduiden van de actieve link in de navigatie

Maak hiervoor gebruik van de `NavLink` component. De actieve link maak je op in CSS met de `active` class. Pas de `Navbar` component aan:

```jsx
// src/components/Navbar.jsx
import { NavLink } from 'react-router-dom';

export default function Navbar() {
  return (
    <nav className='navbar sticky-top mb-4 navbar-light bg-light'>
      <div className='container-fluid flex-column flex-sm-row align-items-start align-items-sm-center'>
        <div className='nav-item my-2 mx-sm-3 my-sm-0'>
          <NavLink className='nav-link' to='/transactions'>
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
            About us
          </NavLink>
        </div>
        <div className='flex-grow-1'></div>
      </div>
    </nav>
  );
}
```

Voeg onderstaande code toe aan de `index.css`:

```css
a.nav-link.active {
  color: blue;
}
```

Zorg ervoor dat je in `main.jsx` refereert naar de CSS:

```jsx
import './index.css';
```

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3-opl 75b3f6d
> yarn install
> yarn dev
> ```

## Oefening 2 - Je eigen project

Denk voor je eigen applicatie na over de navigatie en implementeer.

## Mogelijke extra's voor de examenopdracht

- Gebruik de nieuwe [loader](https://reactrouter.com/en/main/route/loader) en [action](https://reactrouter.com/en/main/route/action) props van de `Route` component van `react-router-dom` om de data op te halen.
  - Dit is een vrij kleine extra, dus zorg ervoor dat je nog een andere extra toevoegt.
