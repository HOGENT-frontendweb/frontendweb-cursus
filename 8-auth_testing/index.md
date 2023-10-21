# E2E testen met authenticatie

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-Web/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les8 d90ece0
> yarn install
> yarn start
> ```
>
> **De [REST API](https://github.com/HOGENT-Web/webservices-budget/) dient ook te draaien op branch `authenticatie`.**

Momenteel moet je voor elke pagina in onze budget applicatie aangemeld zijn (behalve de `/login` en `/logout`). Onze testen gaan er nog steeds van uit dat je niet aangemeld moet zijn en dus zullen deze een voor een falen.

## Cypress commands

Neem eerst [Building Cypress Commands](https://learn.cypress.io/advanced-cypress-concepts/building-the-right-cypress-commands) door.

Met Cypress commands kunnen we zelf functies toevoegen aan het `cy` object. Hiermee is het bv. eenvoudig om veel gebruikte code te abstraheren en te hergebruiken. Hier gaan we dus ook gebruik van maken voor het aanmelden.

We hebben hiervoor een command nodig aangezien Cypress per test de hele browseromgeving reset. We zullen dus steeds uitgelogd zijn aan het begin elke test. Daarom zetten we de code om aan te melden apart.

Maak een nieuw commando aan in het bestand `cypress/support/commands.js`:

```js
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => { // 👈 1
  // 👇 5
  Cypress.log({
    displayName: 'login',
  });

  cy.visit('http://localhost:5173/login'); // 👈 2
  cy.get('[data-cy=email_input]').clear().type(email); // 👈 3
  cy.get('[data-cy=password_input]').clear().type(password); // 👈 3
  cy.get('[data-cy=submit_btn]').click(); // 👈 4
});
```

1. We maken een nieuw commando met de naam `login`. Dit commando heeft twee parameters: `email` en `password`.
2. Als eerste gaat dit commando naar de juiste URL.
3. Vervolgens wordt het e-mailadres en wachtwoord ingevuld. Eerst worden de inputvelden leeggemaakt. Voorzie de nodige `data-cy` attributen in de `Login` component.
4. Dan wordt geklikt op de login-knop. Daarna zijn we normaal ingelogd.
5. Eventueel kan je nog een displayName toevoegen. Dit is handig voor de logging in de Cypress testrunner.

We kunnen dit commando snel even uittesten door een test toe te voegen aan `spec.cy.js`:

```jsx
// cypress/e2e/spec.cy.js
describe("mijn eerste test", () => {

  it("draait de applicatie", () => {
    cy.visit('http://localhost:5173');
    cy.get("h1").should("exist");
  });

  it("should login", () => { // 👈 1
    cy.login('thomas.aelbrecht@hogent.be', '12345678'); // 👈 2
  });
});
```

1. Maak een test `should login` aan.
2. Hier gebruiken we het `login` command.

Voer dit testbestand uit. Je zou Cypress moeten zien inloggen.

Eventueel kan je ook een `logout` command toevoegen:

<!-- TODO: werkt die goToHomePage ? -->

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
    cy.login('thomas.aelbrecht@hogent.be', '12345678')
  });

  // de testen
});
```

Voer de testen uit: ze falen, waarom? Er wordt niet gewacht tot het inloggen voltooid is. We moeten ons `login` commando aanpassen.

```jsx
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  Cypress.log({
    displayName: 'login',
  });

  cy.intercept('/api/users/login').as('login'); // 👈 1
  cy.visit('http://localhost:5173/login');
  cy.get('[data-cy=email_input]').clear().type(email);
  cy.get('[data-cy=password_input]').clear().type(password);
  cy.get('[data-cy=submit_btn]').click();
  cy.wait('@login'); // 👈 2
});
```

1. Vang elk login-request op en geef dit de naam `login`.
2. Laat Cypress wachten tot dit request afgerond is.

Voer de testen opnieuw uit, ze slagen!

## Oefening: add transaction form

- Herhaal hetzelfde voor de testen van het formulier.
- Deze testen falen om verschillende redenen: bv. user veld is niet editable...
- Los alle fouten op en zorg dat de testen slagen.

<!-- markdownlint-disable-next-line -->
+ Oplossing +

  <!-- TODO: oplossing toevoegen -->

  Een voorbeeldoplossing is te vinden op <https://github.com/hogent-web/frontendweb-budget> in de branch `authenticatie`

  ```bash
  git clone https://github.com/hogent-web/frontendweb-budget.git
  cd frontendweb-budget
  git checkout -b oplossing-les8 origin/authenticatie
  yarn install
  yarn dev
  ```

  Je zal een gelijknamige branch vinden in de [back-end repository](https://github.com/hogent-web/webservices-budget).
