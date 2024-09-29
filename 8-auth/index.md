# Authenticatie en autorisatie

> **Startpunt voorbeeldapplicatie**
>
> Het volstaat om uit te checken op de `main` branch
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les7
> yarn install
> yarn dev
> ```
>
> **De [REST API](https://github.com/HOGENT-frontendweb/webservices-budget/) dient ook te draaien op branch `authenticatie`.**

## API calls voor login

Alvorens we kunnen inloggen, moeten we onze API calls definiëren. Dit doen we in het bestand `index.js` in de map `src/api`. Definieer hierin een functie `post` die alle gegeven data als body verzendt naar de gegeven URL. Voer het HTTP request uit binnen deze functie en geef het response terug.

```js
export const post = async (url, { arg }) => {
  const { data } = await axios.post(url, arg);

  return data;
};
```

## AuthProvider

### Context opstellen

We maken gebruik van een context om alles omtrent authenticatie en autorisatie bij te houden. Maak een bestand `src/contexts/Auth.context.jsx` aan met volgende inhoud:

```jsx
import {
  createContext, // 👈 1
  useState, // 👈 4
  useCallback, // 👈 6
  useMemo, // 👈 5
  useContext, // 👈 5
} from 'react';
import useSWRMutation from 'swr/mutation'; // 👈 8
import * as api from '../api'; // 👈 8

const JWT_TOKEN_KEY = 'jwtToken'; // 👈 13
const USER_ID_KEY = 'userId'; // 👈 13
const AuthContext = createContext(); // 👈 1

export const useAuth = () => useContext(AuthContext); // 👈 5

// 👇 2
export const AuthProvider = ({ children }) => {
  const [token, setToken] = useState(localStorage.getItem(JWT_TOKEN_KEY)); // 👈 4 en 13
  const [user, setUser] = useState(null); // 👈 4

  const {
    isMutating: loading,
    error,
    trigger: doLogin,
  } = useSWRMutation('users/login', api.post); // 👈 8

  // 👇 6
  const login = useCallback(
    async (email, password) => {
      try {
        // 👇 7
        const { token, user } = await doLogin({
          email,
          password,
        });

        setToken(token); // 👈 8
        setUser(user); // 👈 8

        localStorage.setItem(JWT_TOKEN_KEY, token); // 👈 13
        localStorage.setItem(USER_ID_KEY, user.id); // 👈 13

        return true; // 👈 10
        // 👇 10
      } catch (error) {
        console.error(error);
        return false;
      }
    },
    [doLogin],
  );

  // 👇 11
  const logout = useCallback(() => {
    setToken(null);
    setUser(null);

    localStorage.removeItem(JWT_TOKEN_KEY);
    localStorage.removeItem(USER_ID_KEY);
  }, []);

  // 👇 5 en 9 en 12
  const value = useMemo(
    () => ({
      token,
      user,
      error,
      loading,
      login,
      logout,
    }),
    [token, user, error, loading, login, logout],
  );

  // 👇 3
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

1. Creëer een nieuwe context.
2. Maak een `AuthProvider` aan.
3. Retourneer reeds de kinderen gewrapped in de `AuthContext.Provider`.
4. Definieer twee state-variabelen om onze JWT en ingelogde gebruiker bij te houden.
5. We zetten deze waarden alvast op de context. Dan moeten we uiteraard ook nog een hook voorzien waarmee we aan deze waarde kunnen. We definiëren eerst een hook om aan onze volledige context-waarde te kunnen, we exporteren deze.
6. We definiëren een functie waarmee we een gebruiker kunnen aanmelden. We wrappen de functie in een `useCallback`.
7. Roep de API aan om een gebruiker aan te melden. We maken hiervoor gebruik van de `useSWRMutation` hook. Deze hook handelt automatisch de loading (via `isMutating`) en error state voor ons af.
8. Als alles goed ging, houden we de JWT en user bij.
9. Voeg ook deze functie toe aan de context.
10. We retourneren ook `true` zodat we kunnen weten of het aanmelden gelukt is. Indien iets fout ging, retourneren we `false`.
11. We voorzien ook een functie om een gebruiker terug uit te loggen. Uitloggen is zo eenvoudig als de token verwijderen en de user op null zetten. Herinner je: een JWT is stateful, de server stateless. M.a.w. een JWT bevat alle nodige informatie, een server valideert deze. Gooien we de JWT weg, dan kunnen we niet meer aan de beveiligde routes.
12. We zetten ook deze functie op de context.
13. Nu moeten we er enkel nog voor zorgen dat de token behouden blijft tussen de verschillende keren dat we naar de website gaan. Hiervoor moeten we de token opslaan in `localStorage`, evenals de userId. We voegen de huidige token uit localStorage toe als initiële waarde van onze state-variabele. We houden de localStorage key bij in een globale constante.

We wrappen de hele app in de `AuthProvider` (in `src/main.jsx`):

```jsx
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
import axiosRoot from 'axios'; // 👈 1

const baseUrl = import.meta.env.VITE_API_URL;

// 👇 2
export const axios = axiosRoot.create({
  baseURL: baseUrl,
});

// 👇 3
export const setAuthToken = (token) => {
  if (token) {
    axios.defaults.headers['Authorization'] = `Bearer ${token}`; // 👈 4
  } else {
    delete axios.defaults.headers['Authorization']; // 👈 5
  }
};

// ...
```

1. We hernoemen de import van axios naar `axiosRoot` omdat we zelf een constante genaamd `axios` gaan maken en dat zorgt voor een duplicate variabele.
2. We creëren een nieuwe instantie van axios met een gegeven `baseUrl`. Elke request zal deze `baseUrl` voor de URL plaatsen. Zo hoeven we dit niet handmatig te doen.
3. We definiëren ook een functie om een JWT in de axios-instantie te plaatsen.
4. Als we een token gekregen hebben, en dus niet `null`, `undefined` of een lege string, dan plaatsen we deze in de `Authorization` header met de prefix `Bearer`.
5. Indien we geen token kregen, verwijderen we de `Authorization` header. Het maakt niet uit of hij al dan niet aanwezig was.

Nu moeten we er nog voor zorgen dat het token toegevoegd wordt aan de `Authorization` header. Dit doen we terug in de `AuthProvider`. Vermits dit pas kan gebeuren als we het token verkregen hebben, maken we hiervoor gebruik van een `useEffect`.

```jsx
import {
  createContext,
  useState,
  useCallback,
  useEffect, // 👈
  useMemo,
  useContext,
} from 'react';

// ...
// 👇 voeg toe aan de AuthProvider, na de state variabelen
useEffect(() => {
  api.setAuthToken(token);
}, [token]);
// ...
```

## Login component

Om te kunnen aanmelden hebben we een `Login` component op de URL `/login` nodig. Deze component bevat een formulier met twee velden: `email` en `password`, beide zijn verplicht. Onderaan het formulier staan ook twee knoppen: "Sign in" en "Cancel". Deze knoppen implementeren we straks.

Vermits we hier de `LabelInput` component uit de `TransactionForm` kunnen hergebruiken, plaatsen we deze eerst in een aparte module `src/components\LabelInput.jsx`:

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
  } = useFormContext();

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
        disabled={isSubmitting}
        className='form-control'
        {...rest}
      />
      {hasError ? (
        <div className='form-text text-danger' data-cy='label_input_error'>
          {errors[name]}
        </div>
      ) : null}
    </div>
  );
}
```

Daarna voegen we de `Login` component toe in `src/pages/Login.jsx`:

```jsx
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
    <FormProvider {...methods}>
      <div className='container'>
        <form className='d-flex flex-column'>
          <h1>Sign in</h1>

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
            name='password'
            validationRules={validationRules.password}
          />

          <div className='clearfix'>
            <div className='btn-group float-end'>
              <button type='submit' className='btn btn-primary'>
                Sign in
              </button>

              <button type='button' className='btn btn-light'>
                Cancel
              </button>
            </div>
          </div>
        </form>
      </div>
    </FormProvider>
  );
}
```

Zorg ervoor dat deze component getoond wordt op de URL `/login`. Voeg deze route toe aan de router in `src/main.jsx`:

```jsx
{
  path: '/login',
  element: <Login />,
}
```

Om aan te melden maken we gebruik van onze `useAuth` hook. Pas `src/pages/Login.jsx` als volgt aan:

```jsx
import { useCallback } from 'react'; // 👈 1
import { useNavigate } from 'react-router-dom'; // 👈 3
import { FormProvider, useForm } from 'react-hook-form';
import LabelInput from '../components/LabelInput';
import { useAuth } from '../contexts/Auth.context'; // 👈 2
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
  const navigate = useNavigate(); // 👈 3

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

      if (loggedIn) {
        navigate({
          pathname: '/',
          replace: true,
        });
      } // 👈 3
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
          {' '}
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
            name='password'
            validationRules={validationRules.password}
          />
          <div className='clearfix'>
            <div className='btn-group float-end'>
              <button
                type='submit'
                className='btn btn-primary'
                disabled={loading}
              >
                {/* 👆 4 */}
                Sign in
              </button>

              <button
                type='button'
                className='btn btn-light'
                onClick={handleCancel}
              >
                {/* 👆 6*/}
                Cancel
              </button>
            </div>
          </div>
        </form>
      </div>
    </FormProvider>
  );
}
```

1. We maken vervolgens onze functie `handleLogin` die opgeroepen zal worden als het formulier gesubmit wordt en stellen de `onSubmit` van het formulier in.
2. We proberen in te loggen.
3. Als het succesvol was, dan keren we terug naar de home. We gebruiken `navigate` met de `replace` optie aangezien we niet terug naar deze component mogen gaan.
4. We schakelen de submit-knop uit indien ons login-request bezig is.
5. Ook tonen we een mogelijke error.
6. We koppelen ook een click handler aan de cancel-knop.
7. We voorzien alle standaardwaarden, dan moeten we niet steeds de gegevens ingeven bij het aanmelden. In een echte applicatie zou dit niet mogen, maar als we aan het ontwikkelen zijn, is dit wel handig.

Je kan testen of de component werkt door eens te navigeren naar `/login`. Na het aanmelden zou je alle transacties in de lijst moeten kunnen zien.

## Routes afschermen

Als laatste moeten we nog routes kunnen afschermen voor ingelogde gebruikers. Hiervoor definiëren we zelf een component `PrivateRoute`:

- Als we de credentials aan het ophalen zijn, dan toont dit een loading indicator.
- Als we aangemeld zijn, dan retourneren we een `Outlet` voor de weergave van de child routes.
- Anders navigeren we naar de login pagina.

Hiervoor dienen we de `AuthProvider` eerst aan te passen. We moeten weten wanneer we de credentials aan het ophalen zijn en of we zijn aangemeld. Hiervoor voegen we 2 state variabelen toe, respectievelijk `ready` en `isAuthed`, die worden ingesteld als het token is opgehaald. We plaatsen deze variabelen ook in de context.

```jsx
// ...
export const AuthProvider = ({ children }) => {
  const [ready, setReady] = useState(false); // 👈
  const [isAuthed, setIsAuthed] = useState(false); // 👈
  // ...

  useEffect(() => {
    api.setAuthToken(token);
    setIsAuthed(Boolean(token)); // 👈
    setReady(true); // 👈
  }, [token]);
  // ...

  // 👇
  const value = useMemo(
    () => ({
      token,
      user,
      error,
      ready,
      loading,
      isAuthed,
      login,
      logout,
    }),
    [token, user, error, ready, loading, isAuthed, login, logout],
  );

  // ...
};
```

Nu kunnen we de `PrivateRoute` component aanmaken. Maak een bestand `src/components/PrivateRoute.jsx` aan met volgende inhoud:

```jsx
import { Navigate, Outlet } from 'react-router-dom'; // 👈 3
import { useAuth } from '../contexts/Auth.context'; // 👈 2

// 👇 1
export default function PrivateRoute() {
  const { ready, isAuthed } = useAuth(); // 👈 2

  // 👇 2
  if (!ready) {
    return (
      <div className='container'>
        <div className='row'>
          <div className='col-12'>
            <h1>Loading...</h1>
            <p>
              Please wait while we are checking your credentials and loading the
              application.
            </p>
          </div>
        </div>
      </div>
    );
  }

  // 👇 3
  if (isAuthed) {
    return <Outlet />;
  }

  return <Navigate replace to='/login' />; // 👈 4
}
```

1. We definiëren een `PrivateRoute` component.
2. Als we de credentials aan het controleren zijn, tonen we de loading indicator.
3. Als de gebruiker ingelogd is, dan retourneren we een Outlet component voor de weergave van de child routes.
4. Als de gebruiker niet ingelogd is, dan sturen we hem door naar de loginpagina.

Tot slot maken we gebruik van deze component om onze routes af te schermen. `Transactions` en `Places` dienen afgeschermd te worden. `ProtectedRoute` wordt de parent component, en in de `Outlet` component worden de children gerenderd als de gebruiker is aangemeld. Pas `src/main.jsx` als volgt aan:

```jsx
// ...
import PrivateRoute from './components/PrivateRoute';

const router = createBrowserRouter([
  {
    element: <Layout />,
    children: [
      {
        path: '/',
        element: <Navigate replace to='/transactions' />,
      },
      {
        path: '/login',
        element: <Login />,
      },
      {
        path: '/transactions',
        element: <PrivateRoute />, // 👈
        children: [
          // 👈
          {
            index: true,
            element: <TransactionsList />,
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
      },
      {
        path: '/places',
        element: <PrivateRoute />, // 👈
        children: [
          // 👈
          {
            index: true,
            element: <PlacesList />,
          },
        ],
      },
      {
        path: '*',
        element: <NotFound />,
      },
    ],
  },
]);
// ...
```

We dienen er ook voor te zorgen dat na het inloggen er geredirect wordt naar de pagina die de gebruiker wou raadplegen. Pas hiervoor de `login` component aan

```jsx
import { useCallback, useMemo } from 'react';// 👈1
//...

export default function Login() {
  const { error, loading, login } = useAuth();
  const navigate = useNavigate();
  const { search } = useLocation();

  const redirect = useMemo(() => {
    const urlParams = new URLSearchParams(search);
    if (urlParams.has("redirect"))
      return urlParams.get("redirect");
    return "/";
  }, [search]);// 👈1

  //..

  const handleLogin = useCallback(
    async ({ email, password }) => {
      const loggedIn = await login(email, password);

      if (loggedIn) {
        navigate({
          pathname: redirect,// 👈2
          replace: true,
        });
      }
    },
    [login, navigate, redirect],// 👈2
  );

  //...
```

1. Haal de querystring op en kijk of de key redirect voorkomt.
2. Navigeer naar de startpagina of naar de redirect

## NavBar: afwerking

Als laatste voegen we nog een login- en logout-knop toe aan onze navbar. Deze knoppen mogen ook niet altijd getoond worden. Pas hiervoor `src/components/Navbar.jsx` aan als volgt:

```jsx
// ...
import { useAuth } from '../contexts/Auth.context';

export default function Navbar() {
  const { isAuthed } = useAuth(); // 👈 2
  // ...

  return (
    <nav className={`navbar sticky-top bg-${theme} mb-4`}>
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
        <div className='flex-grow-1'></div>
        {/* 👈 1*/}

        {
          // 👇 3
          isAuthed ? (
            // 👇 4
            <div className='nav-item my-2 mx-sm-3 my-sm-0'>
              <Link className='nav-link' to='/logout'>
                Logout
              </Link>
            </div>
          ) : (
            // 👇 5
            <div className='nav-item my-2 mx-sm-3 my-sm-0'>
              <Link className='nav-link' to='/login'>
                Login
              </Link>
            </div>
          )
        }

        <button
          className='btn btn-secondary'
          type='button'
          onClick={toggleTheme}
        >
          {theme === 'dark' ? <IoMoonSharp /> : <IoSunny />}
        </button>
      </div>
    </nav>
  );
}
```

1. We voegen een `div` toe om deze knoppen rechts uit te lijnen, deze `div` vult de lege ruimte.
2. We controleren of er een gebruiker ingelogd is.
3. En tonen afhankelijk van de waarde andere knoppen.
4. Als we aangemeld zijn, tonen we de logout-knop.
5. In het andere geval, tonen we de login-knop.

We dienen nog een `Logout` component aan te maken. Maak een nieuw bestand `src/pages/Logout.jsx` met volgende inhoud:

```jsx
import { useEffect } from 'react'; // 👈 1
import { useAuth } from '../contexts/Auth.context'; // 👈 1
import { useThemeColors } from '../contexts/Theme.context';

export default function Logout() {
  const { theme, oppositeTheme } = useThemeColors();
  const { isAuthed, logout } = useAuth(); // 👈 1

  // 👇 1
  useEffect(() => {
    logout();
  }, [logout]);

  // 👇 2
  if (isAuthed) {
    return (
      <div className={`container bg-${theme} text-${oppositeTheme}`}>
        <div className='row'>
          <div className='col-12'>
            <h1>Logging out...</h1>
          </div>
        </div>
      </div>
    );
  }

  // 👇 3
  return (
    <div className={`container bg-${theme} text-${oppositeTheme}`}>
      <div className='row'>
        <div className='col-12'>
          <h1>You were successfully logged out</h1>
        </div>
      </div>
    </div>
  );
}
```

1. Ook hier maken we gebruik van een `useEffect` voor het uitloggen.
2. Als de gebruiker aan het uitloggen is (en dus nog aangemeld is), geven we aan dat we aan het uitloggen zijn.
3. Als de gebruiker is uitgelogd, geven we een melding weer.

Voeg de route naar Logout toe aan `src/main.jsx`:

```jsx
import Logout from './pages/Logout';

// ...
const router = createBrowserRouter([
  {
    element: <Layout />,
    children: [
      {
        path: '/',
        element: <Navigate replace to='/transactions' />,
      },
      {
        path: '/login',
        element: <Login />,
      },
      {
        path: '/logout',
        element: <Logout />,
      },
      // ...
    ],
  },
  // ...
]);
```

## Oefening

Maak een `Register` component op de URL `/register`.

- Deze component bevat een formulier met 4 velden:
  - `name`
  - `email`
  - `password`
  - `confirmPassword`
- Alle velden zijn verplicht + de waarde van het `confirmPassword` veld moet gelijk zijn aan de waarde van het password veld.
  - Gebruik hiervoor de `validate` functie van `register`: <https://react-hook-form.com/docs/useform/register>.
  - Je moet hiervoor de `validationRules` binnen de component zetten, gebruik een `useMemo` om dit object te maken.
  - Om een waarde van een veld op te vragen gebruik je `getValues`.
- Als de gebruiker reeds aangemeld is, moet deze pagina hem doorsturen naar de `/` route.

<!-- markdownlint-disable-next-line -->

- Oplossing +

  Een voorbeeldoplossing is te vinden op <https://github.com/HOGENT-frontendweb/frontendweb-budget> in de branch `authenticatie` op commit `14016d2`:

  ```bash
  git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
  cd frontendweb-budget
  git checkout -b oplossing-les7 14016d2
  yarn install
  yarn dev
  ```
