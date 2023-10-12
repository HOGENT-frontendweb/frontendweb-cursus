# Authenticatie en autorisatie

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-Web/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les3 8539c87
> yarn install
> yarn dev
> ```
>
> **De [REST API](https://github.com/HOGENT-Web/webservices-budget/) dient ook te draaien op branch `feature/auth0`.Plaats in de configuratie voor Auth.disabled op false**

## API calls voor login

Alvorens we kunnen inloggen, moeten we onze API calls definiëren. Dit doen we in het bestand `index.js` in de map `src/api`.

```js
export async function post(url, { arg: { email, password } }) {
  const {
    data,
  } = await axios.post(url, { email, password });

  return data;
}
```

- definieer een functie `post` die een gebruiker met een e-mailadres en wachtwoord probeert in te loggen
- voer het HTTP request uit binnen deze functie en geef het response terug

## AuthProvider

We maken gebruik van een context om alles omtrent authenticatie en autorisatie bij te houden.

`src/contexts/Auth.context.jsx`

```jsx
import {
  createContext, // 👈 1
  useState, // 👈 4
  useCallback, // 👈 6
  useMemo, // 👈 5
  useContext, // 👈 5
} from 'react';
import useSWRMutation from 'swr/mutation'; // 👈 8
import * as api from '../api';// 👈 8

const JWT_TOKEN_KEY = 'jwtToken'; // 👈 15
const USER_ID_KEY = 'userId'; // 👈 15
const AuthContext = createContext(); // 👈 1

export const useAuth = () => useContext(AuthContext); // 👈 5

// 👈 2
export const AuthProvider = ({ children }) => {
  const [loading, setLoading] = useState(false); // 👈 4
  const [error, setError] = useState(''); // 👈 4
  const [token, setToken] = useState(localStorage.getItem(JWT_TOKEN_KEY)); // 👈 4 en 15
  const [user, setUser] = useState(null); // 👈 4

  const {
    trigger: doLogin,
  } = useSWRMutation('users/login', api.post); // 👈 8

  // 👈 6
  const login = useCallback(
    async (email, password) => {
      try {
        setLoading(true); // 👈 7
        setError(''); // 👈 7

        const { token, user } = await doLogin({
          email,
          password,
        }); // 👈 8

        setToken(token); // 👈 10
        setUser(user); // 👈 10

        localStorage.setItem(JWT_TOKEN_KEY, token); // 👈 14
        localStorage.setItem(USER_ID_KEY, user.id); // 👈 14

        return true; // 👈 10
        // 👈 9
      } catch (error) {
        console.error(error);
        setError(error.response?.data?.message || 'Login failed, try again');

        return false;
        // 👈 9
      } finally {
        setLoading(false);
      }
    },
    [doLogin]
  );

  // 👈 12
  const logout = useCallback(() => {
    setToken(null);
    setUser(null);

    localStorage.removeItem(JWT_TOKEN_KEY);
    localStorage.removeItem(USER_ID_KEY);
  }, []);

  const value = useMemo(
    () => ({
      token,
      user,
      error,
      loading,
      login,
      logout,
    }),
    [token, user, error, loading, login, logout]
  ); // 👈 5 en 11 en 13

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>; // 👈 3
};
```

1. Creëer een nieuwe context
2. Maak een `AuthProvider` aan
3. en retourneer reeds de kinderen gewrapped in de `Provider`.
4. Definieer twee state-variabelen om onze JWT en ingelogde gebruiker bij te houden. We voorzien ook een state-variabele voor een fout (bv. wachtwoord verkeerd) en om aan te duiden dat een request bezig is
5. We zetten deze waarden alvast op de context. Dan moeten we uiteraard ook nog een hook voorzien waarmee we aan deze waarde kunnen. We definiëren eerst een hook om aan onze volledige context-waarde te kunnen, we exporteren deze.
6. We definiëren een functie waarmee we een gebruiker kunnen inloggen. We wrappen de functie in een `useCallback`
7. We initialiseren eerst de error (initieel "") en de loading (op true)
8. Roep de API aan om een gebruiker aan te melden. We maken hiervoor gebruik van de useSWRMutation hook
9. Indien er iets fout ging, houden we de fout bij. De loading state wordt terug op false geplaatst
10. Als alles goed ging, houden we de JWT en user bij
11. Voeg ook deze functie toe aan de context
12. We voorzien ook een functie om een gebruiker terug uit te loggen. Uitloggen is zo eenvoudig als de token verwijderen en de user op null zetten. herinner je: een JWT is stateful, de server stateless. M.a.w. een JWT bevat alle nodige informatie, een server valideert deze. Gooien we de JWT weg, dan kunnen we niet meer aan de beveiligde routes
13. We zetten ook deze functie op de context
14. Nu moeten we er enkel nog voor zorgen dat de token behouden blijft tussen de verschillende keren dat we naar de website gaan. Hiervoor moeten we de token opslaan in `localStorage`, evenals de userId
15. We voegen de huidige token uit localStorage toe als initiële waarde van onze state-variabele. we houden de localStorage key bij in een globale constante. We willen ook het userId bijhouden in local storage. Hiervoor voorzien we eveneens een globale constante

We wrappen de hele app in de `AuthProvider`.

`src/main.jsx`

```jsx
import { AuthProvider } from './contexts/Auth.context';
//..
createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <AuthProvider>
      <ThemeProvider>
        <RouterProvider router={router} />
      </ThemeProvider>
    </AuthProvider>
  </React.StrictMode>
);
```

Als we de app opstarten, krijgen we een `HTTP 500` want de server verwacht een token voor o.a. `GET /api/transactions`. Wat is de oorzaak van dit probleem?

OPLOSSING: Axios voegt ons token nog niet toe aan elk request.

## AuthProvider: axios configuratie

We passen `index.js` bestand in de `src/api` map. Hierin gaan we een instantie van axios configureren voor gebruik met het `bearer` token.

`src/api/index.jsx`

```jsx
import axiosRoot from 'axios'; // 👈 1

const baseUrl = import.meta.env.VITE_API_URL;

export const axios = axiosRoot.create({
  baseURL: baseUrl,
}); // 👈 2

// 👈 3
export const setAuthToken = (token) => {
  if (token) {
    axios.defaults.headers['Authorization'] = `Bearer ${token}`; // 👈 4
  } else {
    delete axios.defaults.headers['Authorization']; // 👈 5
  }
};

//..
```

1. We hernoemen de import van axios naar `axiosRoot` omdat we zelf een constante genaamd `axios` gaan maken en dat zorgt voor een duplicate variabele
2. We creëren een nieuwe instantie van axios met een gegeven baseUrl. Elke request zal deze baseUrl voor de URL plaatsen. Zo hoeven we dit niet handmatig te doen.
3. We definiëren ook een functie om een JWT in de axios-instantie te plaatsen
4. Als we een token gekregen hebben, en dus niet null, undefined of een lege string, dan plaatsen we deze in de `Authorization` header met de prefix `Bearer`
5. Indien we geen token kregen, verwijderen we de `Authorization` header. Het maakt niet uit of hij al dan niet aanwezig was

Nu moeten we er nog voor zorgen dat het token toegevoegd wordt aan de `Authorization` header. Dit doen we terug in de `AuthProvider`. Vermits dit pas kan gebeuren als we het token verkregen hebben, maken we hiervoor gebruik van een `useEffect`.

```jsx
import {
  createContext,
  useState,
  useCallback,
  useEffect,
  useMemo,
  useContext,
} from 'react';
// ..
useEffect(() => {
  api.setAuthToken(token);
}, [token]);
//..
```

## Login component

- Maak een Login component op de url /login.
- Deze heeft twee velden: email en password, beide required.
- Onderaan het formulier staan ook twee knoppen: Sign in en Cancel. Deze knoppen implementeren we straks

Vermits we hier de `LabelInput` component uit de `TransactionForm` kunnen herbruiken plaatsen we dit eerst in een aparte module.

`src\components\LabelInput.jsx`

```jsx
import { useFormContext } from 'react-hook-form';

export default function LabelInput({
  label, name, type, ...rest
}) {
  const {
    register,
    formState: {
      errors,
      isSubmitting,
    },
  } = useFormContext();

  const hasError = name in errors;

  return (
    <div className="mb-3">
      <label htmlFor={name} className="form-label">
        {label}
      </label>
      <input
        {...register(name)}
        id={name}
        type={type}
        disabled={isSubmitting}
        className="form-control"
        {...rest}
      />
      {hasError ? (
        <div className="form-text text-danger">
          {errors[name]}
        </div>
      ) : null}
    </div>
  );
}
```

En voeg dan de `Login` component toe.

`src/pages/Login.jsx`

```jsx
import { FormProvider, useForm } from 'react-hook-form';
import LabelInput from '../components/LabelInput';

const validationRules = {
  email: {
    required: true,
  },
  password: {
    required: true,
  },
};

export default function Login() {

const methods = useForm();

  return (
    <FormProvider {...methods}>
      <div className='container'>
        <form
          className='d-flex flex-column'>  
          <h1>Sign in</h1>

          <LabelInput
            label='email'
            type='text'
            name='email'
            placeholder='your@email.com'
            validation={validationRules.email}
          />

          <LabelInput
            label='password'
            type='password'
            name='password'
            validation={validationRules.password}
          />

          <div className='clearfix'>
            <div className='btn-group float-end'>
              <button
                type='submit'
                className='btn btn-primary'
              >
                Sign in
              </button>

              <button
                type='button'
                className='btn btn-light'
              >
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
Maak de url `/login` aan.
### Sign in

Om aan te melden maken we gebruik van de `useAuth` hook.

`src/pages/Login.jsx`

```jsx
import { useCallback } from 'react';// 👈 1
import { useNavigate } from 'react-router-dom';// 👈 3
import { FormProvider, useForm } from 'react-hook-form';
import LabelInput from '../components/LabelInput';
import { useAuth } from '../contexts/Auth.context'; // 👈 2
import Error from '../components/Error';// 👈 5

const validationRules = {
  email: {
    required: true,
  },
  password: {
    required: true,
  },
};

export default function Login() {
  const { error, loading, login} = useAuth(); // 👈 2, 4 en 5 
  const navigate = useNavigate();// 👈 3

  const methods = useForm({
    defaultValues: {
      email: 'thomas.aelbrecht@hogent.be',
      password: '12345678',
    },// 👈 7
  });
  const { handleSubmit, reset } = methods;// 👈 1 en 6

  const handleCancel = useCallback(() => {
    reset();
  }, [reset]);// 👈 6

// 👈 1
  const handleLogin = useCallback(
    async ({ email, password }) => {
      const loggedIn = await login(email, password);// 👈 2

      if (loggedIn) {
        navigate({
          pathname: '/',
          replace: true,
        });
      }// 👈 3
    },
    [login, navigate] // 👈 2 en 3
  );

  return (
    <FormProvider {...methods}>
      <div className='container'>
        <form
          className='d-flex flex-column'
          onSubmit={handleSubmit(handleLogin)}
        >  {/* 👈 1 */}
          <h1>Sign in</h1>

          <Error error={error} /> {/* 👈 5 */}

          <LabelInput
            label='email'
            type='text'
            name='email'
            placeholder='your@email.com'
            validation={validationRules.email}
          />

          <LabelInput
            label='password'
            type='password'
            name='password'
            validation={validationRules.password}
          />

          <div className='clearfix'>
            <div className='btn-group float-end'>
              <button
                type='submit'
                className='btn btn-primary'
                disabled={loading} 
              >{/* 👈4 */}
                Sign in
              </button>

              <button
                type='button'
                className='btn btn-light'
                onClick={handleCancel}
              >{/* 👈 6*/}
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

1. We maken vervolgens onze functie `handleLogin` die opgeroepen zal worden als het formulier gesubmit wordt
   en stellen de `onSubmit` van het formulier in.
2. We proberen in te loggen
3. Als het succesvol was, dan keren we terug naar de home. We gebruiken `navigate` aangezien we niet terug naar deze component mogen gaan
4. we disablen de submit-knop indien ons login-request bezig is
5. Ook tonen we een mogelijke error
6. We handelen ook de Cancel knop af
7. We voorzien al default waarden, dan moeten we niet steeds de gegevens ingeven bij het aanmelden

## Routes afschermen

Als laatste moeten we nog routes kunnen afschermen voor ingelogde gebruikers. Hiervoor definiëren we zelf een component `PrivateRoute`.

- als we de credentials aan het ophalen zijn, dan toont dit een loading indicator
- als we aangemeld zijn, dan retourneren we een Outlet voor de weergave van de child routes
- en anders navigeren we naar de login pagina

Hiervoor dienen we de `AuthProvider` eerst aan te passen. We moeten weten wanneer we de credentials aan het ophalen zijn en of we zijn aangemeld. Hiervoor voegen we 2 state variabelen toe, respectievelijk `ready` en `isAuthed`, die worden ingesteld als het token is opgehaald. We plaatsen deze variabelen ook in de context.

`Auth.context.jsx`

```jsx
//..
export const AuthProvider = ({ children }) => {
  const [ready, setReady] = useState(false); // 👈
  const [isAuthed, setIsAuthed] = useState(false); // 👈
  //..

  useEffect(() => {
    api.setAuthToken(token);
    setIsAuthed(Boolean(token)); // 👈
    setReady(true); // 👈
  }, [token]);
  //..

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
    [token, user, error, ready, loading, isAuthed, login, logout]
  ); // 👈

  //..
};
```

Nu kunnen we de `PrivateRoute` component aanmaken.

`src/components/PrivateRoute.jsx`

```jsx
import { Navigate, Outlet, useLocation } from 'react-router-dom'; // 👈 3 en 4
import { useAuth } from '../contexts/Auth.context'; // 👈 2

// 👈 1
export default function PrivateRoute() {
  const { ready, isAuthed } = useAuth(); // 👈 2
  const { pathname } = useLocation(); // 👈 4

  const loginPath = `/login?redirect=${pathname}`; // 👈 4

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
  } // 👈 2

  if (isAuthed) {
    return <Outlet />;
  } // 👈 3

  return <Navigate replace to={loginPath} />; // 👈 4
}
```

1. We definiëren een `PrivateRoute` component.
2. Als we de credentials aan het controleren zijn, tonen we de loading indicator
3. Als de gebruiker ingelogd is, dan retourneren we een Outlet component voor de weergave van de child routes
4. Als de gebruiker niet ingelogd is, dan sturen we hem door naar de login. Omdat we de from URL niet op voorhand kennen, moeten we deze ophalen via useLocation. Na het aanmelden willen we terug navigeren naar de from pagina

Tot slot maken we gebruik van deze component om onze Routes af te schermen. `Transactions` en `Places` dienen te worden afgeschermd. `ProtectedRoute` wordt de parent component, en in de `Outlet` component worden de children gerendered als de gebruiker is aangemeld.

`src/main.jsx`

```jsx
//..
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
        element: <PrivateRoute />,
        children: [
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
        element: <PrivateRoute />,
        children: [
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
//..
```

## NavBar: afwerking

`src/components/Navbar.jsx`

```jsx
//..
import { useAuth } from '../contexts/Auth.context';

export default function Navbar() {
  const { isAuthed } = useAuth();// 👈 2
  //..

  return (
    <nav className={`navbar sticky-top bg-${theme} mb-4`}>
      <div className="container-fluid flex-column flex-sm-row align-items-start align-items-sm-center">
        <div className="nav-item my-2 mx-sm-3 my-sm-0">
          <Link className="nav-link" to="/">Transactions</Link>
        </div>
        <div className="nav-item my-2 mx-sm-3 my-sm-0">
          <Link className="nav-link" to="/places">Places</Link>
        </div>
        <div className="flex-grow-1"></div>{/* 👈 1*/}

        { // 👈 3
          isAuthed
            ? (
              <div className="nav-item my-2 mx-sm-3 my-sm-0">
                <Link className="nav-link" to="/logout">Logout</Link>
              </div>
            )// 👈 4
            : (
              <div className="nav-item my-2 mx-sm-3 my-sm-0">
                <Link className="nav-link" to="/login">Login</Link>
              </div>
            )// 👈 5
        }

        <button className="btn btn-secondary" type="button" onClick={toggleTheme}>
          {
            theme==='dark'
              ? <IoMoonSharp />
              : <IoSunny />
          }
        </button>
      </div>
    </nav>
  );
}
```

We moeten nog een paar items aan onze navigatie toevoegen:

- login
- uitloggen

Niet alle knopen mogen tegelijk getoond worden, bv. enkel uitloggen als je aangemeld bent.

1. we voegen een div toe om deze knoppen rechts uit te lijnen, deze div vult de lege ruimte
2. we halen op of er een gebruiker ingelogd is
3. en tonen afhankelijk van de waarde andere knoppen
4. als we aangemeld zijn, tonen we de logout-knop
5. in het andere geval, tonen we de login-knop

We dienen nog een `Logout` component aan te maken.

`src/pages/Logout.jsx`

```jsx
import { useEffect } from 'react'; // 👈 1
import { useAuth } from '../contexts/Auth.context'; // 👈 1

export default function Logout() {
  const { isAuthed, logout } = useAuth(); // 👈 1

  useEffect(() => {
    logout();
  }, [logout]); // 👈 1

  if (isAuthed) {
    return (
      <div className='container'>
        <div className='row'>
          <div className='col-12'>
            <h1>Logging out...</h1>
          </div>
        </div>
      </div>
    );
  } // 👈 2

  return (
    <div className='container'>
      <div className='row'>
        <div className='col-12'>
          <h1>You were successfully logged out</h1>
        </div>
      </div>
    </div>
  ); // 👈 3
}
```

1. Ook hier maken we gebruik van een `useEffect` voor het uitloggen.
2. Als de gebruiker aan het uitloggen is (nog aangemeld is), geven we een loading indicator weer
3. Als de gebruiker is uitgelogd, geven we een melding weer

Voeg de route naar Logout toe aan `src/main.jsx`

```jsx
import Logout from './pages/Logout';
//...
const router = createBrowserRouter([
  {
    element: <Layout />,
    children: [
      {
        path: '/',
        element: <Navigate replace to="/transactions" />,
      },
      {
        path: '/login',
        element: <Login />,
      },
      {
        path: '/logout',
        element: <Logout />,
      },//....
      
      ]}])
      //...
```

## Oefening

- maak een Register component op de URL /register
- deze bevat een formulier met 4 velden:
  - name
  - email
  - password
  - confirmPassword
- alle velden zijn verplicht + de waarde van het confirmPassword veld moet gelijk zijn aan de waarde van het password veld
  - gebruik hiervoor de validate functie van register
  - je moet hiervoor de validationRules binnen de component zetten, gebruik een useMemo om dit object te maken
  - om een waarde van een veld op te vragen gebruik je getValues
- als de gebruiker reeds aangemeld is, moet deze pagina hem doorsturen naar de / route
