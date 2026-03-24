# 07 – Services & Dependency Injection

## Lernziele
- Du weisst, was Services sind und wofür sie verwendet werden
- Du kannst Services erstellen und verwenden
- Du verstehst das Konzept der Dependency Injection (DI)
- Du kennst die verschiedenen Providerschichten

---

## 1. Was sind Services?

**Services** sind Klassen, die **Business-Logik, Daten und Funktionen** enthalten, die in mehreren Komponenten verwendet werden können.

### Warum Services?

**Problem ohne Services:**

```
ProduktListeComponent
  └── produkte: Produkt[] = [...]  ← Daten hier

WarenkorbComponent
  └── produkte braucht es auch... ← Daten müssen dupliziert werden
```

**Mit Services:**

```
ProduktService (enthält die Daten)
  ├── ProduktListeComponent  ← Holt Daten vom Service
  └── WarenkorbComponent     ← Holt Daten vom Service
```

### Wann einen Service verwenden?

- Daten die in **mehreren Komponenten** benötigt werden
- **HTTP-Anfragen** (API-Calls)
- **Business-Logik** die von der UI getrennt sein soll
- **Shared State** (gemeinsam genutzter Zustand)

---

## 2. Service erstellen

```bash
ng generate service services/produkt
# Kurzform:
ng g s services/produkt
```

```typescript
// services/produkt.service.ts
import { Injectable } from '@angular/core';

export interface Produkt {
  id: number;
  name: string;
  preis: number;
}

@Injectable({
  providedIn: 'root'  // Verfügbar in der gesamten App (Singleton)
})
export class ProduktService {
  private produkte: Produkt[] = [
    { id: 1, name: 'Laptop', preis: 999 },
    { id: 2, name: 'Maus', preis: 29 },
    { id: 3, name: 'Tastatur', preis: 79 },
  ];

  // Alle Produkte zurückgeben
  getAlleProdukte(): Produkt[] {
    return this.produkte;
  }

  // Ein Produkt nach ID finden
  getProduktById(id: number): Produkt | undefined {
    return this.produkte.find(p => p.id === id);
  }

  // Produkt hinzufügen
  addProdukt(produkt: Omit<Produkt, 'id'>): void {
    const neuesId = Math.max(...this.produkte.map(p => p.id)) + 1;
    this.produkte.push({ id: neuesId, ...produkt });
  }

  // Produkt löschen
  deleteProdukt(id: number): void {
    this.produkte = this.produkte.filter(p => p.id !== id);
  }
}
```

---

## 3. Dependency Injection (DI)

**Dependency Injection** bedeutet, dass Angular automatisch die **Abhängigkeiten** (z.B. Services) in Klassen **injiziert** (einfügt), anstatt dass die Klasse sie selbst erstellen muss.

### Ohne DI (schlechtes Muster)

```typescript
export class ProduktListeComponent {
  private service: ProduktService;

  constructor() {
    this.service = new ProduktService(); // ❌ Komponente erstellt Service selbst
    // Probleme: Schwer testbar, kein Singleton, enge Kopplung
  }
}
```

### Mit DI (gutes Muster)

```typescript
export class ProduktListeComponent {
  constructor(private service: ProduktService) {
    // ✅ Angular injiziert den Service automatisch
  }
}
```

---

## 4. Service in einer Komponente verwenden

```typescript
// produkt-liste.component.ts
import { Component, OnInit } from '@angular/core';
import { NgFor } from '@angular/common';
import { ProduktService, Produkt } from '../services/produkt.service';

@Component({
  selector: 'app-produkt-liste',
  standalone: true,
  imports: [NgFor],
  template: `
    <h1>Produkte ({{ produkte.length }})</h1>
    <ul>
      <li *ngFor="let p of produkte">
        {{ p.name }} – {{ p.preis }} CHF
        <button (click)="loeschen(p.id)">Löschen</button>
      </li>
    </ul>
    <button (click)="addBeispiel()">Produkt hinzufügen</button>
  `
})
export class ProduktListeComponent implements OnInit {
  produkte: Produkt[] = [];

  // Service wird per DI injiziert
  constructor(private produktService: ProduktService) {}

  ngOnInit(): void {
    // Daten beim Starten der Komponente laden
    this.produkte = this.produktService.getAlleProdukte();
  }

  loeschen(id: number): void {
    this.produktService.deleteProdukt(id);
    this.produkte = this.produktService.getAlleProdukte();
  }

  addBeispiel(): void {
    this.produktService.addProdukt({ name: 'Monitor', preis: 299 });
    this.produkte = this.produktService.getAlleProdukte();
  }
}
```

---

## 5. `inject()` Funktion (moderne Alternative)

Ab Angular 14 kann die `inject()` Funktion als **Alternative** zum Konstruktor-Injection verwendet werden.

```typescript
import { Component, OnInit, inject } from '@angular/core';
import { ProduktService } from '../services/produkt.service';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `...`
})
export class DemoComponent implements OnInit {
  // Moderne Alternative zu Konstruktor-Injection
  private produktService = inject(ProduktService);

  ngOnInit(): void {
    const produkte = this.produktService.getAlleProdukte();
  }
}
```

---

## 6. providedIn – Wo wird der Service bereitgestellt?

```typescript
// Root-Level (Standard – empfohlen)
@Injectable({
  providedIn: 'root'
  // → Singleton: Eine Instanz für die gesamte App
})
export class GlobalerService {}

// Component-Level (eigene Instanz pro Komponente)
@Component({
  selector: 'app-demo',
  providers: [LocalerService]
  // → Neue Instanz für diese Komponente
})
export class DemoComponent {}
```

### Wann welches Scope verwenden?

| Scope | Beschreibung | Verwendung |
|---|---|---|
| `providedIn: 'root'` | Globale Singleton-Instanz | HTTP-Services, Auth, Shared State |
| `providers: [Service]` in Komponente | Eigene Instanz pro Komponente | Lokaler Zustand, Formulare |

---

## 7. Vollständiges Beispiel: Todo-App

```typescript
// todo.service.ts
import { Injectable } from '@angular/core';

export interface Todo {
  id: number;
  text: string;
  erledigt: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  private todos: Todo[] = [
    { id: 1, text: 'Angular lernen', erledigt: false },
    { id: 2, text: 'Service erstellen', erledigt: true },
  ];
  private naechsteId = 3;

  getAlle(): Todo[] {
    return [...this.todos]; // Kopie zurückgeben (Immutability)
  }

  getErledigt(): Todo[] {
    return this.todos.filter(t => t.erledigt);
  }

  getOffen(): Todo[] {
    return this.todos.filter(t => !t.erledigt);
  }

  add(text: string): void {
    this.todos.push({ id: this.naechsteId++, text, erledigt: false });
  }

  toggle(id: number): void {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.erledigt = !todo.erledigt;
    }
  }

  delete(id: number): void {
    this.todos = this.todos.filter(t => t.id !== id);
  }
}
```

```typescript
// todo-liste.component.ts
import { Component, OnInit } from '@angular/core';
import { NgFor, NgIf } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { TodoService, Todo } from '../services/todo.service';

@Component({
  selector: 'app-todo-liste',
  standalone: true,
  imports: [NgFor, NgIf, FormsModule],
  template: `
    <h1>Todo-Liste</h1>

    <div>
      <input [(ngModel)]="neuerText" placeholder="Neue Aufgabe...">
      <button (click)="hinzufuegen()">Hinzufügen</button>
    </div>

    <ul>
      <li *ngFor="let todo of todos">
        <input type="checkbox"
               [checked]="todo.erledigt"
               (change)="toggleTodo(todo.id)">
        <span [style.text-decoration]="todo.erledigt ? 'line-through' : 'none'">
          {{ todo.text }}
        </span>
        <button (click)="loeschen(todo.id)">✗</button>
      </li>
    </ul>

    <p>{{ offen }} offen, {{ erledigt }} erledigt</p>
  `
})
export class TodoListeComponent implements OnInit {
  todos: Todo[] = [];
  neuerText = '';

  constructor(private todoService: TodoService) {}

  ngOnInit(): void {
    this.laden();
  }

  laden(): void {
    this.todos = this.todoService.getAlle();
  }

  get offen(): number {
    return this.todos.filter(t => !t.erledigt).length;
  }

  get erledigt(): number {
    return this.todos.filter(t => t.erledigt).length;
  }

  hinzufuegen(): void {
    if (this.neuerText.trim()) {
      this.todoService.add(this.neuerText.trim());
      this.neuerText = '';
      this.laden();
    }
  }

  toggleTodo(id: number): void {
    this.todoService.toggle(id);
    this.laden();
  }

  loeschen(id: number): void {
    this.todoService.delete(id);
    this.laden();
  }
}
```

---

## 8. Services zwischen Komponenten kommunizieren lassen

Services eignen sich gut als **gemeinsamer Zustand** zwischen Komponenten.

```typescript
// warenkorb.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

export interface ProduktKurz {
  id: number;
  name: string;
  preis: number;
}

@Injectable({
  providedIn: 'root'
})
export class WarenkorbService {
  // BehaviorSubject ermöglicht reaktive Updates
  private items$ = new BehaviorSubject<ProduktKurz[]>([]);
  readonly items = this.items$.asObservable();

  hinzufuegen(produkt: ProduktKurz): void {
    const aktuelleItems = this.items$.value;
    this.items$.next([...aktuelleItems, produkt]);
  }

  get anzahl(): number {
    return this.items$.value.length;
  }
}
```

```typescript
// warenkorb-icon.component.ts
import { Component } from '@angular/core';
import { AsyncPipe } from '@angular/common';
import { WarenkorbService } from '../services/warenkorb.service';

@Component({
  selector: 'app-warenkorb-icon',
  standalone: true,
  imports: [AsyncPipe],
  template: `
    <button>
      🛒 ({{ (warenkorbService.items | async)?.length || 0 }})
    </button>
  `
})
export class WarenkorbIconComponent {
  constructor(public warenkorbService: WarenkorbService) {}
}
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| **Service** | Klasse mit Business-Logik und Daten |
| `@Injectable` | Decorator, der eine Klasse als injizierbaren Service markiert |
| `providedIn: 'root'` | Globale Singleton-Instanz |
| **DI** | Angular injiziert Services automatisch |
| Konstruktor-Injection | `constructor(private service: MeinService)` |
| `inject()` | Moderne Alternative zur Konstruktor-Injection |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Services](https://angular.dev/guide/di)
- 📖 [Angular Docs – Dependency Injection](https://angular.dev/guide/di/dependency-injection)
- 🎥 [Angular University – Angular Services & DI](https://angular-university.io/course/angular-core-deep-dive-course)
