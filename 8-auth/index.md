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

Alvorens we kunnen inloggen, moeten we onze API calls definiëren. Dit doen we in het bestand `src/api/index.ts`. Definieer hierin een functie `post` die alle gegeven data als body verzendt naar de gegeven URL. Voer het HTTP request uit binnen deze functie en geef het response terug.

```js
// src/api/index.ts
export const post = async <T, U>(url: string, { arg }: { arg: T }): Promise<U> => {
  const { data } = await axios.post(`${baseUrl}/${url}, arg);
  return data;
};
```

## AuthProvider

Vervolgens maken we een `AuthProvider` aan. Deze provider zal de aangemelde gebruiker bijhouden en de nodige functies voorzien om aan te melden en uit te loggen.

### Context opstellen

We maken gebruik van een context om alles omtrent authenticatie en autorisatie bij te houden. Maak een bestand `src/contexts/auth/Auth.context.tsx` aan met volgende inhoud:

```tsx
// src/contexts/Auth.context.tsx
import {
  createContext, // 👈 1
  useState, // 👈 4
} from 'react';
import useSWRMutation from 'swr/mutation'; // 👈 8
import * as api from '../../api'; // 👈 8
import useSWR from 'swr';

export const JWT_TOKEN_KEY = 'jwtToken'; // 👈 12
export const AuthContext = createContext(undefined); // 👈 1

// 👇 12
export interface AuthUser {
  id: string;
  name: string;
  email: string;
}
// 👇 2
export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [token, setToken] = useState<string | null>(
    localStorage.getItem(JWT_TOKEN_KEY),
  ); // 👈 4

  // 👇 12
  const {
    data: user,
    isLoading: userLoading,
    error: userError,
  } = useSWR<AuthUser>(token ? 'users/me' : null, api.getById);

  const {
    trigger: doLogin,
    isMutating: loginLoading,
    error: loginError,
  } = useSWRMutation('sessions', api.post); // 👈 6

  const setSession = (newToken: string) => {
    setToken(newToken);
    localStorage.setItem(JWT_TOKEN_KEY, newToken);
  }; // 👈 7

  // 👇 5
  const login = async (email: string, password: string): Promise<boolean> => {
    try {
      const { token: newToken } = (await doLogin({ email, password })) as {
        token: string;
      }; // 👈 6
      setSession(newToken); // 👈 7
      return true; // 👈 8
    } catch (error) {
      console.error(error);
      return false; // 👈 8
    }
  };

  // 👇 10
  const logout = () => {
    setToken(null);
    localStorage.removeItem(JWT_TOKEN_KEY);
  };

  // 👇 5 en 9 en 11 en 12
  const value = {
    token,
    user,
    error: loginError || userError,
    loading: loginLoading || userLoading,
    login,
    logout,
  };

  // 👇 3
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

1. Creëer een nieuwe context. Voorlopig defaulten we de waarde naar `undefined`. We zullen later een type toevoegen.
2. Maak een `AuthProvider` aan.
3. Retourneer reeds de kinderen gewrapped in de `AuthContext.Provider`.
4. Definieer een state-variabelen om onze JWT bij te houden. We moeten zorgen dat de token behouden blijft tussen de verschillende keren dat we naar de website gaan. Hiervoor moeten we de token opslaan in `localStorage`. We halen de huidige token uit `localStorage` als initiële waarde van onze state-variabele. We houden de `localStorage` key bij in een globale constante.
5. We definiëren een functie waarmee we een gebruiker kunnen aanmelden.
6. Roep de API aan om een gebruiker aan te melden. We maken hiervoor gebruik van de `useSWRMutation` hook. Deze hook handelt automatisch de loading (via `isMutating`) en error state voor ons af. Voeg `error` en `loading` toe aan de context.
7. Als alles goed ging, houden we de JWT bij en plaatsen we deze in `localStorage`.
8. We retourneren ook `true` zodat we kunnen weten of het aanmelden gelukt is. Indien iets fout ging, retourneren we `false`.
9. Voeg ook deze `login`functie toe aan de context.
10. We voorzien ook een functie om een gebruiker terug uit te loggen. Uitloggen is zo eenvoudig als de token verwijderen. Herinner je: een JWT is stateful, de server stateless. Met andere woorden een JWT bevat alle nodige informatie, een server valideert deze. Gooien we de JWT weg, dan kunnen we niet meer aan de beveiligde routes.
11. We voegen ook deze functie toe aan de context.
12. We halen ook de `user` gegevens op en voegen de waarde toe aan de context. Vervolledig `error` en `loading` in de context.

Dan moeten we uiteraard ook nog een hook voorzien waarmee we aan onze volledige Context waarde kunnen ophalen. Maak een file `index.ts` aan in de `contexts/auth` folder. Plaats ook de andere exports uit `Auth.context.tsx` in deze file en pas de imports in `Auth.context.tsx` aan.

```ts
// src/contexts/auth/index.ts
import { createContext, useContext } from 'react';

// 👇 1
export interface AuthUser {
  id: string;
  name: string;
  email: string;
}

// 👇 2
interface AuthContextType {
  token: string | null;
  user: AuthUser | undefined;
  error: Error | undefined;
  loading: boolean;
  login: (email: string, password: string) => Promise<boolean>;
  logout: () => void;
}

export const JWT_TOKEN_KEY = 'jwtToken'; // 👈 3

export const AuthContext = createContext<AuthContextType | undefined>(
  undefined,
); // 👈 4

export const useAuth = (): AuthContextType => {
  // 👈 5
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within an AuthProvider');
  return context;
};
```

1. `AuthUser` beschrijft de structuur van het ingelogde gebruikersobject. Het wordt geëxporteerd zodat andere bestanden (bv. componenten die de gebruiker tonen) dit type kunnen hergebruiken zonder het opnieuw te definiëren.
2. `AuthContextType` beschrijft alle waarden en functies die de context ter beschikking stelt aan consumers:
   - `token`: de JWT-string, of `null` als de gebruiker niet ingelogd is.
   - `user`: het gebruikersobject van het type `AuthUser`, of `undefined` als er nog geen gebruiker opgehaald is.
   - `error` / `loading`: de fout- en laadtoestand van de actieve authenticatieoperatie.
   - `isAuthed`: `true` als er een geldige token aanwezig is — handig om snel te controleren of de gebruiker ingelogd is.
   - `ready`: `true` zodra de initiële check (bv. token uit `localStorage` laden en valideren) afgerond is. Voorkomt dat de UI kort een "niet ingelogd"-staat toont terwijl de token nog geladen wordt.
   - `login` / `logout` / `register`: de acties die consumers kunnen aanroepen om de authenticatiestatus te wijzigen.
3. `JWT_TOKEN_KEY` is een constante voor de sleutelnaam in `localStorage`.
4. `AuthContext` wordt aangemaakt met `undefined` als standaardwaarde en het type `AuthContextType | undefined`. Dit dwingt consumers via de `useAuth` hook te gaan — wie de context buiten een `AuthProvider` gebruikt krijgt een duidelijke fout.
5. `useAuth` is de custom hook die consumers gebruiken in plaats van `useContext(AuthContext)` rechtstreeks. De check `if (!context)` gooit een begrijpelijke foutmelding als de hook buiten de `AuthProvider` aangeroepen wordt.

We wrappen de hele app in de `AuthProvider` (in `src/main.tsx`):

```tsx
// src/main.tsx
// ...
import { AuthProvider } from './contexts/auth/Auth.contex.tsx';

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

Vervolgens gaan we een instantie van axios configureren voor het gebruik van een Bearer token. We passen hiervoor `src/api/index.ts` aan:

```tsx
// src/api/index.ts
import axiosRoot from 'axios'; // 👈 1
import { JWT_TOKEN_KEY } from '@/contexts/auth'; // 👈 3
import { JWT_TOKEN_KEY } from '../contexts/auth';
import type { PaginatedResponse } from '../types';

const baseUrl = import.meta.env.VITE_API_URL;
if (!baseUrl) {
  throw new Error('VITE_API_URL environment variable is not set');
}

// 👇 2
const axios = axiosRoot.create({
  baseURL: baseUrl,
  timeout: 10_000,
});

// 👇 3
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem(JWT_TOKEN_KEY);
  if (token) {
    config.headers['Authorization'] = `Bearer ${token}`;
  }
  return config;
});

// 👇 4
axios.interceptors.response.use(null, (error) => {
  if (error.response && error.response.status === 401) {
    localStorage.removeItem(JWT_TOKEN_KEY);
    window.location.href = '/login';
  }
  return Promise.reject(error);
});

export async function getAll<T>(url: string): Promise<T[]> {
  const { data } = await axios.get(url); // 👈 2
  return data.items;
}

// ...
```

1. We hernoemen de import van axios naar `axiosRoot` omdat we zelf een constante `axios` gaan aanmaken — zonder hernoeming zou dit een naamconflict geven.
2. We maken een geconfigureerde axios-instantie aan via `axiosRoot.create()`. Door een `baseURL` en `timeout` in te stellen hoeven we die niet bij elke API-call te herhalen. Alle bestaande functies (`getAll`, `getById`, `post`, ...) vervangen hun rechtstreekse `axios`-aanroep door deze nieuwe instantie en verwijderen de `baseUrl`-parameter.
3. Een **request interceptor** onderschept elke uitgaande HTTP-request vóór ze verstuurd wordt. Als er een token aanwezig is in `localStorage`, wordt die als `Authorization: Bearer <token>` header toegevoegd. Zo zijn alle API-calls automatisch geauthenticeerd zonder dat elke functie de token zelf hoeft op te halen.
4. Een **response interceptor** onderschept elke binnenkomende HTTP-response. Bij een `401 Unauthorized` fout — wat betekent dat de token verlopen of ongeldig is — wordt de token uit `localStorage` verwijderd en de gebruiker doorgestuurd naar `/login`. De fout wordt daarna toch doorgegeven (`Promise.reject`) zodat de aanroeper indien nodig zelf ook kan reageren.

## Login component

Om te kunnen aanmelden hebben we een `Login` component op de URL `/login` nodig. Deze component bevat een formulier met twee velden: `email` en `password`, beide zijn verplicht. Onderaan het formulier staan ook twee knoppen: "Sign in" en "Cancel". Deze knoppen implementeren we straks. We kunnen hier de `LabelInput` component hergebruiken.

Voeg de `Login` component toe in `src/pages/Login.tsx`:

```tsx
// src/pages/Login.tsx
import { useForm, FormProvider } from 'react-hook-form';
import * as z from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { Button } from '@/components/ui/button';
import {
  Card,
  CardContent,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { FieldGroup } from '@/components/ui/field';
import LabelInput from '../components/LabelInput';

const loginSchema = z.object({
  email: z.email('Invalid email address'),
  password: z.string().min(1, 'Password is required'),
});

type LoginFormValues = z.infer<typeof loginSchema>;

export default function Login() {
  const {
    control,
    handleSubmit,
    reset,
    formState: { isSubmitting },
  } = useForm<LoginFormValues>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: 'thomas.aelbrecht@hogent.be',
      password: '12345678',
    },
  });

  return (
    <div className='flex justify-center pt-12'>
      <Card className='w-full max-w-sm'>
        <FormProvider {...form}>
          <form>
            <CardHeader>
              <CardTitle className='text-2xl'>Sign in</CardTitle>
            </CardHeader>

            <CardContent className='py-4'>
              <FieldGroup>
                <LabelInput
                  label='Email'
                  name='email'
                  placeholder='your@email.com'
                  type='text'
                />
                <LabelInput
                  label='Password'
                  name='password'
                  placeholder='password'
                  type='password'
                />
              </FieldGroup>
            </CardContent>

            <CardFooter className='flex justify-end gap-2'>
              <Button type='submit'>Sign in</Button>
              <Button type='button' variant='outline'>
                Cancel
              </Button>
            </CardFooter>
          </form>
        </FormProvider>
      </Card>
    </div>
  );
}
```

Zorg ervoor dat deze component getoond wordt op de URL `/login`. Voeg deze route toe aan de router in `src/main.tsx` en importeer de `Login` component:

```tsx
      {
        path: '/login',
        element: <Login />,
      },
```

Om aan te melden maken we gebruik van onze `useAuth` hook. Pas `src/pages/Login.tsx` als volgt aan:

```tsx
// src/pages/Login.tsx
import { useNavigate } from 'react-router';
import { FormProvider, useForm } from 'react-hook-form';
import * as z from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { useAuth } from '../contexts/auth'; // 👈 1
import Error from '../components/Error';
import { Button } from '../components/ui/button';
import {
  Card,
  CardContent,
  CardFooter,
  CardHeader,
  CardTitle,
} from '../components/ui/card';
import { FieldGroup } from '../components/ui/field';
import LabelInput from '../components/LabelInput';

const loginSchema = z.object({
  email: z.email('Invalid email address'),
  password: z.string().min(1, 'Password is required'),
});

type LoginFormValues = z.infer<typeof loginSchema>;

export default function Login() {
  const { error, loading, login } = useAuth(); // 👈 1
  const navigate = useNavigate(); // 👈 3

  // 👇 7
  const form = useForm<LoginFormValues>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: 'thomas.aelbrecht@hogent.be',
      password: '12345678',
    },
  });

  const handleLogin = async ({ email, password }: LoginFormValues) => {
    const loggedIn = await login(email, password); // 👈 2
    if (loggedIn) {
      navigate('/', { replace: true }); // 👈 3
    }
  }; // 👈 1

  return (
    <div className='flex justify-center pt-12'>
      <Card className='w-full max-w-sm'>
        <FormProvider {...form}>
          <form onSubmit={form.handleSubmit(handleLogin)}>
            {/* 👆 1*/}
            <CardHeader>
              <CardTitle className='text-2xl'>Sign in</CardTitle>
            </CardHeader>

            <CardContent className='py-4'>
              <Error error={error} />
              {/* 👈 5 */}
              <FieldGroup>
                <LabelInput
                  label='Email'
                  name='email'
                  placeholder='your@email.com'
                  type='text'
                />
                <LabelInput
                  label='Password'
                  name='password'
                  placeholder='password'
                  type='password'
                />
              </FieldGroup>
            </CardContent>

            <CardFooter className='flex justify-end gap-2'>
              <Button type='submit' disabled={loading}>
                Sign in
              </Button>
              {/* 👆 4*/}
              <Button
                type='button'
                variant='outline'
                onClick={() => form.reset()}
              >
                Cancel
              </Button>
              {/* 👆 6*/}
            </CardFooter>
          </form>
        </FormProvider>
      </Card>
    </div>
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

Pas de interface aan in `src/contexts/auth/index.ts`. Voeg deze 2 props toe aan de `AuthContextType` interface:

```ts
    isAuthed: Boolean(token),
    ready: !userLoading,
```

en pas dan de AuthProvider aan in `src/contexts/auth/Auth.context.tsx`:

```tsx
// src/contexts/auth/Auth.context.tsx
// // ...

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  //...
  // 👇
  const value = {
    token,
    user,
    error: loginError || userError || registerError,
    loading: loginLoading || userLoading || registerLoading,
    isAuthed: Boolean(token),
    ready: !userLoading,
    login,
    logout,
  };

  // ...
};
```

Nu kunnen we de `PrivateRoute` component aanmaken. Maak een bestand `src/components/PrivateRoute.tsx` aan met volgende inhoud:

```tsx
// src/components/PrivateRoute.tsx
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

Tot slot maken we gebruik van deze component om onze routes af te schermen. `Transactions` en `Places` dienen afgeschermd te worden. `PrivateRoute` wordt de parent component, en in de `Outlet` component worden de children gerenderd als de gebruiker is aangemeld. Pas `src/main.tsx` als volgt aan:

```tsx
// src/main.tsx
// ...
import PrivateRoute from './components/PrivateRoute.tsx'; // 👈

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
        path: '/register',
        element: <Register />,
      },
      {
        path: '/transactions',
        element: <PrivateRoute />, // 👈
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
        element: <PrivateRoute />, // 👈
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
            element: <Navigate to='services' replace />,
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
      {
        path: '*',
        element: <NotFound />,
      },
    ],
  },
]);
```

We dienen er ook voor te zorgen dat na het inloggen genavigeerd wordt naar de pagina die de gebruiker wou raadplegen. Pas hiervoor de `Login` component aan:

```tsx
// src/pages/Login.tsx
import { useNavigate, useLocation } from 'react-router'; // 👈
//...

export default function Login() {
  const { search } = useLocation(); // 👈
  //...
  const handleLogin = async ({ email, password }: LoginFormValues) => {
    const loggedIn = await login(email, password);
    // 👇
    if (loggedIn) {
      const params = new URLSearchParams(search);
      const redirect = params.get('redirect');
      const safePath =
        redirect?.startsWith('/') && !redirect.startsWith('//')
          ? redirect
          : '/';
      navigate(safePath, { replace: true });
    }
  };

  //...
}
```

Haal de querystring op en kijk of de key `redirect` voorkomt en navigeer naar de startpagina of naar de waarde van de `redirect` query parameter.

## NavBar: afwerking

Als laatste voegen we nog een login- en logout-knop toe aan onze navbar. Deze knoppen mogen ook niet altijd getoond worden. Pas hiervoor `src/components/Navbar.tsx` aan als volgt:

```tsx
// src/components/Navbar.tsx
// ...
import { Button , buttonVariants} from './ui/button';// 👈 1
import { useAuth } from '../contexts/auth';// 👈 1
import { cn } from '@/lib/utils';// 👈 1

export default function Navbar() {

   // 👇 1
const AuthButtons = () => {
  const { isAuthed } = useAuth();
  return isAuthed ? (
    <Link to="/logout" className={cn(buttonVariants({ variant: 'outline', size: 'sm' }))}>
      Logout
    </Link>
  ) : (
    <>
      <Link to="/login" className={cn(buttonVariants({ size: 'sm' }))}>
        Login
      </Link>
      <Link to="/register" className={cn(buttonVariants({ variant: 'ghost', size: 'sm' }))}>
        Register
      </Link>
    </>
  );
};

  // ...

  return (
    //...  👇 2
    <div className="hidden md:flex items-center gap-2 ml-auto">
          <AuthButtons />
          <ThemeToggle />
        </div>
    //...
    <div className="flex items-center gap-2 pt-2 border-t w-full">
            <AuthButtons />
            <ThemeToggle />
    </div>
     //...
  );
}
```

1. Maak een component `AuthButtons`. Controleer of er een gebruiker ingelogd is, en toon afhankelijk van de waarde andere knoppen.
2. Voeg de `AuthButtons` component toe aan de navbar.

We dienen nog een `Logout` component aan te maken. Maak een nieuw bestand `src/pages/Logout.tsx` met volgende inhoud:

```tsx
// src/pages/Logout.tsx
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
    return <h1>Logging out...</h1>;
  }
  // 👇 3
  return <h1>You were successfully logged out</h1>;
}
```

1. We maken gebruik van een `useEffect` voor het uitloggen.
2. Als de gebruiker aan het uitloggen is (en dus nog aangemeld is), geven we aan dat we aan het uitloggen zijn.
3. Als de gebruiker is uitgelogd, geven we een melding weer.

Voeg de route naar `Logout` component toe aan `src/main.tsx` en importeer de `Logout` component:

```tsx
// src/main.tsx
  {
    path: '/logout',
    element: <Logout />,
  },
```

## Oefening - Pas TransactionForm aan

De gebruiker moet nu niet langer de `userId` kunnen ingeven bij het aanmaken van een nieuwe transactie. De backend gebruikt de userId van de aangemelde gebruiker. Pas hiervoor de `TransactionForm` component aan.

- verwijder de `userId` uit de validatieregels.
- verwijder de defaultwaarde voor `userId` in useForm defaultValues.
- verwijder het `LabelInput` veld voor de `userId`.

## Oefening - Registreren

Een gebruiker dient zich te kunnen registreren op de site.

1. Pas hiervoor `Auth.context.tsx` aan.
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
   - Voeg de route naar de register pagina toe aan `src/main.tsx`.
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

```

```
