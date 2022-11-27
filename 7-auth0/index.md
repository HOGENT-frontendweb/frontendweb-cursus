# Authenticatie & Autorisatie

## Inleiding

Authenticatie is bewijzen wie je bent, heel vaak met een gebruikersnaam en wachtwoord, en meer en meer in combinatie met TOTP (time based one time password), SMS en/of security keys als deel van two factor authentication (2FA).

Autorisatie is dan weer kijken of een gebruiker de juiste rechten heeft om toegang tot (een deel van) je webapplicatie te krijgen.

Zelf gebruikersnamen en wachtwoorden opslaan is niet triviaal, je kan niet zomaar een tabel maken waar je een wachtwoord en gebruikersnaam in opslaat, wachtwoorden moeten gehashed worden, met een salt om rainbow table attacks tegen te gaan.
Voor sommige toepassingen verwachten gebruikers dat ze via hun google of facebook account kunnen inloggen; soms vereist een platform zelfs dat je bepaalde inlogsystemen integreert (iOS en Apple login bijvoorbeeld).
Als je dan ook nog eens alles van 2FA wilt integreren besef je dat dit allemaal maken niet alleen niet triviaal is, maar ook best veel werk.

Daarom wordt er meer en meer gegrepen naar een third party service die deze taken op zich neemt, één zo'n service is [Auth0](https://auth0.com/).

## Auth0

De tagline van [Auth0](https://auth0.com/) is "makes authentication and authorization easy". In plaats van alles zelf te beginnen implementeren gaan we een integratie met Auth0 maken.

### Werking
De verschillende stappen:

![Werking Auth0](./images/auth0.png ':size=80%')

1. De gebruiker wil zich aanmelden in de applicatie.
2. De SPA stuurt een verzoek naar de Auth0 Server om de gebruiker te autoriseren en een [access token](https://auth0.com/docs/secure/tokens/access-tokens) (JWT) te krijgen voor de Backend API. De Backend API moet worden ingesteld en geregistreerd bij Auth0 en het autorisatieverzoek moet een parameter `audience={YOUR_API_IDENTIFIER}` bevatten. De Auth0 server retourneert ook een [ID token](https://auth0.com/docs/secure/tokens/id-tokens) die informatie over de gebruiker bevat.
3. De SPA stuurt namens de gebruiker een verzoek naar de Backend API voor het ophalen van de transacties, waarbij de access token in de header wordt verzonden. De ID token kan niet gebruikt worden om een Backend API aan te spreken!
4. De Backend API verifieert de token. Als de Backend API geen bestaande en geldige Management API token heeft, vraagt deze om een nieuwe. Indien de token gevalideerd is en de gebruiker heeft de juiste rechten, dan retourneert de backend de transacties. Indien de token niet correct is, wordt een `403: Forbidden` geretourneerd.

### JWT
Het zou natuurlijk bijzonder onhandig zijn als je voor elke request opnieuw zou moeten inloggen, zeker bij moderne webapplicaties die vele requests gebruiken om één pagina op te bouwen. We moeten dus ergens kunnen 'onthouden' dat iemand ingelogd is, op een veilige manier.

Hiervoor kan je (o.a.) een [JSON Web Token (JWT)](https://auth0.com/docs/secure/tokens/json-web-tokens) gebruiken, dat is in se een (base64) string die bij elke request meegestuurd wordt in de `Authorization` header. Een JWT bestaat uit drie delen (zie een voorbeeld op https://jwt.io)

- een deel met meta informatie over het token (hash algoritme)
- een deel met de echte data, de payload (wie ingelogd is, wat de rechten zijn, wanneer de token vervalt, etc)
- een hash signature waarmee kan gecontroleerd worden dat het een echte token is.

Het concept is als volgt: de server kan een signature (= handtekening) genereren voor een bepaalde payload. Deze signature wordt berekend o.b.v. een secret dat enkel door de server gekend is. Als iemand anders een token probeert te faken, zal de server dit altijd merken. Als een client correct inlogt, krijgt hij zo'n token van de server. Deze token moet bij elk request door de client meegestuurd worden (natuurlijk enkel waar authenticatie/autorisatie nodig is).

Als zo'n request met token binnenkomt, kan de server de signature opnieuw genereren. Als het overeenkomt met het origineel weet hij dat het token van hem afkomstig is en de payload dus geldig is (en dus de gebruiker is wij hij beweert te zijn).

Dat wil dus zeggen dat iedereen die zo'n token heeft effectief een ingelogde gebruiker is (je hoeft dus iemand zijn username en wachtwoord niet te kennen als je zijn token kan bemachtigen). Daarom vervallen tokens na een tijd, en zal de gebruiker opnieuw moeten inloggen.

## Implementatie in React

De integratie met React is zeer volledig. Daarom gaan hier dus gewoon linken naar een paar tutorials van Auth0. Probeer deze te integreren met je eigen applicatie:

1. Aanmelden/afmelden: <https://auth0.com/docs/quickstart/spa/react/01-login>
2. Een API aanspreken: <https://auth0.com/docs/quickstart/spa/react/02-calling-an-api>
   - Hint: je zal eigen hooks moeten definiëren voor de API calls: `useTransactions` en `usePlaces`. Deze hooks retourneren de nodige functies in een object die de API calls uitvoeren.
3. Routes afschermen: <https://github.com/remix-run/react-router/tree/dev/examples/auth>

Onderstaande secties doorlopen in principe deze tutorials, maar vormen gaandeweg de oplossing op onze GitHub.

### Stap 1 - Aanmelden/afmelden

#### 1. Bepaal de Application keys

*Bron: <https://auth0.com/docs/quickstart/spa/react/01-login#get-your-application-keys>*

Vergeet niet de callback urls en allowed web origins correct in te stellen of inloggen en/of API callen zal niet lukken.

Maak een `.env` bestand aan met volgende inhoud

```json
REACT_APP_AUTH0_DOMAIN={YOUR_DOMAIN}
REACT_APP_AUTH0_CLIENT_ID={YOUR_CLIENTID}
```

De waarden voor domain en Client ID vind je op het Auth0 Dashboard > Applications > Applications > {JOUW APP}

#### 2. Installeer de Auth0 React SDK

*Bron: <https://auth0.com/docs/quickstart/spa/react/01-login#install-the-auth0-react-sdk>*

```bash
yarn add @auth0/auth0-react
```

#### 3. Configureer de Auth0Provider component

*Bron: <https://auth0.com/docs/quickstart/spa/react/01-login#configure-the-auth0provider-component>*

In `src/contexts` folder, maak de file `MyAuth0Provider` component aan.
```jsx
import { Auth0Provider } from '@auth0/auth0-react';
function MyAuth0Provider({ children }) {
  const domain = process.env.REACT_APP_AUTH0_DOMAIN;
  const clientId = process.env.REACT_APP_AUTH0_CLIENT_ID;
  return (
    <Auth0Provider
      domain={domain}
      clientId={clientId}
      redirectUri={`${window.location.origin}/transactions`}
      cacheLocation="localStorage" // 👈 extra, niet in tutorial!
    >
      {children}
    </Auth0Provider>
  );
}
export default MyAuth0Provider;
```

De Auth0 React SDK gebruikt de React Context om de authenticatiestatus van de gebruikers te beheren. Een manier om Auth0 te integreren met de React app is door de root component te verpakken met een `Auth0Provider` uit de SDK. Stel de properties in

- `domain` en `clientId`: zie instellingen van jouw app in het Auth0 Dashboard
- `redirectUri`: de URL waarnaar de gebruiker navigeert als hij geauthenticeerd is door Auth0, deze URL moet toegevoegd zijn aan de `Allowed Callback URLs` in je Auth0 setup.
- `cacheLocation`: waar de auth tokens en andere info opgeslagen worden

LET OP: deze redirectUri moet de juiste zijn, i.e. de pagina die geladen zal worden nadat je ingelogd bent, indien deze niet correct kan je 'rare' fouten krijgen. (isAuthenticated zal false rapporteren maar je krijgt toch geen login te zien als je op login klikt, dat soort dingen)

Verpak de root component in `index.js` met `MyAuth0Provider`:

```jsx
...
import MyAuth0Provider from './contexts/MyAuth0Provider'; // 👈

const root = createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <MyAuth0Provider> {/* 👈*/}
      <ThemeProvider>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </ThemeProvider>
    </MyAuth0Provider>{/* 👈*/}
  </React.StrictMode>
);
reportWebVitals();
```

#### 4. De Login component

*Bron <https://auth0.com/docs/quickstart/spa/react/01-login#add-login-to-your-application>*

Voeg een `LoginButton` component toe in de map `components/authentication`:

```jsx
import { useAuth0 } from '@auth0/auth0-react';
import { useCallback } from 'react';

function LoginButton() {
  const { loginWithRedirect } = useAuth0();

  const handleLogin = useCallback(

    async () => {
      loginWithRedirect();
    },
    [loginWithRedirect],
  );

  return (
    <button
      type="button"
      className="btn btn-primary"
      onClick={handleLogin}
    >
      Log In
    </button>
  );
}

export default LoginButton;
```

Door `loginWithRedirect()` uit te voeren, wordt de gebruiker omgeleid naar de [Auth0 Universal Login Page](https://auth0.com/universal-login), waar Auth0 de gebruiker kan authenticeren. Na succesvolle authenticatie zal Auth0 de gebruiker terugleiden naar de applicatie. Als het aanmelden succesvol is dan keren we terug naar de home pagina.

Voeg de `LoginButton` toe aan de `NavBar`. Bij klik op de knop wordt je omgeleid naar de [Auth0 Universal Login Page](https://auth0.com/universal-login).


#### 6. De Logout component

*Bron: <https://auth0.com/docs/quickstart/spa/react/01-login#add-logout-to-your-application>*

Voeg een `LogoutButton` component toe in de map `components/authentication`:

```jsx
import { useAuth0 } from '@auth0/auth0-react';

function LogoutButton() {
  const { logout } = useAuth0();
  return (
    <button
      type="button"
      className="btn btn-danger"
      onClick={() => logout({
        returnTo: window.location.origin,
      })}
    >
      Log Out
    </button>
  );
}

export default LogoutButton;
```

`logout()` leidt de gebruiker om naar je Auth0 logout endpoint (https://YOUR_DOMAIN/v2/logout). Daarna wordt de gebruiker onmiddellijk omgeleid naar je app.

#### 7. Inbouwen van aanmelden/afmelden

*Bron: <https://auth0.com/docs/quickstart/spa/react/01-login#show-user-profile-information>*

We voorzien een derde component `AuthenticationButton` in de map `components/authentication`:

```jsx
import { useAuth0 } from '@auth0/auth0-react';
import LoginButton from './LoginButton';
import LogoutButton from './LogoutButton';

export default function AuthenticationButton() {
  const {
    isAuthenticated,
    user,
  } = useAuth0(); // 👈 1

  if (isAuthenticated) { // 👈 2
    const { name, picture, givenName } = user;
    return (
      <div className="d-flex flex-row align-items-center">
        <div className="col">
          <img src={picture} alt={givenName} className="rounded" />
        </div>
        <div className="col">
          {name}
        </div>
        <div className="col">
          <LogoutButton />
        </div>
      </div>
    );
  }

  return <LoginButton />;
}
```

1. `isAuthenticated` property geeft aan of Auth0 de gebruiker reeds geauthenticeerd heeft. De `user` property bevat de informatie gerelateerd aan de aangemelde gebruiker.
2. Als de gebruiker nog niet is aangemeld, toon de `LoginButton`. In het andere geval: geef de foto en naam van de aangemelde gebruiker weer, samen met de `LogoutButton`.

#### 8. Pas de NavBar aan

*Bron: <https://auth0.com/docs/quickstart/spa/react/01-login#show-user-profile-information>*

```jsx
import AuthenticationButton from './authentication/AuthenticationButton'; // 👈

// ...
  <div className="d-flex  align-items-center">
    <AuthenticationButton /> {/* 👈*/}
    <button type="button" onClick={toggleTheme}>
      {
        theme === themes.dark ? <IoMoonSharp /> : <IoSunny />
      }
    </button>
  </div>
// ...
```

Start de applicatie. Nu moet je kunnen aanmelden via Auth0.

### Stap 2 - Calling the API

#### 1. Registratie Budget API

In stap 2 dienen we eerst de Budget API te registreren: <https://auth0.com/docs/get-started/auth0-overview/set-up-apis>.

Vul hiervoor onderstaande in

![Registratie API](./images/new_api.png ':size=80%')


In de settings van deze API gaan we Role-Based Access Control (RBAC) aanzetten, zodat we verschillende gebruikers verschillende toegangsrechten kunnen geven.

![Registratie API](./images/rbac_settings.png ':size=80%')

Het uiteindelijke doel is dat de server toegang tot bepaalde REST routes gaat afschermen. Met andere woorden de client zal inloggen bij Auth0, een token krijgen, en die dan meesturen met de headers van elke request. Onze server moet vervolgens kijken of deze token echt is en op basis daarvan requests al dan niet blokkeren.

#### 2. Configureer MyAuth0Provider component

Pas het `.env` bestand aan. Voeg onderstaande variabele toe, dit moet dezelfde waarde bevatten als de `AUTH_AUDIENCE` in onze backend:

```json
REACT_APP_AUTH0_API_AUDIENCE={YOUR_API_AUDIENCE}
```

Pas ook de `MyAuth0Provider` aan:

```jsx
import { Auth0Provider } from '@auth0/auth0-react';

function MyAuth0Provider({ children }) {
  const domain = process.env.REACT_APP_AUTH0_DOMAIN;
  const clientId = process.env.REACT_APP_AUTH0_CLIENT_ID;
  const audience = process.env.REACT_APP_AUTH0_API_AUDIENCE; // 👈 1

  return (
    <Auth0Provider
      domain={domain}
      audience={audience} // 👈 1
      clientId={clientId}
      edirectUri={`${window.location.origin}/transactions`}
      useRefreshTokens // 👈 2
    >
      {children}
    </Auth0Provider>
  );
}
export default MyAuth0Provider;
```

1. Voeg de `audience` toe, dit is de unieke identifier van je API. Auth0 gebruikt deze waarde om te bepalen tot welke API server(s) een gebruiker toegang heeft.
3. `useRefreshTokens`: om refresh tokens te kunnen opvragen (zie verder)

#### 3. Pas de transaction API aan

We moeten het token meesturen met de `Authorization` HTTP header bij elke request naar de server die authenticatie (en authorisatie) vereist. Het token kan worden opgevraagd via de `getAccessTokenSilently()` functie uit de `useAuth0()` hook. Deze functie retourneert een Promise die een access token teruggeeft. Deze token kan je gebruiken om een ​​beveiligde API aan te roepen. 

Deze functie kan het access en ID token vernieuwen gebruik makend van een [refresh token](https://auth0.com/docs/secure/tokens/refresh-tokens). Hiervoor dient `useRefreshTokens` ingesteld zijn in de `Auth0Provider`.

Vervolgens voeg je het toegangstoken toe aan de autorisatieheader van de API call. De API zorgt voor het valideren van de toegangstoken en het verwerken van de request.

Omdat we gebruik willen maken van de `useAuth0` hook en hooks enkel binnen React functies aangeroepen mogen worden, hebben we een probleem. De moeten onze transaction API omzetten naar een hook die de functies voor elk request teruggeeft.

Een custom hook is een JavaScript functie waarvan de naam begint met `use`, en die andere hooks kan aanroepen. Verder bevat het de code uit de API. De hook retourneert de functies in een object.

```jsx
import {
  useAuth0,
} from '@auth0/auth0-react';
import axios from 'axios';
import {
  useCallback,
} from 'react';

const baseUrl = `${process.env.REACT_APP_API_URL}/transactions`;

const useTransactions = () => { // 👈 1
  const {
    getAccessTokenSilently,
  } = useAuth0(); // 👈 2

  const getAll = useCallback(async () => {
    const token = await getAccessTokenSilently(); // 👈 3
    const {
      data,
    } = await axios.get(baseUrl, {
      headers: {
        Authorization: `Bearer ${token}`,
      },// 👈 4
    });

    return data.items;
  }, [getAccessTokenSilently]);

  const getById = useCallback(async (id) => {
    const token = await getAccessTokenSilently(); // 👈 3
    const {
      data,
    } = await axios.get(`${baseUrl}/${id}`, {
      headers: {
        Authorization: `Bearer ${token}`,
      }, // 👈 4
    });
    return data;
  }, [getAccessTokenSilently]);

  const save = useCallback(async (transaction) => {
    const token = await getAccessTokenSilently(); // 👈 3
    const {
      id,
      ...values
    } = transaction;
    await axios({
      method: id ? 'PUT' : 'POST',
      url: `${baseUrl}/${id ?? ''}`,
      data: values,
      headers: {
        Authorization: `Bearer ${token}`,
      }, // 👈 4
    });
  }, [getAccessTokenSilently]);

  const deleteById = useCallback(async (id) => {
    const token = await getAccessTokenSilently(); // 👈 3
    await axios.delete(`${baseUrl}/${id}`, {
      headers: {
        Authorization: `Bearer ${token}`,
      }, // 👈 4
    });
  }, [getAccessTokenSilently]);

  return {
    getAll,
    getById,
    save,
    deleteById,
  }; // 👈 5
};

export default useTransactions;
```

1. Definieer de custom hook.
2. Haal de `getAccessTokenSilently` functie op.
3. Vraag de token op.
4. Voeg de token toe aan de HTTP header. Bearer authentication (wordt ook token authenticatie genoemd) is een HTTP authenticatie methode die gebruik maakt van tokens. Bearer authenticatie betekent *give access to the bearer of this token* (bearer = houder).
5. Retourneer de functies zodat die gebruikt kunnen worden in de componenten of andere hooks.

#### 4. Gebruik de  useTransactions hook

Nu moeten we alle componenten die de transaction API gebruiken aanpassen, zoals bv. de `TransactionsList` component:

```jsx
import useTransactions from '../../api/transactions'; // 👈 1
// ...
export default function TransactionList() {
  // ...
  const transactionApi = useTransactions(); // 👈 2
  // ...
}
```

#### 5. Oefening

- Pas de andere componenten aan zodat ze gebruik maken van de `useTransactions` hook.
- - Pas de places API aan en roep deze hook aan in de betreffende componenten.

### Stap 3 - Routes afschermen

#### 1. Een route afschermen

Als laatste moeten we nog routes kunnen afschermen voor ingelogde gebruikers. Hiervoor definiëren we zelf een component, geïnspireerd op de component in de bovenvermelde documentatie

Maak in map `components/authentication` de component `RequireAuth` aan:

```jsx
import { useAuth0 } from '@auth0/auth0-react';
import { Navigate } from 'react-router';
import Loader from '../Loader';

export default function RequireAuth({ children }) { // 👈 1
  const { isAuthenticated, isLoading } = useAuth0();

  if (isLoading) { // 👈 2
    return <Loader loading />;
  }

  if (isAuthenticated) { // 👈 3
    return children;
  }

  return <Navigate to="/login" />;// 👈 4
}
```
1. De component ontvangt React children (= de component(en) die afgeschermd moeten worden).
2. Zorg ervoor dat het laden van de Auth0 SDK is voltooid voordat je toegang krijgt tot de property `isAuthenticated`. `isLoading` zal `false` zijn zolang Auth0 nog niet klaar is.
3. A.d.h.v. de property `isAuthenticated` van `useAuth0` hook controleer je of Auth0 de gebruiker geverifiëerd heeft voordat de component wordt weergegeven.
4. Indien niet aangemeld, ga naar de login pagina.

#### 2. Landingspagina

Het kan handig zijn om één landingspagina te maken. Deze pagina staat op de callback URL van Auth0, bv. `http://localhost:3000/login`. Het enige doel van deze pagina is om elke mogelijke auth state op te vangen:

- Er ging iets fout: de fout tonen
- Auth state is niet aan het laden en we zijn aangemeld: naar de home (`/`) navigeren
- Auth state is niet aan het laden en we zijn niet aangemeld: melding geven dat je aangemeld moet zijn om naar deze pagina te komen
- Alle andere gevallen: zeggen dat we de gebruiker aan het aanmelden zijn

Dit is ook de pagina waarnaar de `RequireAuth` component navigeert indien de gebruiker niet aangemeld is.

```jsx
import { useAuth0 } from '@auth0/auth0-react';
import { Navigate } from 'react-router-dom';
import Error from '../Error';
import LoginButton from './LoginButton';

export default function AuthLanding() {
  const { error, isAuthenticated, isLoading } = useAuth0();

  if (error) {
    <div className="container">
      <div className="row">
        <div className="col">
          <h1>Login failed</h1>
          <p>
            Sorry, we were unable to sign you in, the error below might be useful.
          </p>
          <Error error={error} />
          <LoginButton />
        </div>
      </div>
    </div>;
  }

  if (!isLoading && isAuthenticated) {
    return <Navigate to="/" />;
  }

  if (!isLoading && !isAuthenticated) {
    return (
      <div className="container">
        <div className="row">
          <div className="col">
            <h1>Login required</h1>
            <p>You need to login to access this page.</p>
            <LoginButton />
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="container">
      <div className="row">
        <div className="col">
          <h1>Signing in</h1>
          <p>
            Please wait while we sign you in!
          </p>
        </div>
      </div>
    </div>
  );
}
```

Pas in `MyAuthProvider` component ook de redirect pagina aan:

```jsx
<Auth0Provider
  domain={domain}
  audience={audience}
  clientId={clientId}
  redirectUri={`${window.location.origin}/login`}{/* 👈 */}
  useRefreshTokens
>
  {children}
</Auth0Provider>
```

Pas ook de callback URL aan in de instellingen van jouw applicatie via het Auth0 Dashboard.

#### 3. De routes

Als laatste schermen we de nodige routes af met onze `RequireAuth` component. We mogen ook de landingspagina niet vergeten definiëren op de URL `/login`, uiteraard zonder `RequireAuth`.

```jsx
import { Routes, Route, Navigate } from 'react-router-dom';
import TransactionList from './components/transactions/TransactionList';
import TransactionForm from './components/transactions/TransactionForm';
import PlacesList from './components/places/PlacesList';
import {
  useTheme,
} from './contexts/Theme.context';
import Navbar from './components/Navbar';
import RequireAuth from './components/authentication/RequireAuth';
import AuthLanding from './components/authentication/AuthLanding';

function App() {
  const {
    theme,
    oppositeTheme,
  } = useTheme();

  return (
    <div className={`container-xl bg-${theme} text-${oppositeTheme}`}>
      <Navbar />

      <Routes>
        <Route path="/" element={<Navigate replace to="/transactions" />} />
        <Route path="/transactions">
          <Route
            index
            element={(
              <RequireAuth>
                <TransactionList />
              </RequireAuth>
            )}
          />
          <Route
            path="add"
            element={(
              <RequireAuth>
                <TransactionForm />
              </RequireAuth>
            )}
          />
          <Route
            path="edit/:id"
            element={(
              <RequireAuth>
                <TransactionForm />
              </RequireAuth>
            )}
          />
        </Route>

        <Route
          path="/places"
          element={(
            <RequireAuth>
              <PlacesList />
            </RequireAuth>
          )}
        />

        <Route path="/login" element={<AuthLanding />} />
      </Routes>
    </div>
  );
}
export default App;
```

## Oplossing

Zoals altijd vind je een werkend voorbeeld in de [budget applicatie](https://github.com/HOGENT-Web/frontendweb-budget). Dit keer staat dit op de branch `feature/auth0`, je zal een gelijknamige branch vinden in de [backend](https://github.com/HOGENT-Web/webservices-budget). Zo kan je kiezen om wel of niet met authenticatie te werken.
