# React Router

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-Web/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les4 bbc6743
> yarn install
> yarn dev
> ```

Zoals je al weet, maken we met React een Single Page Application (SPA). Daardoor bestaat onze applicatie uit slechts één `index.html`. In deze HTML-pagina worden alle door Vite gegenereerde scripts en stylesheets geïnjecteerd.

Wanneer een client een React-applicatie opent (a.k.a. naar de URL ervan surft), wordt die ene `index.html` gedownload en gebeuren alle andere acties client-side. Het probleem is dat deze `index.html` enkel gedownload wordt indien we naar de `/` navigeren. Er zijn verder geen statische of server-side gegenereerde pagina's. Bijgevolg kunnen we enkel naar `/` navigeren, de browser downloadt dan `index.html` by default.

Daarnaast is React een library en geen framework, zoals Angular dat wel is. Dit zorgt ervoor dat er dus geen ingebouwde router beschikbaar is. Daarvoor biedt [React Router](https://reactrouter.com/) een oplossing.

!> Merk op: wij gebruiken hier [React Router versie 6.4](https://reactrouter.com/en/main/start/overview). Andere versies hebben mogelijks andere implementaties/APIs.

## React Router

React Router wordt aangeboden via de npm repository en biedt routing aan voor zowel React als React Native.

We hebben volgende dependencies nodig om met React Router aan de slag te kunnen (in React):

- [react-router](https://www.npmjs.com/package/react-router): de core van React Router (gedeeld met React Native)
- [react-router-dom](https://www.npmjs.com/package/react-router-dom): implementaties specifiek voor routing in webapplicaties

> Let op: react-router wordt niet automatisch geïnstalleerd als je react-router-dom installeert!

`react-router` is nodig voor de werking, maar tijdens implementatie zal je enkel componenten en hooks importeren uit `react-router-dom`.

### Achter de schermen

De werking van React Router is vrij intuïtief:

1. Vang een URL-wijziging op (bv. in de adresbalk of programmatisch gewijzigd).
2. Kijk of de URL gekend is:
   1. Indien gekend: toon de juiste pagina (lees: component).
   2. Indien niet gekend: stop (toont normaal niets maar je kan ook een eigen 404-pagina maken).

Deze werking is wel overmatig versimpeld, maar het geeft toch een idee...

## Voorbeeldapplicatie

Voor dit hoofdstuk maken we een simpele voorbeeldapplicatie, los van onze budgetapplicatie. Zo kunnen we alle aspecten van React Router tonen zonder gefoefel. Het is hét klassieke voorbeeld met volgende pagina's:

- Home (`/`)
- Over ons (`/over`)
- Contact (`/contact`)
- 404 Not Found (alle andere)

Als eerste maak je een nieuw project:

```bash
yarn create vite routing-demo --template react
```

Installeer daarin React Router

```bash
yarn add react-router-dom react-router
```

En start de applicatie (er gebeurt nog niets fancy, hoor):

```bash
yarn dev
```

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
import React from 'react';
import ReactDOM from 'react-dom/client';
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

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} /> {/* 👈 */}
  </React.StrictMode>
);
```

[createBrowserRouter](https://reactrouter.com/en/main/routers/create-browser-router) creëert een `RemixRouter` die zal functioneren als een `BrowserRouter`. Deze waarde moeten we doorgeven aan de [RouterProvider](https://reactrouter.com/en/main/routers/router-provider). De `BrowserRouter` gebruikt de DOM History API om een URL aan te passen en beheert de history stack. We geven een array met [Route](https://reactrouter.com/en/main/route/route) objecten mee. Deze koppelen een URL (`path`) aan een component (`element`).

Alle routes van de applicatie moeten doorgegeven worden aan de `RouterProvider`, deze moet dus de root-component zijn van de applicatie.

## Routes definiëren

Alvorens we routes kunnen definiëren, hebben we eerst een aantal componenten nodig. Omdat we te lui zijn om deze zelf te vullen met tekst, gaan we gebruik maken van `react-lorem-ipsum`.

Installeer dit package:

```bash
yarn add react-lorem-ipsum
```

Vervolgens maak je een bestand `src/pages.jsx` met de componenten `Home`, `About`, `Contact` en `NotFound`:

```jsx
import { LoremIpsum } from 'react-lorem-ipsum';

export const Home = () => (
  <div>
    <h1>Welkom!</h1>
    <p>Hier is nog niet zoveel te zien.</p>
  </div>
);

export const About = () => (
  <div>
    <h1>Over ons</h1>
    <LoremIpsum p={2} />
  </div>
);

export const Contact = () => (
  <div>
    <h1>Contact</h1>
    <LoremIpsum p={2} />
  </div>
);

export const NotFound = () => {
  return (
    <div>
      <h1>Pagina niet gevonden</h1>
      <p>Er is geen pagina met op deze url, probeer iets anders.</p>
    </div>
  );
};
```

Nu we de nodige componenten hebben, hoeven we enkel nog de routes te configureren. Hiervoor gaan we naar de `main.jsx` en voegen de extra routes toe.

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { NotFound, About, Contact, Home } from './pages.jsx'; // 👈 1

const router = createBrowserRouter([
  {
    path: '/',
    element: <Home />, // 👈 3
  },
  { path: 'over', element: <About /> }, // 👈 2
  { path: 'contact', element: <Contact /> }, // 👈 2
  { path: '*', element: <NotFound /> }, // 👈 4
]);

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

1. Importeer de gewenste componenten.
2. Vul de array met route objecten aan, één voor elke route. We geven telkens de component die getoond moet worden mee aan de optie `element`. Wanneer de URL in de browser wijzigt, zal de `RouterProvider` doorheen zijn routes zoeken naar een geschikte match. Een route definiëren we door gebruik te maken van het `Route` object.
3. Vervang de `App` component door de `Home` component. Je mag de `App` component verwijderen.
4. Dit zorgt ervoor dat de `NotFound` component getoond wordt indien de gebruiker op een URL uitkomt die niet bestaat. **Test dit zelf eens uit!**
   - Deze route hoeft niet als laatste staan. Waarom? React Router zoekt de meest exacte match en `*` is veel te algemeen.

Uit de route voor de `NotFound` component blijkt dat je ook reguliere expressies kan meegeven aan de `path` optie.

![How to regex](./images/how-to-regex.jpg ':size=50%')

## Navigeren tussen pagina's

Om te navigeren tussen pagina's kunnen we gebruik maken van de `Link` component. We voegen enkele links toe aan de `Home` component:

```jsx
// src/pages.jsx
import { LoremIpsum } from 'react-lorem-ipsum';
import { Link } from 'react-router-dom'; // 👈

export const Home = () => (
  <div>
    <h1>Welkom!</h1>
    <p>Kies één van de volgende links:</p>
    <ul>
      <li>
        <Link to='/over'>Over ons</Link> {/* 👈 */}
      </li>
      <li>
        <Link to='/contact'>Contact</Link> {/* 👈 */}
      </li>
    </ul>
  </div>
);
```

Je geeft de URL waarnaar genavigeerd moet worden mee aan de `to` prop. Achter de schermen wordt een anchor-element uit HTML gebruikt. De tekst tussen de `Link` tags wordt getoond als tekst voor de link.

## Route properties

Om eigenschappen over de huidige route op te vragen bestaat de hook `useLocation`. Pas de `NotFound` component aan:

```jsx
// src/pages.jsx
import { Link, useLocation } from 'react-router-dom'; // 👈
//..
export const NotFound = () => {
  const { pathname } = useLocation(); // 👈

  return (
    <div>
      <h1>Pagina niet gevonden</h1>
      <p>Er is geen pagina met als url {pathname}, probeer iets anders.</p> {/* 👈 */}
    </div>
  );
};
```

Deze hook retourneert nog diverse keys, **lees hierover volgende documentatie:**

- [useLocation](https://reactrouter.com/en/main/hooks/use-location)
- [Location interface van history package](https://github.com/remix-run/history/blob/main/docs/api-reference.md#location)

![How to use docs](./images/how-to-docs.jpg ':size=50%')

## Routes nesten

Stel we willen nog drie extra routes die starten met `/over`: `/over/services`, `/over/history` en `/over/location`. We voegen enkele links toe aan onze `About` component:

```jsx
// src/pages.jsx
export const About = () => (
  <div>
    <h1>Over ons</h1>
    <LoremIpsum p={2} />

    <ul>
      <li>
        <Link to='/over/services'>Onze diensten</Link> {/* 👈 */}
      </li>
      <li>
        <Link to='/over/history'>Geschiedenis</Link> {/* 👈 */}
      </li>
      <li>
        <Link to='/over/location'>Locatie</Link> {/* 👈 */}
      </li>
    </ul>
  </div>
);
```

En we voorzien deze pagina's in `pages.jsx`.

```jsx
export const Services = () => (
  <div>
    <h1>Our services</h1>
    <LoremIpsum p={2} />
  </div>
);

export const History = () => (
  <div>
    <h1>History</h1>
    <LoremIpsum p={2} />
  </div>
);

export const Location = () => (
  <div>
    <h1>Location</h1>
    <LoremIpsum p={2} />
  </div>
);
```

Daarna passen we de definitie van `/over` aan, de drie nieuwe routes dienen als kind van de `/over` route te worden aangemaakt (vergeet de nodige imports niet):

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import {
  NotFound,
  About,
  Contact,
  Home,
  Services,
  History,
  Location,
} from './pages.jsx'; // 👈

const router = createBrowserRouter([
  {
    path: '/',
    element: <Home />,
    errorElement: <NotFound />,
  },
  {
    path: 'over',
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
  { path: 'contact', element: <Contact /> },
]);

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

Nu willen we de subroutes `/over/services`, `/over/history` en `/over/location` tonen op de `About` component. Met andere woorden `About` moet altijd getoond worden met `Services`, `History` of `Location` onderaan deze component. Hiervoor bestaat de `Outlet` component van React Router ([bekijk de documentatie](https://reactrouter.com/en/main/components/outlet)).

Voeg onderaan de `About` component de `Outlet` toe:

```jsx
// src/pages.jsx
import { Outlet } from 'react-router-dom'; // 👈

export const About = () => (
  <div>
    {/* de rest */}
    <Outlet /> {/* 👈 */}
  </div>
);
```

## Redirects

Stel we willen dat gebruikers die naar `/services` navigeren naar `/over/services` doorgestuurd worden. Daarvoor voeg je volgende route toe aan de `main.jsx`:

```jsx
{
  path: 'services',
  element: <Navigate to='/over/services' replace />
}
```

Deze route rendert de `Navigate` component wanneer de gebruiker naar `/services` navigeert. Deze component is onderdeel van React Router en zal naar de URL in de `to` prop navigeren. De `replace` prop zorgt ervoor dat de URL `/services` vervangen wordt en bijgevolg verwijderd wordt uit de geschiedenis. Daarom kunnen we dus niet meer terugkeren naar `/services`, gebruikmakend van de terugknop van de browser.

## URL parameters

In sommige gevallen wil je ook stukken in de URL kunnen invullen met bv. een id van een entiteit. Om dit te demonstreren maken we een component die een lijst van producten toont. Voeg volgende code toe aan `src/pages.jsx`:

```jsx
// src/pages.jsx
const products = [
  {
    id: 1,
    name: 'Confituur',
    price: 2.5,
  },
  {
    id: 2,
    name: 'Choco',
    price: 3.5,
  },
  {
    id: 3,
    name: 'Coco-cola',
    price: 3.2,
  },
  {
    id: 4,
    name: 'Fanta',
    price: 3.0,
  },
  {
    id: 5,
    name: 'Sprite',
    price: 2.9,
  },
];

export const Products = () => (
  <div>
    <ul>
      {products.map(({ id, name }) => (
        <li key={id}>{name}</li>
      ))}
    </ul>
  </div>
);
```

Vervolgens definiëren we twee nieuwe routes in `main.jsx`:

```jsx
{
  path: '/products',
  children: [
    {
      index: true,
      element: <Products />,
    },
    {
      path: ':id',
      element: <Product />,
    },
  ],
}
```

De eerste route is een index-route die enkel getoond wordt op `/products`. Bij de eerste route valt op dat we een `index` optie meegeven. Zonder deze optie kunnen we niet navigeren naar `/`, wat de zogenaamde index-pagina is. Per definitie deelt een route met `index` hetzelfde pad als zijn ouder-route. De tweede route zal getoond worden wanneer een id (eigenlijk eender welke string) opgegeven wordt na `/products`, bv. `/products/1` maar ook `/products/foo`.

Om dit id op te halen uit de URL maken we gebruik van de `useParams` hook. Deze hook retourneert een object van key/value pairs met alle URL parameters met hun waarde uit de huidige URL. Deze hook zal een nieuw object retourneren telkens wanneer een URL parameter wijzigt.

### Voorbeeld

Stel we hebben volgende routes gedefinieerd:

```jsx
{
  path: '/products/:id',
  element: <Product />
},
{
  path: '/posts/:year/:month',
  element: <Posts />
}
```

Wanneer we navigeren naar `/products/263`, dan zal de `Product` component getoond worden. Zijn `useParams` zal volgend object retourneren:

```js
{
  id: '263'
}
```

Wanneer we navigeren naar `/posts/2021/1`, dan zal de `Posts` component getoond worden. Zijn `useParams` zal volgend object retourneren:

```js
{
  year: '2021',
  month: '1'
}
```

> Merk op dat elke value een string is en niet een number in dit geval. We zullen de string dus zelf nog moeten parsen via de Number-functie.

### Details van een product

We moeten nog enkel de `Product` component implementeren zodat we de details van een product kunnen tonen. Maak een nieuwe component in `src/pages.jsx`:

```jsx
// src/pages.jsx
import { Link, Outlet, useParams, useLocation } from 'react-router-dom';

export const Product = () => {
  const { id } = useParams();
  const idAsNumber = Number(id);

  const product = products.find((p) => p.id === idAsNumber);

  if (!product) {
    return (
      <div>
        <h1>Product niet gevonden</h1>
        <p>Er is geen product met id {id}.</p>
      </div>
    );
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>
        <b>Price:</b> &euro; {product.price}
      </p>
    </div>
  );
};
```

Deze component zal eerst het id uit de URL ophalen en omvormen naar een `number`. Daarna zoekt het een product met het opgegeven id. Indien dit product niet bestaat, zal een gepaste boodschap getoond worden. In het andere geval wordt de product-informatie getoond.

### Oefening 1 - Navigeren naar een product

Zorg ervoor dat je op de `Products` pagina kan navigeren naar de detailpagina van een product door te klikken op de naam van een product.

## De Root component

Nu willen we een navigatiebalk toevoegen aan de website (we houden het heel eenvoudig). Deze navigatiebalk komt op elke pagina voor. Om globale layout voor de app toe te voegen maak je een `Root` component aan in de `src` map. Deze bevat de navigatiebalk en de `Outlet` component voor de weergave van de kind-routes.

```jsx
import { Outlet, Link } from 'react-router-dom';

export default function Root() {
  return (
    <div>
      <nav>
        <ul>
          <li>
            <Link to='/'>Home</Link> {/* 👈 */}
          </li>
          <li>
            <Link to='/over'>Over ons</Link> {/* 👈 */}
          </li>
          <li>
            <Link to='/contact'>Contact</Link> {/* 👈 */}
          </li>
          <li>
            <Link to='/products'>Products</Link> {/* 👈 */}
          </li>
        </ul>
      </nav>

      <Outlet /> {/* 👈 */}
    </div>
  );
}
```

Pas `main.jsx` aan, alle paden zijn nu kinderen van de `Root` component:

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    children: [
      {
        index: true,
        element: <Home />,
      },
      {
        path: '/',
        element: <Home />,
      },
      {
        path: 'over',
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
      { path: 'contact', element: <Contact /> },
      { path: 'services', element: <Navigate to='/over/services' replace /> },
      {
        path: '/products',
        children: [
          {
            index: true,
            element: <Products />,
          },
          {
            path: ':id',
            element: <Product />,
          },
        ],
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);
```

Voeg onderstaande css toe in `index.css`, zo ziet de navigatiebalk er al iets beter uit:

```css
nav ul {
  list-style-type: none;
  display: flex;
  background-color: rgb(16, 167, 218);
  margin: 0;
  padding: 5px;
}

nav ul li a {
  color: white;
  text-align: center;
  padding: 16px;
  text-decoration: none;
}

nav li {
  font-size: 16px;
}

nav ul li a:hover {
  background-color: darkblue;
}
```

## Scroll restoration

Bij routing in SPA's wordt de scroll-positie niet automatisch hersteld naar linksboven in de browser. Indien gewenst, moet je hier zelf voor zorgen. Maak hiervoor gebruik van de `ScrollRestoration` component. Elke keer als de URL wijzigt, vraagt deze de browser om naar boven te scrollen. Pas hiervoor de `Root` component aan.

```jsx
import { Outlet, Link, ScrollRestoration } from 'react-router-dom'; // 👈

export default function Root() {
  return (
    <div>
      <nav>
        <ul>
          <li>
            <Link to='/'>Home</Link>
          </li>
          <li>
            <Link to='/over'>Over ons</Link>
          </li>
          <li>
            <Link to='/contact'>Contact</Link>
          </li>
          <li>
            <Link to='/products'>Products</Link>
          </li>
        </ul>
      </nav>

      <Outlet />
      <ScrollRestoration /> {/* 👈 */}
    </div>
  );
}
```

## Navigeren vanuit code

Soms wil je navigeren vanuit code, daarvoor bestaat de `useNavigate` hook. Deze hook geeft een functie terug met o.a. de URL waarnaar genavigeerd wordt als parameter. Meer informatie staat uiteraard in de [useNavigate documentatie](https://reactrouter.com/docs/en/v6/hooks/use-navigate). Je kan bijvoorbeeld ook vragen om de huidige URL te vervangen zodat deze verdwijnt uit de "terugkeer-geschiedenis" van de browser.

Als voorbeeld gaan we onderaan elke pagina een knop zetten waarmee we terug naar de home-pagina kunnen. Dit doen we door volgende code toe te voegen aan `main.jsx`:

```jsx
import { Outlet, Link, useNavigate, ScrollRestoration } from 'react-router-dom'; // 👈

export default function Root() {
  const navigate = useNavigate(); // 👈

  // 👇
  const handleGoHome = () => {
    navigate('/', { replace: true });
  };

  return (
    <div>
      <nav>
        <ul>
          <li>
            <Link to='/'>Home</Link>
          </li>
          <li>
            <Link to='/over'>Over ons</Link>
          </li>
          <li>
            <Link to='/contact'>Contact</Link>
          </li>
          <li>
            <Link to='/products'>Products</Link>
          </li>
        </ul>
      </nav>

      <Outlet />
      <button onClick={handleGoHome}>Go home!</button> {/* 👈 */}
      <ScrollRestoration />
    </div>
  );
}
```

Hiermee maken we een knop met een `onClick` handler. Deze functie zal via React Router terug naar de home-pagina navigeren en de huidige URL hierdoor vervangen.

## Aanduiden van de actieve link in de navigatie

Maak hiervoor gebruik van de `NavLink` component. De actieve link maak je op in css met de `active` class. Pas de `Root` component aan:

```jsx
import {
  Outlet,
  NavLink, // 👈
  useNavigate,
  ScrollRestoration,
} from 'react-router-dom';

export default function Root() {
  const navigate = useNavigate();

  const handleGoHome = () => {
    navigate('/', { replace: true });
  };

  return (
    <div>
      <nav>
        <ul>
          <li>
            <NavLink to='/'>Home</NavLink> {/* 👈 */}
          </li>
          <li>
            <NavLink to='/over'>Over ons</NavLink> {/* 👈 */}
          </li>
          <li>
            <NavLink to='/contact'>Contact</NavLink> {/* 👈 */}
          </li>
          <li>
            <NavLink to='/products'>Products</NavLink> {/* 👈 */}
          </li>
        </ul>
      </nav>
      <Outlet />
      <button onClick={handleGoHome}>Go home!</button>
      <ScrollRestoration />
    </div>
  );
}
```

Voeg onderstaande code toe aan de `index.css`:

```css
nav a.active {
  color: darkblue;
}
```

## Oplossing mini-voorbeeldapplicatie

De oplossing van deze mini-voorbeeldapplicatie is te vinden op <https://github.com/HOGENT-Web/RoutingDemo>.

### Oefeningen

### Oefening 2 - Basis routing

Voorzie routing in de budget-applicatie (of in je eigen app?). Check uit op commit `bbc6743` en maak volgende routes:

- `/`: doorsturen naar `/transactions`
- `/transactions`: een lijst van transacties
- `/places`: een lijst van places
- `/transactions/add`: een nieuwe transactie toevoegen

Maak voor elke route een nieuwe component aan die de pagina voorstelt. Deze component haalt o.a. de nodige data op uit de REST API en voorziet triggers om gegevens door te sturen naar de REST API. Je kan deze componenten in een nieuwe map `src/pages` plaatsen, met een submap `src/pages/transactions` en `src/pages/places`.

Om eenvoudig te testen voeg je best een navbar toe, gebruik hiervoor [bootstrap](https://getbootstrap.com/docs/5.2/components/navbar/).

### Oefening 3 - URL parameters

Voorzie volgende bijkomende routes in de budget-applicatie (of in je eigen app?):

- `/transactions/edit/:id`: een transactie bewerken
- `/transactions/:id`: details van één transactie
- `/places/:id`: details van één place

Let op er zijn aanpassingen nodig aan de `TransactionForm` component om de huidige transactie te kunnen ophalen en te tonen in het formulier.

Maak voor elke route een nieuwe component aan die de pagina voorstelt. Deze component haalt o.a. de nodige data op uit de REST API en voorziet triggers om gegevens door te sturen naar de REST API. Je kan deze componenten in een nieuwe map `src/pages` plaatsen, met een submap `src/pages/transactions` en `src/pages/places`.

### Oplossing oefeningen

Een voorbeeldoplossing is te vinden op <https://github.com/hogent-web/frontendweb-budget> in commit `50e2881`:

```bash
git clone https://github.com/hogent-web/frontendweb-budget.git
git checkout -b oplossing-les5 50e2881
yarn install
yarn start
```
