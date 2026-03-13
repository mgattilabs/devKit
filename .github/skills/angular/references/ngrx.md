# NgRx State Management

> **Due approcci disponibili:**  
> 1. **NgRx SignalStore** — Moderno, lightweight, basato su signals (raccomandato per nuovi progetti)  
> 2. **NgRx Classic Store** — Pattern tradizionale con actions/reducers/effects (per progetti esistenti)

---

## NgRx SignalStore (Raccomandato)

### Setup Base

```typescript
// Installa: npm install @ngrx/signals

import { signalStore, withState, withComputed, withMethods } from '@ngrx/signals';
import { rxMethod } from '@ngrx/signals/rxjs-interop';
import { computed, inject } from '@angular/core';
import { pipe, switchMap, tap } from 'rxjs';
import { tapResponse } from '@ngrx/operators';

interface UsersState {
  users: User[];
  selectedId: string | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UsersState = {
  users: [],
  selectedId: null,
  isLoading: false,
  error: null,
};

export const UsersStore = signalStore(
  { providedIn: 'root' },    // Global store
  withState(initialState),
  
  // Computed signals (derived state)
  withComputed(({ users, selectedId }) => ({
    selectedUser: computed(() => 
      users().find(u => u.id === selectedId()) ?? null
    ),
    totalCount: computed(() => users().length),
    hasUsers: computed(() => users().length > 0),
  })),
  
  // Methods (actions + reducers)
  withMethods((store, usersService = inject(UsersService)) => ({
    // Metodo semplice per aggiornare stato
    selectUser(id: string): void {
      patchState(store, { selectedId: id });
    },
    
    // rxMethod per side effects (API calls)
    loadAll: rxMethod<void>(
      pipe(
        tap(() => patchState(store, { isLoading: true, error: null })),
        switchMap(() =>
          usersService.getAll().pipe(
            tapResponse({
              next: (users) => patchState(store, { users, isLoading: false }),
              error: (error: Error) => 
                patchState(store, { error: error.message, isLoading: false }),
            })
          )
        )
      )
    ),
    
    addUser: rxMethod<Omit<User, 'id'>>(
      pipe(
        tap(() => patchState(store, { isLoading: true })),
        switchMap((payload) =>
          usersService.create(payload).pipe(
            tapResponse({
              next: (newUser) => patchState(store, { 
                users: [...store.users(), newUser],
                isLoading: false 
              }),
              error: (error: Error) => 
                patchState(store, { error: error.message, isLoading: false }),
            })
          )
        )
      )
    ),
  }))
);
```

### Component Integration (SignalStore)

```typescript
import { Component, inject, effect } from '@angular/core';
import { UsersStore } from './users.store';

@Component({
  selector: 'app-users-list',
  standalone: true,
  template: `
    @if (store.isLoading()) {
      <mat-spinner />
    } @else if (store.error()) {
      <mat-error>{{ store.error() }}</mat-error>
    } @else {
      <mat-list>
        @for (user of store.users(); track user.id) {
          <mat-list-item (click)="store.selectUser(user.id)">
            {{ user.name }}
          </mat-list-item>
        }
      </mat-list>
      <p>Total: {{ store.totalCount() }}</p>
    }
  `
})
export class UsersListComponent {
  protected readonly store = inject(UsersStore);

  constructor() {
    // Carica dati all'avvio
    effect(() => {
      this.store.loadAll();
    }, { allowSignalWrites: true });
  }
}
```

### Feature-Scoped Store

```typescript
// Per store locale a una feature invece che globale
export const ProductsStore = signalStore(
  // RIMUOVI providedIn per renderlo locale
  withState(initialState),
  // ... rest
);

// Fornisci lo store a livello di route
{
  path: 'products',
  providers: [ProductsStore],  // ← Store feature-scoped
  loadComponent: () => import('./products.component')
}
```

---

## NgRx Classic Store (Legacy)

> **Usa questo pattern solo per:**
> - Progetti esistenti che già usano NgRx Classic
> - Team requirements specifici
> - Integrazione con librerie che richiedono classic store

### Store Setup

```typescript
// app.config.ts
import { provideStore } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { provideStoreDevtools } from '@ngrx/store-devtools';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore({
      users: usersReducer,
      products: productsReducer
    }),
    provideEffects([UsersEffects, ProductsEffects]),
    provideStoreDevtools({
      maxAge: 25,
      logOnly: !isDevMode()
    })
  ]
};
```

## Actions (Classic Store)

```typescript
// users.actions.ts
import { createActionGroup, emptyProps, props } from '@ngrx/store';
import { User } from './user.model';

export const UsersActions = createActionGroup({
  source: 'Users',
  events: {
    'Load Users': emptyProps(),
    'Load Users Success': props<{ users: User[] }>(),
    'Load Users Failure': props<{ error: string }>(),

    'Add User': props<{ user: User }>(),
    'Add User Success': props<{ user: User }>(),
    'Add User Failure': props<{ error: string }>(),

    'Update User': props<{ id: string; changes: Partial<User> }>(),
    'Update User Success': props<{ user: User }>(),

    'Delete User': props<{ id: string }>(),
    'Delete User Success': props<{ id: string }>()
  }
});
```

## Reducer with Entity Adapter (Classic Store)

```typescript
// users.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { createEntityAdapter, EntityAdapter, EntityState } from '@ngrx/entity';
import { UsersActions } from './users.actions';
import { User } from './user.model';

export interface UsersState extends EntityState<User> {
  loading: boolean;
  error: string | null;
  selectedUserId: string | null;
}

export const usersAdapter: EntityAdapter<User> = createEntityAdapter<User>({
  selectId: (user: User) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

const initialState: UsersState = usersAdapter.getInitialState({
  loading: false,
  error: null,
  selectedUserId: null
});

export const usersReducer = createReducer(
  initialState,

  // Load users
  on(UsersActions.loadUsers, (state) => ({
    ...state,
    loading: true,
    error: null
  })),

  on(UsersActions.loadUsersSuccess, (state, { users }) =>
    usersAdapter.setAll(users, {
      ...state,
      loading: false
    })
  ),

  on(UsersActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),

  // Add user
  on(UsersActions.addUserSuccess, (state, { user }) =>
    usersAdapter.addOne(user, state)
  ),

  // Update user
  on(UsersActions.updateUserSuccess, (state, { user }) =>
    usersAdapter.updateOne(
      { id: user.id, changes: user },
      state
    )
  ),

  // Delete user
  on(UsersActions.deleteUserSuccess, (state, { id }) =>
    usersAdapter.removeOne(id, state)
  )
);
```

## Selectors (Classic Store)

```typescript
// users.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { usersAdapter, UsersState } from './users.reducer';

export const selectUsersState = createFeatureSelector<UsersState>('users');

// Entity adapter selectors
const {
  selectIds,
  selectEntities,
  selectAll,
  selectTotal
} = usersAdapter.getSelectors();

export const selectUserIds = createSelector(
  selectUsersState,
  selectIds
);

export const selectUserEntities = createSelector(
  selectUsersState,
  selectEntities
);

export const selectAllUsers = createSelector(
  selectUsersState,
  selectAll
);

export const selectUsersTotal = createSelector(
  selectUsersState,
  selectTotal
);

export const selectUsersLoading = createSelector(
  selectUsersState,
  (state) => state.loading
);

export const selectUsersError = createSelector(
  selectUsersState,
  (state) => state.error
);

// Parameterized selector
export const selectUserById = (id: string) =>
  createSelector(
    selectUserEntities,
    (entities) => entities[id]
  );

// Composed selector
export const selectActiveUsers = createSelector(
  selectAllUsers,
  (users) => users.filter(user => user.isActive)
);

// Selector with multiple inputs
export const selectUserWithPosts = createSelector(
  selectUserById,
  selectAllPosts,
  (user, posts) => ({
    user,
    posts: posts.filter(post => post.userId === user?.id)
  })
);
```

## Effects (Classic Store)

```typescript
// users.effects.ts
import { Injectable, inject } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, mergeMap, exhaustMap } from 'rxjs/operators';
import { of } from 'rxjs';
import { UsersService } from './users.service';
import { UsersActions } from './users.actions';

@Injectable()
export class UsersEffects {
  private actions$ = inject(Actions);
  private usersService = inject(UsersService);

  // Load users effect
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UsersActions.loadUsers),
      mergeMap(() =>
        this.usersService.getAll().pipe(
          map(users => UsersActions.loadUsersSuccess({ users })),
          catchError(error =>
            of(UsersActions.loadUsersFailure({ error: error.message }))
          )
        )
      )
    )
  );

  // Add user effect (exhaustMap prevents duplicate submits)
  addUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UsersActions.addUser),
      exhaustMap(({ user }) =>
        this.usersService.create(user).pipe(
          map(createdUser => UsersActions.addUserSuccess({ user: createdUser })),
          catchError(error =>
            of(UsersActions.addUserFailure({ error: error.message }))
          )
        )
      )
    )
  );

  // Update user effect
  updateUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UsersActions.updateUser),
      mergeMap(({ id, changes }) =>
        this.usersService.update(id, changes).pipe(
          map(user => UsersActions.updateUserSuccess({ user })),
          catchError(error =>
            of(UsersActions.loadUsersFailure({ error: error.message }))
          )
        )
      )
    )
  );

  // Delete user effect
  deleteUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UsersActions.deleteUser),
      mergeMap(({ id }) =>
        this.usersService.delete(id).pipe(
          map(() => UsersActions.deleteUserSuccess({ id })),
          catchError(error =>
            of(UsersActions.loadUsersFailure({ error: error.message }))
          )
        )
      )
    )
  );

}
```

## Component Integration (Classic Store)

```typescript
// users-list.component.ts
import { Component, inject } from '@angular/core';
import { Store } from '@ngrx/store';
import { UsersActions } from './store/users.actions';
import {
  selectAllUsers,
  selectUsersLoading,
  selectUsersError
} from './store/users.selectors';

@Component({
  selector: 'app-users-list',
  standalone: true,
  template: `
    @if (loading()) {
      <div>Loading...</div>
    } @else if (error(); as err) {
      <div>Error: {{ err }}</div>
    } @else {
      @for (user of users(); track user.id) {
        <div>
          {{ user.name }}
          <button (click)="onDelete(user.id)">Delete</button>
        </div>
      }
    }
  `
})
export class UsersListComponent {
  private store = inject(Store);

  // Select data as signals
  users = toSignal(this.store.select(selectAllUsers), { initialValue: [] });
  loading = toSignal(this.store.select(selectUsersLoading), { initialValue: false });
  error = toSignal(this.store.select(selectUsersError), { initialValue: null });

  ngOnInit() {
    this.store.dispatch(UsersActions.loadUsers());
  }

  onDelete(id: string) {
    this.store.dispatch(UsersActions.deleteUser({ id }));
  }
}
```

## Quick Reference

| Concept | Usage |
|---------|-------|
| Actions | `createActionGroup()` |
| Reducer | `createReducer()`, `on()` |
| Entity | `createEntityAdapter()` |
| Selectors | `createSelector()`, `createFeatureSelector()` |
| Effects | `createEffect()`, `ofType()` |
| Store | `inject(Store)`, `store.select()`, `store.dispatch()` |
| DevTools | `provideStoreDevtools()` |
| Testing | Mock store, marble testing |