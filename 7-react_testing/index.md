# Testing

<!-- TODO: startpunt en oplossing toevoegen -->

Vite komt standaard niet met een test framework, dat geeft ons de vrijheid om zelf te kiezen. Wij kiezen hier voor UI testen m.b.v. [Cypress](https://www.cypress.io/). Naast UI testen kan je bv. ook unit testen schrijven voor de componenten (m.b.v. [Jest](https://jestjs.io/)), maar deze testen vallen buiten de scope van deze cursus.

## Cypress

Cypress draait in een browser en niet via een WebDriver. Dus het draait (zo goed als) onafhankelijk van onze React code, m.a.w. wat je hier vandaag leert kan je op een identieke manier gebruiken als je een ander framework gebruikt. Het leukste aan Cypress is dat je vrij eenvoudig testen kan schrijven en toevoegen. Want iedereen weet dat het moeilijkste van testen is om luie developers testen te laten schrijven.

Om met Cypress aan de slag te gaan, moet je dit eerst installeren als dev dependency:

```bash
yarn add --dev cypress
```

We voegen ook de [Cypress ESLint plugin](https://github.com/cypress-io/eslint-plugin-cypress/blob/HEAD/FLAT-CONFIG.md) toe.

```bash
yarn add --dev eslint-plugin-cypress
```

Pas vervolgens de configuratie van ESLint aan. Importeer de plugin en exporteer de `recommended rules` . Zonder deze plugin krijgen we een `no-undef` foutmeldingen voor `describe`, `it`...

```js
import pluginCypress from 'eslint-plugin-cypress/flat';

//...

export default [
  pluginCypress.configs.recommended,
  //...
];
```

Voeg vervolgens een nieuw commando toe aan de `package.json`:

```json
"scripts": {
  "test": "cypress open"
}
```

Vervolgens kan je Cypress openen met onderstaand commando:

```bash
yarn test
```

Wanneer je Cypress voor de eerste keer opent in een project, dien je een wizard te doorlopen om Cypress te configureren. Sla een eventuele melding van de nieuwtjes in Cypress over.

In de eerste stap dien je te kiezen voor **E2E Testing** (= je runt de applicatie en je bezoekt pagina's om ze te testen) of **Component testing** (= je monteert afzonderlijke componenten en test die afzonderlijk). Kies hier voor `E2E Testing`.

![Eerste keer Cypress runnen (stap 1)](./images/7_1_startscherm_cypress.png ':size=80%')

In de volgende stap maakt Cypress de nodige configuratiemappen/-bestanden aan en geeft hiervan een overzicht. [Bekijk de documentatie voor de details](https://docs.cypress.io/guides/references/configuration).

![Eerste keer Cypress runnen (stap 2)](./images/7_2_startscherm_cypress.png ':size=80%')

Klik op `Continue`, kies vervolgens de browser waarin je de testen wenst te draaien (uit een lijst van browsers die op je machine staan) en klik op `Start E2E testing`.

![Eerste keer Cypress runnen (stap 3)](./images/7_3_startscherm_cypress.png ':size=80%')

Cypress opent **The Launchpad** in de browser. The Launchpad is het portaal naar Cypress dat helpt bij onboarding, het kiezen van een testtype en het starten van een browser.

![Eerste keer Cypress runnen (stap 4)](./images/7_4_startscherm_cypress.png ':size=80%')

## De eerste test

Dan is het tijd voor onze eerste test. Kies `Create new empty spec`. Behoud de standaard naam voor de spec en klik op `Create spec`. Een dialoogvenster met de gegenereerde spec wordt getoond. De test zal controleren of het surfen naar de [Example app](https://example.cypress.io) werkt.

```jsx
describe('template spec', () => {
  it('passes', () => {
    cy.visit('https://example.cypress.io');
  });
});
```

Klik bovenaan op 'x' om het venster te sluiten. De test verschijnt onmiddellijk in de [**Spec Explorer**](https://docs.cypress.io/guides/core-concepts/cypress-app#The-Spec-Explorer) en werd gecreëerd in de `cypress/e2e` map. Cypress controleert de specificatiebestanden op eventuele wijzigingen en geeft automatisch eventuele wijzigingen weer.

Klik op het bestand `spec.cy.js` om de spec uit te voeren.

De test draait in de [**Test Runner**](https://docs.cypress.io/guides/core-concepts/cypress-app#Project-Runs) in een browser, tegen een website die bezocht wordt door de browser. M.a.w. je kan niet enkel een site die in ontwikkeling is testen, maar ook om het even welke site die ergens draait. Hier wordt dus [https://example.cypress.io](https://example.cypress.io) getest. Indien je kiest voor `Scaffold example specs`, dan draaien deze voorbeelden tegen [https://example.cypress.io/todo](https://example.cypress.io/todo).

We passen de test nu aan. Ga terug naar Visual Studio Code en open het bestand `cypress/e2e/spec.cy.js`

```js
// cypress/e2e/spec.cy.js

// 👇 1
describe('mijn eerste test', () => {
  // 👇 2
  it('doet niet veel', () => {
    // 👇 3
    expect(true).to.equal(true);
  });
});
```

1. [`describe()`](https://mochajs.org/#bdd) bundelt een aantal testen. Je geeft het een beschrijving en een functie die uitgevoerd wordt. De syntax is overgenomen van het [Mocha](https://mochajs.org/) framework.
2. Elke test is dan een `it()` functie met een beschrijving en een functie die de test uitvoert
3. De `asserts` (= vergelijkingen/checks) in de testen zijn **BDD (behavior driven development) asserts**. De syntax is overgenomen van het [Chai](https://www.chaijs.com/) framework. De mogelijkheden hiervan worden verderop behandeld.

Geef aan `describe` en `it` een degelijke beschrijving mee, dit zal je alleen maar helpen tijdens het debuggen van gefaalde testen.

Als je de wijzigingen opslaat, voert de Test Runner een reload uit. De test voert met success uit. Verander gerust een `true` in `false` om een gefaalde test te bekijken.

We willen natuurlijk onze applicatie testen. Een eerste test nuttige kan zijn om te checken of de applicatie effectief beschikbaar is.

```js
describe('mijn eerste test', () => {
  // 👇 1
  it('draait de applicatie', () => {
    cy.visit('http://localhost:5173'); // 👈 2
  });
});
```

1. Geef een betekenisvolle naam aan de test
2. Bezoek de website

**Zorg er wel voor dat de front-end én back-end draaien**, anders faalt de test.

Ga naar de **Test Runner** en voer de test uit:

1. De Command Log toont de VISIT action. De VISIT toont een blue pending state totdat de pagina geladen is.
2. De budget applicatie wordt geladen in het Preview pane.
3. De test kleurt groen, alhoewel we geen assertions hebben toegevoegd. Dit komt omdat veel van de Cypress opdrachten zijn gebouwd om te mislukken als ze niet vinden wat ze verwachten te vinden. Dit staat bekend als een [default assertion](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress#Default-Assertions).

Nu controleren we ook of er een `h1` element gevonden kan worden:

```js
describe('mijn eerste test', () => {
  it('draait de applicatie', () => {
    cy.visit('http://localhost:5173');
    cy.get('h1').should('exist'); // 👈
  });
});
```

- [get()](https://docs.cypress.io/api/commands/get) wordt gebruikt om een element te selecteren, hier op basis van de tagnaam.
- [should()](https://docs.cypress.io/api/commands/should) is een assertion. In de documentatie lezen we: "Assertions describe the desired state of your elements, your objects, and your application.".

Neem de documentatie [Introduction to Cypress](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress) door.

## Anatomie van een UI test

Onze testen zullen vaak een gelijkaardig stramien hebben: een URL bezoeken, interageren met elementen op de pagina (iets typen in een inputveld, op een knop klikken...) en/of kijken of het we het gewenste resultaat te zien krijgen.

In beide gevallen moeten we elementen van de DOM kunnen identificeren. Gewoon checken of er 'een' `h1` aanwezig is, zoals in de vorige test, zal niet volstaan. Als er meerdere inputs, buttons, etc. zijn, moeten we zeker zijn dat we met de juiste elementen interageren.

Neem volgende stukje HTML als voorbeeld:

```html
<button id="main" class="btn btn-large" name="submission" role="button">
  Submit
</button>
```

Stel dat we deze Submit-button willen terugvinden.

We zouden kunnen opteren om gewoon 'de' button op te vragen. Maar dat werkt enkel als er maar één button is.

```js
cy.get('button').click();
```

Een alternatief is om naar de CSS-klassen te kijken. Maar dat hangt sterk samen met de styling, en verandert dus veel te makkelijk.

```js
cy.get('.btn.btn-large').click();
```

Naar het `id` kijken is al iets beter, het is op zijn minst uniek. Maar bij een goede refactor riskeert die ook te wijzigen.

```js
cy.get('#main').click();
```

Je zou ook naar de inhoud van de tag kunnen kijken. Dit is soms valabel, maar wat met i18n (= internationalization of vertalingen)?

```js
cy.contains('Submit').click();
```

De beste optie is om gewoon een extra [`data-attribute`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes) toe te voegen. We zijn zeker dat we altijd hetzelfde element gaan terugvinden en iemand die de code wijzigt weet ook direct dat het element deel uitmaakt van een test.

```html
<button
  id="main"
  class="btn btn-large"
  name="submission"
  role="button"
  data-cy="submit"
>
  <!-- 👆 -->
  Submit
</button>
```

In onze test wordt dit dus:

```js
cy.get('[data-cy=submit]').click();
```

Kortom, we gaan telkens `data-cy` attributen toevoegen waar nodig.

## Add transaction test

Als voorbeeld zullen we het toevoegen van een transactie testen. Eerst en vooral moeten we overal de juiste `data-cy` attributen toevoegen. Dit hoef je natuurlijk maar éénmaal te doen per component waar je testen voor schrijft.

```jsx
<form onSubmit={handleSubmit(onSubmit)} className='mb-5'>
  <LabelInput
    label='User ID'
    name='user'
    type='number'
    validationRules={validationRules.user}
    data-cy='user_input' // 👈 1
  />
  <LabelInput
    label='Date'
    name='date'
    type='date'
    validationRules={validationRules.date}
    data-cy='date_input' // 👈 1
  />
  <SelectList
    label='Place'
    name='placeId'
    placeholder='-- Select a place --'
    items={places}
    validationRules={validationRules.placeId}
    data-cy='place_input' // 👈 1
  />
  <LabelInput
    label='Amount'
    name='amount'
    type='number'
    validationRules={validationRules.amount}
    data-cy='amount_input' // 👈 1
  />
  <div className='clearfix'>
    <div className='btn-group float-end'>
      <button
        type='submit'
        disabled={isSubmitting || !isValid}
        className='btn btn-primary'
        data-cy='submit_transaction' // 👈 2
      >
        {transaction?.id ? 'Save transaction' : 'Add transaction'}
      </button>
      <Link
        disabled={isSubmitting}
        className='btn btn-light'
        to='/transactions'
      >
        Cancel
      </Link>
    </div>
  </div>
</form>
```

1. Bij elke `input` zetten we een `data-cy` attribuut. De `LabelInput` component geeft alle onbekende props door aan het `input` element (via `{...rest}`).
2. Natuurlijk heeft de submit button ook een `data-cy` attribuut nodig.

Op een gelijkaardige manier passen we `Transaction` aan zodat we nadien kunnen checken of de transactie goed toegevoegd is.

```jsx
import { memo, useCallback } from 'react';
// ...

const TransactionMemoized = memo(function Transaction({
  id,
  user,
  date,
  amount,
  place,
  onDelete,
}) {
  const handleDelete = useCallback(() => {
    onDelete(id);
  }, [id, onDelete]);

  return (
    {/* 👇 */}
    <tr data-cy='transaction'>
      {/* 👇 */}
      <td data-cy='transaction_date'>
        {dateFormat.format(new Date(date))}
      </td>
      {/* 👇 */}
      <td data-cy='transaction_user'>{user.name}</td>
      {/* 👇 */}
      <td data-cy='transaction_place'>{place.name}</td>
      {/* 👇 */}
      <td data-cy='transaction_amount' className='text-end'>
        {amountFormat.format(amount)}
      </td>
      <td>
        {onDelete ? (
          <>
            <Link
              to={`/transactions/edit/${id}`}
              className='btn btn-primary'
              data-cy='transaction_edit_btn'
            >
              {/* 👆 */}
              <IoPencilOutline />
            </Link>
            <button
              className='btn btn-danger'
              onClick={handleDelete}
              data-cy='transaction_remove_btn'
            >
              {/* 👆 */}
              <IoTrashOutline />
            </button>
          </>
        ) : null}
      </td>
    </tr>
  );
});

export default TransactionMemoized;
```

Uiteindelijk kunnen we de echte testcode schrijven. Voeg een nieuw bestand `cypress/e2e/addTransaction.cy.js` toe.

```js
// cypress/e2e/addTransaction.cy.js
describe('Add transaction', () => {
  it('should add a transaction', () => {
    cy.visit('http://localhost:5173/transactions/add'); // 👈 1

    cy.get('[data-cy=user_input]').type('2'); // 👈 2
    cy.get('[data-cy=date_input]').type('2021-11-01'); // 👈 2
    cy.get('[data-cy=place_input]').select('Irish Pub'); // 👈 2
    cy.get('[data-cy=amount_input]').type('200'); // 👈 2
    cy.get('[data-cy=submit_transaction]').click(); // 👈 3

    cy.get('[data-cy=transaction_user]').eq(9).contains('Pieter'); // 👈 4
    // 👇 5
    cy.get('[data-cy=transaction_amount]').each((el, idx) => {
      if (idx === 9) {
        expect(
          Number(el[0].textContent.replace(/^\D+/g, '').replace(/,/, '.')),
        ).to.equal(200);
      }
    });
    cy.get('[data-cy=transaction]').should('have.length', 10); // 👈 6
  });
});
```

1. Om nu het formulier te testen, gaan we eerst naar de juiste pagina.
2. Dan vragen we alle input fields op en geven we zinvolle data in.
   - Bij text input fields kan je gewoon de [`.type()`](https://docs.cypress.io/api/commands/type) functie gebruiken.
   - Voor select inputs de functie [`.select()`](https://docs.cypress.io/api/commands/select). Hierbij kan de waarde zowel de value, als de content, of zelfs de index zijn.
3. Als laatste klikken (m.b.v. [`click()`](https://docs.cypress.io/api/commands/click)) we op de submit button. Submitten zorgt ervoor dat we terug naar onze overzichtspagina gaan, dat gebeurt ook in de testomgeving.
4. Daar kunnen we kijken of de transactie toegevoegd is. We hebben `data-cy` op elk deel van een Transaction, maar er zijn meerdere transacties, dus we kunnen niet gewoon bv. 'de' `transaction_user` opvragen (`cy.get("[data-cy=transaction_user]")`). Met `eq()` kan je één specifiek element opvragen a.d.h.v. zijn index.
5. Of je kan een functie meegeven, die voor elk element aangeroepen wordt, en zo je checks doen. `el` is hier een array waar het 1ste element het echte DOM element bevat.
6. Vaak is het gewoon al nuttig om te kijken of er effectief één toegevoegd is, los van de inhoud, dat kan natuurlijk ook.

> Merk op: In een e2e test zijn we niet beperkt tot één enkele assertion in een bepaalde test. Veel interacties in een toepassing kunnen zelfs meerdere stappen vereisen en zullen de toepassingsstatus waarschijnlijk op meer dan één manier veranderen.

### Page transitions

De test surft naar twee pagina's. Cypress detecteert automatisch een paginaovergangsgebeurtenis en stopt automatisch met het uitvoeren van opdrachten totdat de volgende pagina is geladen.

Als de volgende pagina de laadfase niet had voltooid, zou Cypress de test hebben beëindigd en een foutmelding hebben gegeven.

Cypress wacht 4 seconden voordat er een time-out wordt gegenereerd bij het vinden van een DOM-element. In geval van een paginaovergangsgebeurtenis krijg je pas een time-out na 60 seconden. Met andere woorden, op basis van de opdrachten en de gebeurtenissen die plaatsvinden, past Cypress automatisch de verwachte time-outs aan om overeen te komen met het gedrag van de webtoepassing.

Deze verschillende time-outs worden gedefinieerd in het [configuratiebestand](https://docs.cypress.io/guides/references/configuration#Timeouts)

### Reproduceerbaarheid

Als je aan het mee typen was, zal je merken dat er nu iets lastig aan de hand is. Elke keer als je opslaat, wordt het toevoegen van een transactie uitgevoerd, en de lijst groeit en groeit.

M.a.w. de check om te kijken of er vier transacties na het toevoegen zijn werkt maar éénmaal en faalt vervolgens altijd. Dat is natuurlijk niet werkbaar.

We moeten ervoor zorgen dat onze testen geen blijvende wijzigingen veroorzaken, we kunnen dat op twee manieren bereiken:

- niet met de echte databank werken, **mocks** gebruiken (straks meer hierover)
- alle bewerkingen ook weer 'omkeren' (wij kiezen voor deze optie)

Als we onze `add transaction test` telkens opnieuw willen kunnen uitvoeren, moeten we de toegevoegde transactie nadien weer verwijderen (en dan hebben we direct een verwijder test ook):

```js
describe('Add transaction', () => {
  // ...

  it('should remove the transaction', () => {
    cy.visit('http://localhost:5173/transactions/'); // 👈 1
    cy.get('[data-cy=transaction_remove_btn]').eq(9).click(); // 👈 2
    cy.get('[data-cy=transaction]').should('have.length', 9); // 👈 3
  });
});
```

1. We bezoeken weer de juiste pagina.
2. We klikken op de verwijder-knop van de net toegevoegde transactie (index 9).
3. Vervolgens controleren we of er effectief maar 9 transacties meer overblijven.

Nu kunnen we de testen opnieuw en opnieuw draaien zonder dat ze falen. Mogelijks moet je wel eerst manueel de lijst van transacties herstellen naar wat de test verwacht.

## Oefening 1: foutboodschappen

We hebben getest of ons formulier werkt. Er wordt een transactie toegevoegd als alle input fields een geldige waarde krijgen.

Vaak is het echter minstens even interessant (zo niet interessanter) om alle edge cases te gaan testen. Worden de foutboodschappen wel goed getoond als de gebruiker foutieve informatie ingeeft? Het heeft erg veel zin om hiervoor testen te schrijven. Als alles goed gaat, komen foutieve situaties niet bijzonder vaak voor. Dus je wilt ze opmerken in je testen en niet (te) laat bij de gebruiker.

<!-- markdownlint-disable-next-line -->

- Oplossing +

  1. Voeg een `data-cy` attribuut aan de tags met foutboodschappen.
  2. Voeg een nieuwe test toe aan het `addTransaction.cy.js` bestand.
  3. Ga naar de juiste URL, typ een negatief getal in het veld voor de gebruiker.
  4. Klik op het toevoegen van een transactie.
  5. Controleer of een foutboodschap verschijnt.
  6. Extra: test zowel niets als een negatief getal invullen en controleer dat in beide gevallen de juiste foutboodschap verschijnt.

Deze [cheat sheet](https://cheatography.com/aiqbal/cheat-sheets/cypress-io/) kan je misschien helpen. Bekijk zeker ook de voorbeeldtesten voor inspiratie als je niet verder kan.

## Mocks

Alles op een echte back-end testen heeft zeker zijn nut (om zeker te zijn dat alles wel werkt), maar zo wil je niet alle testen schrijven. Het is relatief traag, altijd alles 'terugzetten' kan knap lastig worden, en het beperkt wat je allemaal kan testen. Hoe zou je testen of de front-end alles juist toont als de back-end onbereikbaar is bijvoorbeeld?

De oplossing hiervoor heet **mocken**. Hierbij stellen we een fake server op die vóór onze test uitgevoerd wordt en beschrijven hoe die moet reageren op bepaalde API calls. Vervolgens doen we onze testen en kunnen we checken of onze front-end voor die bepaalde back-end alles correct afhandelt.

Neem de documentatie [Network requests](https://docs.cypress.io/guides/guides/network-requests) door tot aan Fixtures.

Laat ons een nieuwe test toevoegen die kijkt of de lijst van transacties wel correct getoond wordt. Maak hiervoor een nieuw bestand `cypress/e2e/transactions.cy.js`.

```js
// cypress/e2e/transactions.cy.js
describe('Transactions list', () => {
  it('should show the transactions', () => {
    // 👇 1
    cy.intercept(
      'GET',
      'http://localhost:9000/api/transactions',
      `{"items":[{"id":1,"amount":-97,"date":"2021-11-01","user":{"id":1,"name":"Pieter"},
      "place":{"id":4,"name":"Chinees Restaurant"}}],"count":1}`,
    );

    // 👇 2
    cy.visit('http://localhost:5173');
    cy.get('[data-cy=transaction]').should('have.length', 1);
    cy.get('[data-cy=transaction_place]').eq(0).contains('Chinees Restaurant');
    cy.get('[data-cy=transaction_date]').eq(0).should('contain', '01/11/2021');
  });
});
```

1. Routes mocken doen we met de [`intercept`](https://docs.cypress.io/api/commands/intercept) functie
   - 1ste argument: de HTTP methode
   - 2de argument: het URL van de route om op te vangen
   - 3de argument: het gewenst antwoord
2. Nadien schrijven we de test. Als de pagina geladen wordt, zal dit resulteren in de API call die wij gemockt hebben. Daarom zullen de componenten gerenderd worden met onze fake data.

## Fixtures

De data inline plaatsen in een `intercept` is meestal niet zo handig (of leesbaar). Je kan dit oplossen door `fixtures` te gebruiken. Neem de documentatie over [Fixtures](https://docs.cypress.io/guides/guides/network-requests#Fixtures) door.

Creëer een nieuw bestand `transactions.json` in de `fixtures` map van cypress

```json
// fixtures/transactions.json
{
  "items": [
    {
      "id": 1,
      "amount": -97,
      "date": "2021-11-01",
      "user": {
        "id": 2,
        "name": "Pieter"
      },
      "place": {
        "id": 4,
        "name": "Chinees Restaurant"
      }
    }
  ],
  "count": 1
}
```

Pas vervolgens de test aan om deze fixture terug te geven i.p.v. de hardgecodeerde string:

```js
// cypress/e2e/transactions.cy.js
describe('Transactions list', () => {
  it('should show the transactions', () => {
    cy.intercept(
      'GET',
      'http://localhost:9000/api/transactions',
      { fixture: 'transactions.json' }, // 👈
    );

    cy.visit('http://localhost:5173');
    cy.get('[data-cy=transaction]').should('have.length', 1);
    cy.get('[data-cy=transaction_place]').eq(0).contains('Chinees Restaurant');
    cy.get('[data-cy=transaction_date]').eq(0).should('contain', '01/11/2021');
  });
});
```

Zo kunnen we eenvoudiger dit soort data hergebruiken/aanpassen. Zo'n fixture bestand is bovendien een pak leesbaarder dan zo'n lange string.

## Waiting

Je kan nog veel meer doen dan simpel antwoorden terugsturen. Neem de documentatie over [Waiting](https://docs.cypress.io/guides/guides/network-requests#Waiting) door.

Als voorbeeld gaan we een request eens sterk vertragen. Dan kunnen we testen of onze loading indicator wel effectief getoond wordt. Voeg deze test toe aan het bestand `cypress/e2e/transactions.cy.js`:

```js
// cypress/e2e/transactions.cy.js
describe('Transactions list', () => {
  // ...

  it('should show a loading indicator for a very slow response', () => {
    cy.intercept(
      'http://localhost:9000/api/transactions', // 👈 1
      (req) => {
        req.on('response', (res) => {
          res.setDelay(1000);
        }); // 👈 2
      },
    ).as('slowResponse'); // 👈 5
    cy.visit('http://localhost:5173'); // 👈 3
    cy.get('[data-cy=loader]').should('be.visible'); // 👈 4
    cy.wait('@slowResponse'); // 👈 6
    cy.get('[data-cy=loader]').should('not.exist'); // 👈 7
  });
});
```

1. We starten weer met onze `intercept` op de URL die we willen vertragen.
2. In plaats van een statisch antwoord te formuleren gebruiken we een callback, die het antwoord met 1000ms (= 1 seconde) vertraagt.
3. Nadien bezoeken we weer de pagina.
4. En kijken we of de loading indicator zichtbaar is. Uiteraard vereist dit dat je in de `Loader` component een `data-cy` attribuut toevoegt aan de eerste `div` tag.
5. Als je nu ook wil checken of de indicator weer verdwijnt kan dat ook betrouwbaar. Geef het request een naam.
6. En wacht iets later op dat request (m.a.w. tot hij afgehandeld is, hoelang de delay ook).
7. Kijk dan of de loading indicator niet langer voorkomt.

## Oefening 2: zoekfunctie van transacties

Schrijf volgende testen voor de zoekfunctie van onze transacties:

- correcte invoer
- invoer zonder resultaten
- fouten in de back-end

Hieronder worden de testgevallen afzonderlijk uitgelegd.

### Correcte invoer

Als naar 'Ir' gezocht wordt, willen we enkel de transacties van Irish Pub zien.

- Voeg `data-cy` attributen toe waar nodig.
- Check of er drie transacties in de lijst voorkomen.
- Check of de 3 transacties 'Ir' bevatten. Dit kan je het makkelijkst bereiken door gebruik te maken van een match met regular expressions (zie <https://glebbahmutov.com/cypress-examples/recipes/contains-regular-expression.html>).

### Invoer zonder resultaten

Als er naar 'xyz' gezocht wordt mag er geen enkel element getoond worden. Check hier ook of er geen fouten getoond worden.

### Fouten in de back-end

Als de back-end fouten geeft bij het ophalen van de transacties, dan zijn er geen transacties zichtbaar maar wel een foutboodschap. Maak gebruik van status code in de intercept om dit te bereiken (zie <https://docs.cypress.io/api/commands/intercept#StaticResponse-objects>).

## Oefening 3 - README

Pas `README.md` aan zodat de gebruiker weet hoe de testen uitgevoerd moeten worden.

### Oplossing

TODO: voorbeeldoplossing toevoegen

## Authenticatie

In het volgende hoofdstuk zullen we authenticatie toevoegen aan ons project. Dit zal wijzigen hoe onze testen uitgevoerd moeten worden, dit behandelen we in een later hoofdstuk.
