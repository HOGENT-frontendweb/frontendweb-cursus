# Testing

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les7 6b8dff1
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.
>
> Vanaf nu heb je ook de bijbehorende backend nodig:
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/webservices-budget.git
> cd webservices-budget
> git checkout -b les6 3386a40
> pnpm install
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#web-services-budget) voor meer informatie.

Vite komt standaard niet met een test framework, dat geeft ons de vrijheid om zelf te kiezen. Wij kiezen hier voor UI testen m.b.v. [Playwright](https://playwright.dev/). Naast UI testen kan je bv. ook unit testen schrijven voor de componenten (m.b.v. [Vitest](https://vitest.dev/)), maar deze testen vallen buiten de scope van deze cursus.

TODO: @Andreas : Playwright CLI en Playwright MCP???

## Playwright

[Playwright](https://playwright.dev/) is een end-to-end testframework van Microsoft dat testen uitvoert in een echte browser (Chromium, Firefox of WebKit). In tegenstelling tot oudere tools werkt Playwright niet via een WebDriver maar rechtstreeks via het browserprotocol, wat het sneller en betrouwbaarder maakt. Playwright ondersteunt automatisch wachten op elementen, netwerk-interceptie en het draaien van testen in meerdere browsers tegelijk.

Om met Playwright aan de slag te gaan, installeer je het als dev dependency en installeer je de browsers:

```bash
pnpm create playwright
```

Beantwoord de onderstaande vragen

- Where to put your end-to-end tests? · tests
- Add a GitHub Actions workflow? (Y/n) · false
- Install Playwright browsers (can be done manually via 'pnpm exec playwright install')? (Y/n) · true

Playwright genereert een configuratiebestand `playwright.config.ts` in de root van het project. Een minimale configuratie voor onze applicatie:

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests', // 👈 1
  fullyParallel: true, // 👈 2
  retries: process.env.CI ? 2 : 0, // 👈 3
  workers: process.env.CI ? 1 : undefined, // 👈 4
  reporter: 'html', // 👈 5
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  }, // 👈 6
  projects: [{ name: 'chromium', use: { ...devices['Desktop Chrome'] } }], // 👈 7
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  }, // 👈 8
});
```

1. **`testDir`**: de map waar Playwright naar testbestanden zoekt. Alle bestanden die eindigen op `.spec.ts` worden herkend als testbestand.
2. **`fullyParallel`**: alle testen draaien gelijktijdig in aparte browsercontexten. Dat maakt de suite veel sneller. Opgelet: testen mogen dan geen gedeelde toestand hebben (bv. dezelfde database-rij aanpassen tegelijk).
3. **`retries`**: bij een gefaalde test probeert Playwright het op CI automatisch twee keer opnieuw. Lokaal wordt niet herhaald zodat je snel feedback krijgt. De variabele `process.env.CI` is `true` op een CI-server (GitHub Actions, GitLab CI...) en `undefined` lokaal.
4. **`workers`**: het aantal parallelle browserprocessen. Op CI beperken we dit tot 1 om resource-conflicten te vermijden. Lokaal kiest Playwright automatisch een zinvol aantal op basis van de CPU.
5. **`reporter`**: na elke testrun genereert Playwright een HTML-rapport in de map `playwright-report/`. Je kan dit openen met `pnpm exec playwright show-report`. Andere opties zijn `list` (terminal output), `dot` (compact) of `json`.
6. **`use`**: globale instellingen die voor alle testen gelden.
   - `baseURL`: de basis-URL van de applicatie. In tests kan je dan `page.goto('/')` schrijven in plaats van de volledige URL te herhalen. Pas url aan in `http://localhost:5173`
   - `trace`: Playwright kan een volledige trace opnemen (screenshots, netwerklogs, console-output). Met `'on-first-retry'` wordt de trace enkel bewaard als een test opnieuw geprobeerd wordt — handig om gefaalde CI-runs te debuggen zonder teveel schijfruimte te gebruiken.
7. **`projects`**: welke browsers en apparaten Playwright gebruikt. `devices['Desktop Chrome']` is een preset met de viewport en user-agent van een desktop Chrome-browser. Je kan meerdere projecten toevoegen (bv. ook Firefox of mobiele viewports) om cross-browser te testen.
8. **`webServer`**: Playwright start automatisch de Vite dev-server vóór de testen en stopt hem daarna. `reuseExistingServer: !process.env.CI` zorgt ervoor dat een reeds draaiende server lokaal hergebruikt wordt (handig tijdens ontwikkeling), terwijl op CI altijd een nieuwe server opgestart wordt. Pas url aan in `http://localhost:5173`

Opdat `process.env` geen fout meer zou geven, pas je `tsnode.config.json`aan. Voeg onderstaande toe aan de include prop

```ts
 "include": ["vite.config.ts", "playwright.config.ts", "tests/**/*.ts"]
```

Voeg de testcommando's toe aan `package.json`:

```json
"scripts": {
  "test": "playwright test",
  "test:ui": "playwright test --ui"
}
```

- `pnpm test` voert alle testen uit in de terminal.
- `pnpm test:ui` opent de **Playwright UI mode**: een interactieve browser-interface waarin je testen stap voor stap kan volgen, inspecteren en debuggen. ALs je wil kan je ook de [Playwright VS Code extension](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright) installeren. Meer info op [https://playwright.dev/docs/getting-started-vscode](https://playwright.dev/docs/getting-started-vscode)

Bekijk het bestand `example.spec.ts` en voer de testen uit. Bekijk de documentatie: [Running and debugging tests](https://playwright.dev/docs/running-tests) en [UI Mode](https://playwright.dev/docs/test-ui-mode)

```bash
pnpm test:ui
```

Pas de code aan en zorg dat de test faalt. Bekijk de foutmelding in de terminal en in de Playwright UI. Herstel de code en zorg dat de test opnieuw slaagt.

De `playwright-report` folder bevat het HTML-rapport van de laatste test run. Het wordt bij elke test run volledig overschreven. Playwright maakt dit automatisch aan door de reporter: 'html' instelling in `playwright.config.ts`. De inhoud:

- index.html: visueel rapport met een overzicht van alle tests (geslaagd/gefaald)
- trace: screenshots, traces en videos gekoppeld aan elke test
- data: JSON-bestanden met testresultaten
  Je kan het rapport openen met:

```bash
pnpm playwright show-report
```

De `test-results` folder wordt automatisch aangemaakt door Playwright wanneer een test mislukt. Hij bevat debuginformatie om de fout te analyseren:

- error-context.md: beschrijving van de fout met stacktrace
- screenshots: snapshot van de pagina op het moment van de fout
- traces (trace.zip) : een opname van de volledige test die je kan bekijken via `pnpm playwright show-trace
videos` als video opname geconfigureerd is
  Bij een geslaagde test wordt er niets bewaard (of de folder wordt opgeruimd).

Negeer beide folders in git:

```gitignore
test-results/
playwright-report/
```

## De eerste test

Neem de documentatie [Writing tests](https://playwright.dev/docs/writing-tests) door.

In de map `tests` in de root van het project, maak het bestand `general.spec.ts` aan:

```ts
// tests/general.spec.ts
import { test, expect } from '@playwright/test'; // 👈 1

// 👇 2
test('app loads', async ({ page }) => {
  await page.goto('/'); // 👈 3
  await expect(page.getByText('Budget')).toBeVisible(); // 👈 4
});
```

1. Importeer `test` en `expect` uit `@playwright/test`. Dit zijn de enige imports die je nodig hebt — geen extra plugins of configuratie.
2. Elke test is een `test()` functie met een beschrijving en een `async` callback. De callback ontvangt `{ page }` als parameter: het `page`-object stelt de browsertab voor en biedt alle methodes om met de pagina te interageren.
3. De meeste testen starten met het [navigeren](https://playwright.dev/docs/writing-tests#navigation) naar een url. `page.goto('/')` navigeert naar de `baseURL` die in `playwright.config.ts` geconfigureerd is. Je hoeft de volledige URL niet te herhalen in elke test.
4. `expect(page.getByText('Budget')).toBeVisible()` is een [**assertion**](https://playwright.dev/docs/writing-tests#assertions). Playwright wacht automatisch tot het element zichtbaar (`toBeVisible`) is of tot de timeout verloopt — je hoeft niet handmatig te wachten.

!> **Zorg er voor dat de back-end draait** voor de testen die echte API calls maken, anders falen ze.

## Anatomie van een UI test

Onze testen zullen vaak een gelijkaardig stramien hebben: een URL bezoeken, interageren met elementen op de pagina (iets typen in een inputveld, op een knop klikken...) en/of kijken of we het gewenste resultaat te zien krijgen.

In beide gevallen moeten we elementen van de DOM kunnen identificeren. Gewoon checken of er 'een' `h1` aanwezig is zal niet volstaan. Als er meerdere inputs, buttons, etc. zijn, moeten we zeker zijn dat we met de juiste elementen interageren.

Playwright voorziet in een aantal locators. Hieronder vind je de meest gebruikte.

### getByTestId: via `data-testid` attribuut

Je voegt zelf een `data-testid` toe aan het HTML-element:

```html
<button type="submit" disabled="{isSubmitting}" data-testid="submit">
  {transaction?.id ? 'Save transaction' : 'Add transaction'}
</button>
```

Playwright heeft hiervoor een ingebouwde methode:

```ts
page.getByTestId('submit').click();
```

Je bent 100% zeker welk element je vindt. Als iemand de tekst of het CSS aanpast, breekt de test niet. De [`data-testid`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes) maakt ook duidelijk dat het element door een test gebruikt wordt.

### getByRole: via ARIA-rol en naam

Gebruikt voor elementen die een semantische rol hebben, zoals knoppen, koppen en formuliervelden. Merk op dat veel HTML-elementen zoals `<button>` een impliciet gedefinieerde rol hebben die herkend wordt door de role locator.

```ts
page.getByRole('button', { name: 'Add transaction' }).click();
page.getByRole('heading', { name: 'Add Transaction' }).waitFor();
page.getByRole('table').waitFor({ state: 'visible' });
page.getByRole('combobox').click(); // een <select> of dropdown
page.getByRole('option').first().click(); // een optie in een dropdown
page.getByRole('alert'); // een foutmelding
```

Dit is hoe een gebruiker (en schermlezers) de pagina ervaren. De `name` is de zichtbare tekst op de knop of het label van het veld. Gebruik role locators waar mogelijk, omdat dit de manier is waarop gebruikers en ondersteunende technologie (zoals schermlezers) de pagina ervaren.

### getByText: via zichtbare tekst

```ts
page.getByText('Budget');
```

Handig voor een snelle check of bepaalde tekst zichtbaar is op de pagina. Minder precies dan `getByTestId`: als de tekst verandert, breekt de test.

### getByPlaceholder: via placeholder-tekst

```ts
page.getByPlaceholder('Search by place…').fill('Ir');
```

Gebruikt voor inputvelden met een `placeholder` attribuut. Praktisch als er geen `data-testid` is, maar minder robuust.

Meer over [locators](https://playwright.dev/docs/locators)

## Add and remove transaction test

Als voorbeeld zullen we het toevoegen van een transactie testen. Eerst en vooral moeten we overal de juiste `data-testid` attributen toevoegen. Dit hoef je natuurlijk maar éénmaal te doen per component.

```tsx
// src/components/transactions/TransactionForm.tsx
 return (
    <FormProvider {...form}>
      <Error error={saveError} />
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <FieldGroup>
          <LabelInput
            label="User Id"
            name="userId"
            placeholder="user id"
            type="number"
            data-testid="user-input" // 👈 1
          />
          <LabelInput
            label="Amount"
            name="amount"
            placeholder="0.00"
            type="number"
            data-testid="amount-input" // 👈 1
          />
          <LabelDatePicker
            label="Date"
            name="date"
            disabled={{ after: new Date() }}
            testId="date-picker-trigger" // 👈 3
          />
          <LabelSelectList
            label="Place"
            name="placeId"
            items={placesSelectItems}
            placeholder="Place"
          />// 👈 3
        </FieldGroup>
        <div className="flex justify-end gap-2 pt-6">
          <Button type="submit" disabled={isSubmitting}>
            {transaction?.id ? 'Save transaction' : 'Add transaction'}
          </Button>// 👈 4
          <Link
            to="/transactions"
            className={cn(buttonVariants({ variant: 'outline' }))}
          >
            Cancel
          </Link>
        </div>
      </form>
    </FormProvider>
  );
}
```

1. Bij elke `input` zetten we een `data-testid` attribuut. De `LabelInput` component geeft alle onbekende props door aan het `input` element (via `{...rest}`).
2. Voor de `LabelSelectList` voegen we geen `data-testid` toe, want we kunnen de `combobox` en `option` vinden via hun ARIA-rol.
3. Hier maken we een andere prop aan vermits we dit willen toevoegen aan de `Button` in de `PopoverTrigger`
4. De submit button heeft geen `data-testid` nodig want we kunnen hem vinden via zijn zichtbare tekst met `getByRole('button', { name: 'Add transaction' })`.

In `LabelDatePicker.tsx` wijzigen we onderstaande

```tsx
//..
interface LabelDatePickerFieldProps extends Omit<
  ComponentProps<typeof Calendar>,
  'mode' | 'selected' | 'onSelect'
> {
  label: string;
  name: string;
  placeholder?: string;
  testId?: string;// 👈
}

const LabelDatePicker = ({
  label,
  name,
  placeholder = 'Pick a date',
  testId,// 👈
  ...rest
}: LabelDatePickerFieldProps) => {
  //...
 <PopoverTrigger
              render={
                <Button
                  variant="outline"
                  data-empty={!field.value}
                  className="justify-start text-left font-normal data-[empty=true]:text-muted-foreground"
                  data-testid={testId}// 👈
                />
              }
              className="flex w-full justify-between"
            >
//...
```

We moeten ook een `data-testid` toevoegen aan de `Button` in de ui component `calendar.tsx`.

```tsx
// src/components/ui/calendar.tsx
function CalendarDayButton({
  className,
  day,
  modifiers,
  locale,
  ...props
}: React.ComponentProps<typeof DayButton> & { locale?: Partial<Locale> }) {
  const defaultClassNames = getDefaultClassNames();

  const ref = React.useRef<HTMLButtonElement>(null);
  React.useEffect(() => {
    if (modifiers.focused) ref.current?.focus();
  }, [modifiers.focused]);

  return (
    <Button
      variant='ghost'
      size='icon'
      data-day={day.date.toLocaleDateString(locale?.code)}
      data-testid={day.date.toISOString().split('T')[0]} // 👈
      data-selected-single={
        modifiers.selected &&
        !modifiers.range_start &&
        !modifiers.range_end &&
        !modifiers.range_middle
      }
      //...
      {...props}
    />
  );
}
```

We hebben `data-testid` toegevoegd aan `CalendarDayButton` in `calendar.tsx` met als waarde de ISO-datumstring van de dag (bv. `'2025-10-01'`). Zo kunnen we in de test een specifieke dag opzoeken via `getByTestId(dateStr)`, waarbij `dateStr` op dezelfde manier berekend is (`toISOString().split('T')[0]`), en zijn de waarden gegarandeerd gelijk ongeacht de locale. Je zou ook de bestaande `data-day` kunnen gebruiken, maar die is afhankelijk van de locale en kan dus verschillen per gebruiker.

We passen de `TableRow` component in `Transaction` aan, zodat we kunnen berekenen hoeveel rijen er zijn voor en na het toevoegen van een transactie.

```tsx
// src/components/transactions/Transaction.tsx
<TableRow data-testid="transaction">
```

Voeg nu een nieuw bestand `tests/addTransaction.spec.ts` toe:

```ts
// tests/addTransaction.spec.ts
import { test, expect } from '@playwright/test';

test('should add a transaction', async ({ page }) => {
  // 👇1
  await page.goto('/transactions');
  await page.getByRole('table').waitFor({ state: 'visible' });

  // 👇2
  const transactions = page.getByTestId('transaction');
  const countBefore = await transactions.count();

  // 👇3
  await page.goto('/transactions/add');
  await page
    .getByRole('heading', { name: 'Add Transaction' })
    .waitFor({ state: 'visible' });

  // 👇4
  const now = new Date();
  now.setDate(now.getDate() - 2);
  const dateStr = now.toISOString().split('T')[0];
  await page.getByTestId('date-picker-trigger').click();
  await page.getByTestId(dateStr).click({ timeout: 2000 });
  await page.keyboard.press('Escape');

  // 👇5
  // Select first place
  await page.getByRole('combobox').click();
  await page.getByRole('option').first().waitFor({ state: 'visible' });
  await page.getByRole('option').first().click();

  // 👇6
  // Fill in amount
  await page.getByTestId('amount-input').fill('200');
  await page.getByTestId('amount-input').blur();

  // 👇7
  // Fill in user id
  await page.getByTestId('user-input').fill('1');
  await page.getByTestId('user-input').blur();

  // 👇8
  await page
    .getByRole('button', { name: 'Add transaction' })
    .click({ timeout: 2000 });
  await page.waitForURL('/transactions');

  // 👇9
  await expect(transactions).toHaveCount(countBefore + 1);
  await expect(transactions.first()).toContainText('Thomas');
  await expect(transactions.first()).toContainText('200');
});
```

1. Navigeer eerst naar `/transactions` en wacht tot de tabel zichtbaar is.
2. Tel daarna het huidig aantal rijen op — zo zijn we niet afhankelijk van een vast getal en werkt de test ook na een database-reset.
3. Navigeer naar het toevoegformulier en wacht met `waitFor({ state: 'visible' })` tot de heading geladen is. Dit is een expliciete wacht voor situaties waar Playwright het element nog net niet kan onderscheppen bij de navigatie.
4. **Datumkiezer**: de shadcn `DatePicker` is geen standaard `<input type="date">`. We klikken de trigger open met `getByTestId('date-picker-trigger')`, selecteren de datum via `getByTestId(dateStr)`, en sluiten de picker daarna met `Escape`.
5. **Plaatsselectie**: de shadcn `Select` wordt opengeklikt via zijn ARIA-rol `combobox`. De opties zijn zichtbaar als items met de rol `option`. We wachten tot de eerste optie zichtbaar is en klikken erop.
6. **Bedrag**: `fill('200')` vult het veld in. `blur()` verlaat het veld, wat de `onBlur`-validatie van React Hook Form triggert
7. **UserId**: `fill('1')` vult het veld in. `blur()` verlaat het veld, wat de `onBlur`-validatie van React Hook Form triggert
8. Klik de submit-knop op basis van zijn zichtbare tekst met `getByRole('button', { name: 'Add transaction' })`. `waitForURL('/transactions')` wacht tot de navigatie naar de overzichtspagina volledig afgerond is.
9. Controleer of er één transactie meer is dan vóór de test (`countBefore + 1`). De nieuwste transactie staat bovenaan (`transactions.first()`), dus we checken of die de naam `Thomas` en het ingevulde bedrag bevat.

> Merk op: In een e2e test zijn we niet beperkt tot één enkele assertion. Veel interacties vereisen meerdere stappen en zullen de toestandverandering op meer dan één manier verifiëren.

### Auto-wachten

Playwright wacht automatisch tot elementen beschikbaar, zichtbaar en interactief zijn vóór het er iets mee doet. Je hoeft geen expliciete `sleep()` of `wait()` toe te voegen. Dit maakt de testen stabieler dan bij tools die dit niet doen. Meer info: [Auto-waiting](https://playwright.dev/docs/actionability).

### Reproduceerbaarheid

Elke keer dat de test draait wordt een transactie toegevoegd en groeit de lijst.

We moeten ervoor zorgen dat onze testen geen blijvende wijzigingen veroorzaken. Dat kan op twee manieren:

- niet met de echte databank werken — **mocks** gebruiken (straks meer hierover)
- alle bewerkingen ook weer 'omkeren' (wij kiezen voor deze optie)

Voeg een verwijdertest toe in hetzelfde bestand:

```ts
// tests/addTransaction.spec.ts
// Run these tests in serial to avoid conflicts with the same transaction being added/removed by multiple tests
test.describe.configure({ mode: 'serial' }); // 👈1

test('should remove the transaction', async ({ page }) => {
  await page.goto('/transactions'); // 👈2
  await page.getByRole('table').waitFor({ state: 'visible' }); // 👈2

  const transactions = page.getByTestId('transaction'); // 👈3
  const countBefore = await transactions.count(); // 👈3

  await transactions
    .first()
    .getByRole('button', { name: 'Delete transaction' })
    .click(); // 👈4

  await expect(transactions).toHaveCount(countBefore - 1); // 👈5
});
```

1. **`test.describe.configure({ mode: 'serial' })`**: de testen in dit bestand draaien na elkaar (niet parallel). Dat is nodig omdat de eerste test een transactie toevoegt en de tweede die daarna verwijdert — bij parallel uitvoering zou de volgorde niet gegarandeerd zijn.
2. Navigeer naar `/transactions` en wacht tot de tabel zichtbaar is.
3. Tel het huidig aantal rijen
4. Zoek binnen de eerste rij (`transactions.first()`) de verwijder-knop op via zijn ARIA-rol en naam (`getByRole('button', { name: 'Delete transaction' })`). Omdat de nieuwst toegevoegde transactie bovenaan staat, is dit steeds de transactie die de vorige test toevoegde.
5. Controleer of het aantal rijen daarna met één gedaald is (`countBefore - 1`).

Nu kunnen we de testen opnieuw en opnieuw draaien zonder dat ze falen. Mogelijks moet je eerst de database resetten naar de beginstand.

## Oefening 1 - Foutboodschappen in TransactionForm

We hebben getest of ons formulier werkt bij geldige invoer. Minstens even interessant is het testen van foutgevallen: worden foutboodschappen correct getoond?

<!-- markdownlint-disable-next-line -->

Stappenplan

1. Voeg een nieuwe test toe aan `tests/addTransaction.spec.ts`.
2. Navigeer naar de juiste URL en typ 0 in het amount-veld.
3. Verlaat het veld (blur) zodat validatie getriggerd wordt.
4. Controleer of de foutboodschap verschijnt. Voeg indien nodig een `data-testid` toe aan de foutboodschap.
5. Extra: test ook het geval waarbij het veld leeg blijft.

Bekijk de [Playwright assertions documentatie](https://playwright.dev/docs/test-assertions) voor inspiratie.

- Oplossing +

  ```ts
  test('should show error message for an invalid amount', async ({ page }) => {
    await page.goto('/transactions/add');

    await page.getByTestId('amount-input').fill('0');
    await page.getByTestId('amount-input').blur();
    await expect(
      page.getByRole('button', { name: 'Add transaction' }),
    ).toBeDisabled();
  });
  ```

  TODO: @Thomas. IK heb de test op IsValid bij de knop weggehaald, want als ik niks heb ingevuld kan ik niet klikken op de Add knop en krijg ik geen melding te zien. Dus dan toch testen of ik de juiste melding krijg, of terug !isValid toevoegen aan de knop?

## Mocks

Alles op een echte back-end testen heeft zeker zijn nut, maar zo wil je niet alle testen schrijven. Het is relatief traag, alles terugzetten kan lastig zijn, en het beperkt wat je kan testen. Hoe zou je testen of de front-end alles correct toont als de back-end onbereikbaar is?

De oplossing heet **mocken**: we onderscheppen netwerkverzoeken en geven een nep-antwoord terug. Playwright biedt hiervoor [`page.route()`](https://playwright.dev/docs/api/class-page#page-route).

Maak een nieuw bestand `tests/transactions.spec.ts`:

```ts
// tests/transactions.spec.ts
import { test, expect } from '@playwright/test';

test('should show the transactions', async ({ page }) => {
  // 👇 1
  await page.route('**/api/transactions?page=1&pageSize=10', async (route) => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        items: [
          {
            id: 1,
            amount: -97,
            date: '2025-10-01',
            user: { id: 1, name: 'Pieter' },
            place: { id: 4, name: 'Chinees Restaurant' },
          },
        ],
      }),
    });
  });

  // 👇 2
  await page.goto('/');
  const rows = page.getByTestId('transaction');
  await expect(rows).toHaveCount(1);
  await expect(rows.first()).toContainText('Chinees Restaurant');
  await expect(rows.first()).toContainText('01/10/2025');
});
```

1. [`page.route()`](https://playwright.dev/docs/api/class-page#page-route) onderschept netwerkverzoeken die overeenkomen met het patroon (`**` matcht elk domein). De callback geeft het nep-antwoord terug via [`route.fulfill()`](https://playwright.dev/docs/api/class-route#route-fulfill).
2. Daarna schrijven we de test. De componenten renderen met onze fake data omdat de echte API call nooit de back-end bereikt.

## Fixtures

Inline JSON in een `route.fulfill()` is niet handig bij grotere datasets. Je kan dit oplossen door JSON-bestanden te gebruiken. Maak een map `tests/fixtures/` aan en voeg daarin `transactions.ts` toe:

```ts
// tests/fixtures/transactions.ts
export const mockGetAllTransactions = {
  items: [
    {
      id: 1,
      amount: 97,
      date: '2025-10-01',
      user: { id: 2, name: 'Pieter' },
      place: { id: 4, name: 'Chinees Restaurant' },
    },
    {
      id: 2,
      amount: 100,
      date: '2025-10-01',
      user: { id: 2, name: 'Pieter' },
      place: { id: 3, name: 'Irish Pub' },
    },
  ],
  page: 1,
  pageSize: 10,
  total: 2,
};
```

Pas de test aan om het fixture-bestand te laden:

```ts
// tests/transactions.spec.ts
import { test, expect } from '@playwright/test';
import { mockGetAllTransactions } from './fixtures/transactions'; // 👈

test('should show the transactions', async ({ page }) => {
  await page.route('**/api/transactions?page=1&pageSize=10', (route) =>
    route.fulfill({ json: mockGetAllTransactions }),
  ); // 👈
  await page.goto('/');

  const rows = page.getByTestId('transaction');
  await expect(rows).toHaveCount(2); // 👈
  await expect(rows.first()).toContainText('Chinees Restaurant');
  await expect(rows.first()).toContainText('01/10/2025');
});
```

Zo is de testdata makkelijker te hergebruiken en aan te passen. Het fixture-bestand is ook veel leesbaarder dan een inline JSON-string.

## beforeEach

We wensen de json op te halen voor elke test:

```tsx
import { test, expect } from '@playwright/test';
import { mockGetAllTransactions } from './fixtures/transactions';

test.beforeEach(async ({ page }) => {
  await page.route('**/api/transactions?page=1&pageSize=10', (route) =>
    route.fulfill({ json: mockGetAllTransactions }),
  );
}); // 👈

test('should show the transactions', async ({ page }) => {
  await page.goto('/');

  const rows = page.getByTestId('transaction');
  await expect(rows).toHaveCount(2);
  await expect(rows.first()).toContainText('Chinees Restaurant');
  await expect(rows.first()).toContainText('01/10/2025');
});
```

## Vertraagde responses

Je kan ook simuleren dat de back-end traag reageert, om te testen of je loading indicator getoond wordt. Voeg deze test toe aan `tests/transactions.spec.ts`:

```ts
// tests/transactions.spec.ts
test('should show a loading indicator for a very slow response', async ({
  page,
}) => {
  // 👇 1
  await page.route('**/api/transactions?page=1&pageSize=10', async (route) => {
    await new Promise((resolve) => setTimeout(resolve, 1000)); // 👈 2
    await route.continue(); // 👈 3
  });

  await page.goto('/'); // 👈 4

  await expect(page.getByTestId('loader')).not.toBeVisible({ timeout: 3000 }); // 👈 5
});
```

1. We onderscheppen de API call met `page.route()`. Deze handler overschrijft de `beforeEach`-mock omdat Playwright routes in omgekeerde volgorde van registratie verwerkt — de laatste geregistreerde route wint.
2. In plaats van `route.fulfill()` voegen we hier een vertraging van 1 seconde in met `setTimeout` vóór de `continue()`. Dit simuleert een trage netwerkverbinding.
3. `route.continue()` stuurt het request wél door naar de echte back-end, maar pas na de vertraging. Zo testen we het laadgedrag zonder een nep-antwoord te hoeven schrijven.
4. Navigeer naar de pagina. De loading indicator verschijnt terwijl het vertraagde request nog onderweg is.
5. `expect(...).not.toBeVisible({ timeout: 3000 })` controleert dat de loading indicator uiteindelijk verdwijnt. De `timeout: 3000` geeft Playwright 3 seconden de tijd — ruim genoeg na de 1 seconde vertraging. Voeg een `data-testid='loader'` toe aan de `Loader` component zodat Playwright hem kan vinden.

## Oefening 2 - Zoekfunctie van transacties

Schrijf volgende testen voor de zoekfunctie van onze transacties:

- correcte invoer
- invoer zonder resultaten
- fouten in de back-end

### Correcte invoer

Als naar "Ir" gezocht wordt, willen we enkel de transacties van "Irish Pub" zien.

- Voeg `data-testid` attributen toe waar nodig.
- Gebruik de fixture `transactions.ts`.
- Check of er 1 transactie in de lijst voorkomt.
- Check of die transactie "Irish Pub" bevat.

### Invoer zonder resultaten

Als er naar "xyz" gezocht wordt, mag er geen enkel element getoond worden. Check ook of de boodschap verschijnt.

### Fouten in de back-end

Als de back-end een fout geeft, zijn er geen transacties zichtbaar maar wel een foutboodschap. Gebruik `route.fulfill()` met `status: 500`.

- Oplossing +

  ```ts
  test('should show all transactions for the Irish pub', async ({ page }) => {
    await page.route(
      '**/api/transactions?page=1&pageSize=10&search=Ir',
      (route) =>
        route.fulfill({
          json: {
            items: [mockGetAllTransactions.items[1]],
            page: 1,
            pageSize: 10,
            total: 1,
          },
        }),
    );

    await page.goto('/');

    await page.getByPlaceholder('Search by place…').fill('Ir');
    await page.getByRole('button', { name: 'Search' }).click();

    const rows = page.getByTestId('transaction');
    await expect(rows).toHaveCount(1);
    await expect(rows.first()).toContainText('Irish Pub');
  });

  test('should show a message when no transactions are found', async ({
    page,
  }) => {
    await page.route(
      '**/api/transactions?page=1&pageSize=10&search=xyz',
      (route) =>
        route.fulfill({
          json: {
            items: [],
            page: 1,
            pageSize: 10,
            total: 0,
          },
        }),
    );
    await page.goto('/');

    await page.getByPlaceholder('Search by place…').fill('xyz');
    await page.getByRole('button', { name: 'Search' }).click();

    await expect(page.getByTestId('no_transactions_message')).toBeVisible();
  });

  test('should show an error if the API call fails', async ({ page }) => {
    await page.route('\*\*/api/transactions?page=1&pageSize=10', (route) =>
      route.fulfill({ status: 500, json: { error: 'Internal server error' } }),
    );

    await page.goto('/');

    await expect(page.getByRole('alert')).toBeVisible();
  });
  ```

## Oefening 3 - Paginatie

Controleer of de paginatie correct werkt.

- Oplossing +

  ```ts
  test('should navigate to the next and previous page', async ({ page }) => {
    await page.route('**/api/transactions?page=1&pageSize=10', (route) =>
      route.fulfill({
        json: { ...mockGetAllTransactions, total: 12 },
      }),
    );
    await page.route('**/api/transactions?page=2&pageSize=10', (route) =>
      route.fulfill({
        json: {
          items: [
            {
              id: 3,
              amount: 50,
              date: '2026-11-01',
              user: { id: 1, name: 'Thomas' },
              place: { id: 1, name: 'HoGent' },
            },
            {
              id: 4,
              amount: 75,
              date: '2026-11-02',
              user: { id: 1, name: 'Thomas' },
              place: { id: 2, name: 'HoGent' },
            },
          ],
          page: 2,
          pageSize: 10,
          total: 12,
        },
      }),
    );

    await page.goto('/');
    await expect(page.getByTestId('transaction')).toHaveCount(2);
    await expect(page.getByText('1 / 2')).toBeVisible();

    await page.getByRole('button', { name: 'Go to next page' }).click();
    await expect(page.getByTestId('transaction')).toHaveCount(2);
    await expect(page.getByText('2 / 2')).toBeVisible();
    await expect(page.getByTestId('transaction').first()).toContainText(
      'HoGent',
    );
    await expect(
      page.getByRole('button', { name: 'Go to next page' }),
    ).toHaveAttribute('aria-disabled', 'true');

    await page.getByRole('button', { name: 'Go to previous page' }).click();
    await expect(page.getByTestId('transaction')).toHaveCount(2);
    await expect(page.getByText('1 / 2')).toBeVisible();
    await expect(page.getByTestId('transaction').first()).toContainText(
      'Chinees Restaurant',
    );
  });
  ```

## Oefening 4 - README

Pas `README.md` aan zodat de gebruiker weet hoe de testen uitgevoerd moeten worden:

```bash
pnpm test          # alle testen in de terminal
pnpm test:ui       # interactieve UI mode
```

## Mogelijke extra's voor de examenopdracht

- [Cypress](https://www.cypress.io/)
