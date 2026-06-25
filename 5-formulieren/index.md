# Formulieren & hooks

> **Startpunt voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les5 6d60fc7
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.
>
> In dit hoofdstuk heb je de bijbehorende backend nodig:
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/webservices-budget.git
> cd webservices-budget
> git checkout -b les5 13add05
> pnpm install
> pnpm db:migrate
> pnpm db:seed
> pnpm start:dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/webservices-budget?tab=readme-ov-file#webservices-budget) voor meer informatie.

In dit hoofdstuk maken we een component aan voor het toevoegen en wijzigen van een transactie. We bekijken ook hoe we de performantie verder kunnen verbeteren.

## Routing

We maken een component voor het toevoegen en wijzigen van een transactie en voorzien de nodige routes.

### Oefening 1 - Routing

Maak een bestand `AddOrEditTransaction.tsx` aan in de map `src/pages/transactions`. Voeg hieraan een placeholder toe zodat je weet dat de pagina correct wordt weergegeven.

Voorzie volgende bijkomende routes in de budget-applicatie:

- `/transactions/add`: een nieuwe transactie toevoegen, via de `AddOrEditTransaction` component
- `/transactions/edit/:id`: een transactie bewerken, via de `AddOrEditTransaction` component

<br />

- Oplossing +

  De `AddOrEditTransaction` component:

  ```tsx
  // src/pages/AddOrEditTransaction.tsx
  export default function AddOrEditTransaction() {
    return <h1>Add transaction</h1>;
  }
  ```

  De routes worden toegevoegd in `main.tsx`:

  ```tsx
  // src/main.tsx
  import AddOrEditTransaction from './pages/transactions/AddOrEditTransaction.tsx';
  //...
  {
    path: '/transactions',
    children: [
      {
        index: true,
        element: <TransactionList /> ,
      },
      {
        path: 'add',
        element: <AddOrEditTransaction />,
      },
      {
        path: 'edit/:id',
        element: <AddOrEditTransaction />,
      },
    ],
  }
  ```

### Oefening 2 - Toevoegen van knoppen

Voorzie een knop "Add Transaction" naast de zoekbalk in `TransactionList.tsx` en een potloodknop in de lijst voor elke transactie (`Transaction.tsx`). Zorg dat de verwijderknop en edit knop niet getoond wordt als we de detail van een plaats bekijken.

- Oplossing +

  In `TransactionList.tsx` voeg je onderstaande code toe:

  ```tsx
  // src/pages/transactions/TransactionList.tsx
  import { Link } from 'react-router'; // 👈
  import { Button, buttonVariants } from '@/components/ui/button'; // 👈
  //...

  <div className='flex justify-between mb-4 gap-2'>
    <div className='flex gap-2 w-1/2'>
      <Input
        type='search'
        placeholder='Search by place…'
        value={text}
        onChange={handleSearchChange}
        onKeyDown={handleKeyDown}
      />
      <Button variant='outline' onClick={() => setSearch(text)}>
        Search
      </Button>
    </div>

    <Link to='/transactions/add' className={cn(buttonVariants())}>
      Add transaction
    </Link>
    {/* 👈 */}
  </div>;
  //..
  ```

`cn(buttonVariants())` zorgt ervoor dat de <Link> er visueel uitziet als een knop.

- `buttonVariants()`: genereert de Tailwind CSS-klassen voor de standaard knopstijl (achtergrondkleur, padding, border-radius…)
- `cn(...)`: combineert die klassen samen (en filtert eventuele conflicten weg)
- `className={...}`: past die klassen toe op de `<Link>`

In `Transaction.tsx` voeg je onderstaande code toe:

```tsx
// src/components/transactions/Transaction.tsx
import { Button, buttonVariants } from '../ui/button'; // 👈
import { Pencil, Trash2 } from 'lucide-react'; // 👈
import { Link } from 'react-router'; // 👈
import { cn } from '@/lib/utils'; // 👈
//...
<TableCell className='text-right'>
  <div className='flex justify-end gap-1'>
    <Link
      to={`/transactions/edit/${id}`}
      className={cn(buttonVariants({ variant: 'ghost', size: 'icon' }))}
    >
      <Pencil className='h-4 w-4' />
    </Link>
    {/* 👈 */}
    {onDelete && (
      <Button
        variant='ghost'
        size='icon'
        onClick={handleDelete}
        aria-label='Delete transaction'
        className='text-destructive hover:text-destructive'
      >
        <Trash2 className='h-4 w-4' />
      </Button>
    )}
  </div>
</TableCell>;
//...
```

## Het formulier

Maak een bestand `TransactionForm.tsx` aan in de map `src/components/transactions`. Dit bevat een formulier met 2 input velden (userId, amount), een datepicker en één select lijst (placeId). Het userId zal later geschrapt worden en vervangen worden door het id van de aangemelde gebruiker. Deze component krijgt de `places` door als prop voor het vullen van de select lijst. We beginnen met een leeg formulier

```tsx
// src/components/transactions/TransactionForm.tsx
import type { Place } from '../../types';

interface TransactionFormProps {
  places?: Place[];
}

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  return <form>Hier komt het formulier</form>;
}
```

In HTML houden formulierelementen zoals `input`, `textarea` en `select` doorgaans hun eigen state bij. Ze werken deze bij op basis van gebruikersinvoer. Formulierelementen in React zijn read-only. Door state toe te voegen, kan de component zich aanpassen. In het vorige hoofdstuk hebben we een eenvoudig voorbeeld van een formulier behandeld maar validatie, foutafhandeling, formArrays... ontbreken nog. Je kan dit allemaal zelf implementeren of je kan gebruik maken van een package, zoals bv. [react-hook-form](https://react-hook-form.com/).

?> Voor simpele formulieren zoals bv. een zoekbalk kan je gebruik maken van controlled components. Voor complexere formulieren is het aangeraden om een library te gebruiken zoals `react-hook-form`.

### Oefening 3 - AddOrEditTransaction

De pagina `AddOrEditTransaction` gebruikt de component `TransactionForm` die het formulier zal bevatten om een transactie te creëren en te wijzigen. We dienen alvast de places op te halen aangezien de gebruiker de plaats waar de transactie plaatsvindt zal moeten selecteren. Maak de component aan.

- Oplossing +

  ```tsx
  // src/pages/transactions/AddOrEditTransaction.tsx
  import useSWR from 'swr'; // 👈 1
  import { getAll } from '../../api'; // 👈 1
  import TransactionForm from '../../components/transactions/TransactionForm'; // 👈 2
  import AsyncData from '../../components/AsyncData'; // 👈 3
  import type { Place } from '../../types'; // 👈 1

  export default function AddOrEditTransaction() {
    const {
      data: places = [],
      error: placesError,
      isLoading: placesLoading,
    } = useSWR<Place[]>('places', getAll); // 👈 1

    return (
      <>
        <h1 className='text-2xl font-semibold mb-6'>Add transaction</h1>
        {/* 👇 3 */}
        <AsyncData error={placesError} loading={placesLoading}>
          {/* 👇 2 */}
          <TransactionForm places={places} />
        </AsyncData>
      </>
    );
  }
  ```

1. We maken gebruik van swr om alle plaatsen op te halen.
2. De `TransactionForm` component bevat het formulier voor de ingave van een transactie. We geven de plaatsen door.
3. Zorg voor foutafhandeling en loading indicator.

### React-hook-form

We maken gebruik van de React-hook-form voor het formulierbeheer in React. Dit bevat

- `useForm` hook voor het beheren van de formulierstatus.
- `<Controller />` component om controlled components te integreren in je formulier.
- Client-side validatie met `Zod` en bijhorende `zodResolver`.

Voeg deze packages toe aan het project:

```bash
pnpm add react-hook-form zod @hookform/resolvers
```

Neem eerst de [documentatie over React-hook-form en shadcn](https://ui.shadcn.com/docs/forms/react-hook-form) door.

We beginnen met een eenvoudig formulier met 2 inputvelden `userId` en het `bedrag`. Later voegen we ook een Select lijst en datepicker toe.

### Stap 1. Maak een formulierschema aan.

We beginnen met het definiëren van de vorm van ons formulier met behulp van een `Zod`-schema. Het bepaalt:

- Welke velden je formulier heeft
- Welke types ze moeten hebben
- Welke validatieregels gelden (zie verder)

```tsx
// src/components/transactions/TransactionForm.tsx
import type { Place } from '../../types';
import * as z from 'zod'; // 👈 1

interface TransactionFormProps {
  places?: Place[];
}

const formSchema = z.object({
  userId: z.number(),
  amount: z.number(),
}); // 👈 2

type TransactionFormValues = z.infer<typeof formSchema>; // 👈 3

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  return <form>Hier komt het formulier</form>;
}
```

1. Importeer zod
2. Zod laat je een schema definiëren dat beschrijft hoe de data er moet uitzien adhv `z.object({})`. Het zegt uit welke velden je formulier bestaat en welk type ze hebben. Later breiden we dit verder uit met validatieregels
3. Genereer automatisch een TypeScript type op basis van je schema. Het schema en het type blijven zo automatisch gesynchroniseerd.

### Stap 2. Stel het formulier in

We maken gebruik van de [useForm](https://react-hook-form.com/docs/useform) hook uit het `react-hook-form` package. De hook beheert de volledige toestand van je formulier: de ingevulde waarden, of er fouten zijn, of het formulier al ingediend is, enzovoort. Zonder deze hook moet je dat allemaal zelf bijhouden met `useState`.

```tsx
// src/components/transactions/TransactionForm.tsx
import type { Place } from '../../types';
import * as z from 'zod';
import { useForm } from 'react-hook-form'; // 👈 1
import { zodResolver } from '@hookform/resolvers/zod'; // 👈 2

interface TransactionFormProps {
  places?: Place[];
}

const formSchema = z.object({
  userId: z.number(),
  amount: z.number(),
});

type TransactionFormValues = z.infer<typeof formSchema>;

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  const form = useForm<TransactionFormValues>({
    resolver: zodResolver(formSchema), // 👈 2
  }); // 👈 1

  const onSubmit = (values: TransactionFormValues) => {
    console.log(JSON.stringify(values));
  }; // 👈 4

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>Hier komt het formulier</form>
  ); // 👈 5
}
```

1. importeer `useForm`.
2. importeer `zodResolver`: Dit is een "brug" tussen `react-hook-form` en `zod`. `react-hook-form` weet op zich niet hoe hij met een zod-schema moet werken. `zodResolver` vertaalt het zod-schema zodat `react-hook-form` het kan gebruiken voor validatie.
3. Roep `useForm` aan. Configureer de hook met de optie `resolver: zodResolver(formSchema)`, zodat de validatie via het zod-schema moet verlopen. Het resultaat (`form`) bevat alle methodes, properties nodig om het formulier te besturen: veldregistratie, foutmeldingen, het indienen, enzovoort.
4. `onSubmit` functie wordt uitgevoerd als het formulier succesvol gevalideerd en ingediend wordt. Ze krijgt automatisch de ingevulde waarden (`values`) binnen als parameter van het type `TransactionFormValues`. Log de `values` tijdelijk naar de console; later stuur je ze naar een API.

5.`form.handleSubmit` is een wrapper van `react-hook-form`. Als de gebruiker het formulier indient, doet hij eerst de validatie. Pas als alles geldig is, roept hij `onSubmit` aan. Als er fouten zijn, worden die automatisch opgeslagen in `form.formState.errors` en wordt `onSubmit` niet opgeroepen.

### Stap 3. Maak het formulier

Voor de bouw van het formulier maken we gebruik van

- [field](https://ui.shadcn.com/docs/components/base/field) component voor het combineren van de labels en de input velden
- [input](https://ui.shadcn.com/docs/components/radix/input) component voor de ingave van tekst

```bash
pnpm dlx shadcn@latest add field input
```

Voeg inputvelden toe voor User ID en amount

```tsx
// src/components/transactions/TransactionForm.tsx
import type { Place } from '../../types';
import * as z from 'zod';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Field, FieldLabel, FieldGroup } from '@/components/ui/field'; // 👈
import { Input } from '@/components/ui/input'; // 👈
import { Button } from '@/components/ui/button'; // 👈

interface TransactionFormProps {
  places?: Place[];
}

const formSchema = z.object({
  userId: z.number(),
  amount: z.number(),
});

type TransactionFormValues = z.infer<typeof formSchema>;

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  const form = useForm<TransactionFormValues>({
    resolver: zodResolver(formSchema),
  });

  const onSubmit = (values: TransactionFormValues) => {
    console.log(JSON.stringify(values));
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <FieldGroup>
        <Field>
          <FieldLabel htmlFor='userId'>User ID</FieldLabel>
          <Input id='userId' type='number' />
        </Field>
        <Field>
          <FieldLabel htmlFor='amount'>Bedrag</FieldLabel>
          <Input id='amount' type='number' placeholder='0.00' />
        </Field>
      </FieldGroup>
      <div className='flex justify-end gap-2 pt-6'>
        <Button type='submit'>Add transaction</Button>
      </div>
    </form>
  ); // 👈
}
```

De input velden zijn nog niet gekoppeld aan ract-hook-form. Dit doen we in de volgende stap.

### Stap 4. Controller

React-hook-form houdt de state bij voor elke control. [Controller](https://react-hook-form.com/docs/useform/register) is een wrapper van react-hook-form die een UI-library component integreert met het formulier. De `Controller` toevoegen doe je als volgt:

```tsx
<Controller
  control={form.control}
  name='theName' //naam overeenkomstig formObject
  render={({ field }) => (
    <Input
      {...field}
      type='number'
      onChange={(e) => field.onChange(e.target.valueAsNumber)} // overschrijft onChange voor type conversie
    />
  )}
/>
```

- `form.control` is het "brein" van het formulier, het bevat alle waarden en validatieregels.
- `name` is de naam van het overeenkomstig form object
- Het `render` prop is een functie die je component rendert en een `field` object doorgeeft aan je component. Dit bevat volgende props :
  - `field.value`: de huidige waarde uit de form state
  - `field.onChange`: update de form state bij een wijziging
  - `field.name`: de naam van het veld
  - `field.ref`: verbindt het element met react-hook-form voor focus bij errors
  - `field.onBlur`: geeft door dat het veld 'touched' is. Zo kan de validatie getriggerd worden afhankelijk van de modus
- `{...field}` spreidt alle field props uit op de input. Dus o.a.

  ```tsx
  <Input
    value={field.value} // waarde komt uit React-hook-form's state
    onChange={field.onChange} // elke toetsaanslag updatet React-hook-form's state
  />
  ```

  `value/onChange` zorgen ervoor dat de state in het formulier gesynchroniseerd wordt.

- `onChange={(e) => field.onChange(e.target.valueAsNumber)` overschrijft de standaard `onChange`. Normaal geeft een input een string terug, maar .`valueAsNumber` converteert het meteen naar een `number` — dat is nodig omdat het Zod schema `z.number()` verwacht en anders de conversie zou falen.

Het formulier wordt na toevoeging van de `Controller`:

```tsx
// src/components/transactions/TransactionForm.tsx
import type { Place } from '../../types';
import * as z from 'zod';
import { Controller, useForm } from 'react-hook-form'; // 👈 1
import { zodResolver } from '@hookform/resolvers/zod';
import { Field, FieldLabel, FieldGroup } from '@/components/ui/field';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';

interface TransactionFormProps {
  places?: Place[];
}

const formSchema = z.object({
  userId: z.number(),
  amount: z.number(),
});

type TransactionFormValues = z.infer<typeof formSchema>;

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  const form = useForm<TransactionFormValues>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      userId: 0,
      amount: 0,
    }, // 👈 2
  });

  const onSubmit = (values: TransactionFormValues) => {
    console.log(JSON.stringify(values));
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <FieldGroup>
        {/* 👇 3 */}
        <Controller
          control={form.control}
          name='userId'
          render={({ field }) => (
            <Field>
              <FieldLabel htmlFor={field.name}>User Id</FieldLabel>
              <Input
                {...field}
                id={field.name}
                type='number'
                onChange={(e) => field.onChange(e.target.valueAsNumber)}
              />
            </Field>
          )}
        />
        {/* 👇 3 */}
        <Controller
          control={form.control}
          name='amount'
          render={({ field }) => (
            <Field>
              <FieldLabel htmlFor={field.name}>Amount</FieldLabel>
              <Input
                {...field}
                id={field.name}
                type='number'
                placeholder='0.00'
                onChange={(e) => field.onChange(e.target.valueAsNumber)}
              />
            </Field>
          )}
        />
      </FieldGroup>
      <div className='flex justify-end gap-2 pt-6'>
        <Button type='submit'>Add transaction</Button>
      </div>
    </form>
  );
}
```

1. importeer de `Controller`
2. `defaultValues`: de beginwaarden van de formuliervelden bij het laden van de pagina. Deze zijn 0 en niet leeg omdat het schema `z.number()` verwacht. Dit moet je opgeven als je met de Controller werkt , anders krijg je een fout in de Console: "installHook.js:1 Base UI: A component is changing the uncontrolled value state of Input to be controlled. Elements should not switch from uncontrolled to controlled (or vice versa). Decide between using a controlled or uncontrolled Input element for the lifetime of the component. The nature of the state is determined during the first render. It's considered controlled if the value is not `undefined`."
3. Voeg de `Controller` toe.

### Stap 5. Validatie

In een applicatie kan je niet alleen werken met server-side validatie. In dat geval moet nl. de data eerst verzonden worden alvorens de validatie kan gebeuren. Je kan ook niet alleen vertrouwen op client-side validatie. Deze is eenvoudig uit te schakelen waardoor toch verkeerde data naar de server (en in de databank) kan komen.

React-hook-form ondersteunt ook schema-validatie met Yup, Zod, Superstruct & Joi. De validatie is afgestemd op de HTML-standaard voor formuliervalidatie. We maken gebruik van [zod](https://zod.dev/).

Neem eerst de [documentatie](https://zod.dev/basics) door

```tsx
// src/components/transactions/TransactionForm.tsx
import type { Place } from '../../types';
import * as z from 'zod';
import { Controller, useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import {
  Field,
  FieldLabel,
  FieldGroup,
  FieldError,
} from '@/components/ui/field';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';

interface TransactionFormProps {
  places?: Place[];
}

const formSchema = z.object({
  userId: z
    .number({ error: 'User Id is required and must be a number' })
    .min(1, 'User Id must be minimum 1'),
  amount: z
    .number({ error: 'Amount is required and must be a number' })
    .refine((value) => !isNaN(value), {
      message: 'Amount is required and must be a number',
    })
    .refine((value) => value !== 0, { message: '0 is not a valid amount' }),
}); // 👈 1

type TransactionFormValues = z.infer<typeof formSchema>;

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  const form = useForm<TransactionFormValues>({
    mode: 'onBlur', // 👈 2
    resolver: zodResolver(formSchema),
    defaultValues: {
      userId: 0,
      amount: 0,
    },
  });

  const onSubmit = (values: TransactionFormValues) => {
    if (!form.formState.isValid) return; // 👈 3
    console.log(JSON.stringify(values));
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <FieldGroup>
        <Controller
          control={form.control}
          name='userId'
          render={(
            { field, fieldState }, //👈 4
          ) => (
            <Field data-invalid={fieldState.invalid}>
              {/* 👈 5 */}
              <FieldLabel htmlFor={field.name}>User Id</FieldLabel>
              <Input
                {...field}
                id={field.name}
                type='number'
                onChange={(e) => field.onChange(e.target.valueAsNumber)}
              />
              {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
              {/* 👈 6 */}
            </Field>
          )}
        />

        <Controller
          control={form.control}
          name='amount'
          render={(
            { field, fieldState }, //👈 4
          ) => (
            <Field data-invalid={fieldState.invalid}>
              {/* 👈 5 */}
              <FieldLabel htmlFor={field.name}>Amount</FieldLabel>
              <Input
                {...field}
                id={field.name}
                type='number'
                placeholder='0.00'
                onChange={(e) => field.onChange(e.target.valueAsNumber)}
              />
              {fieldState.invalid && <FieldError errors={[fieldState.error]} />}{' '}
              {/* 👈 6 */}
            </Field>
          )}
        />
      </FieldGroup>
      <div className='flex justify-end gap-2 pt-6'>
        <Button type='submit'>Add transaction</Button>
      </div>
    </form>
  );
}
```

1. Voeg de validatie toe aan het schema.
   - userId — moet een getal zijn (minimaal 1), verplicht.
   - amount — het bedrag, met twee extra checks via `.refine()`: Mag geen NaN zijn (niet-getal) — dit is een extra veiligheidscheck bovenop z.number() en mag niet 0 zijn
     De `{ error: '...' }` en`{ message: '...' }` opties bepalen de foutmeldingen die de gebruiker te zien krijgt als de validatie faalt.
2. React-hook-form ondersteunt verschillende `validatiemodi`.
   - `onChange`: Bij elke wijziging wordt een validatie geactiveerd.
   - `onBlur`: Validatie wordt geactiveerd bij het verlaten van het scherm.
   - `onSubmit`: Validatie wordt geactiveerd bij het verzenden (standaard).
   - `onTouched`: De validatie wordt geactiveerd bij de eerste keer dat het scherm de focus verliest, en vervolgens bij elke wijziging.
   - `all`: Validatie wordt geactiveerd bij het verlaten van het scherm en bij wijzigingen.
3. We controlleren of er fouten voorkomen in het formulier
4. De Controller geeft nu ook de `fieldState` door. Di de validatiestatus van 1 specifiek veld.
   - `fieldState.invalid`: true als het veld niet voldoet aan de Zod-validatieregels
   - `fieldState.error`: het foutobject met de foutmelding (bv. { message: 'User is required' })
   - `fieldState.isDirty`: true als de gebruiker de waarde heeft gewijzigd t.o.v. de defaultValue
   - `fieldState.isTouched`: true als de gebruiker het veld heeft gefocust en er terug uit is gegaan

5. Voeg de `data-invalid` prop toe aan het `<Field />`component voor de styling. De CSS in `field.tsx` reageert hierop met `group-data-[invalid=true]:text-destructive` — het label en de rand worden rood.
6. Geef de foutmeldingen weer onder het veld met behulp van `<FieldError />`.

### Stap 6. Select voor plaatsen

Voeg een extra formulierveld toe voor de keuze van een plaats. Maak hiervoor gebruik van de Select component. [Lees de documentatie voor integratie met react-form-hooks](https://ui.shadcn.com/docs/forms/react-hook-form#select).

```tsx
//...
// src/components/transactions/TransactionForm.tsx
import {
  Select,
  SelectContent,
  SelectTrigger,
  SelectValue,
  SelectItem,
} from '@/components/ui/select';

//...
const formSchema = z.object({
//...
  placeId: z.number({ error: 'Place is required' }).min(1, 'Place is required'),// 👈 1
});

export default function TransactionForm({ places = [] }: TransactionFormProps) {
  const form = useForm<TransactionFormValues>({
    mode: 'onBlur',
    resolver: zodResolver(formSchema),
    defaultValues: {
      userId: 0,
      amount: 0,
      placeId: 0, // 👈 2
    },
  });

  //...
  const placesSelectItems = places.map((place) => ({
    value: place.id,
    label: place.name,
  }));// 👈 4

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      //...
        {/* 👇 3 */}
        <Controller
          control={form.control}
          name="placeId"
          render={({ field, fieldState }) => (
            <Field data-invalid={fieldState.invalid}>
              <FieldLabel htmlFor={field.name}>Place</FieldLabel>
              <Select
                value={field.value || null}
                items={placesSelectItems}
                onValueChange={field.onChange}
                onOpenChange={() => field.onBlur()}
              > {/* 👈 5 */}
                <SelectTrigger id={field.name} className="w-45">
                  <SelectValue placeholder="Place" />
                </SelectTrigger>
                <SelectContent>
                  {placesSelectItems.map((item) => (
                    <SelectItem key={item.value} value={item.value}>
                      {item.label}
                    </SelectItem>
                  ))}{/* 👈 4 */}
                </SelectContent>
              </Select>
              {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
            </Field>
          )}
        />
     //...
  );
}
```

1. Voeg een `placeId` veld toe aan het `formSchema` (verplicht getal, minimaal 1).
2. Voeg `placeId` toe aan de `defaultValues` in `useForm`.
3. Voeg een `<Controller>` toe voor `placeId` met een [Select component](https://ui.shadcn.com/docs/components/radix/select). De Select component verwacht een lijst van het type `{value: number;label: string;}[]`, zie `placesSelectItems`. `SelectTrigger` is de zichtbare knop. `SelectContent` is het uitklapmenu en bevat de items.
4. De places prop bevat de beschikbare plaatsen. Zet ze om naar het formaat { value: number; label: string }[] dat de Select verwacht.

5.`<select>`:

- `value` toont de waarde bijgehouden in react-hook-form. `field.value || null`: Base UI toont de placeholder "Place" als de value null is. (Indien 0 toont Base UI de waarde 0).
- `onValueChange` stuurt de gekozen waarde terug naar react-hook-form (geen native event, maar een directe waarde — vandaar `onValueChange` i.p.v. `onChange`)
- `onBlur` vertelt React-hook-form dat de gebruiker klaar is met dit veld, zodat validatie getriggerd wordt

### Stap 7. Een datepicker voor de datum

Maak gebruik van de [DatePicker component](https://ui.shadcn.com/docs/components/base/date-picker) voor de datum, die op zijn beurt gebruik maakt van de [Calendar](https://ui.shadcn.com/docs/components/base/calendar) en [PopOver](https://ui.shadcn.com/docs/components/base/popover) component. `PopOver` toont/verbergt de calender. `PopoverTrigger` is de knop die je aanklikt, het toont de gekozen datum. `PopoverContent` klapt uit en toont de kalender.

```bash
pnpm dlx shadcn@latest add field popover calendar
```

En voeg toe aan het formulier.

```tsx
// src/components/transactions/TransactionForm.tsx
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover';
import { Calendar } from '@/components/ui/calendar';
import { ChevronDownIcon } from 'lucide-react';
import { LocalizedDate } from '../LocalizedDate';
//...
// In het zod schema
  date: z
    .date()
    .max(
      new Date().setDate(new Date().getDate() + 1),
      'Date cannot be in the future',
    ),
//...
export default function TransactionForm({ places = [] }: TransactionFormProps) {
  const form = useForm<TransactionFormValues>({
    mode: 'onBlur',
    resolver: zodResolver(formSchema),
    defaultValues: {
      userId: 0,
      amount: 0,
      placeId: 0,
      date: new Date(),// 👈
    },
  });
  //...
    <Controller
      control={form.control}
      name='date'
      render={({ field, fieldState }) => (
        <Field data-invalid={fieldState.invalid}>
          <FieldLabel id={field.name}>Date</FieldLabel>
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
                <span>Pick a date</span>
              )}
              <ChevronDownIcon className='size-4' />
            </PopoverTrigger>
            <PopoverContent className='w-auto p-0'>
              <Calendar
                mode='single'
                selected={field.value}
                onSelect={field.onChange}
                weekStartsOn={1}
                disabled={{ after: new Date() }}
              />
            </PopoverContent>
          </Popover>
          {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
        </Field>
      )}
    />
  ));
```

1. Voeg een `date` veld toe aan het `formSchema` (verplicht, moet een Date-object zijn, en mag niet in de toekomst liggen )
2. Voeg `date` toe aan de `defaultValues` in `useForm` en stel in op de datum van vandaag.
3. Voeg een `<Controller>` toe voor `amount` met een [DatePicker component](https://ui.shadcn.com/docs/components/base/date-picker).
4. Maak gebruik van `LocalizedDate`voor weergave van de datum in de `Button`.
5. `selected/onSelected` prop voor de synchronisatie met de state in react-hook-form

### Stap 8: Reset

[reset](https://react-hook-form.com/docs/useform/reset) zet alle velden terug op de standaardwaarde (indien opgegeven) of maakt ze leeg.

```tsx
// src/components/transactions/TransactionForm.tsx
const onSubmit = (values: TransactionFormValues) => {
  if (!form.formState.isValid) return;
  console.log(JSON.stringify(values));
  form.reset();
};
```

Vermits er meerdere invoervelden op ons formulier voorkomen en we steeds dezelfde code moeten schrijven, zouden we een aparte component moeten maken. Deze component zal gebruik moeten maken van [useFormContext](https://react-hook-form.com/docs/useformcontext). Dit komt in het volgende hoofdstuk aan bod.
In React-hook-form zijn dit twee verschillende concepten:

## POST /api/transactions

De volgende stap van de CRUD operaties is de 'C', een nieuwe transactie aanmaken. Pas `index.js` in de map `api` aan:

```tsx
// src/api/index.js
// 👇 1
export const save = async <T>(
  url: string,
  { arg }: { arg: T },
): Promise<void> => {
    await axios.post(`${baseUrl}/${url}`, arg);
};
```

1. De parameter `url` zal van `swr` de `key` ontvangen. We krijgen ook de `transaction` mee als argument `arg` .
2. We voeren een `POST` request uit naar de API. Axios zal de `transaction` automatisch omzetten naar JSON en versturen als body van het HTTP request. Het antwoord heeft als HTTP status code 201 en als response body de nieuw gecreëerde transactie. We negeren dat antwoord hier.

We maken een nieuwe mutation in `TransactionForm`:

```tsx
// src/components/transactions/TransactionForm.tsx
// ... (imports)
import useSWRMutation from 'swr/mutation'; // 👈 1
import { getAll, save } from '../../api'; // 👈 1
import Error from '../Error'; // 👈 2

export default function TransactionForm({places = []) {

const { trigger: saveTransaction, error: saveError } = useSWRMutation('transactions', api.save);// 👈 2

const onSubmit = async (values: TransactionFormValues) => {
      if (!form.formState.isValid) return;
    // 👇 3
      await saveTransaction(
        {
          ...values,
        },
        {
          throwOnError: false,
          onSuccess: () => form.reset()
        },
      );
    };
// ...
   return (
    <>
      <Error error={saveError} />{/* 👈 2 */}

      <form onSubmit={form.handleSubmit(onSubmit)}>
    </>
}
```

1. Importeer de `useSWRMutation` hook en de `save` functie.
2. Maak een trigger-functie die een transactie zal opslaan. We gebruiken dezelfde key als bij het ophalen van de transacties, dus `transactions`. We geven als fetcher onze `save` functie mee. We krijgen o.a. terug:
   - `trigger`: een functie die we kunnen aanroepen om het request effectief uit te voeren en dus de transactie toe te voegen. Deze functie ontvangt de `transaction` als argument. We hernoemen dit naar `saveTransaction`.
   - `error`: een eventuele fout die zich voordoet bij het opslaan van de transactie. We hernoemen deze naar `saveError`.
3. Wijzig de `onSubmit` zodat `saveTransaction` aangeroepen wordt. We geven de values mee als argument . **Let op:** de functie is `async`, dus we moeten `await` gebruiken. Het optioneel 2de argument definieert dat `throw error` niet moet worden aangeroepen als de update faalt, en bij succes wordt het formulier gereset.

Later verwijderen we het userId-veld, daarom doen we hier geen moeite om de gebruiker op te zoeken. We geven gewoon het id mee. 4. We tonen een eventuele fout na het opslaan

### Oefening 5 - POST in je eigen project

Implementeer een formulier om een entiteit te creëren in je eigen project:

- Maak een functie die een POST request uitvoert aan in `api/index.js`.
- Gebruik de `useSWRMutation` hook om de data toe te voegen.
- Controleer of je formulier een item kan toevoegen.

## PUT /api/transactions/:id

Dan rest nog de 'U' van CRUD, maar die is een beetje speciaal. De API call zelf is geen probleem, dit is basically hetzelfde als het toevoegen maar met een extra `id` parameter.

Maar we willen natuurlijk niet dat een gebruiker een volledig object juist moet invoeren om het aan te passen. We willen dat hij ergens op 'bewerk' kan klikken bij een bestaand element.

In de API hebben we een functie nodig om de aan te passen transactie op te halen:

```tsx
// src/api/index.js

export async function getById<T>(url: string): Promise<T> {
  const { data } = await axios.get(`${baseUrl}/${url}`);
  return data;
}
```

In de API kan je een aparte functie aanmaken om iets te updaten:

```tsx
export const updateById = async (
  url: string,
  { arg: { id, ...data } }: { arg: { id?: number } & Record<string, unknown> },
) => {
  await axios.put(`${baseUrl}/${url}/${id}`, data);
};
```

Of we kunnen de `save` functie aanpassen:

```tsx
// src/api/index.js
export async function save(
  url: string,
  { arg: { id, ...data } }: { arg: { id?: number } & Record<string, unknown> },
): Promise<void> {
  await axios({
    method: id ? 'PUT' : 'POST',
    url: `${url}/${id ?? ''}`,
    data,
  });
}
```

Wij kiezen voor de laatste (compacte) oplossing. Pas ook `PlacesList.tsx` aan zodat nu van de `save` methode gebruik gemaakt wordt.

Als we in de `Transaction` component klikken op de potlood-knop, navigeren we naar `/transactions/edit/${id}`. In `AddOrEditTransaction` kijken we of het om een add of edit gaat (id al dan niet gekend). In het laatste geval halen we de betreffende transactie op en geven dit door aan de `TransactionForm`, anders geven we `null` door. Indien swr `null` ontvangt, zal het geen request uitvoeren.

```tsx
// src/pages/transactions/AddOrEditTransaction.tsx
import { useParams } from 'react-router'; // 👈 1
import useSWR from 'swr';
import { getById, getAll } from '../api'; // 👈 3
import TransactionForm from '../../components/transactions/TransactionForm';
import AsyncData from '../../components/AsyncData';
import type { Transaction, Place } from '../../types';

export default function AddOrEditTransaction() {
  const { id } = useParams<{ id: string }>(); // 👈 2

  const {
    data: transaction,
    error: transactionError,
    isLoading: transactionLoading,
  } = useSWR<Transaction>(id ? `transactions/${id}` : null, getById); // 👈 3

  const {
    data: places = [],
    error: placesError,
    isLoading: placesLoading,
  } = useSWR<Place[]>('places', getAll);

  return (
    <>
      <h1 className='text-2xl font-semibold mb-6'>
        {id ? 'Edit transaction' : 'Add transaction'}
      </h1>
      {/* 👇 5 */}
      <AsyncData
        error={transactionError ?? placesError}
        loading={transactionLoading || placesLoading}
      >
        {/* 👇 4 */}
        <TransactionForm places={places} transaction={transaction} />
      </AsyncData>
    </>
  );
}
```

1. Importeer `useParams`.
2. Extraheer de id uit de url.
3. Maak gebruik van `useSWR` om de aan te passen transactie op te halen. Indien geen id parameter werd meegegeven wordt `null` meegegeven.
4. Geef de aan te passen transactie door aan `TransactionForm`.
5. Zorg voor foutafhandeling en laadindicator.

In het `TransactionForm` voorzien we de prop `transaction` met standaardwaarde `EMPTY_TRANSACTION`, zo werkt het aanmaken van een nieuwe transactie ook en passen we de code verder aan.

```tsx
// src/components/transactions/TransactionForm.tsx
// ... (imports)
import { useNavigate, Link } from 'react-router'; // 👈 3,5
import { Button, buttonVariants } from '@/components/ui/button';// 👈 2
import { cn } from '@/lib/utils';// 👈 2
//..
interface TransactionFormProps {
  places?: Place[];
  transaction?:Transaction// 👈 1
}

const EMPTY_TRANSACTION: Partial<Transaction> = {
  id: undefined,
  amount: 0,
  date: new Date().toISOString(),
  place: { id: 0, name: '', rating: 0 },
  user: { id: 0, name: ''},
};// 👈 1

export default function TransactionForm({
  places = [],
  transaction = EMPTY_TRANSACTION as Transaction,// 👈 2
}: TransactionFormProps) {


export default function TransactionForm({places = [], transaction = EMPTY_TRANSACTION}) {
  const navigate = useNavigate(); // 👈 3

  const { trigger: saveTransaction, error: saveError } = useSWRMutation('transactions', api.save);

  const form = useForm<TransactionFormValues>({
    mode: 'onBlur',
    resolver: zodResolver(formSchema),
    defaultValues: {
      date: transaction?.date ? new Date(transaction.date) : new Date(),
      placeId: transaction?.place.id,
      amount: transaction?.amount ?? 0,
      userId:transaction?.user.id
    },
    values: transaction
      ? {
          date: transaction?.date ? new Date(transaction.date) : new Date(),
          placeId: transaction.place.id,
          amount: transaction.amount,
          userId: transaction.user.id
        }
      : undefined,
  });// 👈 4

  const { isSubmitting, isValid } = form.formState;// 👈 5

  const onSubmit = async (values: TransactionFormValues) => {
      if (!isValid) return;

      await saveTransaction(
        {
          id: transaction?.id,
          ...values,
        },
        {
          throwOnError: false,
          onSuccess: () => {
            void navigate('/transactions');
          },
        },
      );
    } // 👈 5


  //..
  return (
    {/* ... */}
    {/* 👇 3  */}
        <div className="flex justify-end gap-2 pt-6">
          <Button type="submit" disabled={isSubmitting || !isValid}>
            {transaction?.id ? 'Save transaction' : 'Add transaction'}
          </Button>
          <Link to="/transactions" className={cn(buttonVariants({ variant: 'outline' }))}>
            Cancel
          </Link>
        </div>
    {/* ... */}
  );
}
```

1. `EMPTY_TRANSACTION`: Definieer een leeg transaction object. Plaats dit buiten de component omdat we geen nieuwe objecten willen aanmaken bij elke render! Di t object blijft steeds hetzelfde.
2. Ontvang `transaction` als prop. Stel de standaardwaarde in op `EMPTY_TRANSACTION`.
3. Pas de tekst op de knop aan i.f.v. of het om een update of een create gaat en voeg een `Cancel`knop toe
4. Pas `useForm` aan
   - `defaultValues`: Worden éénmalig ingesteld bij de eerste render van het formulier. Deze worden niet bijgewerkt als transaction van buiten verandert.
   - `values`: Worden gesynchroniseerd met de huidige waarde van de prop, als transaction van buiten verandert, wordt het formulier automatisch gereset met de nieuwe waarden. Bedoeld voor gecontroleerde formulieren waarbij de externe data kan wijzigen (bijv. na een fetch of navigatie naar een andere transactie).
5. Bij het opslaan van de transactie geven we ook de id mee. En we navigeren terug naar de `TransactionList` pagina. We hoeven het formulier niet meer te resetten.

Pas ook de titel `Add transaction` aan in de `AddOrEditTransaction` component.

## Verbeteren van de performantie

In een React applicatie worden componenten heel vaak gerenderd. De performantie kan je verbeteren door het voorkomen van onnodige renders en het verminderen van de tijd die een render in beslag neemt. Een oplossing voor dit probleem is **memoization**.

React biedt een paar vormen van memoization:

- `memo`: creatie van pure componenten (let op: dit is **geen** hook)
- `useMemo`: retourneert een memoized **waarde**
- `useCallback`: retourneert een memoized **functie**

We hebben reeds `useMemo` uitgelegd. Nu komen de andere vormen aan bod.

### Hooks

**Componenten** zijn functies die de UI renderen. Het renderen gebeurt wanneer de app voor het eerst geladen wordt en wanneer state waarden wijzigen. Renderen van code moet "[puur](https://react.dev/learn/keeping-components-pure)" zijn. Componenten mogen enkel 'hun' JSX retourneren en mogen objecten of variabelen die bestaan voor de rendering niet wijzigen (bv. geen nieuwe waarde toekennen aan props). Gegeven dezelfde input, dient het dezelfde output te retourneren. Net als een wiskundige formule zou het alleen het resultaat moeten berekenen, maar niets anders doen.

Een veel gemaakte denkfout is dat alle waarden (i.e. variabelen) in een component bewaard blijven, dat is **niet waar**. Een component is een functie, en functies hebben lokale variabelen. Eens de component zijn JSX geretourneerd heeft, zijn alle lokale variabelen weg. Je hebt nood aan de magie van React Hooks om waarden te bewaren tussen renders.

**Events** zijn functies binnen de component die worden uitgevoerd als reactie op een actie van een gebruiker. Een event handler kan state aanpassen, bv. een HTTP POST request uitvoeren om een transactie toe te voegen. Event handlers bevatten **side-effects** veroorzaakt door een interactie. React biedt daarnaast ook de mogelijkheid voor side-effects na bv. een state-wijziging. Hierover in een volgend hoofdstuk meer.

In een vorige hoofdstuk hebben we kennis gemaakt met de `useState`, `useReducer`, `useMemo` en `useEffect` hooks. React heeft nog heel wat meer hooks (zie <https://react.dev/reference/react>) en er zijn reeds heel wat nuttige custom hooks te vinden op internet (zie bv. <https://nikgraf.github.io/react-hooks/>).

Hooks hebben ervoor gezorgd dat je met function components hetzelfde kan bereiken als met class components. Toch kan het zijn alsof hooks vreemd aanvoelen, alsof React de bal mis geslagen heeft in vergelijking met andere frameworks als [Solid.js](https://www.solidjs.com/) (zie <https://jakelazaroff.com/words/were-react-hooks-a-mistake/>).

### Regels voor hooks

Hooks zijn niet meer dan JavaScript functies. Echter moet je twee regels volgen wanneer je er gebruik van maakt. Je kan hiervoor een linter plugin installeren, dit gebeurt automatisch bij het gebruik van `vite`.

- Gebruik hooks enkel op het top niveau. Gebruik hooks niet binnen een if, andere condities, loops of geneste functies.
  - Reden: React valt terug op de volgorde waarin hooks worden aangeroepen om een waarde terug te geven. React houdt dit bij in een array. De volgorde moet dezelfde zijn bij elke render. Benieuwd naar meer info? Lees verder in [The First Rule of React Hooks, In Plain English](https://itnext.io/the-first-rule-of-react-hooks-in-plain-english-1e0d5ae32009)
- Roep hooks enkel aan vanuit React functies. Dit wil zeggen: enkel vanuit function components of vanuit eigen geschreven hooks.

Hooks maken gebruik van closures, let dus op voor stale closures! [Zie hier voor enkele voorbeelden](https://dmitripavlutin.com/react-hooks-stale-closures/).

### React.memo en pure functions

Voeg een `console.log` instructie toe voor elke `return` in onderstaande componenten:

```tsx
// src/pages/transactions/TransactionList.tsx
export default function TransactionList() {
  // ...
  console.log('Rendering transactionlist...');
  return (...);
}

// src/components/transactions/Transaction.tsx
export default function Transaction(props) {
  // ...
  console.log('Rendering transaction...');
  return (...);
}
```

Telkens als we een letter ingeven in het zoekveld worden alle componenten opnieuw gerenderd, hoewel er niets wijzigt aan de output van de component. De `Transaction` component heeft als prop een transaction en deze blijft ongewijzigd als de gebruiker een letter ingeeft in het zoekveld. Toch wordt de component opnieuw gerenderd.

Een **pure component** is een component die gegeven dezelfde props dezelfde output genereert. `Transaction` is een pure component. Gegeven dezelfde props, wordt dezelfde output gegenereerd. We willen een pure component niet opnieuw renderen als de properties niet gewijzigd zijn. De `memo` functie wordt gebruikt om een component te creëren die enkel opnieuw zal renderen als de props wijzigen.

```tsx
// src/components/transactions/Transaction.tsx
import { memo } from 'react'; // 👈

// 👇
const TransactionMemoized = memo(function Transaction({
  id,
  user,
  date,
  amount,
  place,
  onDelete,
}) {
  //...
});
export default TransactionMemoized;
```

Start de app en bekijk de console. Verwijder nadien de toegevoegd `console.log` statements.

### useCallback hook

Pas de `TransactionList` aan en voeg onderstaande functie toe. We geven een extra alert als de transactie verwijderd is.

```js
const handleDeleteTransaction = async (id) => {
  await deleteTransaction(id);
  alert('Transaction is removed');
};
```

Geef deze door als property `onDelete` aan de `TransactionTable` component:

```tsx
<TransactionsTable
  transactions={filteredTransactions}
  onDelete={handleDeleteTransaction}
/>
```

Van zodra we een letter ingeven in de zoekbalk worden alle transacties toch opnieuw gerenderd.

`TransactionTable` bevat een prop `onDelete`. Deze wordt doorgegeven door de parent component `TransactionList`. `handleDeleteTransaction` is de event handler functie. JavaScript gaat er vanuit dat de functie `handleDeleteTransaction` bij elke render verschillend is. Echter is dit niet het geval. `useCallback` cachet een functie tussen twee renders en dit totdat de dependency array wijzigt.

Pas de code van de functie in de `TransactionList` component aan:

```tsx
// src/components/transactions/TransactionList.tsx
import { useState, useMemo, useCallback } from 'react'; // 👈

// ...

// 👇
const handleDeleteTransaction = useCallback(
  async (id) => {
    await deleteTransaction(id);
    alert('Transaction is removed');
  },
  [deleteTransaction],
);
```

Start de app en bekijk de console. De functie wordt nu gecachet. Merk op dat swr dit ook doet.

Gebruik `useCallback` niet zomaar overal: `useCallback` introduceert zelf ook een beetje overhead. Gebruik useCallback enkel als je een functie doorgeeft als prop (bij grote lijsten) of als dependency van een andere hook (bv. useEffect, useMemo, useCallback). Dit is nodig omdat functies in JavaScript referentietypes zijn. Bij elke render wordt een nieuwe functie aangemaakt, ook al is de code identiek. Hierdoor worden pure componenten onnodig opnieuw gerenderd of worden hooks onnodig opnieuw uitgevoerd.

> **Oplossing voorbeeldapplicatie**
>
> ```bash
> git clone https://github.com/HOGENT-frontendweb/frontendweb-budget.git
> cd frontendweb-budget
> git checkout -b les5-opl 53ef032
> pnpm install
> pnpm dev
> ```
>
> Vergeet geen `.env` aan te maken! Bekijk de [README](https://github.com/HOGENT-frontendweb/frontendweb-budget?tab=readme-ov-file#budgetapp) voor meer informatie.

## Mogelijke extra's voor de examenopdracht

- [Formik](https://www.npmjs.com/package/formik)
- [react-jsonschema-form](https://www.npmjs.com/package/react-jsonschema-form): formulieren genereren op basis van een JSON schema
- Validatie in [react-hook-form](https://www.npmjs.com/package/react-hook-form) met [Joi](https://www.npmjs.com/package/joi), [Yup](https://npmjs.com/package/yup) of dergelijke
  - Dit is een vrij kleine extra, dus zorg ervoor dat je nog een andere extra toevoegt.
- Maak gebruik van **meerdere** custom hooks uit:
  - <https://nikgraf.github.io/react-hooks/>
  - <https://github.com/streamich/react-use>
  - Dit is een vrij kleine extra, dus zorg ervoor dat je nog een andere extra toevoegt.
  - Zelf custom hooks schrijven is **geen** extra

## Must reads

- [Collection of React Hooks](https://nikgraf.github.io/react-hooks/)
- [react-use: Collection of essential React Hooks.](https://github.com/streamich/react-use)
- [Are You Making This React State Mistake?](https://www.youtube.com/watch?v=NZqMVUEiDIw)
- [useState: Asynchronous or what?](https://youtu.be/RAJD4KpX8LA)
- [3 React Mistakes, 1 App Killer](https://youtu.be/QuLfCUh-iwI)
- [Were React Hooks a Mistake?](https://jakelazaroff.com/words/were-react-hooks-a-mistake/)
- [How React 18 Improves Application Performance](https://vercel.com/blog/how-react-18-improves-application-performance)
- [Free & Open Source Animated ReactJS Components](https://animata.design/)

```

```

```

```
