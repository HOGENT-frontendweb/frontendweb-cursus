# FAQ

In dit hoofdstuk behandelen we nog enkele leuke extra's die nuttig kunnen zijn tijdens het ontwikkelen, alsook antwoorden
op de meest gestelde vragen.

## Debugging

Een applicatie ontwikkelen zonder eens te moeten debuggen is een utopie, ook in React.

Net zoals in vanilla JavaScript kan je hier gebruik maken van o.a. `console.log`, maar op die manier debuggen is tijdrovend en lastig.

Uiteraard heb je ook het [`debugger`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger?retiredLocale=nl ':ignore') statement, maar daarvoor moet je in de browser de inspector tools open staan hebben.

Het zou handig zijn als we in VS Code konden debuggen... Uiteraard kan dit ook. [Lees in deze tutorial hoe je dit opzet](https://profy.dev/article/debug-react-vscode ':ignore').

## Linting

Linting is statische analyse van code om problemen zoals verkeerde syntax en twijfelachtig gebruik van code te detecteren. Waarom zou je gebruiken maken van linting en formatting? Het kan vroegtijdig fouten, typo's en syntax errors vinden. Het verplicht developers dezelfde codeerstijl te gebruiken, best practices te volgen en vermijdt het committen van slechte code.

[ESLint](https://github.com/eslint/eslint ':ignore'), gecreëerd door Nicholas C. Zakas in 2013, is een linting tool voor JavaScript. [Airbnb](https://github.com/airbnb/javascript ':ignore') heeft een eigen coding style opgesteld. Je kan hiervan vertrekken, of van de standaard aanbevolen instellingen van ESLint.

Installeer ESLint:

```bash
yarn add eslint --dev
```

Verwijder de bestaande ESLint configuratie uit de `package.json`.

Voer onderstaand commando uit om een standaard configuratie voor ESLint te krijgen:

```bash
npx eslint --init
```

Kies hierbij volgende settings:

- How would you like to use ESLint?: To check syntax, find problems and enforce code style
- What type of modules does your project use?: JavaScript modules
- Which framework does your project use?: React
- Does your project use TypeScript?: No
- Where does your code run?: Browser
- How would you like to define a style for your project?: Use a popular style guide
- Which style guide do you want to follow?: Airbnb
- What format do you want your config file to be in? JSON
- Would you like to install them now?: Yes
- Which package manager do you want to use?: yarn

Aangezien we gebruik maken van React 18, moeten we `plugin:react/jsx-runtime` toevoegen aan de `extends` property van `.eslintrc.json` (zie [documentatie](https://github.com/jsx-eslint/eslint-plugin-react#configuration-legacy-eslintrc)).

Voeg een extra script toe aan de `package.json`:

```json
{
  "scripts": {
    "lint": "npx eslint . --fix --ext jsx,js"
  }
}
```

Voer dit script uit en fix de fouten, een paar hints:

- Voeg onderstaande property toe aan de `.eslintrc.json`:

```json
{
  "ignorePatterns": [
    "node_modules",
    "*.config.js"
  ],
}
```

- `App.test.js` mag je simpelweg verwijderen aangezien we geen component testen schrijven.
- Sommige fouten kan je oplossen door bestanden met JSX een extensie `jsx` te geven (i.p.v. `js`).
