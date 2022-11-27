# E2E testen met authenticatie

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-Web/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les8 50b4dad
> yarn install
> yarn start
> 
> ```
>
> **De [REST API](https://github.com/HOGENT-Web/webservices-budget/) dient ook te draaien op branch `feature/auth0`.**

Momenteel moet je voor elke pagina in onze budget applicatie aangemeld zijn (behalve de `/login`). Onze testen gaan er nog steeds van uit dat je niet aangemeld moet zijn en dus zullen deze een voor een falen.

Alvorens meteen de oplossing te geven, laten we jullie aan de slag gaan met een tutorial van Auth0: <https://auth0.com/blog/end-to-end-testing-with-cypress-and-auth0/>. Neem deze eerst door en probeer de testen te doen slagen zonder naar de oplossing hieronder te kijken. Een paar opmerkingen en hints:

- We raden aan om ook naar volgend code-voorbeeld te kijken wanneer je het Cypress commando definieert: <https://github.com/charklewis/auth0-cypress/blob/master/cypress/support/commands.js>. De tutorial van Auth0 is, jammer genoeg, niet helemaal up-to-date. Je hebt een combinatie van de twee nodig.
  - Je hoeft de user niet door te geven aan het command, dit mag hard gecodeerd in het `login` command staan.
- Je kiest als scope best `openid profile email offline_access`, anders zal je nog steeds niet kunnen aanmelden.
- Implementeer een simpele login-test zoals in de tutorial om te checken of het werkt. Nadien kan je de andere testen fixen.

## Testgebruiker

Om via Cypress te kunnen aanmelden gaan we een testgebruiker aanmaken in het [Auth0 Dashboard](https://manage.auth0.com/). Ga hiervoor naar User Management > Users en klik op `Create User`. Kies onderstaande instellingen:

- Email: e2e-testing@budgetapp.be
- Password: kies zelf een wachtwoord
- Repeat Password: (zou duidelijk moeten zijn)
- Connection: Username-Password-Authentication

![Create a testuser](./images/1-create_testuser.png ':size=60%')

Na het aanmaken van de gebruiker opent het detailscherm.

![View user details](./images/2-userdetails.png ':size=60%')

Klik hier op `Edit` onder het e-mailadres van de testgebruiker. Klik onder het e-mailveld op `Set email as verified`.

![Verify email](./images/3-verify_email.png ':size=60%')

Zorg ervoor dat deze gebruiker de rol `boekhouder` toegekend krijgt.

![Verify email](./images/4-assign_role.png ':size=60%')

Vervolgens ga je naar de settings van jouw applicatie in het Auth0 Dashboard. Daar open je de Advanced Settings en ga je naar het tabblad Grant Types. Daar selecteer je `Password`. Vergeet de wijzigingen niet op te slaan.

![Allow password grant](./images/5-password_grant.png ':size=60%')

Als laatste moeten we username-password-authentication instellen als de default directory voor Auth0:

- Ga naar het [Auth0 Dashboard](https://manage.auth0.com/).
- Open Settings in het linkermenu
- Ga naar API Authorization Settings
- Vul `Username-Password-Authentication` bij Default Directory
- Sla de wijzigingen op

## Aanmelden

Het zou een beetje stom zijn om Cypress te laten werken met de hele login UI van Auth0. Dat is lastig om te onderhouden aangezien selectors mogelijks wijzigen en onze testen dan falen om de verkeerde reden.

Daarom gaan we manueel een token aanvragen via een HTTP request, net zoals Auth0 achter de schermen doet. Dat is veel eenvoudiger dan Cypress met de Auth0 login UI te laten werken en de signatuur van de API call wijzigt nooit. Stel dat de API drastisch wijzigt, dan maakt Auth0 een andere URL voor de nieuwe versie (merk de `v2` in de URL op).

Neem eerst [Building Cypress Commands](https://learn.cypress.io/advanced-cypress-concepts/building-the-right-cypress-commands) door.

Met Cypress commands kunnen we zelf functies toevoegen aan het `cy` object. Hiermee is het bv. eenvoudig om veel gebruikte code te abstraheren en te hergebruiken. We gaan hier dus ook gebruik van maken voor het aanmelden.

We hebben hiervoor een command nodig aangezien Cypress per test de hele browseromgeving reset. We zullen dus steeds uitgelogd zijn aan het begin elke test. Daarom zetten we de code om aan te melden apart.

Maak twee nieuw commando's aan in het bestand `cypress/support/commands.js`:

```js
Cypress.Commands.add('goToHomePage', () => { // 👈 1
  cy.visit('http://localhost:3000/');
});

Cypress.Commands.add('login', () => {
  cy.goToHomePage();
  cy.clearLocalStorage(); // 👈 2

  // 👇 3
  Cypress.log({
    displayName: 'login',
    message: `Signing in as ${Cypress.env('auth_username')}`,
  });

  const clientId = Cypress.env('auth_client_id');
  const audience = Cypress.env('auth_audience');
  const scope = 'openid profile email offline_access';

  // 👇 4
  cy.request({
    method: 'POST',
    url: Cypress.env('auth_url'),
    body: {
      grant_type: 'password',
      username: Cypress.env('auth_username'),
      password: Cypress.env('auth_password'),
      audience,
      scope,
      client_id: clientId,
      client_secret: Cypress.env('auth_client_secret'),
    },
  }).then(({
    // 👇 5
    body: {
      access_token: accessToken,
      expires_in: expiresIn,
      id_token: idToken,
      token_type: tokenType,
    },
  }) => {
    cy.window() // 👈 6
      .then((win) => {
        // 👇 7
        win.localStorage.setItem(
          `@@auth0spajs@@::${clientId}::${audience}::${scope}`,
          JSON.stringify({
            body: {
              client_id: clientId,
              access_token: accessToken,
              id_token: idToken,
              scope,
              expires_in: expiresIn,
              token_type: tokenType,
              decodedToken: {
                user: JSON.parse(
                  Buffer.from(idToken.split('.')[1], 'base64').toString('ascii'),
                ),
              },
              audience,
            },
            expiresAt: Math.floor(Date.now() / 1000) + expiresIn,
          }),
        );
        cy.reload(); // 👈 8
      });
  });
});
```

1. Definieer een commando om naar de home pagina terug te keren.
2. [`clearLocalStorage`](https://docs.cypress.io/api/commands/clearlocalstorage) maakt `localStorage` helemaal leeg.
3. [`Cypress.log`](https://docs.cypress.io/api/cypress-api/cypress-log) print een bericht naar de Cypress command log (linkerpaneel in de testomgeving).
4. We maken manueel een request naar het endpoint van Auth0 waarmee een token verkregen wordt. We hebben hiervoor heel wat geheime informatie nodig (username, password, client secret), net zoals informatie die kan wijzigen in productie. Daarom maken we hiervoor environment variables die enkel voor Cypress beschikbaar zijn (zie verder). Via `Cypress.env` vraag je de waarde hiervan op.
5. We halen de nodige tokens, het type en de vervaldatum uit het response.
6. We vragen het window object op.
7. Vervolgens zetten we de nodige informatie in `localStorage`. Hiervoor kijken we manueel wat Auth0 opslaat als we aanmelden in de applicatie, dit bootsen we na.
8. Vervolgens herladen we de pagina zodat React de nieuwe waarde in `localStorage` opmerkt.

Vervolgens hebben we nog een `cypress.env.json` nodig. Dit bestand bevat alle environment variales die enkel voor Cypress beschikbaar zijn. Je past dit bestand aan volgens de specifieke Auth0 settings van jouw applicatie en API.

```json
{
  "auth_audience": "{YOUR_API_IDENTIFIER}",
  "auth_url": "https://{YOUR_DOMAIN}/oauth/token",
  "auth_client_id": "{YOUR_CLIENT_ID}",
  "auth_client_secret": "{YOUR_CLIENT_SECRET}",
  "auth_username": "e2e-testing@budgetapp.be",
  "auth_password": "{YOUR_PASSWORD}"
}
```

**Voeg dit bestand zeker toe aan je `.gitignore`, dit mag NIET op GitHub terug komen!**

Eventueel kan je ook een `logout` command toevoegen:

```js
Cypress.Commands.add('logout', () => {
  Cypress.log({
    displayName: 'logout',
  });
  cy.goToHomePage();
  cy.get('[data-cy=logout_btn]').click();
});
```

## Testen aanpassen

Voeg bovenaan elke test suite een `beforeEach` toe die de testgebruiker aanmeldt in de applicatie. `beforeEach` wordt uitgevoerd voor elke test. Dit is nodig aangezien Cypress steeds o.a. `localStorage` leegmaakt voor elke test.

```js
describe('...', () => {

  beforeEach(() => {
    cy.login();
  });

  // de testen
});
```

## Oplossing

Zoals altijd vind je een werkend voorbeeld in de [budget applicatie](https://github.com/HOGENT-Web/frontendweb-budget). Dit keer staat dit op de branch `feature/auth0`, je zal een gelijknamige branch vinden in de [backend](https://github.com/HOGENT-Web/webservices-budget). Zo kan je kiezen om wel of niet met authenticatie te werken.
