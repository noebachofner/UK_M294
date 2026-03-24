# 20 – Standalone Components (Vertiefung)

## Lernziele
- Du verstehst vollständig was Standalone Components sind
- Du kannst eine App komplett ohne NgModule aufbauen
- Du kennst die Unterschiede zwischen Standalone und Module-basiertem Ansatz
- Du kannst Module-basierte Code zu Standalone migrieren

---

## 1. Was sind Standalone Components?

Bis Angular 13 mussten alle Komponenten in einem **NgModule** registriert werden. Ab Angular 14 sind **Standalone Components** verfügbar. Ab Angular 17 sind sie der **Standard**.

### Früher (NgModule-basiert)

```typescript
// ❗ Komplexe Modul-Struktur nötig
@NgModule({
  declarations: [
    AppComponent,
    ProduktListeComponent,
    ProduktKarteComponent,
    HeaderComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    RouterModule.forRoot(routes),
    FormsModule
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### Jetzt (Standalone)

```typescript
// ✅ Direkter Import – kein Modul nötig!
@Component({
  selector: 'app-produkt-liste',
  standalone: true,
  imports: [ProduktKarteComponent, NgFor, NgIf],
  template: `...`
})
export class ProduktListeComponent {}
```

---

## 2. Standalone Component erstellen

```typescript
import { Component } from '@angular/core';
import { NgFor, NgIf, AsyncPipe } from '@angular/common';
import { RouterLink, RouterLinkActive } from '@angular/router';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-meine-komponente',
  standalone: true,               // ← Macht die Komponente standalone
  imports: [                      // ← Direkte Imports (kein Modul nötig)
    NgFor,                        // Built-in Direktiven
    NgIf,
    AsyncPipe,
    RouterLink,                   // Router-Direktiven
    RouterLinkActive,
    FormsModule,                  // Angular-Module
    AndereStandaloneComponent,    // Andere Standalone-Komponenten
  ],
  templateUrl: './meine-komponente.component.html',
  styleUrl: './meine-komponente.component.css'
})
export class MeineKomponente {
  // ...
}
```

---

## 3. App bootstrappen ohne AppModule

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

```typescript
// app.config.ts – Zentrale Provider-Konfiguration
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { routes } from './app.routes';
import { authInterceptor } from './interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimationsAsync(),
    // Weitere Provider...
  ]
};
```

---

## 4. Routing mit Standalone Components

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full'
  },
  {
    path: 'home',
    // Lazy Loading einer Standalone-Komponente
    loadComponent: () =>
      import('./home/home.component').then(m => m.HomeComponent)
  },
  {
    path: 'produkte',
    loadComponent: () =>
      import('./produkte/produkt-liste.component').then(m => m.ProduktListeComponent)
  },
  {
    path: 'produkte/:id',
    loadComponent: () =>
      import('./produkte/produkt-detail.component').then(m => m.ProduktDetailComponent)
  }
];
```

---

## 5. Services mit Standalone

Services funktionieren gleich wie vorher:

```typescript
// produkt.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'  // Globaler Service – verfügbar überall
})
export class ProduktService {
  constructor(private http: HttpClient) {}
  // ...
}
```

```typescript
// Lokal bereitgestellter Service (eigene Instanz pro Komponente)
@Component({
  providers: [LokalerService],  // Neue Instanz für diese Komponente
  template: `...`
})
export class MeineKomponente {
  constructor(private lokalerService: LokalerService) {}
}
```

---

## 6. Shared Imports – Häufige Imports bündeln

```typescript
// shared/common-imports.ts
// Häufig verwendete Imports an einem Ort bündeln
export const COMMON_IMPORTS = [
  NgFor,
  NgIf,
  NgClass,
  AsyncPipe,
  DatePipe,
  CurrencyPipe,
  RouterLink,
] as const;
```

```typescript
// Verwendung
@Component({
  imports: [...COMMON_IMPORTS, MatCardModule],
  template: `...`
})
export class MeineKomponente {}
```

---

## 7. Migration von NgModule zu Standalone

Angular bietet ein automatisches Migrations-Tool:

```bash
# Automatische Migration
ng generate @angular/core:standalone

# Optionen:
# 1. Komponenten zu Standalone migrieren
# 2. NgModule aufräumen
# 3. Bootstrap migrieren
```

### Manuelle Migration

**Schritt 1: `standalone: true` hinzufügen**

```typescript
// Vorher:
@Component({ selector: 'app-demo', template: `...` })
export class DemoComponent {}

// Nachher:
@Component({ selector: 'app-demo', standalone: true, template: `...` })
export class DemoComponent {}
```

**Schritt 2: Imports direkt in der Komponente angeben**

```typescript
@Component({
  standalone: true,
  imports: [NgFor, NgIf, MatButtonModule],
  template: `...`
})
```

**Schritt 3: Aus NgModule entfernen**

```typescript
@NgModule({
  declarations: [
    AppComponent,
    // DemoComponent hier entfernen
  ],
  imports: [
    BrowserModule,
    // Imports die DemoComponent brauchte hier entfernen (falls nur sie es brauchte)
  ]
})
```

---

## 8. Vollständiges Standalone-Projekt Beispiel

```
meine-app/
├── src/
│   ├── app/
│   │   ├── app.component.ts          ← Root-Komponente (standalone)
│   │   ├── app.component.html
│   │   ├── app.config.ts             ← Provider-Konfiguration
│   │   ├── app.routes.ts             ← Routing
│   │   │
│   │   ├── home/
│   │   │   └── home.component.ts     ← Standalone
│   │   │
│   │   ├── produkte/
│   │   │   ├── produkt-liste.component.ts  ← Standalone
│   │   │   └── produkt-detail.component.ts ← Standalone
│   │   │
│   │   └── services/
│   │       └── produkt.service.ts    ← providedIn: 'root'
│   │
│   ├── main.ts                       ← bootstrapApplication()
│   └── index.html
```

---

## 9. Vergleich: NgModule vs. Standalone

| Aspekt | NgModule | Standalone |
|---|---|---|
| **Boilerplate** | Viel (separate Modul-Dateien) | Wenig (direkte Imports) |
| **Transparenz** | Indirekt (über Module) | Direkt (sieht man was importiert wird) |
| **Tree-Shaking** | Auf Modul-Ebene | Auf Komponenten-Ebene (besser!) |
| **Lazy Loading** | `loadChildren` | `loadComponent` (einfacher) |
| **Migration** | – | Automatisch möglich |
| **Standard ab** | Angular 2–16 | Angular 17+ |

---

## Zusammenfassung

- Standalone Components sind seit Angular 17 der **Standard**
- Mit `standalone: true` braucht man **kein NgModule**
- Abhängigkeiten werden direkt in `imports: []` angegeben
- `bootstrapApplication()` startet die App ohne AppModule
- `app.config.ts` enthält die zentrale Provider-Konfiguration
- **Lazy Loading** wird einfacher mit `loadComponent`

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Standalone Components](https://angular.dev/guide/components/importing)
- 📖 [Angular Docs – Migration](https://angular.dev/reference/migrations/standalone)
