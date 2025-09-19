# E2E testen met authenticatie

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les9 56dc080
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
> pnpm prisma migrate dev
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#web-services-budget) voor meer informatie.

Momenteel moet je voor elke pagina in onze budgetapplicatie aangemeld zijn (behalve de `/login` en `/logout`). Onze testen gaan er nog steeds van uit dat je niet aangemeld moet zijn en dus zullen deze één voor één falen.

## Cypress commands

Neem eerst [Building Cypress Commands](https://learn.cypress.io/advanced-cypress-concepts/building-the-right-cypress-commands) door.

Met Cypress commands kunnen we zelf functies toevoegen aan het `cy` object. Hiermee is het bv. eenvoudig om veel gebruikte code te abstraheren en te hergebruiken. Hier gaan we dus ook gebruik van maken voor het aanmelden.

Aangezien Cypress per test de hele browseromgeving reset, hebben we hiervoor een command nodig. We zullen dus steeds uitgelogd zijn aan het begin van elke test. Daarom zetten we de code om aan te melden apart, dan kunnen we deze code hergebruiken in elke test.

Maak een nieuw commando aan in het bestand `cypress/support/commands.js`:

```js
// cypress/support/commands.js
// 👇 1
Cypress.Commands.add('login', (email, password) => {
  // 👇 5
  Cypress.log({
    displayName: 'login',
  });

  cy.visit('http://localhost:5173/login'); // 👈 2

  cy.get('[data-cy=email_input]').clear(); // 👈 3
  cy.get('[data-cy=email_input]').type(email); // 👈 3

  cy.get('[data-cy=password_input]').clear(); // 👈 3
  cy.get('[data-cy=password_input]').type(password); // 👈 3

  cy.get('[data-cy=submit_btn]').click(); // 👈 4
});
```

1. We maken een nieuw commando met de naam `login`. Dit commando heeft twee parameters: `email` en `password`.
2. Als eerste gaat dit commando naar de juiste URL.
3. Vervolgens wordt het e-mailadres en wachtwoord ingevuld. Eerst worden de inputvelden leeggemaakt.
   - **Voorzie de nodige `data-cy` attributen in de `Login` component.**
4. Dan wordt geklikt op de login-knop. Daarna zijn we normaal ingelogd.
5. Eventueel kan je nog een displayName toevoegen. Dit is handig voor de logging in de Cypress testrunner.

We kunnen dit commando snel even uittesten door een test toe te voegen aan `spec.cy.js`:

```jsx
// cypress/e2e/spec.cy.js
describe('General', () => {
  it('draait de applicatie', () => {
    cy.visit('http://localhost:5173');
    cy.get('h1').should('exist');
  });

  // 👇 1
  it('should login', () => {
    cy.login('pieter.vanderhelst@hogent.be', '12345678'); // 👈 2
  });
});
```

1. Maak een nieuwe test met als naam `should login`.
2. Hier gebruiken we het `login` command.

Voer dit testbestand uit. Je zou Cypress moeten zien inloggen.

Eventueel kan je ook een `logout` commando toevoegen:

```js
// cypress/support/commands.js
Cypress.Commands.add('logout', () => {
  Cypress.log({
    displayName: 'logout',
  });

  cy.visit('http://localhost:5173');
  cy.get('[data-cy=logout_btn]').click();
});
```

Voeg ook een `data-cy` attribuut toe aan de logout knop.

## Testen aanpassen

Voeg bovenaan elke test suite een `beforeEach` toe die de testgebruiker aanmeldt in de applicatie. `beforeEach` wordt uitgevoerd voor elke test. Dit is nodig aangezien Cypress steeds o.a. `localStorage` leegmaakt voor elke test.

```js
describe('...', () => {
  beforeEach(() => {
    cy.login('pieter.vanderhelst@hogent.be', '12345678');
  });

  // de testen
});
```

Voer de testen uit: ze falen, waarom?

- **Antwoord** +

  Er wordt niet gewacht tot het inloggen voltooid is. We moeten Cypress laten wachten totdat het login request voltooid is.

We moeten ons `login` commando aanpassen.

```jsx
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  Cypress.log({
    displayName: 'login',
  });

  cy.intercept('/api/sessions').as('login'); // 👈 1
  cy.visit('http://localhost:5173/login');

  cy.get('[data-cy=email_input]').clear();
  cy.get('[data-cy=email_input]').type(email);

  cy.get('[data-cy=password_input]').clear();
  cy.get('[data-cy=password_input]').type(password);

  cy.get('[data-cy=submit_btn]').click();
  cy.wait('@login'); // 👈 2
});
```

1. Vang elk login-request op en geef dit de naam `login`.
2. Laat Cypress wachten tot dit request afgerond is.

Voer de testen opnieuw uit. Bekijk ook de mock-data en ga na of dit nu overeenstemt met de transacties die je terug zou krijgen van de backend voor de aangemelde gebruiker. Je kan ook een versie maken voor de admin en voor de gebruiker.

## Oefening - Foutboodschappen in TransactionForm

- Deze testen falen om verschillende redenen: bv. het `userId` veld is verdwenen.
- Los alle fouten op en zorg dat de testen slagen.
- De test op een ongeldige userId vervang je door een test voor een ongeldige amount.

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les9-opl b9f9671
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.
