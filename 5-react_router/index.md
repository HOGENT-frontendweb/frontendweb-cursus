# React Router

Zoals je al weet, maken we met React een zogenaamde Single Page Application (SPA). Daardoor bestaat onze applicatie uit slechts één `index.html`. In deze HTML-pagina worden alle door webpack gegenereerde scripts en stylesheets geïnjecteerd.

Wanneer een client een React-applicatie opent (a.k.a. naar de URL ervan surft), wordt die ene `index.html` gedownload en gebeuren alle andere acties client-side. Het probleem is dat deze `index.html` enkel gedownload indien we naar de `/` navigeren. Er zijn verder geen statische of server-side gegenereerde pagina's. Bijgevolg kunnen we enkel naar `/` navigeren, de browser download dan `index.html` by default.

Daarnaast is React een library en geen framework, zoals Angular dat wel is. Dit zorgt ervoor dat er dus geen ingebouwde router beschikbaar is. Daarvoor biedt [React Router](https://reactrouter.com/) een oplossing.

> Merk op: wij gebruiken hier React Router versie 6 (andere versies hebben mogelijks andere implementaties/APIs)

## React Router

React Router wordt aangeboden via de npm repository en biedt een declaratieve routing voor React, dit wil zeggen routing o.b.v. componenten in JSX.

We hebben volgende dependencies nodig om met React Router aan de slag te kunnen (in React):

- [react-router](https://www.npmjs.com/package/react-router): de core van React Router (gedeeld met React Native)
- [react-router-dom](https://www.npmjs.com/package/react-router-dom): implementaties specifiek voor routing in webapplicaties

> Let op: react-router wordt niet automatisch geïnstalleerd als je react-router-dom installeert

`react-router` is nodig voor de werking, maar tijdens implementatie zal je enkel componenten en hooks importeren uit `react-router-dom`.

### Achter de schermen

De werking van React Router is vrij intuïtief:

1. vang een URL-wijziging op (bv. in de adresbalk of programmatisch gewijzigd)
2. kijk of de URL gekend is:
    1. indien gekend: toon de juiste pagina (lees: component)
    2. indien niet gekend: stop (toont normaal niets maar je kan ook een eigen 404-pagina maken)

Deze werking is wel overmatig versimpeld, maar het geeft toch een idee...

## Voorbeeldapplicatie

Voor dit hoofdstuk maken we een simpele voorbeeldapplicatie, los van onze budgetapplicatie. Zo kunnen we alle aspecten van React Router tonen zonder gefoefel. Het is hét klassieke voorbeeld met volgende pagina's:

- Home (`/`)
- Over ons (`/over`)
- Contact (`/contact`)
- 404 Not Found (alle andere)

Als eerste maak je een nieuw project:

```bash
npx create-react-app routing-demo
```

Installeer daarin React Router

```bash
yarn add react-router-dom@6 react-router@6
```

En start de applicatie (er gebeurt nog niets fancy, hoor):

```bash
yarn start
```

## BrowserRouter vs HashRouter

### BrowserRouter

De `BrowserRouter` is een van de twee mogelijkheden in `react-router-dom` om te gebruiken als router. Dit type router zal functioneren zoals je verwacht dat een router funtioneert: hij gebruikt het deel na de `/` om naar een pagina te navigeren. Dit is zeer gelijkaardig aan hoe server-side gerenderde pagina's werken.

Een probleem hierbij is dat browsers standaard refreshen wanneer de URL na de `/` wijzigt. In dit geval zal `react-router-dom` dit probleem opvangen en voorkomen.

Het voordeel met dit soort routers is dat je webapplicatie werkt zoals een _old-school website_, met alle features die een URL te bieden heeft.

### HashRouter

Het tweede type routes is de `HashRouter`. Deze router gebruikt de hash (of dus de `#`) in de url om te navigeren tussen pagina's.

Dit heeft als voordeel dat de browser by default niet zal refreshen. Het nadeel is dat je de hash niet meer kan gebruiken om te scrollen naar een element in de pagina.

Dit soort routers wordt typisch weinig gebruikt, we raden aan om `BrowserRouter` te gebruiken.

### In de voorbeeldapplicatie

De voorbeeldapplicatie zal dus gebruik maken van een `BrowserRouter`. Je importeert deze component en wrapped je hele `App` component hiermee.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
// ...
import { BrowserRouter } from "react-router-dom"; // 👈

ReactDOM.render(
    <React.StrictMode>
        <BrowserRouter> // 👈
            <App />
        </BrowserRouter> // 👈
    </React.StrictMode>,
    document.getElementById('root')
);
```

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
      <p>
        Er is geen pagina met op deze url, probeer iets anders.
      </p>
    </div>
  );
};
```

Nu we de nodige componenten hebben, hoeven we enkel nog de routes te configureren. Hiervoor gaan we naar de `App` component, maken deze helemaal leeg en plaatsen een `Routes` component:

```jsx
import { Routes } from 'react-router-dom' // 👈

function App() {
  return (
    <Routes> // 👈

    </Routes> // 👈
  );
}

export default App;
```

URLs binnen de app worden gedefinieerd onder deze `Routes` component. Wanneer de URL in de browser wijzigt, zal deze component doorheen zijn routes zoeken naar een geschikte match. Een route definiëren we door gebruik te maken van de `Route` component:

```jsx
import { Routes, Route } from 'react-router-dom' // 👈
import { Home, About, Contact, NotFound } from './pages'; // 👈

function App() {
  return (
    <Routes>
      <Route index element={<Home />} /> // 👈
      <Route path="over" element={<About />} /> // 👈
      <Route path="contact" element={<Contact />} /> // 👈
      <Route path="*" element={<NotFound />} /> // 👈
    </Routes>
  );
}

export default App;
```

We geven telkens de component die getoond moet worden mee aan de prop `element`.

Bij de eerste route valt op dat we een `index` prop meegeven. Zonder deze prop kunnen we niet navigeren naar `/`, wat de zogenaamde index-pagina is. Je zou ook `path="/"` kunnen schrijven maar `index` is een best practice. Per definitie deelt een route met `index` hetzelfde pad als zijn ouder-route.

De laatste route is een beetje speciaal, deze matchet met eender welke URL (door `path="*"`). Dit zorgt ervoor dat de `NotFound` component getoond wordt indien de gebruiker op een URL uitkomt die niet bestaat. **Test zelf eens uit:** deze route moet niet als laatste staan. Waarom? React Router zoekt de meest exacte match en `*` is veel te algemeen.

Uit de route voor de `NotFound` component blijkt dus dat je ook reguliere expressies kan meegeven aan de `path` prop.

![How to regex](./images/how-to-regex.jpg ':size=50%')

## Navigeren tussen pagina's

Om te navigeren tussen pagina's kunnen we gebruik maken van de `Link` component. We voegen enkele links toe aan de `Home` component:

```jsx
// src/pages.jsx
export const Home = () => (
  <div>
    <h1>Welkom!</h1>
    <p>Kies één van de volgende links:</p>
    <ul>
      <li>
        <Link to="/over">Over ons</Link> // 👈
      </li>
      <li>
        <Link to="/contact">Contact</Link> // 👈
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
export const NotFound = () => {
  const { pathname } = useLocation(); // 👈

  return (
    <div>
      <h1>Pagina niet gevonden</h1>
      <p>Er is geen pagina met als url {pathname}, probeer iets anders.</p> // 👈
    </div>
  );
};
```

Deze hook retourneert nog diverse keys, **lees hierover volgende documentatie:**

- [useLocation](https://reactrouter.com/docs/en/v6/hooks/use-location)
- [Location interface van history package](https://github.com/remix-run/history/blob/main/docs/api-reference.md#location)

![How to use docs](./images/how-to-docs.jpg ':size=50%')

## Routes nesten

Stel we willen nog drie extra routes die starten met `/over`: `/over/services`, `/over/history` en `/over/location`. We voegen enkel links toe aan onze `About` component:

```jsx
// src/pages.jsx
export const About = () => (
  <div>
    <h1>Over ons</h1>
    <LoremIpsum p={2} />

    <ul>
      <li>
        <Link to="/over/services">Onze diensten</Link> // 👈
      </li>
      <li>
        <Link to="/over/history">Geschiedenis</Link> // 👈
      </li>
      <li>
        <Link to="/over/services">Locatie</Link> // 👈
      </li>
    </ul>
  </div>
);
```

Daarna passen we de definitie van `/over` aan (vergeet de nodige imports niet):

```jsx
<Route path="over">
  <Route index element={<About />} />
  <Route path="services" element={<Services />} />
  <Route path="history" element={<History />} />
  <Route path="location" element={<Location />} />
</Route>
```

Merk volgende wijzigingen op:

- er is geen `element` meer op de `Route`-component voor `/over`
- er is een nieuwe index-route (waarvan de component enkel getoond wordt op `/over`)
- er zijn drie nieuwe routes die allemaal onder `/over` vallen, React Router zal telkens voor elk van deze paden `/over` verwachten

Onze drie nieuwe component bevatten telkens weer wat lorem ipsum tekst:

```jsx
// src/pages.jsx
const Services = () => (
  <div>
    <h1>Our services</h1>
    <LoremIpsum p={2} />
  </div>
);

const History = () => (
  <div>
    <h1>History</h1>
    <LoremIpsum p={2} />
  </div>
);

const Location = () => (
  <div>
    <h1>Location</h1>
    <LoremIpsum p={2} />
  </div>
);
```

### Oefening

Stel we willen de subroutes `/over/services`, `/over/history` en `/over/location` tonen op de `About` component. Met andere woorden `About` moet altijd getoond worden met `Services`, `History` of `Location` eronder. Hiervoor bestaat de `Outlet` component van React Router. **Probeer deze component eerst zelf uit o.b.v. [de documentatie](https://reactrouter.com/docs/en/v6/components/outlet).**

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  Pas de definitie van `/over` aan:

  ```jsx
  <Route path="over">
    <Route index element={<About />} />
    <Route path="services" element={<Services />} />
    <Route path="history" element={<History />} />
    <Route path="location" element={<Location />} />
  </Route>
  ```

  Voeg onderaan de `About` component de `Outlet` toe:

  ```jsx
  // src/pages.jsx
  import { Outlet } from 'react-router-dom';

  export const About = () => (
    <div>
      {/* de rest */}
      <Outlet />
    </div>
  );
  ```

## Redirects

Stel we willen dat gebruikers die naar `/services` navigeren naar `/over/services` doorgestuurd worden. Daarvoor voeg je volgende route toe aan de `App` component:

```jsx
<Route path="services" element={<Navigate to="/over/services" replace />} />
```

Deze route rendert de `Navigate` component wanneer de gebruiker naar `/services` navigeert. Deze component is onderdeel van React Router en zal naar de URL in de `to` prop navigeren. De `replace` prop zorgt ervoor dat de URL `/services` vervangen wordt en bijgevolg verwijderd wordt uit de geschiedenis. Daarom kunnen we dus niet meer terugkeren naar `/services` gebruik makend van de terugknop van de browser.

## URL parameters

In sommige gevallen wil je ook stukken in de URL kunnen invullen met bv. een id van een entiteit. Om dit te demonstreren maken we een component die een lijst van producten toont:

```jsx
// src/pages.jsx
const products = [{
  id: 1,
  name: 'Confituur',
  price: 2.50
}, {
  id: 2,
  name: 'Choco',
  price: 3.50
}, {
  id: 3,
  name: 'Coco-cola',
  price: 3.20
}, {
  id: 4,
  name: 'Fanta',
  price: 3.00
}, {
  id: 5,
  name: 'Sprite',
  price: 2.90
}];

export const Products = () => (
  <div>
    <ul>
      {products.map(({ id, name }) => (
        <li key={id}>
          {name}
        </li>
      ))}
    </ul>
  </div>
);
```

Vervolgens definiëren we twee nieuwe routes in de `App` component:

```jsx
<Route path="products">
  <Route index element={<Products />} />
  <Route path=":id" element={<Product />} />
</Route>
```

De eerste route is opnieuw een index-route die enkel getoond wordt op `/products`. De tweede route zal getoond worden wanneer een id (eigenlijk eender welke string) opgegeven wordt na `/products`, bv. `/products/1` maar ook `/products/foo`.

Om dit id op te halen uit de URL maken we gebruik van de `useParams` hook. Deze hook retourneert een object van key/value pairs met alle URL parameters met hun waarde uit de huidige URL. Deze hook zal een nieuw object retourneren telkens wanneer een URL parameter wijzigt.

### Voorbeeld

Stel we hebben volgende routes gedefinieerd:

```jsx
function App() {
  return (
    <Routes>
      <Route path="/products/:id" element={<Product />} />
      <Route path="/posts/:year/:month" element={<Posts />} />
    </Routes>
  );
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

We moeten nog enkel de Product component implementeren zodat we de details van een product kunnen tonen. Maak een nieuwe component in `src/pages.jsx`:

```jsx
// src/pages.jsx
import { useParams } from 'react-router';

export const Product = () => {
  const { id } = useParams();
  const idAsNumber = Number(id);

  const product = products.find((p) => p.id === idAsNumber);

  if (!product) {
    return (
      <div>
        <h1>Product niet gevonden</h1>
        <p>
          Er is geen product met id {id}.
        </p>
      </div>
    )
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p><b>Price:</b> &euro; {product.price}</p>
    </div>
  )
};
```

Deze component zal eerst het id uit de URL ophalen en omvormen naar een `Number`. Daarna zoekt het een product met het opgegeven id. Indien dit product niet bestaat, zal een gepaste boodschap getoond worden. In het andere geval wordt de product-informatie getoond.

## Scroll restoration

Bij routing in SPA's wordt de scroll-positie niet automatisch hersteld naar linksboven in de browser. Dit is vrij logisch aangezien de routing niet door de browser afgehandeld wordt maar door JavaScript (React Router in dit geval). Indien gewenst, moet hier zelf voor gezorgd worden met de useEffect hook.

Maak een nieuw bestand `src/ScrollToTop.jsx` met volgende code:

```jsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

export default function ScrollToTop() {
  const { pathname } = useLocation();

  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);

  return null;
}
```

Deze component is niet zichtbaar voor de gebruiker door de `return null`. Zo zie je dat componenten ook enkel gedrag kunnen hebben en niet per se iets moeten tonen!

Deze component haalt de huidige URL op. Wanneer deze URL wijzigt, zal de code van het effect uitgevoerd worden. Deze code herstelt de scroll-positie naar (0, 0), wat dus linksboven is.

Je kan deze component toevoegen aan de `App` component om ervoor te zorgen dat de scroll-positie hersteld wordt:

```jsx
import ScrollToTop from './ScrollToTop';

function App() {
  return (
    <>
      <ScrollToTop />
      <Routes>
        { /* ... */ }
      </Routes>
    </>
  );
}
```

**Merk op:** we moeten de code van de `App` component wrappen in een `React.Element` (of verkort `<>`) aangezien `Routes` niet meer het enige kind is van de component.

Hoe kan je dit testen? Simpel: voeg op een willekeurige pagina onderaan een `div` met een gigantische hoogte toe (hoogte via CSS bv.) en plaats hierna een `Link` naar eender waar. Uiteraard werkt dit niet met een `Link` naar de pagina waar je nu bent.

### Smooth scroll

Je kan het scrollen ook fancy maken door de code in het effect te vervangen door onderstaande code. Hierdoor zal de browser scrollen met een animatie. Indien de modernde `scrollTo` niet ondersteund wordt, wordt teruggevallen op de oude implementatie (in de `catch`).

```js
try {
  window.scrollTo({
    top: 0,
    left: 0,
    behavior: 'smooth'
  });
} catch {
  // Fallback voor oude browsers
  window.scrollTo(0, 0);
}
```

## Navigeren vanuit code

Soms wil je navigeren vanuit code, daarvoor bestaat de `useNavigate` hook. Deze hook geeft een functie terug met o.a. de URL waarnaar genavigeerd wordt als parameter. Meer informatie staat uiteraard in de [useNavigate documentatie](https://reactrouter.com/docs/en/v6/hooks/use-navigate). Je kan bijvoorbeeld ook vragen om de huidige URL te vervangen zodat deze verdwijnt uit de "terugkeer-geschiedenis" van de browser.

Als voorbeeld gaan we onderaan elke pagina een knop zetten waarmee we terug naar de home-pagina kunnen. Dit doen we door volgende code toe te voegen aan de `App` component:

```jsx
import { useCallback } from 'react';
import { useNavigate } from 'react-router-dom';

function App() {
  const navigate = useNavigate();

  const handleGoHome = useCallback(() => {
    navigate('/', { replace: true });
  }, [navigate]);

  return (
    <>
      { /* de rest */ }
      <button onClick={handleGoHome}>Go home!</button>
    </>
  );
}
```

Hiermee maken we een knop met een `onClick` handler. Deze functie zal via React Router terug naar de home-pagina navigeren en de huidige URL hierdoor vervangen.

## Oefening: basis routing

Voorzie routing in de budget-applicatie (of in je eigen app?):

- `/`: doorsturen naar `/transactions`
- `/transactions`: een lijst van transacties (`TransactionList` component)
- `/places`: een lijst van places (`PlacesList` component)
- `/transactions/add`: een nieuwe transactie toevoegen (`TransactionForm` component)

## Oefening: URL parameters

Voorzie volgende bijkomende routes in de budget-applicatie (of in je eigen app?):

- `/transactions/edit/:id`: een transactie bewerken (`TransactionForm` component)
- `/transactions/:id`: details van één transactie (nieuwe component nodig)
- `/places/:id`: details van één place (nieuwe component nodig)

## Oplossing oefeningen

Zie GitHub: [HOGENT-Web/frontendweb-budget](https://github.com/hogent-web/frontendweb-budget).
