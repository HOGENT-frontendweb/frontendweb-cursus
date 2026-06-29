# Context API

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les6 6ad1816
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
> git checkout -b les6 3386a40
> pnpm install
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#webservices-budget) voor meer informatie.

We creëren geneste componenten om de UI te bouwen. De state plaatsen we in de root component en wordt via props doorgegeven aan de kinderen. Dit kan echter heel complex worden als je sommige props tot diep in de boom dient door te geven of als heel wat componenten dezelfde props nodig hebben.

De **Context API** laat toe om data globaal bij te houden en door te geven aan child components, zonder dat we via props de data tot in deze kinderen moeten doorgeven. Dus Context API is een alternatief voor het doorgeven van props.

**Let op**: de Context API wordt vaak ten onrechte gebruikt als oplossing. Gebruik het enkel en alleen als de state echt globaal moet zijn. Gebruik de context ook zo laag mogelijk in de component tree. Alle componenten onder de context worden nl. opnieuw gerenderd als de waarde in de context wijzigt! Een aantal use cases voor context zijn o.a. theming, taalkeuze, huidige gebruiker...

De Context API bestaat uit drie bouwstenen:

1. Een **Context Object**, aangemaakt door de factory functie `createContext`
2. Een **Context Provider**: voorziet de onderliggende componenten van data
3. Meerdere **Context Consumers** : ontvangen de data van de context (= onze eigen componenten)

## Licht of donker thema in shadcn

shadcn/ui gebruikt de `class` strategie van Tailwind CSS: dark mode wordt geactiveerd door de class `dark` toe te voegen aan het `<html>` element. Alle child elementen erven dan automatisch de dark mode stijlen.

shadcn maakt gebruik van **CSS-variabelen**. In `index.css` worden kleuren gedefinieerd via `:root` (light) en `.dark` (dark) blokken:

```css
@layer base {
  :root {
    --background: oklch(1 0 0);
    --foreground: oklch(0.145 0 0);
  }
  .dark {
    --background: oklch(0.145 0 0);
    --foreground: oklch(0.985 0 0);
  }
}
```

shadcn-componenten gebruiken semantische utility klassen zoals `bg-background` en `text-foreground` die naar deze variabelen verwijzen. Hierdoor schakelen **alle** shadcn-componenten automatisch tussen light en dark mode zonder dat je per component `dark:bg-gray-900` hoeft te schrijven.

## Licht of donker thema in onze applicatie

Om de werking van de React Context API te demonstreren kan de gebruiker van onze budgetapplicatie een licht of donker thema kiezen. We gaan hiervoor een context gebruiken aangezien het thema van de website door heel veel componenten gekend moet zijn om bv. de kleuren, beelden,... aan te passen.

Het aanmaken van een context gebeurt typisch in 3 stappen:

1. Creëer een context.
2. Bied de context aan via een provider.
3. Neem data uit de context via een consumer (of dus in een child component).

### Stap 1: Creëer de context

In de navigatiebalk voorzien we een knop om het thema te kiezen. We maken in de `main.tsx`, een context aan m.b.v. `createContext`. Deze factory-functie heeft één argument, de defaultwaarde. Exporteer `ThemeContext` zodat de consumers dit kunnen gebruiken.

```tsx
// src/main.tsx
//...
import { createContext } from 'react'; // 👈 1

// 👇 2
interface ThemeContextType {
  darkmode: boolean;
}

export const ThemeContext = createContext<ThemeContextType>({
  // 👈 3
  isDark: false,
});

//...
```

1. Importeer `createContext` uit React.
2. `ThemeContextType` is een TypeScript interface die de vorm van de context beschrijft. Door dit type te definiëren krijg je autocomplete en typeveiligheid in alle consumers.
3. `createContext<ThemeContextType>` koppelt het type aan de context. De verplichte standaardwaarde `{ darkmode: false }` wordt enkel gebruikt als een consumer de context aanroept **buiten** een provider — in de praktijk zal dit zelden voorkomen, maar TypeScript vereist een waarde die aan `ThemeContextType` voldoet.

### Stap 2: Bied de context aan

Voeg toe in `main.tsx`:

```tsx
// src/main.tsx
//...
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <ThemeContext.Provider value={{ isDark: true }}>
      <RouterProvider router={router} />
    </ThemeContext.Provider>
  </StrictMode>,
);
```

Elk **context object** wordt beschikbaar gemaakt met een **context provider** component waarin de data geplaatst wordt. Een context provider plaats je rond de volledige component tree of bepaalde secties ervan. Alle kinderen (de **context consumers**) binnen de context provider hebben toegang tot de data en kunnen zich abonneren op wijzigingen. De `value` property bevat de data die in de context geplaatst word. We overschrijven hier de default waarde. Geef een object door (vandaar `{{}}`). Alle kinderen onder de provider zullen opnieuw renderen wanneer de waarde van de `Provider` verandert.

> Het is niet verplicht om een context steeds in de hoogste component uit de component tree te zetten, je zet hem zo laag mogelijk zodat de nodige child componenten aan de data kunnen.

### Stap 3: Consume de context

De data hoeft niet langer doorgegeven te worden via props. Gebruik bv. de darkmode in de `Star` component. De `Star` component zal de data consumeren, en is een context consumer. De `dark` mode zal de kleur van de ster bepalen.

```tsx
// src/components/places/StarRating.tsx
import { useContext } from 'react'; // 👈 1
import { ThemeContext } from '../../main'; // 👈 1
// ...

function Star({ index, selected = false, onSelect = () => {} }: StarProps) {
  const handleSelect = useCallback(
    () => onSelect(index + 1),
    [index, onSelect],
  );
  const { isDark } = useContext(ThemeContext); // 👈 1
  return (
    <StarIcon
      size={22}
      className={cn('cursor-pointer transition-all duration-150 fill-current', {
        'text-amber-400 scale-110 drop-shadow-sm': selected && !isDark, // 👈 2
        'text-red-400 scale-110 drop-shadow-sm': selected && isDark, // 👈 2
        'text-muted-foreground/40': !selected,
      })}
      onClick={handleClick}
    />
  );
}
```

1. De `useContext` hook wordt gebruikt om met de context te connecteren en heeft als parameter een context object `ThemeContext`. Het retourneert de `value` van de huidige context (zie value property van de context provider).
2. De kleur van de geselecteerde ster hangt af van de `darkmode` waarde uit de context. Zo demonstreren we visueel dat de context de UI beïnvloedt.

De context provider kan data in de context plaatsen, maar het kan de data in de context niet aanpassen. Willen we ook nog functies toevoegen aan de Context om te wisselen van dark naar light mode of omgekeerd? Dan moeten we een aparte component aanmaken.

## ThemeContext en Provider

Maak een map `contexts` aan in de map `src` met daarbinnen en folder `theme` en het bestand `Theme.context.tsx`:

```tsx
// src/contexts/theme/Theme.context.tsx
import { createContext } from 'react'; // 👈 1

interface ThemeContextType {
  isDark: boolean;
  toggleTheme: () => void;
} // 👈 1
createContext<ThemeContextType>({
  isDark: false,
  toggleTheme: () => {},
}); // 👈 1

// 👇 2
export const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  // 👇 3
  return <ThemeContext.Provider>{children}</ThemeContext.Provider>;
};
```

1. `ThemeContextType` beschrijft wat de context aanbiedt: `isDark` (huidige toestand) en `toggleTheme` (functie om te wisselen). De standaardwaarde `{ isDark: false, toggleTheme: () => {} }` wordt enkel gebruikt als een consumer de context aanroept buiten een provider. Omdat het type `ThemeContextType` is (zonder `| undefined`), moet de standaardwaarde een volledig geldig object zijn dat aan de interface voldoet — vandaar de lege functie `() => {}` als placeholder voor `toggleTheme`.
2. `ThemeProvider` is een component die de `children` als prop ontvangt. `children` stelt de volledige component tree voor waar rond je de provider plaatst. De provider beheert de state en stelt die ter beschikking van alle children. Dit implementeren we in de volgende stap.
3. De provider rendert `ThemeContext.Provider` en geeft de children mee. Voorlopig ontbreekt de `value` prop nog — die voegen we toe zodra we de state en functies definiëren.

De `ThemeProvider` beheert de state en functies, en stelt deze ter beschikking aan de children. Hou hier als state het thema bij en een functie om te wisselen van thema.

```tsx
// src/contexts/theme/Theme.context.tsx
import { useState, useEffect } from 'react';
import { createContext } from 'react';

interface ThemeContextType {
  isDark: boolean;
  toggleTheme: () => void;
}

export const ThemeContext = createContext<ThemeContextType>({
  isDark: false,
  toggleTheme: () => {},
});

export const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  // 👇 1
  const [isDark, setIsDark] = useState(
    () => localStorage.getItem('themeMode') === 'dark',
  );

  // 👇 2
  useEffect(() => {
    if (isDark) {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
    localStorage.setItem('themeMode', isDark ? 'dark' : 'light');
  }, [isDark]);

  // 👇 3
  const toggleTheme = () => setIsDark((prev) => !prev);

  const value = { isDark, toggleTheme };

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  ); // 👈 5
};
```

1. `isDark` is de state-variabele die bijhoudt of dark mode actief is. De lazy initializer (`() => ...`) wordt slechts één keer uitgevoerd bij de eerste render: als `localStorage` de waarde `'dark'` bevat start de applicatie meteen in dark mode, anders in light mode.
2. De `useEffect` synchroniseert de state met de DOM en `localStorage` telkens wanneer `isDark` wijzigt. De DOM-update voegt de class `dark` toe aan `document.documentElement` (het `<html>` element) of verwijdert ze — dit is precies hoe shadcn dark mode werkt. `localStorage` bewaart de voorkeur zodat ze bewaard blijft na een page refresh.
3. `toggleTheme` is de functie die de state omschakelt via `setIsDark`. Door ze via de `value` door te geven kunnen consumers het thema aanpassen zonder de state rechtstreeks te manipuleren.
4. `value` is een **object** met de state en de toggle-functie. Consumers destructureren dit als `const { isDark, toggleTheme } = useContext(ThemeContext)`. De eigenschapsnamen van `value` moeten overeenkomen met de interface `ThemeContextType`.
5. De `ThemeProvider` geeft de `value` door aan alle children via de context provider. Elk child dat de context consumeert wordt opnieuw gerenderd wanneer `isDark` wijzigt.

We krijgen een eslint fout `Fast refresh only works when a file only exports components`. Fast refresh is een feature van React die ervoor zorgt dat de applicatie snel herladen wordt bij wijzigingen in de code. Dit werkt enkel als een bestand enkel componenten exporteert. Maak daarom een nieuw bestand `index.ts` aan in de `contexts/theme` folder en plaats alle exports die geen component zijn in dit bestand. Pas eventueel de verwijzingen aan.

```ts
// src/contexts/theme/index.ts
import { createContext } from 'react';

// 👇 1
interface ThemeContextType {
  isDark: boolean;
  toggleTheme: () => void;
}

// 👇 2
export const ThemeContext = createContext<ThemeContextType>({
  isDark: false,
  toggleTheme: () => {},
});

);
```

### Providing ThemeContext

Stel in `main.tsx` de `ThemeProvider` ter beschikking aan alle children. Verwijder de reeds toegevoegde code.

```tsx
// src/main.tsx
// ... imports
import { ThemeProvider } from './contexts/theme/Theme.context.tsx'; // 👈

// ...

createRoot(document.getElementById('root')).render(
  <StrictMode>
    {/* 👇 */}
    <ThemeProvider>
      <RouterProvider router={router} />
    </ThemeProvider>
    {/* 👆 */}
  </StrictMode>,
);
```

### Consuming ThemeContext

Voeg in `Navbar.tsx` een knop toe om van thema te wisselen:

```tsx
// src/components/Navbar.tsx
import { NavLink, Link, useLocation } from 'react-router';
import { useState } from 'react';
import { useContext } from 'react'; //👈 1
import { ThemeContext } from '../contexts/theme'; // 👈 1
import { PiggyBankIcon, Moon, Sun, Menu, X } from 'lucide-react'; // 👈 4
import { Button } from './ui/button';// 👈 2
//...

{/* 👇 2*/}
const ThemeToggle = () => {
  const { isDark, toggleTheme } = useContext(ThemeContext);
  return (
    <Button
      variant='ghost'
      size='icon'
      onClick={toggleTheme}
      aria-label='Toggle theme'
    >
      {isDark ? <Moon className='h-4 w-4' /> : <Sun className='h-4 w-4' />}
    </Button>
  );
};

 return (
    <header className="sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur">
      <div className="container mx-auto flex h-14 max-w-5xl items-center px-4">
        <Link to="/transactions" className="flex items-center gap-2 font-semibold text-primary mr-6">
          <PiggyBankIcon className="size-5" />
          Budget
        </Link>

        {/* Desktop nav */}
        <div className="hidden md:flex flex-1">
          <NavMenu />
        </div>
        {/* 👇 3 */}
        <div className="hidden md:flex items-center gap-2 ml-auto">
          <ThemeToggle />
        </div>

        {/* Mobile toggle */}
        <button className="ml-auto md:hidden" onClick={() => setIsOpen((prev) => !prev)} aria-label="Toggle menu">
          {isOpen ? <X className="h-5 w-5" /> : <Menu className="h-5 w-5" />}
        </button>
      </div>

      {/* Mobile menu */}
      {isOpen && (
        <div className="border-t md:hidden bg-background">
          <div className="container mx-auto px-4 py-4 max-w-5xl">
            <NavMenu vertical />
          </div>
          {/* 👇 3 */}
          <div className="flex items-center gap-2 pt-2 border-t w-full">
            <ThemeToggle />
          </div>
        </div>
      )}
    </header>
  );
}
```

1. Importeer de nodige componenten.
2. Maak een component aan met een knop om het thema te kiezen. Roep de `useContext` hook aan en gebruik destructuring om de properties, die de component nodig heeft, eruit te halen.
3. Voeg de knop toe aan de navbar.

Pas ook de `StarRating` component aan.

## Custom hooks

Een custom hook is een JavaScript functie die begint met `use` en die andere hooks kan aanroepen. Custom hooks laten toe om logica te delen tussen componenten. Je kan zelf custom hooks schrijven of zoeken naar bestaande custom hooks via bv. <https://nikgraf.github.io/react-hooks/> of <https://npmjs.com>.

In elke component die gebruik maakt van de context dienen we volgende code te schrijven:

```tsx
import { ThemeContext } from '../../contexts/Theme.context';

const { darkmode, toggleDarkmode } = useContext(ThemeContext);
```

Om duplicate code te vermijden kunnen we gebruik maken van een **custom hook**. Neem hiervoor eerst [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) door.

Maak vervolgens volgende custom hook aan in `index.ts`:

```tsx
// src/contexts/index.ts
import { createContext, useContext } from 'react'; // 👈1

interface ThemeContextType {
  isDark: boolean;
  toggleTheme: () => void;
}

export const ThemeContext = createContext<ThemeContextType | undefined>(
  undefined,
); // 👈 2

export const useTheme = (): ThemeContextType => {
  // 👈 3
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within a ThemeProvider');
  return context;
};
```

1. Importeer `useContext`.
2. **Waarom nu `undefined` en daarvoor een object?** In de eerste versie (`Theme.context.tsx`) gebruikten we een concrete standaardwaarde `{ isDark: false, toggleTheme: () => {} }`. Dat werkt, maar heeft een nadeel: als een consumer per ongeluk buiten de `ThemeProvider` gebruikt wordt, krijgt hij de standaardwaarde — zonder enige foutmelding. De bug is dan moeilijk te vinden. In deze `index.ts` kiezen we bewust voor `undefined` als standaardwaarde. Daardoor moet het type `ThemeContextType | undefined` zijn. Het voordeel: de `useTheme` hook kan nu controleren of de context wel degelijk voorzien werd. Ontbreekt de provider, dan gooit de hook onmiddellijk een duidelijke fout (zie punt 2) in plaats van verder te werken met een stille placeholder.
3. `useTheme` is een **custom hook** die de ruwe `useContext`-aanroep verbergt. De check `if (!context)` gooit een duidelijke fout als de hook per ongeluk buiten de `ThemeProvider` gebruikt wordt — veel nuttiger dan de cryptische `Cannot destructure property of undefined` die je anders zou krijgen. Consumers importeren voortaan `useTheme` in plaats van `useContext(ThemeContext)` rechtstreeks aan te roepen.

Zo kan de code in `Navbar.tsx` als volgt aangepast worden:

```tsx
// src/components/Navbar.tsx
//import { useContext } from 'react'; // 👈 1
//import { ThemeContext } from '../contexts/theme/Theme.context'; // 👈 1
import { useTheme } from '../contexts/theme'; // 👈 1

const ThemeToggle = () => {
  const { darkmode, toggleDarkmode } = useTheme(); // 👈 2
  //...
};
```

1. Verwijder de import `useContext`, `ThemeContext`, en importeer `useTheme`.
2. Destructure de waarden die in deze component gebruikt worden.

### Oefening 2 - useTheme

Pas de StarRating component aan zodat deze ook van de `useTheme` hook gebruik maakt om i.g.v. isDark een geselecteerde ster rood te kleuren.

## Anti-patterns

Er zitten een paar anti-patterns in ons formulier. Waarschijnlijk zijn deze ook aanwezig in jouw eigen project.

1. Gebruik geen constante object literals/arrays binnen de component, bv. validatieregels. Plaats deze buiten de component.
2. Definieer geen pure functies binnen de component (functies zonder afhankelijkheden van variabelen). Plaats deze buiten de component.
3. Definieer geen componenten inline in een andere component, bv. `Star` component. Plaats deze buiten de component.
4. Gebruik een id als waarde voor de `key` prop in lijsten, gebruik geen index.
   - Deze fout hebben we reeds opgelost in hoofdstuk 1.

### Duplicate code

De combinatie `label` en `input` tag komen vaak voor. Kunnen we hier aparte component van maken?

Componenten mag je niet definiëren binnen een andere component. Ofwel maak je een functiecomponent `LabelInput` in het bestand van het formulier ofwel als aparte component als je deze wil hergebruiken. We kiezen voor de 2de optie. Plaats de code van het invoerveld van de gebruiker hierin en maak van de hardgecodeerde waarden props.

```tsx
// src/components/LabelInput.tsx
import type { ComponentPropsWithoutRef } from 'react'; // 👈 1
import { Field, FieldLabel, FieldError } from '@/components/ui/field'; // 👈 2
import { Input } from '@/components/ui/input'; // 👈 2
import { Controller } from 'react-hook-form'; // 👈 3

// 👇 4
interface LabelInputInterface extends ComponentPropsWithoutRef<'input'> {
  label: string;
  name: string;
}

// 👇 5
const LabelInput = ({
  label,
  name,
  placeholder,
  type,
  ...rest
}: LabelInputInterface) => {
  return (
    <Controller
      control={control}
      name={name}
      render={({ field, fieldState }) => (
        <Field data-invalid={fieldState.invalid}>
          <FieldLabel id={field.name}>{label}</FieldLabel>
          <Input
            {...field}
            type={type}
            placeholder={placeholder}
            onChange={(e) =>
              field.onChange(
                type === 'number' ? e.target.valueAsNumber : e.target.value, // 👈 7
              )
            }
            {...rest} // 👈 6
          />
          {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
        </Field>
      )}
    />
  );
};

export default LabelInput;
```

1. Import van het type `ComponentPropsWithoutRef`.
2. Import shadcn-componenten .
3. Import `Controller` uit `react-hook-form`
4. `LabelInputInterface` erft alle standaard HTML `<input>` attributen via `React.ComponentPropsWithoutRef<'input'>` en voegt `label` en `name` als verplichte props toe. Zo kan de component gebruikt worden met alle `<input>` attributen die HTML ondersteunt (bv. `min`, `max`, `disabled`).
5. We maken een component `LabelInput`. We destructuren alle props(`label`, `name`, `placeholder`) die we nodig hebben. `...rest` vangt alle overige props op (bv. `disabled`, `min`, `max`) en geeft ze door aan het `<Input>` element. Zo hoef je de component niet aan te passen als je een extra attribuut wil doorgeven.
6. We spreiden ook alle rest props
7. Voor `type='number'` geeft `e.target.value` altijd een string terug. `e.target.valueAsNumber` converteert dit naar een getal, wat nodig is voor correcte validatie en verwerking van numerieke velden.

We krijgen nog fouten (zie verder). Importeer eerst de `LabelInput` component in `TransactionForm` component en pas de invoervelden aan. De code voor het `userId` inputveld wordt:

```tsx
<LabelInput
  label='User Id'
  name='userId'
  placeholder='user id'
  type='number'
/>;
{
  /* Herhaal dit voor de overige input fields */
}
```

We krijgen de fouten: `control not defined`. Oplossing: `useFormContext` en `FormProvider`. Definitie van `useFormContext` uit de documentatie:

> This custom hook allows you to access the FormContext. useFormContext is intended to be used in deeply nested structures, where it would become inconvenient to pass the context as a prop.

Met `FormProvider` creëren we een nieuwe context provider voor een formulier. Met `useFormContext` halen we hieruit de waarde op en gebruiken dit in een (diep) geneste component tree.

In de documentatie lezen we ook het volgende over de `FormProvider`:

> React Hook Form's FormProvider is built upon React's Context API. It solves the problem where data is passed through the component tree without having to pass props down manually at every level. This also causes the component tree to trigger a rerender when React Hook Form triggers a state update.

```tsx
// src/components/transactions/TransactionForm.tsx
import { useForm, FormProvider } from 'react-hook-form'; // 👈 1
// ...

export default function TransactionForm({
  places = [],
  transaction = EMPTY_TRANSACTION as Transaction,
}: TransactionFormProps) {
  // ...
  // 👇 2
   return (
    <FormProvider {...form}>
      <form ...>
      //...
      </form>
    </FormProvider>
  );
}
```

Plaats eventjes de `select` lijst in commentaar. Verderop wordt dit ook een aparte component.

1. Importeer de `FormProvider`.
2. Plaats de `FormProvider` rond het formulier en geef alles door om de `useFormContext` correct te laten werken voor gebruik in `LabelInput`.

Pas nu ook de `LabelInput` component aan:

```tsx
import { useFormContext } from 'react-hook-form'; // 👈

const LabelInput = ({
  label,
  name,
  placeholder,
  type,
  ...rest
}: LabelInputInterface) => {
  const { control } = useFormContext();// 👈
  return (
    <Controller
      control={control} // 👈
  //...
}
```

Importeer `useFormContext`. `useFormContext` haalt de `control` op uit de dichtstbijzijnde `FormProvider`. `control` is het object waarmee `Controller` de waarden en validatiestatus van een veld beheert.

### Oefening 3 - LabelSelectList

Maak een `LabelSelectList` component aan.

- Oplossing +

  ```tsx
  // src/components/LabelSelectList.tsx
  import type {} from 'react';
  import { Controller, useFormContext } from 'react-hook-form';
  import { Field, FieldLabel, FieldError } from '@/components/ui/field';
  import {
    Select,
    SelectContent,
    SelectTrigger,
    SelectValue,
    SelectItem,
  } from '@/components/ui/select';

  // 👇 1
  interface SelectListItem {
    value: number | string;
    label: string;
  }

  // 👇 2
  interface LabelSelectList extends Omit<
    ComponentPropsWithoutRef<typeof Select>, // 👈 3
    'value' | 'items' | 'onValueChange' | 'onOpenChange' // 👈 4
  > {
    label: string; // 👈 5
    name: string; // 👈 5
    items: SelectListItem[]; // 👈 5
    placeholder?: string; // 👈 5
  }

  const LabelSelectList = ({
    label,
    name,
    items,
    placeholder,
    ...rest
  }: SelectListProps) => {
    const { control } = useFormContext();
    return (
      <Controller
        control={control}
        name={name}
        render={({ field, fieldState }) => (
          <Field data-invalid={fieldState.invalid}>
            <FieldLabel htmlFor={field.name}>{label}</FieldLabel>
            <Select
              value={field.value || null}
              items={items}
              onValueChange={field.onChange}
              onOpenChange={() => field.onBlur()}
              {...rest}
            >
              <SelectTrigger id={field.name}>
                <SelectValue placeholder={placeholder} />
              </SelectTrigger>
              <SelectContent>
                {items.map((item) => (
                  <SelectItem key={item.value} value={item.value}>
                    {item.label}
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
            {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
          </Field>
        )}
      />
    );
  };

  export default LabelSelectList;
  ```

  1. `SelectListItem` beschrijft de vorm van één optie in de keuzelijst: een `value` (het getal of de string die verstuurd wordt bij submit) en een `label` (de tekst die de gebruiker ziet).
  2. `LabelSelectListProps` beschrijft alle props die `LabelSelectList` accepteert. Door te extenden van een bestaand type hergebruiken we alle beschikbare props van de shadcn `Select` component zonder ze manueel te moeten oplijsten.
  3. `ComponentPropsWithoutRef<typeof Select>` geeft het volledige type van alle props van de shadcn `Select` component terug — inclusief `disabled`, `required`, `className`, enzovoort. Zo kan de gebruiker van `LabelSelectList` al deze attributen doorgeven zonder dat wij ze expliciet moeten definiëren.
  4. `Omit<..., 'value' | 'items' | 'onValueChange' | 'onOpenChange'>` verwijdert vier specifieke props uit het overgeërfde type, omdat wij ze zelf beheren via de `Controller`:
     - `value` en `onValueChange` worden door `Controller` doorgegeven via `field.value` en `field.onChange`.
     - `onOpenChange` gebruiken we om `field.onBlur()` aan te roepen zodat validatie getriggerd wordt bij sluiten van de dropdown.
     - `items` definiëren we zelf als `SelectListItem[]` (punt 5) in plaats van het generiekere type van de shadcn `Select`.
       Door deze props te verwijderen met `Omit` vermijd je TypeScript-conflicten: als je ze in de interface zou laten staan én ze zelf invult bij de `<Select>`, zou TypeScript klagen over dubbele definitie.
  5. De eigen props die `LabelSelectList` verwacht: `label` voor het label boven het veld, `name` voor de koppeling met React Hook Form, `items` als de lijst opties, en `placeholder` (optioneel) als de begintekst in de dropdown.

### Oefening 3 - LabelDatePicker

Maak een `LabelDatePicker` component aan.

- Oplossing +

  ```tsx
  // src/components/LabelDatePicker.tsx
  import type { ComponentProps } from 'react';
  import { Controller, useFormContext } from 'react-hook-form';
  import { Field, FieldLabel, FieldError } from '@/components/ui/field';
  import { Button } from '@/components/ui/button';
  import {
    Popover,
    PopoverContent,
    PopoverTrigger,
  } from '@/components/ui/popover';
  import { Calendar } from '@/components/ui/calendar';
  import { ChevronDownIcon } from 'lucide-react';
  import { LocalizedDate } from './LocalizedDate';

  interface LabelDatePickerFieldProps extends Omit<
    ComponentProps<typeof Calendar>,
    'mode' | 'selected' | 'onSelect'
  > {
    label: string;
    name: string;
    placeholder?: string;
  }

  const LabelDatePicker = ({
    label,
    name,
    placeholder = 'Pick a date',
    ...rest
  }: LabelDatePickerFieldProps) => {
    const { control } = useFormContext();
    return (
      <Controller
        control={control}
        name={name}
        render={({ field, fieldState }) => (
          <Field data-invalid={fieldState.invalid}>
            <FieldLabel id={field.name}>{label}</FieldLabel>
            <Popover>
              <PopoverTrigger
                render={
                  <Button
                    variant='outline'
                    data-empty={!field.value}
                    className='justify-start text-left font-normal data-[empty=true]:text-muted-foreground'
                  />
                }
                className='flex w-full justify-between'
              >
                {field.value ? (
                  <LocalizedDate date={field.value} />
                ) : (
                  <span>{placeholder}</span>
                )}
                <ChevronDownIcon className='size-4' />
              </PopoverTrigger>
              <PopoverContent className='w-auto p-0'>
                <Calendar
                  mode='single'
                  selected={field.value}
                  onSelect={field.onChange}
                  weekStartsOn={1}
                  {...rest}
                />
              </PopoverContent>
            </Popover>
            {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
          </Field>
        )}
      />
    );
  };

  export default LabelDatePicker;
  ```

### Uitschakelen inputvelden bij submit

Je kan er ook voor zorgen dat de inputvelden en knoppen in het formulier _disabled_ worden als het formulier gesubmit wordt.
`useForm` geeft een boolean [isSubmitting](https://react-hook-form.com/api/useform/formstate) terug die `true` is als het formulier gesubmit wordt en `false` bij een reset.

```tsx
export default function TransactionForm({...}){
// ...
const { isSubmitting, isValid } = form.formState; // 👆

// ...
return (
  <>
    <FormProvider {...form}>
      {/* ... */}
      <div className='flex justify-end'>
            <button
            type='submit'
            disabled={isSubmitting}
            className='bg-blue-500 text-white font-medium py-2 px-4 rounded'
          >
            {transaction?.id ? 'Save transaction' : 'Add transaction'}
          </button>
          {/* 👇 */}
       <Link to="/transactions" className={cn(buttonVariants({ variant: 'outline' }))}>
            Cancel
          </Link>
        </div>
      </div>
      {/* ... */}
    </FormProvider>
  </>
);
}
```

Disable het inputveld tijdens submit, doe hetzelfde voor de `LabelSelectList` component.

```tsx
import { useFormContext } from 'react-hook-form';

export default function LabelInput({...}) {
  const {
    control,
    formState: { isSubmitting },
  } = useFormContext(); // 👆

  const hasError = name in errors;

  return (
    //...
      <input ...
        disabled={isSubmitting}

      /> {/* 👆 */}
    //...
  );
}
```

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les6-opl 5d642b4
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.

### Oefening 4 - Je eigen project

Controleer je eigen project op anti-patterns, duplicate code en refactor.
Denk na over global state in je project. Indien van toepassing, maak hiervoor een Context aan.
