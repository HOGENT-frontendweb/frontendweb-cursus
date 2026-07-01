# E2E testen met authenticatie

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les9 b333700
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
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#web-services-budget) voor meer informatie.

Momenteel moet je voor elke pagina in onze budgetapplicatie aangemeld zijn (behalve de `/login` en `/logout`). Onze testen gaan er nog steeds van uit dat je niet aangemeld moet zijn en dus zullen deze één voor één falen.

## Authenticatie

Playwright biedt een ingebouwde manier om authenticatie te hergebruiken via **`storageState`**. Het idee is eenvoudig: één keer inloggen, de browsertoestand (cookies + localStorage) opslaan in een bestand, en daarna alle testen met die opgeslagen toestand starten. Zo vermijd je dat elke test opnieuw moet inloggen.

Neem de documentatie [Authentication](https://playwright.dev/docs/auth) door.

### Setup project

Voeg een apart **setup project** toe aan `playwright.config.ts`:

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';
import path from 'node:path';

export default defineConfig({
  // ...
  projects: [
    // 👇 1
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/,
    },
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        storageState: 'playwright/.auth/user.json', // 👈 2
      },
      dependencies: ['setup'], // 👈 3
    },
  ],
});
```

1. Het `setup`-project voert alle bestanden uit die eindigen op `.setup.ts`. Deze worden uitgevoerd **vóór** de eigenlijke testen.
2. `storageState` laadt de opgeslagen browserstate (cookies, localStorage) zodat elke test als aangemelde gebruiker start.
3. `dependencies: ['setup']` zorgt ervoor dat het `chromium`-project pas start nadat het `setup`-project volledig uitgevoerd is.

Voeg ook de gegenereerde bestanden toe aan `.gitignore` — ze bevatten sessiedata die je niet wil committen:

```gitignore
playwright/.auth/
```

### Auth setup bestand

Maak het bestand `tests/setup/auth.setup.ts` aan:

```ts
// tests/setup/auth.setup.ts
import { test as setup } from '@playwright/test'; // 👈 1
import path from 'node:path';
import process from 'node:process';

const authFile = path.join(process.cwd(), 'playwright/.auth/user.json'); // 👈 2

setup('authenticate', async ({ page }) => {
  // 👈 3
  await page.goto('/login'); // 👈 4
  await page
    .getByPlaceholder('your@email.com')
    .fill('thomas.aelbrecht@hogent.be'); // 👈 5
  await page.getByPlaceholder('password').fill('12345678'); // 👈 5
  await page.getByRole('button', { name: 'Sign in' }).click(); // 👈 5

  await page.waitForURL('/'); // 👈 6

  await page.context().storageState({ path: authFile }); // 👈 7
});
```

1. We importeren `test` onder de alias `setup` om duidelijk te maken dat dit geen gewone test is maar een voorbereidingsstap.
2. Het pad naar het bestand waar de browserstate opgeslagen wordt. De map `playwright/.auth/` wordt automatisch aangemaakt.
3. De setup-test heeft een beschrijvende naam (`'authenticate'`) die zichtbaar is in de testoutput.
4. Navigeer naar de loginpagina. De `baseURL` uit de config wordt als prefix gebruikt.
5. Vul het e-mailadres en wachtwoord in via placeholder-tekst en klik op de inlogknop. Dit zijn robuuste selectors die niet afhangen van `data-testid` of CSS.
6. Wacht tot de navigatie naar de homepage voltooid is — dan weten we zeker dat het inloggen gelukt is.
7. Sla de volledige browserstate op (cookies, localStorage, sessionStorage) naar het JSON-bestand. Playwright laadt dit bestand automatisch voor elke test in projecten die `storageState` configureren.

## Testen aanpassen

Dankzij `storageState` starten alle testen automatisch als aangemelde gebruiker — je hoeft geen `beforeEach` met een loginactie toe te voegen. De bestaande testen in `tests/addTransaction.spec.ts` en `tests/transactions.spec.ts` werken daardoor zonder aanpassingen.

Voer de testen uit:

```bash
pnpm test:ui
```

Playwright voert eerst het `setup`-project uit (inloggen + opslaan), daarna alle andere testen met de opgeslagen sessie.
Als de andere testen niet langer getoond worden: In de Playwright UI staat er linksboven een Projects filter. Momenteel staat waarschijnlijk alleen setup aangevinkt. Vink ook chromium aan om alle testbestanden te zien.

Als `auth.setup` faalt, controleer dan `playwright/.auth/` map bestaat. Indien niet, maak hem aan:

```bash
mkdir -p playwright/.auth
```

## Oefening - Foutboodschappen in TransactionForm

- De bestaande testen falen mogelijk om verschillende redenen nu authenticatie vereist is: bv. het `userId`-veld is verdwenen omdat de backend de gebruiker nu uit de token haalt.
- Los alle fouten op en zorg dat de testen slagen.

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les9-opl 1e5986a
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.
