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

[ESLint](https://github.com/eslint/eslint ':ignore'), gecreëerd door Nicholas C. Zakas in 2013, is een linting tool voor JavaScript. [Airbnb](https://github.com/airbnb/javascript ':ignore') heeft een eigen coding style opgesteld. Je kan hiervan vertrekken, of van de standaard aanbevolen instellingen van ESLint of het Vite template.

Installeer ESLint en specifieke React plugins:

```bash
yarn add --dev eslint eslint-plugin-react eslint-plugin-react-hooks
```

- `eslint`: de linting tool
- `eslint-plugin-react`: de plugin met specifieke regels voor React
- `eslint-plugin-react-hooks`: de plugin met specifieke regels rond React hooks

Verwijder eventueel bestaande ESLint configuratie uit de `package.json` en maak een `.eslintrc.js` bestand aan in de root van je project. Voeg volgende inhoud toe:

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2020: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended',
    'plugin:import/recommended',
  ],
  ignorePatterns: [
    'dist',
    'node_modules',
    '.eslintrc.cjs',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  settings: {
    react:{
      version: '18.2',
    },
    "import/resolver": {
      node: {
        extensions: [
          ".js",
          ".jsx",
        ],
      },
    },
  },
  rules: {
    'react/prop-types': 'off',
    'comma-dangle': ['error', 'always-multiline'],
    semi: ['error', 'always'],
  },
};
```

Voeg een extra script toe aan de `package.json`:

```json
{
  "scripts": {
    "lint": "eslint . --ext js,jsx --fix --report-unused-disable-directives --max-warnings 0"
  }
}
```

De opties van het `eslint` script betekenen:

- `--ext js,jsx`: enkel `.js` en `.jsx` bestanden linten
- `--fix`: automatisch op te lossen fouten oplossen
- `--report-unused-disable-directives`: rapporteer ongebruikte `eslint-disable` commentaar
- `--max-warnings 0`: er mogen ook geen warning optreden (als er zijn, krijg je een exit code != 0)
