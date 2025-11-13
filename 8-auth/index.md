# Authenticatie en autorisatie

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les8 5d642b4
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
> pnpm install
> docker compose up -d
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#web-services-budget) voor meer informatie.

In dit hoofdstuk voegen we authenticatie en autorisatie toe aan onze applicatie. We maken hiervoor gebruik van JSON Web Tokens (JWT). We zullen de gebruiker toelaten om zich aan te melden en uit te loggen, maar ook te registreren. We zullen ook bepaalde routes afschermen voor niet-aangemelde gebruikers.

?> De code die je in de olods Front-end Web Development en Web Services opbouwt, werd in een bachelorproef door een student gecontroleerd op gebied van security. Een aantal zaken werden reeds in de applicatie aangepast. Voel je vrij om deze bachelorproef te lezen als inspiratie voor eigen projecten: <br /><br />Vermeersch, J. (2024). Cybersecuritymaatregelen in de opleidingsonderdelen over webapplicatieontwikkeling aan Hogeschool Gent: een analyse en integratie van aanbevelingen in de voorbeeldapplicaties. Gent: s.n. Geraadpleegd op 3 november 2024, via <https://catalogus.hogent.be/catalog/hog01:003132172>.

## API calls voor login

Alvorens we kunnen inloggen, moeten we onze API calls definiëren. Dit doen we in het bestand `src/api/index.js`. Definieer hierin een functie `post` die alle gegeven data als body verzendt naar de gegeven URL. Voer het HTTP request uit binnen deze functie en geef het response terug.

```js
// src/api/index.js
export const post = async (url, { arg }) => {
  const { data } = await axios.post(`${baseUrl}/${url}`, arg);

  return data;
};
```

## AuthProvider

Vervolgens maken we een `AuthProvider` aan. Deze provider zal de aangemelde gebruiker bijhouden en de nodige functies voorzien om aan te melden en uit te loggen.

### Context opstellen

We maken gebruik van een context om alles omtrent authenticatie en autorisatie bij te houden. Maak een bestand `src/contexts/Auth.context.jsx` aan met volgende inhoud:

```jsx
// src/contexts/Auth.context.jsx
import {
  createContext, // 👈 1
  useState, // 👈 4
  useCallback, // 👈 6
  useMemo, // 👈 5
} from 'react';
import useSWRMutation from 'swr/mutation'; // 👈 8
import * as api from '../api'; // 👈 8
import useSWR from 'swr';

export const JWT_TOKEN_KEY = 'jwtToken'; // 👈 13
export const AuthContext = createContext(); // 👈 1

// 👇 2
export const AuthProvider = ({ children }) => {
  const [token, setToken] = useState(localStorage.getItem(JWT_TOKEN_KEY)); // 👈 4 en 13

  // 👇 14
  const {
    data: user,
    loading: userLoading,
    error: userError,
  } = useSWR(token ? 'users/me' : null, api.getById);

  const {
    trigger: doLogin,
    isMutating: loginLoading,
    error: loginError,
  } = useSWRMutation('sessions', api.post); // 👈 8

  // 👇 5 en 6
  const login = useCallback(
    async (email, password) => {
      try {
        // 👇 7
        const { token } = await doLogin({
          email,
          password,
        });

        setToken(token); // 👈 8

        localStorage.setItem(JWT_TOKEN_KEY, token); // 👈 13

        return true; // 👈 10
      } catch (error) {
        // 👇 10
        console.error(error);
        return false;
      }
    },
    [doLogin],
  );

  // 👇 11
  const logout = useCallback(() => {
    setToken(null);

    localStorage.removeItem(JWT_TOKEN_KEY);
  }, []);

  // 👇 5 en 7 en 9 en 12 en 14
  const value = useMemo(
    () => ({
      user,
      error: loginError || userError,
      loading: loginLoading || userLoading,
      login,
      logout,
    }),
    [user, loginError, loginLoading, userError, userLoading, login, logout],
  );

  // 👇 3
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

1. Creëer een nieuwe context.
2. Maak een `AuthProvider` aan.
3. Retourneer reeds de kinderen gewrapped in de `AuthContext.Provider`.
4. Definieer een state-variabelen om onze JWT bij te houden.
5. We definiëren een functie waarmee we een gebruiker kunnen aanmelden.
6. We wrappen de functie in een `useCallback`.
7. Roep de API aan om een gebruiker aan te melden. We maken hiervoor gebruik van de `useSWRMutation` hook. Deze hook handelt automatisch de loading (via `isMutating`) en error state voor ons af. Voeg `error` en `loading` toe aan de context.
8. Als alles goed ging, houden we de JWT bij.
9. Voeg ook deze functie toe aan de context.
10. We retourneren ook `true` zodat we kunnen weten of het aanmelden gelukt is. Indien iets fout ging, retourneren we `false`.
11. We voorzien ook een functie om een gebruiker terug uit te loggen. Uitloggen is zo eenvoudig als de token verwijderen. Herinner je: een JWT is stateful, de server stateless. Met andere woorden een JWT bevat alle nodige informatie, een server valideert deze. Gooien we de JWT weg, dan kunnen we niet meer aan de beveiligde routes.
12. We voegen ook deze functie toe aan de context.
13. Nu moeten we er enkel nog voor zorgen dat de token behouden blijft tussen de verschillende keren dat we naar de website gaan. Hiervoor moeten we de token opslaan in `localStorage`. We voegen de huidige token uit `localStorage` toe als initiële waarde van onze state-variabele. We houden de `localStorage` key bij in een globale constante.
14. We halen ook de `user` gegevens op en voegen de waarde toe aan de context. Vervolledig `error` en `loading` in de context.

Dan moeten we uiteraard ook nog een hook voorzien waarmee we aan onze volledige Context waarde kunnen ophalen. Maak een file `auth.js` aan in de `contexts` folder. Plaats ook de andere exports uit `Auth.context.jsx` in deze file en pas de imports in `Auth.context.jsx` aan.

```js
// src/contexts/auth.js
import { createContext, useContext } from 'react';
export const JWT_TOKEN_KEY = 'jwtToken';
export const AuthContext = createContext();
export const useAuth = () => useContext(AuthContext);
```

We wrappen de hele app in de `AuthProvider` (in `src/main.jsx`):

```jsx
// src/main.jsx
// ...
import { AuthProvider } from './contexts/Auth.context';

// ...
createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <AuthProvider>
      <ThemeProvider>
        <RouterProvider router={router} />
      </ThemeProvider>
    </AuthProvider>
  </React.StrictMode>,
);
```

Als we de app opstarten, krijgen we een `HTTP 401` want de server verwacht een token voor o.a. `GET /api/transactions`. Wat is de oorzaak van dit probleem?

<!-- markdownlint-disable-next-line -->

- Oplossing +

  Axios voegt ons token nog niet toe aan elk request.

### Axios configuratie

Vervolgens gaan we een instantie van axios configureren voor het gebruik van een Bearer token. We passen hiervoor `src/api/index.js` aan:

```jsx
// src/api/index.js
import axiosRoot from 'axios'; // 👈 1
import { JWT_TOKEN_KEY } from '../contexts/auth';

const baseUrl = import.meta.env.VITE_API_URL;

// 👇 2
export const axios = axiosRoot.create({
  baseURL: baseUrl,
});

// 👇 3
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem(JWT_TOKEN_KEY);

  if (token) {
    config.headers['Authorization'] = `Bearer ${token}`;
  }

  return config;
});

export async function getAll(url) {
  const { data } = await axios.get(url); // 👈 2
  return data.items;
}

// ...
```

1. We hernoemen de import van axios naar `axiosRoot` omdat we zelf een constante genaamd `axios` gaan maken en dat zorgt voor een duplicate variabele.
2. We creëren een nieuwe instantie van axios met een gegeven `baseUrl`. Elke request zal deze `baseUrl` voor de URL plaatsen. Zo hoeven we dit niet handmatig te doen.
   - Pas alle methodes aan, verwijder de `baseUrl`.
3. We onderscheppen de uitgaande HTTP requests en wijzigen deze alvorens ze naar de server gestuurd wordt. We plaatsen de token, indien aanwezig, in de `Authorization` header met de prefix `Bearer`.

## Login component

Om te kunnen aanmelden hebben we een `Login` component op de URL `/login` nodig. Deze component bevat een formulier met twee velden: `email` en `password`, beide zijn verplicht. Onderaan het formulier staan ook twee knoppen: "Sign in" en "Cancel". Deze knoppen implementeren we straks. We kunnen hier de `LabelInput` component hergebruiken.

Voeg de `Login` component toe in `src/pages/Login.jsx`:

```jsx
// src/pages/Login.jsx
import { FormProvider, useForm } from 'react-hook-form';
import LabelInput from '../components/LabelInput';

const validationRules = {
  email: {
    required: 'Email is required',
  },
  password: {
    required: 'Password is required',
  },
};

export default function Login() {
  const methods = useForm();

    return (
    <div className='w-full max-w-md'>
      <h1>Sign in</h1>
      <FormProvider {...methods}>
        <div className='container'>
          <form className='d-flex flex-column'>

            <LabelInput
              label='email'
              type='text'
              name='email'
              placeholder='your@email.com'
              validationRules={validationRules.email}
            />

            <LabelInput
              label='password'
              type='password'
              placeholder='password'
              name='password'
              validationRules={validationRules.password}
            />

            <div className='flex justify-end'>
              <button type='submit' className="primary">
                Sign in
              </button>
              <button type='submit' className="secondary ml-2">
                Cancel
              </button>
            </div>
          </form>
        </div>
      </FormProvider>
    </div>
  );
}
```

Zorg ervoor dat deze component getoond wordt op de URL `/login`. Voeg deze route toe aan de router in `src/main.jsx` en importeer de `Login` component:

```jsx
{
  path: '/login',
  Component : Login,
}
```

Om aan te melden maken we gebruik van onze `useAuth` hook. Pas `src/pages/Login.jsx` als volgt aan:

```jsx
// src/pages/Login.jsx
import { useCallback } from 'react'; // 👈 1
import { useNavigate } from 'react-router'; // 👈 3
import { FormProvider, useForm } from 'react-hook-form';
import LabelInput from '../components/LabelInput';
import { useAuth } from '../contexts/auth'; // 👈 2
import Error from '../components/Error'; // 👈 5

const validationRules = {
  email: {
    required: 'Email is required',
  },
  password: {
    required: 'Password is required',
  },
};

export default function Login() {
  const { error, loading, login } = useAuth(); // 👈 2, 4 en 5
  const navigate = useNavigate();

  // 👇 7
  const methods = useForm({
    defaultValues: {
      email: 'thomas.aelbrecht@hogent.be',
      password: '12345678',
    },
  });
  const { handleSubmit, reset } = methods; // 👈 1 en 6

  // 👇 6
  const handleCancel = useCallback(() => {
    reset();
  }, [reset]);

  // 👇 1
  const handleLogin = useCallback(
    async ({ email, password }) => {
      const loggedIn = await login(email, password); // 👈 2
      // 👇 3
      if (loggedIn) {
        navigate({
          pathname: '/',
          replace: true,
        });
      }
    },
    [login, navigate], // 👈 2 en 3
  );

  return (
    <FormProvider {...methods}>
      <div className='container'>
        <form
          className='d-flex flex-column'
          onSubmit={handleSubmit(handleLogin)}
        >
          {/* 👆 1 */}
          <h1>Sign in</h1>
          <Error error={error} /> {/* 👈 5 */}
          <LabelInput
            label='email'
            type='text'
            name='email'
            placeholder='your@email.com'
            validationRules={validationRules.email}
          />
          <LabelInput
            label='password'
            type='password'
            placeholder='password'
            name='password'
            validationRules={validationRules.password}
          />
          <div className='flex justify-end'>
            <button
              type='submit'
              className='primary'
              disabled={loading}
            >
              {/* 👆 4 */}
              Sign in
            </button>

            <button
              type='button'
              className='secondary ml-2'
              onClick={handleCancel}
            >
              {/* 👆 6*/}
              Cancel
            </button>
          </div>
        </form>
      </div>
    </FormProvider>
  );
}
```

1. Maak een functie `handleLogin` aan die opgeroepen zal worden als het formulier gesubmit wordt en stel de `onSubmit` van het formulier in.
2. We proberen in te loggen.
3. Als het succesvol was, dan keren we terug naar de home page. We gebruiken `navigate` met de `replace` optie aangezien we niet terug naar deze component mogen gaan.
4. We schakelen de submit-knop uit indien ons login-request bezig is.
5. Ook tonen we een mogelijke error.
6. We koppelen ook een click handler aan de cancel-knop.
7. We voorzien alle standaardwaarden, dan moeten we niet steeds de gegevens ingeven bij het aanmelden.
   - In een echte applicatie zou dit niet mogen, maar als we aan het ontwikkelen zijn, is dit wel handig.

Je kan testen of de component werkt door eens te navigeren naar `/login`. Na het aanmelden zou je alle transacties in de lijst moeten kunnen zien.

## Routes afschermen

Als laatste moeten we nog routes kunnen afschermen voor ingelogde gebruikers. Hiervoor definiëren we zelf een component `PrivateRoute`:

- Als we de credentials aan het ophalen zijn, dan toont dit een loading indicator.
- Als we aangemeld zijn, dan retourneren we een `Outlet` voor de weergave van de child routes.
- Anders navigeren we naar de login pagina.

Hiervoor dienen we de `AuthProvider` eerst aan te passen. We moeten weten wanneer we de credentials aan het ophalen zijn en of we zijn aangemeld. Hiervoor voegen `ready` en `isAuthed` toe aan de context.

?> Je kan de `isAuthed` nog uitbreiden met een check of de token niet verlopen is. Zie de bachelorproef van [Joren Vermeersch](https://catalogus.hogent.be/catalog/hog01:003132172) voor meer informatie.

```jsx
// src/contexts/Auth.context.jsx
// ...

export const AuthProvider = ({ children }) => {
  // 👇
  const value = useMemo(
    () => ({
      token,
      user,
      error: loginError || userError,
      loading: loginLoading || userLoading,
      isAuthed: Boolean(token),
      ready: !userLoading,
      login,
      logout,
    }),
    [
      token,
      user,
      loginError,
      loginLoading,
      userError,
      userLoading,
      login,
      logout,
    ],
  );

  // ...
};
```

Nu kunnen we de `PrivateRoute` component aanmaken. Maak een bestand `src/components/PrivateRoute.jsx` aan met volgende inhoud:

```jsx
// src/components/PrivateRoute.jsx
import { Navigate, Outlet, useLocation } from 'react-router'; // 👈 3 en 4
import { useAuth } from '../contexts/auth'; // 👈 2

// 👇 1
export default function PrivateRoute() {
  const { ready, isAuthed } = useAuth(); // 👈 2
  const { pathname } = useLocation(); // 👈 4

  // 👇 2
 if (!ready) {
    return (
      <div>
        <h1>Loading...</h1>
        <p>
          Please wait while we are checking your credentials and loading the
          application.
        </p>
      </div>
    );
  }

  // 👇 3
  if (isAuthed) {
    return <Outlet />;
  }

  return <Navigate replace to={`/login?redirect=${pathname}`} />; // 👈 4
}
```

1. We definiëren een `PrivateRoute` component.
2. Als we de credentials aan het controleren zijn, tonen we de loading indicator.
3. Als de gebruiker ingelogd is, dan retourneren we een `Outlet` component voor de weergave van de child routes.
4. Als de gebruiker niet ingelogd is, dan sturen we hem door naar de login pagina. We geven een redirect mee zodat de gebruiker eens aangemeld, terug op deze pagina komt.

Tot slot maken we gebruik van deze component om onze routes af te schermen. `Transactions` en `Places` dienen afgeschermd te worden. `PrivateRoute` wordt de parent component, en in de `Outlet` component worden de children gerenderd als de gebruiker is aangemeld. Pas `src/main.jsx` als volgt aan:

```jsx
// src/main.jsx
// ...
import PrivateRoute from './components/PrivateRoute';

const router = createBrowserRouter([
  {
    Component: Layout,
    children: [
      {
        path: '/',
        element: <Navigate replace to='/transactions' />,
      },
      {
        path: '/login',
        Component: Login ,
      },
      {
        path: '/transactions',
        Component:PrivateRoute ,// 👈
        children: [
          {
            index: true,
            Component: TransactionList ,
          },
          {
            path: 'add',
            Component: AddOrEditTransaction ,
          },
          {
            path: 'edit/:id',
            Component: AddOrEditTransaction ,
          },
        ],
      },
      {
        path: '/places',
        Component:PrivateRoute,// 👈
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
        ],
      },
      {
        path: 'services',
        element: <Navigate to='/about/services' replace />,
      },
      { path: '*', Component: NotFound },
    ],
  }]);
// ...
```

We dienen er ook voor te zorgen dat na het inloggen genavigeerd wordt naar de pagina die de gebruiker wou raadplegen. Pas hiervoor de `Login` component aan:

```jsx
// src/pages/Login.jsx
import { useNavigate, useLocation } from 'react-router'; // 👈
//...

export default function Login() {
  const { search } = useLocation(); // 👈
  //...
  const handleLogin = useCallback(
    async ({ email, password }) => {
      const loggedIn = await login(email, password);

      if (loggedIn) {
        // Redirect to the page the user was on before logging in, if present in the URL
        // 👇
        const params = new URLSearchParams(search);
        navigate({
          pathname: params.get('redirect') || '/',
          replace: true,
        });
      }
    },
    [login, navigate, search], // 👈
  );

  //...
}
```

Haal de querystring op en kijk of de key `redirect` voorkomt en navigeer naar de startpagina of naar de waarde van de `redirect` query parameter.

## NavBar: afwerking

Als laatste voegen we nog een login- en logout-knop toe aan onze navbar. Deze knoppen mogen ook niet altijd getoond worden. Pas hiervoor `src/components/Navbar.jsx` aan als volgt:

```jsx
// src/components/Navbar.jsx
// ...
import { useAuth } from '../contexts/auth';// 👈 1

export default function Navbar() {

   // 👇 1
  const AuthButtons = ()=> {
    const { isAuthed } = useAuth();
    return  isAuthed ? (
      <Link className='primary' to='/logout'>
        Logout
      </Link>
    ) : (
      <Link className='primary' to='/login'>
        Login
      </Link>
    );
  };

  // ...

  return (
    //...  👇 2
    <div className="hidden lg:flex lg:items-center lg:space-x-4">
          <AuthButtons/>
          <ThemeToggle/>
        </div>
    //...
     <div className="flex items-center mb-8">
        <Logo/>
        <AuthButtons/>
        <ThemeToggle/>
     //...
  );
}
```

1. Maak een component `AuthButtons`. Controleer of er een gebruiker ingelogd is, en toon afhankelijk van de waarde andere knoppen.
2. Voeg de `AuthButtons` component toe aan de navbar.

We dienen nog een `Logout` component aan te maken. Maak een nieuw bestand `src/pages/Logout.jsx` met volgende inhoud:

```jsx
// src/pages/Logout.jsx
import { useEffect } from 'react'; // 👈 1
import { useAuth } from '../contexts/auth'; // 👈 1

export default function Logout() {
  const { isAuthed, logout } = useAuth(); // 👈 1

  // 👇 1
  useEffect(() => {
    logout();
  }, [logout]);

  // 👇 2
  if (isAuthed) {
    return (
      <h1>Logging out...</h1>
    );
  }

  // 👇 3
  return (
    <h1>You were successfully logged out</h1>
  );
}
```

1. We maken gebruik van een `useEffect` voor het uitloggen.
2. Als de gebruiker aan het uitloggen is (en dus nog aangemeld is), geven we aan dat we aan het uitloggen zijn.
3. Als de gebruiker is uitgelogd, geven we een melding weer.

Voeg de route naar `Logout` component toe aan `src/main.jsx` en importeer de `Logout` component:

```jsx
// src/main.jsx
{
  path: '/logout',
  Component: Logout,
}
```

## Oefening - Pas TransactionForm aan

De gebruiker moet nu niet langer de `userId` kunnen ingeven bij het aanmaken van een nieuwe transactie. De backend gebruikt de userId van de aangemelde gebruiker. Pas hiervoor de `TransactionForm` component aan.

- verwijder de `userId` uit de validatieregels.
- verwijder de defaultwaarde voor `userId` in useForm defaultValues.
- verwijder het `LabelInput` veld voor de `userId`.

## Oefening - Registreren

Een gebruiker dient zich te kunnen registreren op de site.

1. Pas hiervoor `Auth.context.jsx` aan.
   - Maak gebruik van `useSWRMutation` om een gebruiker te registreren.
   - Voorzie een methode `register`. Stel het `token` in als de gebruiker geregistreerd is.
   - Voeg de `register` functie toe aan de context.
2. Maak een `Register` component op de URL `/register`.
   - Deze component bevat een formulier met 4 velden:
     - `name`
     - `email`
     - `password`
     - `confirmPassword`
   - Alle velden zijn verplicht én de waarde van het `confirmPassword` veld moet gelijk zijn aan de waarde van het `password` veld.
     - Gebruik hiervoor de `validate` functie van `register`: <https://react-hook-form.com/docs/useform/register>.
     - Je moet hiervoor de `validationRules` binnen de component zetten, gebruik een `useMemo` om dit object te maken.
     - Om een waarde van een veld op te vragen gebruik je `getValues`.
   - Als de gebruiker is geregistreerd, moet deze pagina de gebruiker doorsturen naar de `/` route.
   - Voeg de route naar de register pagina toe aan `src/main.jsx`.
   - Pas ook de `NavBar` component aan. Voorzie naast de login ook een register link.

> **Oplossing voorbeeldapplicatie**
>
> Een voorbeeldoplossing is te vinden in de voorbeeldapplicatie:
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les8-opl b333700
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.

## Mogelijke extra's voor de examenopdracht

- Gebruik van een externe authenticatieprovider (bv. [Auth0](https://auth0.com/), [Userfront](https://userfront.com/)...)
- Voeg een wachtwoordsterkte-indicator toe
  - Dit is een vrij kleine extra, dus zorg ervoor dat je nog een andere extra toevoegt.
