# 08 – Routing & Navigation

## Lernziele
- Du kannst den Angular Router konfigurieren
- Du kannst zwischen Seiten navigieren
- Du verstehst Route-Parameter, Query-Parameter und Child Routes
- Du kannst Route Guards für geschützte Routen verwenden

---

## 1. Was ist Routing?

**Routing** ermöglicht die Navigation zwischen verschiedenen **Ansichten/Seiten** in einer Single-Page Application, ohne die Seite neu zu laden.

```
URL: /produkte           → Zeigt ProduktListeComponent
URL: /produkte/1         → Zeigt ProduktDetailComponent (ID: 1)
URL: /warenkorb          → Zeigt WarenkorbComponent
URL: /login              → Zeigt LoginComponent
```

---

## 2. Routing einrichten

### app.routes.ts

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { ProduktListeComponent } from './produkte/produkt-liste.component';
import { ProduktDetailComponent } from './produkte/produkt-detail.component';
import { LoginComponent } from './auth/login.component';
import { NichtGefundenComponent } from './nicht-gefunden.component';

export const routes: Routes = [
  // Standard-Route (Weiterleitung)
  { path: '', redirectTo: '/home', pathMatch: 'full' },

  // Seiten
  { path: 'home', component: HomeComponent },
  { path: 'produkte', component: ProduktListeComponent },

  // Route mit Parameter (:id)
  { path: 'produkte/:id', component: ProduktDetailComponent },

  // Login
  { path: 'login', component: LoginComponent },

  // 404 – muss am Ende sein!
  { path: '**', component: NichtGefundenComponent },
];
```

### app.config.ts

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes)
  ]
};
```

### router-outlet im Template

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink],
  template: `
    <nav>
      <a routerLink="/home">Home</a>
      <a routerLink="/produkte">Produkte</a>
    </nav>

    <!-- Hier werden die Routen gerendert -->
    <router-outlet></router-outlet>
  `
})
export class AppComponent {}
```

---

## 3. Navigation mit RouterLink

```typescript
import { Component } from '@angular/core';
import { RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  selector: 'app-nav',
  standalone: true,
  imports: [RouterLink, RouterLinkActive],
  template: `
    <!-- Einfache Navigation -->
    <a routerLink="/home">Home</a>

    <!-- Mit RouterLinkActive (CSS-Klasse wenn Route aktiv) -->
    <a routerLink="/produkte" routerLinkActive="aktiv">Produkte</a>

    <!-- Mit exaktem Match -->
    <a routerLink="/" routerLinkActive="aktiv" [routerLinkActiveOptions]="{exact: true}">
      Home (exact)
    </a>

    <!-- Mit Parameter -->
    <a [routerLink]="['/produkte', produkt.id]">{{ produkt.name }}</a>

    <!-- Mit Query-Parametern -->
    <a [routerLink]="['/produkte']" [queryParams]="{seite: 1, filter: 'aktiv'}">
      Produkte (Seite 1)
    </a>
  `
})
export class NavComponent {
  produkt = { id: 1, name: 'Laptop' };
}
```

---

## 4. Programmatische Navigation (Router)

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-login',
  standalone: true,
  template: `
    <button (click)="login()">Login</button>
    <button (click)="goBack()">Zurück</button>
  `
})
export class LoginComponent {
  constructor(private router: Router) {}

  login(): void {
    // Nach dem Login weiterleiten
    this.router.navigate(['/dashboard']);

    // Mit Query-Parametern
    this.router.navigate(['/produkte'], { queryParams: { filter: 'neu' } });

    // Relativ navigieren
    this.router.navigate(['../'], { relativeTo: this.route });
  }

  goBack(): void {
    // Zur vorherigen Seite zurück (Browser-History)
    window.history.back();
  }
}
```

---

## 5. Route-Parameter lesen

```typescript
// produkt-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  selector: 'app-produkt-detail',
  standalone: true,
  template: `
    <h1>Produkt #{{ produktId }}</h1>
    <button (click)="zurueck()">← Zurück</button>
  `
})
export class ProduktDetailComponent implements OnInit {
  produktId: number = 0;

  constructor(
    private route: ActivatedRoute, // Aktuelle Route
    private router: Router
  ) {}

  ngOnInit(): void {
    // Route-Parameter lesen (snapshot = einmalig lesen)
    this.produktId = Number(this.route.snapshot.paramMap.get('id'));

    // Oder reaktiv (bei Änderungen reagieren)
    this.route.paramMap.subscribe(params => {
      this.produktId = Number(params.get('id'));
    });
  }

  zurueck(): void {
    this.router.navigate(['/produkte']);
  }
}
```

---

## 6. Query-Parameter lesen

```typescript
// produkt-liste.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-produkt-liste',
  standalone: true,
  template: `
    <p>Aktuelle Seite: {{ seite }}</p>
    <p>Filter: {{ filter }}</p>
  `
})
export class ProduktListeComponent implements OnInit {
  seite = 1;
  filter = '';

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    // URL: /produkte?seite=2&filter=aktiv
    this.route.queryParamMap.subscribe(params => {
      this.seite = Number(params.get('seite') || '1');
      this.filter = params.get('filter') || '';
    });
  }
}
```

---

## 7. Child Routes (Unterrouten)

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    children: [
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: AdminDashboardComponent },
      { path: 'benutzer', component: AdminBenutzerComponent },
      { path: 'einstellungen', component: AdminEinstellungenComponent },
    ]
  }
];
```

```typescript
// admin.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';

@Component({
  selector: 'app-admin',
  standalone: true,
  imports: [RouterOutlet, RouterLink],
  template: `
    <h1>Admin-Bereich</h1>
    <nav>
      <a routerLink="dashboard">Dashboard</a>
      <a routerLink="benutzer">Benutzer</a>
    </nav>
    <!-- Child Routes werden hier gerendert -->
    <router-outlet></router-outlet>
  `
})
export class AdminComponent {}
```

---

## 8. Lazy Loading

Grosse Apps können **Lazy Loading** verwenden, um Module erst bei Bedarf zu laden:

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    // Wird erst geladen wenn der User zu /admin navigiert
    loadChildren: () =>
      import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  },
  {
    path: 'shop',
    loadComponent: () =>
      import('./shop/shop.component').then(m => m.ShopComponent)
  }
];
```

---

## 9. Route Guards (Zugriffsschutz)

Guards schützen Routen vor unberechtigtem Zugriff.

```bash
ng generate guard guards/auth
```

### canActivate Guard

```typescript
// guards/auth.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true; // Zugriff erlaubt
  }

  // Weiterleitung zur Login-Seite
  router.navigate(['/login'], { queryParams: { returnUrl: state.url } });
  return false; // Zugriff verweigert
};
```

### Guard in Routen einsetzen

```typescript
// app.routes.ts
export const routes: Routes = [
  { path: 'login', component: LoginComponent },
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [authGuard]  // Geschützte Route
  },
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [authGuard, adminGuard]  // Mehrere Guards
  }
];
```

### Auth Service (Beispiel)

```typescript
// services/auth.service.ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private eingeloggt = false;

  isLoggedIn(): boolean {
    return this.eingeloggt || !!localStorage.getItem('token');
  }

  login(token: string): void {
    localStorage.setItem('token', token);
    this.eingeloggt = true;
  }

  logout(): void {
    localStorage.removeItem('token');
    this.eingeloggt = false;
  }
}
```

---

## 10. Resolver (Daten vor dem Routing laden)

```typescript
// resolvers/produkt.resolver.ts
import { inject } from '@angular/core';
import { ResolveFn } from '@angular/router';
import { ProduktService, Produkt } from '../services/produkt.service';

export const produktResolver: ResolveFn<Produkt | null> = (route) => {
  const id = Number(route.paramMap.get('id'));
  return inject(ProduktService).getProduktById(id) ?? null;
};
```

```typescript
// In der Route:
{ 
  path: 'produkte/:id',
  component: ProduktDetailComponent,
  resolve: { produkt: produktResolver }
}
```

```typescript
// In der Komponente:
export class ProduktDetailComponent implements OnInit {
  produkt!: Produkt;

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.produkt = this.route.snapshot.data['produkt'];
  }
}
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `Routes` | Array mit Route-Definitionen |
| `RouterOutlet` | Platzhalter für geroutete Komponenten |
| `RouterLink` | Navigation über HTML-Links |
| `Router.navigate()` | Programmatische Navigation |
| `ActivatedRoute` | Zugriff auf Route-Parameter |
| Child Routes | Unterrouten für verschachtelte Navigation |
| Lazy Loading | Module erst bei Bedarf laden |
| Route Guards | Zugriffsschutz für Routen |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Routing](https://angular.dev/guide/routing)
- 📖 [Angular Docs – Route Guards](https://angular.dev/guide/routing/common-router-tasks#preventing-unauthorized-access)
- 🎥 [Angular University – Angular Router](https://angular-university.io/course/angular-router-course)
