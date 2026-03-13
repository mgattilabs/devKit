---
name: angular
description: Generates Angular 17+ standalone components, configures advanced routing with lazy loading and guards, implements NgRx state management, applies RxJS patterns, and optimizes bundle performance. Use when building Angular 17+ applications with standalone components or signals, setting up NgRx stores, establishing RxJS reactive patterns, performance tuning, or writing Angular tests for enterprise apps.
license: MIT
metadata:
  author: https://github.com/mgattilabs
  version: "2.0.0"
  domain: frontend
  triggers: Angular, Angular 17, standalone components, signals, RxJS, NgRx, Angular performance, Angular routing, Angular testing, zoneless
  role: specialist
  scope: implementation
  output-format: code
  related-skills: typescript-pro, test-master
---

---

## Core Workflow

1. **Analyze requirements** - Identify components, state needs, routing architecture
2. **Design architecture** - Plan standalone components, signal usage, state flow with NgRx SignalStore
3. **Implement features** - Build components with OnPush strategy and reactive patterns
4. **Manage state** - Setup NgRx SignalStore (`@ngrx/signals`), effects, computed signals as needed; verify store hydration and state flow before proceeding
5. **Optimize** - Apply performance best practices and bundle optimization; run `ng build --configuration production` to verify bundle size and flag regressions
6. **Test** - Write unit and integration tests with Jest + Testing Library; verify >85% coverage threshold is met

---

## Stack Obbligatorio

| Categoria        | Tecnologia                              | Note                              |
|------------------|-----------------------------------------|-----------------------------------|
| Framework        | Angular (ultima versione stabile)       | Standalone components sempre      |
| Change Detection | Zoneless                                | `provideZonelessChangeDetection()` |
| State            | NgRx SignalStore (`@ngrx/signals`)      | Solo quando lo store è necessario |
| UI Components    | Angular Material                        | MAI componenti HTML naked per UI  |
| HTTP             | `HttpClient` + interceptor centralizzato | Nessuna chiamata diretta nel componente |
| DI               | `inject()` function                     | MAI constructor injection         |
| Styling          | SCSS + Angular Material theming         | No utility classes esterne        |
| Testing          | Jest + Testing Library                  | Un test per ogni store e service  |

---

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Components | `references/components.md` | Standalone components, signals, input/output |
| RxJS | `references/rxjs.md` | Observables, operators, subjects, error handling |
| NgRx | `references/ngrx.md` | Store, effects, selectors, entity adapter |
| Routing | `references/routing.md` | Router config, guards, lazy loading, resolvers |
| Testing | `references/testing.md` | TestBed, component tests, service tests |

---

## Quick Reference

> **Vedi le references per esempi completi** (`references/components.md`, `references/rxjs.md`, `references/ngrx.md`)

```typescript
// Standalone component con signals
@Component({
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [MatButtonModule, MatCardModule]
})
export class UserCardComponent {
  firstName = input.required<string>();          // signal input
  selected = output<string>();                   // signal output
  fullName = computed(() => `${this.firstName()}`); // computed
}

// RxJS subscription management
export class UsersComponent {
  private destroyRef = inject(DestroyRef);
  
  ngOnInit() {
    this.service.getUsers()
      .pipe(takeUntilDestroyed(this.destroyRef))  // auto-cleanup
      .subscribe(users => { /* ... */ });
  }
}

// NgRx SignalStore
export const UsersStore = signalStore(
  { providedIn: 'root' },
  withState({ items: [], loading: false }),
  withComputed(({ items }) => ({
    total: computed(() => items().length)
  })),
  withMethods((store) => ({
    loadAll: rxMethod(/* ... */)
  }))
);
```

---

## Struttura Cartelle — Vertical Slice Angular

```
src/
├── app/
│   ├── core/
│   │   ├── interceptors/
│   │   │   └── http.interceptor.ts      ← unico interceptor centralizzato
│   │   ├── guards/
│   │   └── core.providers.ts
│   │
│   ├── shared/
│   │   ├── components/                  ← solo componenti riutilizzabili puri
│   │   ├── pipes/
│   │   └── utils/
│   │
│   └── features/
│       └── [feature-name]/              ← una cartella per feature
│           ├── components/              ← componenti smart e presentazionali
│           │   └── [name]/
│           │       ├── [name].component.ts
│           │       ├── [name].component.html
│           │       └── [name].component.scss
│           ├── [feature-name].store.ts  ← NgRx SignalStore (solo se necessario)
│           ├── [feature-name].service.ts
│           ├── [feature-name].routes.ts
│           ├── models/
│           │   └── [feature-name].model.ts
│           └── index.ts                 ← barrel: esporta solo l'essenziale
```

---

## Routing — Template Obbligatorio

### `app.routes.ts` — routing principale, zero import diretti di componenti:

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    redirectTo: 'dashboard',
    pathMatch: 'full',
  },
  {
    path: 'dashboard',
    loadChildren: () =>
      import('./features/dashboard/dashboard.routes')
        .then(m => m.DASHBOARD_ROUTES),
  },
  {
    path: 'users',
    loadChildren: () =>
      import('./features/users/users.routes')
        .then(m => m.USERS_ROUTES),
  },
  {
    path: '**',
    loadComponent: () =>
      import('./shared/components/not-found/not-found.component')
        .then(m => m.NotFoundComponent),
  },
];
```

### `features/[feature]/[feature].routes.ts` — routing della feature:

```typescript
import { Routes } from '@angular/router';

export const [FEATURE]_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./components/[feature]-list/[feature]-list.component')
        .then(m => m.[Feature]ListComponent),
  },
  {
    path: ':id',
    loadComponent: () =>
      import('./components/[feature]-detail/[feature]-detail.component')
        .then(m => m.[Feature]DetailComponent),
  },
];
```

**Regole routing:**
- `app.routes.ts` non importa MAI un componente direttamente
- Ogni route usa `loadChildren` (per gruppi) o `loadComponent` (per singolo)
- I bundle per feature vengono creati automaticamente dal build

---

## Componente Angular — Template Obbligatorio

```typescript
import { ChangeDetectionStrategy, Component, inject } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
// altri import Material necessari...
import { FeatureStore } from '../feature.store';

@Component({
  selector: 'app-[feature]-[name]',
  standalone: true,
  imports: [
    // SOLO moduli Angular Material necessari
    MatButtonModule,
  ],
  templateUrl: './[name].component.html',
  styleUrl: './[name].component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class [Name]Component {
  // Se la feature ha uno store:
  protected readonly store = inject(FeatureStore);

  // Se la feature NON ha store (stato locale):
  // protected readonly items = signal<Item[]>([]);
  // protected readonly isLoading = signal(false);

  // Il componente NON contiene logica di business.
  // Delega allo store o chiama il service via azioni semplici.
}
```

**Regole componente:**
- `standalone: true` — sempre, nessun NgModule in codice nuovo
- `ChangeDetectionStrategy.OnPush` — sempre, senza eccezioni
- `inject()` — mai constructor injection
- Nessun metodo HTTP nel componente
- Nessun `ngOnInit` con logica complessa — usa store effects o `afterNextRender`
- Template: `@if`, `@for`, `@switch` — mai `*ngIf`, `*ngFor`

---

## Stato Locale (quando lo store NON serve)

Per componenti semplici senza stato condiviso, uso Angular signals direttamente:

```typescript
import { ChangeDetectionStrategy, Component, inject, signal } from '@angular/core';
import { FeatureService } from '../feature.service';

@Component({
  selector: 'app-simple-form',
  standalone: true,
  imports: [/* ... */],
  templateUrl: './simple-form.component.html',
  styleUrl: './simple-form.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class SimpleFormComponent {
  private readonly service = inject(FeatureService);

  protected readonly isSubmitting = signal(false);
  protected readonly errorMessage = signal<string | null>(null);

  protected async onSubmit(data: FormData): Promise<void> {
    this.isSubmitting.set(true);
    this.errorMessage.set(null);

    this.service.create(data).subscribe({
      next: () => {
        this.isSubmitting.set(false);
        // navigate o feedback
      },
      error: (err) => {
        this.errorMessage.set(err.message);
        this.isSubmitting.set(false);
      },
    });
  }
}
```

---

## NgRx SignalStore — Template (quando lo store SERVE)

```typescript
import { patchState, signalStore, withComputed, withMethods, withState } from '@ngrx/signals';
import { rxMethod } from '@ngrx/signals/rxjs-interop';
import { computed, inject } from '@angular/core';
import { pipe, switchMap, tap } from 'rxjs';
import { tapResponse } from '@ngrx/operators';
import { [Feature]Service } from './[feature].service';
import { [Feature]Model } from './models/[feature].model';

interface [Feature]State {
  items: [Feature]Model[];
  selectedId: string | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: [Feature]State = {
  items: [],
  selectedId: null,
  isLoading: false,
  error: null,
};

export const [Feature]Store = signalStore(
  { providedIn: 'root' },    // oppure feature-scoped se appropriato
  withState(initialState),

  withComputed(({ items, selectedId }) => ({
    selectedItem: computed(() =>
      items().find(i => i.id === selectedId()) ?? null
    ),
    totalCount: computed(() => items().length),
  })),

  withMethods((store, service = inject([Feature]Service)) => ({
    loadAll: rxMethod<void>(
      pipe(
        tap(() => patchState(store, { isLoading: true, error: null })),
        switchMap(() =>
          service.getAll().pipe(
            tapResponse({
              next: items => patchState(store, { items, isLoading: false }),
              error: (err: Error) =>
                patchState(store, { error: err.message, isLoading: false }),
            })
          )
        )
      )
    ),

    selectItem(id: string): void {
      patchState(store, { selectedId: id });
    },
  }))
);
```

---

## HTTP Service — Template Obbligatorio

```typescript
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { [Feature]Model } from './models/[feature].model';

@Injectable({ providedIn: 'root' })
export class [Feature]Service {
  readonly #http = inject(HttpClient);
  readonly #baseUrl = '/api/[feature]';

  getAll(): Observable<[Feature]Model[]> {
    return this.#http.get<[Feature]Model[]>(this.#baseUrl);
  }

  getById(id: string): Observable<[Feature]Model> {
    return this.#http.get<[Feature]Model>(`${this.#baseUrl}/${id}`);
  }

  create(payload: Omit<[Feature]Model, 'id'>): Observable<[Feature]Model> {
    return this.#http.post<[Feature]Model>(this.#baseUrl, payload);
  }

  update(id: string, payload: Partial<[Feature]Model>): Observable<[Feature]Model> {
    return this.#http.patch<[Feature]Model>(`${this.#baseUrl}/${id}`, payload);
  }

  delete(id: string): Observable<void> {
    return this.#http.delete<void>(`${this.#baseUrl}/${id}`);
  }
}
```

---

## Interceptor Centralizzato — Template Obbligatorio

```typescript
import { HttpErrorResponse, HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';

export const httpInterceptor: HttpInterceptorFn = (req, next) => {
  const authReq = req.clone({
    headers: req.headers
      .set('Content-Type', 'application/json'),
  });

  return next(authReq).pipe(
    catchError((error: HttpErrorResponse) => {
      // Gestione centralizzata errori:
      // 401 → redirect login
      // 403 → pagina forbidden
      // 500 → notifica globale
      console.error('[HTTP Error]', error.status, error.message);
      return throwError(() => error);
    })
  );
};
```

Registrato in `core.providers.ts`:

```typescript
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { httpInterceptor } from './interceptors/http.interceptor';

export const coreProviders = [
  provideHttpClient(withInterceptors([httpInterceptor])),
];
```

---

## Setup Nuovo Progetto

### `app.config.ts` base obbligatorio

```typescript
import { ApplicationConfig, provideZonelessChangeDetection } from '@angular/core';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { routes } from './app.routes';
import { coreProviders } from './core/core.providers';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZonelessChangeDetection(),
    provideRouter(routes, withComponentInputBinding()),
    provideAnimationsAsync(),
    ...coreProviders,
  ],
};
```

### Tema Angular Material

File `src/styles/_theme.scss`:

```scss
@use '@angular/material' as mat;

$primary: mat.define-palette(mat.$azure-palette);
$accent:  mat.define-palette(mat.$teal-palette, A200, A100, A400);
$warn:    mat.define-palette(mat.$red-palette);

$light-theme: mat.define-light-theme((
  color: (
    primary: $primary,
    accent:  $accent,
    warn:    $warn,
  ),
  typography: mat.define-typography-config(),
  density: 0,
));

@include mat.all-component-themes($light-theme);

@import 'custom-theme';
```

File `src/styles/_custom-theme.scss` — spazio per personalizzazioni, inizialmente vuoto:

```scss
// CUSTOM THEME
// Sovrascrivi qui i token del tema Angular Material.
// Documentazione: https://material.angular.io/guide/theming
```

In `styles.scss`:

```scss
@use 'styles/theme';

html, body {
  height: 100%;
  margin: 0;
  font-family: Roboto, 'Helvetica Neue', sans-serif;
}
```

---

## Constraints & Best Practices

### ✅ MUST DO

- **Use standalone components** (Angular 17+ default) — non creare più NgModules
- **Use signals for reactive state** — dove appropriato per semplificare la reattività
- **Use OnPush change detection** — sempre, nessuna eccezione
- **Use strict TypeScript configuration** — mantieni `strict: true` in `tsconfig.json`
- **Implement proper error handling** — in tutti gli RxJS streams (`.pipe(catchError(...))`)
- **Use `trackBy` functions** — in tutti i `@for` loops per performance
- **Write tests with >85% coverage** — per store, service e logica critica
- **Follow Angular style guide** — nomi file kebab-case, classi PascalCase
- **Use `provideZonelessChangeDetection()`** — per migliori performance
- **Use inject() function** — per dependency injection invece di constructor
- **Use Angular Material** — per tutti i componenti UI, nessun HTML naked
- **Lazy load features** — via `loadChildren` o `loadComponent` nel routing
- **Use `HttpInterceptorFn` (functional interceptors)** — never class-based interceptors

### ❌ MUST NOT DO

- **Use NgModule-based components** — eccetto quando strettamente necessario per compatibilità legacy
- **Forget to unsubscribe from observables** — usa `takeUntilDestroyed`, `rxMethod`, o `async` pipe
- **Use async operations without error handling** — ogni Observable deve avere `error` callback o `catchError`
- **Skip accessibility attributes** — aggiungi sempre `aria-label`, `role`, etc. dove necessario
- **Expose sensitive data in client-side code** — mai token, segreti, o dati sensibili hardcoded
- **Use `any` type** — senza una giustificazione documentata nel commento
- **Mutate state directly in NgRx** — usa sempre `patchState` o reducer immutabili
- **Skip unit tests for critical logic** — ogni store e service devono avere test
- **Use Zone.js** — se presente in un progetto nuovo, rimuovilo e usa zoneless
- **Use `*ngIf` / `*ngFor`** — usa la nuova sintassi `@if` / `@for` / `@switch`
- **Use constructor injection** — solo `inject()` function
- **Hardcode colors in component SCSS** — usa sempre i token del tema Angular Material

