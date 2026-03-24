# 14 – State Management mit NgRx

## Lernziele
- Du verstehst das Problem das State Management löst
- Du kennst die Grundkonzepte von NgRx (Store, Actions, Reducers, Effects, Selectors)
- Du kannst einen einfachen NgRx Store aufsetzen
- Du weisst wann NgRx sinnvoll ist

---

## 1. Was ist State Management?

### Das Problem ohne State Management

Bei grossen Apps entsteht das Problem, dass Daten in vielen Komponenten geteilt werden müssen:

```
AppComponent
├── HeaderComponent        (braucht: Benutzername, Warenkorb-Anzahl)
├── ProduktListeComponent  (braucht: Produktliste, Ladestatus)
├── WarenkorbComponent     (braucht: Warenkorb-Items, Gesamtpreis)
└── BenutzerComponent      (braucht: Benutzerdaten, Einstellungen)
```

Wenn mehrere Komponenten dieselben Daten brauchen und verändern, wird die Kommunikation komplex.

### Die Lösung: Zentraler State Store

```
Alle Daten in einem zentralen "Store"
                    ↓
         NgRx Store (Einzige Quelle der Wahrheit)
         ├── benutzer: Benutzer
         ├── produkte: Produkt[]
         ├── warenkorb: WarenkorbItem[]
         └── isLoading: boolean
                    ↓
        Alle Komponenten lesen aus dem Store
```

---

## 2. NgRx Kernkonzepte

```
Component → dispatch(Action) → Reducer → Store (State)
                                                ↓
Component ←  select(Selector) ←────────────────┘

HTTP-Call → Effect → dispatch(Action) → Reducer → Store
```

| Konzept | Beschreibung |
|---|---|
| **Store** | Zentraler Speicher für den App-Zustand |
| **State** | Das aktuelle Datenobjekt im Store |
| **Action** | Beschreibt WAS passiert ist |
| **Reducer** | Berechnet neuen State basierend auf Action |
| **Selector** | Liest bestimmte Teile des States |
| **Effect** | Behandelt Nebeneffekte (HTTP, etc.) |

---

## 3. NgRx installieren

```bash
ng add @ngrx/store
ng add @ngrx/effects
ng add @ngrx/store-devtools  # Chrome DevTools Erweiterung
```

---

## 4. Vollständiges NgRx Beispiel – Todo-App

### 4.1 State Interface

```typescript
// store/todo.state.ts
export interface Todo {
  id: number;
  text: string;
  erledigt: boolean;
}

export interface TodoState {
  todos: Todo[];
  isLoading: boolean;
  fehler: string | null;
}

// Anfangszustand
export const initialTodoState: TodoState = {
  todos: [],
  isLoading: false,
  fehler: null
};
```

### 4.2 Actions

```typescript
// store/todo.actions.ts
import { createAction, props } from '@ngrx/store';
import { Todo } from './todo.state';

// Todos laden
export const ladeTodos = createAction('[Todo] Lade Todos');
export const ladeTodosErfolg = createAction(
  '[Todo] Lade Todos Erfolg',
  props<{ todos: Todo[] }>()
);
export const ladeTodosFehler = createAction(
  '[Todo] Lade Todos Fehler',
  props<{ fehler: string }>()
);

// Todo hinzufügen
export const addTodo = createAction(
  '[Todo] Add Todo',
  props<{ text: string }>()
);

// Todo umschalten (erledigt/offen)
export const toggleTodo = createAction(
  '[Todo] Toggle Todo',
  props<{ id: number }>()
);

// Todo löschen
export const deleteTodo = createAction(
  '[Todo] Delete Todo',
  props<{ id: number }>()
);
```

### 4.3 Reducer

```typescript
// store/todo.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { initialTodoState, TodoState } from './todo.state';
import * as TodoActions from './todo.actions';

export const todoReducer = createReducer(
  initialTodoState,

  // Laden gestartet
  on(TodoActions.ladeTodos, state => ({
    ...state,
    isLoading: true,
    fehler: null
  })),

  // Laden erfolgreich
  on(TodoActions.ladeTodosErfolg, (state, { todos }) => ({
    ...state,
    todos,
    isLoading: false
  })),

  // Laden fehlgeschlagen
  on(TodoActions.ladeTodosFehler, (state, { fehler }) => ({
    ...state,
    fehler,
    isLoading: false
  })),

  // Todo hinzufügen (lokal)
  on(TodoActions.addTodo, (state, { text }) => ({
    ...state,
    todos: [
      ...state.todos,
      {
        id: Date.now(),
        text,
        erledigt: false
      }
    ]
  })),

  // Todo umschalten
  on(TodoActions.toggleTodo, (state, { id }) => ({
    ...state,
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, erledigt: !todo.erledigt } : todo
    )
  })),

  // Todo löschen
  on(TodoActions.deleteTodo, (state, { id }) => ({
    ...state,
    todos: state.todos.filter(todo => todo.id !== id)
  }))
);
```

### 4.4 Selectors

```typescript
// store/todo.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { TodoState } from './todo.state';

// Feature-Selektor (Root-Zugriff)
export const selectTodoState = createFeatureSelector<TodoState>('todos');

// Abgeleitete Selektoren
export const selectAlleTodos = createSelector(
  selectTodoState,
  state => state.todos
);

export const selectIsLoading = createSelector(
  selectTodoState,
  state => state.isLoading
);

export const selectFehler = createSelector(
  selectTodoState,
  state => state.fehler
);

// Berechnete Selektoren
export const selectErledigteTodos = createSelector(
  selectAlleTodos,
  todos => todos.filter(t => t.erledigt)
);

export const selectOffeneTodos = createSelector(
  selectAlleTodos,
  todos => todos.filter(t => !t.erledigt)
);

export const selectTodosAnzahl = createSelector(
  selectAlleTodos,
  todos => todos.length
);
```

### 4.5 Effects (für HTTP-Calls)

```typescript
// store/todo.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, switchMap } from 'rxjs/operators';
import { of } from 'rxjs';
import { TodoApiService } from '../services/todo-api.service';
import * as TodoActions from './todo.actions';

@Injectable()
export class TodoEffects {
  // Effect: Todos vom Server laden
  ladeTodos$ = createEffect(() =>
    this.actions$.pipe(
      ofType(TodoActions.ladeTodos),       // Auf diese Action reagieren
      switchMap(() =>
        this.todoApiService.getAlleTodos().pipe(
          map(todos => TodoActions.ladeTodosErfolg({ todos })),  // Erfolg
          catchError(fehler =>
            of(TodoActions.ladeTodosFehler({ fehler: fehler.message })) // Fehler
          )
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private todoApiService: TodoApiService
  ) {}
}
```

### 4.6 Store konfigurieren

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideStore } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { provideStoreDevtools } from '@ngrx/store-devtools';
import { todoReducer } from './store/todo.reducer';
import { TodoEffects } from './store/todo.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore({ todos: todoReducer }),
    provideEffects([TodoEffects]),
    provideStoreDevtools({ maxAge: 25 })  // DevTools
  ]
};
```

### 4.7 Store in Komponenten verwenden

```typescript
// todo-liste.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AsyncPipe, NgFor, NgIf } from '@angular/common';
import { FormsModule } from '@angular/forms';
import * as TodoActions from '../store/todo.actions';
import * as TodoSelectors from '../store/todo.selectors';

@Component({
  selector: 'app-todo-liste',
  standalone: true,
  imports: [AsyncPipe, NgFor, NgIf, FormsModule],
  template: `
    <h1>Todos</h1>

    <!-- Lade-Anzeige -->
    <div *ngIf="isLoading$ | async">Lädt...</div>

    <!-- Fehler-Anzeige -->
    <div *ngIf="fehler$ | async as fehler" class="fehler">{{ fehler }}</div>

    <!-- Neue Todo -->
    <div>
      <input [(ngModel)]="neuerText" placeholder="Neue Aufgabe...">
      <button (click)="hinzufuegen()">Hinzufügen</button>
    </div>

    <!-- Todo-Liste -->
    <ul>
      <li *ngFor="let todo of todos$ | async">
        <input type="checkbox"
               [checked]="todo.erledigt"
               (change)="toggle(todo.id)">
        {{ todo.text }}
        <button (click)="loeschen(todo.id)">✗</button>
      </li>
    </ul>

    <!-- Zähler -->
    <p>Offen: {{ (offeneTodos$ | async)?.length }}</p>
    <p>Erledigt: {{ (erledigteTodos$ | async)?.length }}</p>
  `
})
export class TodoListeComponent implements OnInit {
  // Observables aus dem Store
  todos$ = this.store.select(TodoSelectors.selectAlleTodos);
  isLoading$ = this.store.select(TodoSelectors.selectIsLoading);
  fehler$ = this.store.select(TodoSelectors.selectFehler);
  offeneTodos$ = this.store.select(TodoSelectors.selectOffeneTodos);
  erledigteTodos$ = this.store.select(TodoSelectors.selectErledigteTodos);

  neuerText = '';

  constructor(private store: Store) {}

  ngOnInit(): void {
    // Action dispatchen → lädt Todos vom Server (via Effect)
    this.store.dispatch(TodoActions.ladeTodos());
  }

  hinzufuegen(): void {
    if (this.neuerText.trim()) {
      this.store.dispatch(TodoActions.addTodo({ text: this.neuerText.trim() }));
      this.neuerText = '';
    }
  }

  toggle(id: number): void {
    this.store.dispatch(TodoActions.toggleTodo({ id }));
  }

  loeschen(id: number): void {
    this.store.dispatch(TodoActions.deleteTodo({ id }));
  }
}
```

---

## 5. Wann NgRx verwenden?

### NgRx empfohlen wenn:
- Viele Komponenten **teilen denselben State**
- **HTTP-Calls** müssen koordiniert werden
- Der State muss **vorhersehbar** und **testbar** sein
- Die App ist **gross und komplex**

### NgRx nicht nötig wenn:
- Die App ist **klein** (wenige Komponenten)
- State wird nur **lokal** in einer Komponente verwendet
- Einfache **Services** mit BehaviorSubject reichen aus

### Alternative: Services mit BehaviorSubject

```typescript
// Einfacher Shared State ohne NgRx
@Injectable({ providedIn: 'root' })
export class TodoStore {
  private todos$ = new BehaviorSubject<Todo[]>([]);

  readonly todos = this.todos$.asObservable();

  add(text: string): void {
    this.todos$.next([...this.todos$.value, { id: Date.now(), text, erledigt: false }]);
  }

  toggle(id: number): void {
    this.todos$.next(
      this.todos$.value.map(t => t.id === id ? { ...t, erledigt: !t.erledigt } : t)
    );
  }
}
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| **Store** | Zentraler Zustandsspeicher |
| **Action** | Beschreibt ein Ereignis |
| **Reducer** | Reine Funktion: (State, Action) → neuer State |
| **Selector** | Liest Teile des States |
| **Effect** | Behandelt Nebeneffekte (HTTP) |
| `store.dispatch()` | Action auslösen |
| `store.select()` | State-Daten als Observable lesen |

---

## Weiterführende Ressourcen

- 📖 [NgRx Dokumentation](https://ngrx.io/docs)
- 🎥 [Angular University – NgRx Store](https://angular-university.io/course/angular-ngrx-course)
- 🛠️ [Redux DevTools Extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
