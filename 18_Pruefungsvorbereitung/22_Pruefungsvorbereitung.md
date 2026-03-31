# 22 – Prüfungsvorbereitung: Angular & Keycloak Gesamtübersicht

> Dieses Dokument fasst alle prüfungsrelevanten Themen mit Codebeispielen zusammen.  
> Lese es von oben nach unten durch – es deckt alle Bereiche der Prüfungsvorbereitung ab.

---

## Inhaltsverzeichnis

1. [Allgemeines – GitHub Workflow](#1-allgemeines--github-workflow)
2. [Angular Version nachweisen](#2-angular-version-nachweisen)
3. [Angular Material Installation & Verwendung](#3-angular-material-installation--verwendung)
4. [Services erstellen](#4-services-erstellen)
5. [Components erstellen](#5-components-erstellen)
6. [REST-API Zugriff aus dem Service](#6-rest-api-zugriff-aus-dem-service)
7. [JWT-Token im HTTP-Request (Interceptor)](#7-jwt-token-im-http-request-interceptor)
8. [Keycloak Login & Logout](#8-keycloak-login--logout)
9. [Routes mit Guards schützen](#9-routes-mit-guards-schützen)
10. [Rollen-basierte UI (Buttons ein-/ausblenden)](#10-rollen-basierte-ui-buttons-einausblenden)
11. [JWT-Token zwischenspeichern & Rollen auslesen](#11-jwt-token-zwischenspeichern--rollen-auslesen)
12. [Client Routing (Redirect & Catch-All)](#12-client-routing-redirect--catch-all)
13. [Menübar erstellen](#13-menübar-erstellen)
14. [Tabellenartige Liste mit Edit & Delete](#14-tabellenartige-liste-mit-edit--delete)
15. [Detailseite mit Routing-Daten](#15-detailseite-mit-routing-daten)
16. [Neu/Bearbeiten-Seite mit Feldvalidierung](#16-neubearbeiten-seite-mit-feldvalidierung)
17. [Lösch-Bestätigungsdialog](#17-lösch-bestätigungsdialog)
18. [Checkbox für Boolean-Werte](#18-checkbox-für-boolean-werte)
19. [Redirect nach Aktionen](#19-redirect-nach-aktionen)
20. [Styling in Component.scss](#20-styling-in-componentscss)
21. [Vollständiges App-Beispiel (Zusammenfassung)](#21-vollständiges-app-beispiel-zusammenfassung)

---

## 1. Allgemeines – GitHub Workflow

### Issues erstellen (mit Zeitaufwandschätzung)

Issues repräsentieren Aufgaben, Bugs oder Features. In GitHub werden sie unter dem Tab **Issues** angelegt.

**Gutes Issue-Template:**
```
Titel:  [Feature] Benutzer-Listenseite erstellen

Beschreibung:
Als Benutzer möchte ich eine Liste aller Benutzer sehen,
damit ich einen Überblick über alle Einträge habe.

Akzeptanzkriterien:
- [ ] Tabelle zeigt alle Benutzer aus der REST-API
- [ ] Jeder Eintrag hat Edit- und Delete-Button
- [ ] Löschen nur für Admin-Rolle sichtbar

Zeitaufwand-Schätzung: 3h
Milestone: Sprint 1
Labels: feature, frontend
```

### Milestones erstellen

Milestones gruppieren Issues zu einem Sprint oder Release:
- **Settings → Milestones → New Milestone**
- Name: `Sprint 1`, Fälligkeitsdatum setzen
- Issues werden dem Milestone zugewiesen

### Tags / Releases erstellen

```bash
# Tag lokal erstellen
git tag v1.0.0

# Tag mit Nachricht
git tag -a v1.0.0 -m "Erstes Release – Login & Benutzerverwaltung"

# Tag auf Remote pushen
git push origin v1.0.0

# Alle Tags pushen
git push origin --tags
```

### Branches erstellen

```bash
# Neuen Branch erstellen und wechseln
git checkout -b feature/benutzer-liste

# Branch pushen
git push -u origin feature/benutzer-liste

# Branch auflisten
git branch -a
```

### Merge-Requests (Pull Requests)

1. Feature-Branch auf GitHub pushen
2. Auf GitHub: **Compare & pull request**
3. Reviewer zuweisen, Labels und Milestone setzen
4. Code Review abwarten
5. Merge ausführen (Squash or Merge)
6. Feature-Branch löschen

### GitFlow Workflow

```
main          ●───────────────────────────────●  (Produktion)
               \                             /
develop         ●──●──●──●──●──●──●──●──●──●    (Integration)
                    \     /    \     /
feature/login        ●───●      ●───●           (Feature-Branches)
                              feature/user-list
```

| Branch | Beschreibung |
|---|---|
| `main` | Stabiler Produktions-Code |
| `develop` | Integrationsstand für nächstes Release |
| `feature/*` | Einzelne Features, werden in `develop` gemergt |
| `release/*` | Release-Vorbereitung, dann in `main` + `develop` |
| `hotfix/*` | Kritische Bugfixes direkt auf `main` |

---

## 2. Angular Version nachweisen

```bash
# Angular CLI Version anzeigen
ng version

# In package.json prüfen
cat package.json | grep "@angular/core"
```

**Ausgabe von `ng version` (Beispiel Angular 21.x):**
```
Angular CLI: 21.x.x
Node: 22.x.x
Angular: 21.x.x
... 
Package                         Version
---------------------------------------------------------
@angular/animations             21.x.x
@angular/common                 21.x.x
@angular/compiler               21.x.x
@angular/core                   21.x.x
@angular/forms                  21.x.x
@angular/platform-browser       21.x.x
@angular/router                 21.x.x
```

**Neues Angular-Projekt mit aktueller Version:**
```bash
# Aktuellste Version installieren
npm install -g @angular/cli@latest

# Neues Projekt erstellen
ng new mein-projekt --style=scss --routing=true

# Mit Standalone-Komponenten (Standard ab Angular 17+)
ng new mein-projekt --standalone
```

---

## 3. Angular Material Installation & Verwendung

### Installation

```bash
ng add @angular/material
# Wähle ein Theme (z.B. Azure/Blue)
# Typography: Yes
# Animations: Yes
```

### app.config.ts – Animations aktivieren

```typescript
// app.config.ts
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
    provideAnimationsAsync()
  ]
};
```

### Toolbar

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink, MatToolbarModule, MatButtonModule, MatIconModule],
  template: `
    <mat-toolbar color="primary">
      <span>Meine App</span>
      <span class="spacer"></span>
      <a mat-button routerLink="/benutzer">Benutzer</a>
      <button mat-icon-button (click)="logout()">
        <mat-icon>logout</mat-icon>
      </button>
    </mat-toolbar>
    <div class="content">
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent {
  logout(): void { /* ... */ }
}
```

### Buttons

```html
<button mat-button>Einfach</button>
<button mat-raised-button color="primary">Primär (erhöht)</button>
<button mat-flat-button color="accent">Accent (flach)</button>
<button mat-stroked-button color="warn">Warn (Umriss)</button>
<button mat-icon-button><mat-icon>delete</mat-icon></button>
<button mat-fab color="primary"><mat-icon>add</mat-icon></button>
```

### Checkboxen

```html
<mat-checkbox [(ngModel)]="aktiv">Aktiv</mat-checkbox>
<mat-checkbox [checked]="benutzer.aktiv" [disabled]="true">Aktiv (readonly)</mat-checkbox>
```

```typescript
import { MatCheckboxModule } from '@angular/material/checkbox';
import { FormsModule } from '@angular/forms';
```

---

## 4. Services erstellen

Services bündeln die Geschäftslogik und den Datenzugriff. Sie werden per Dependency Injection in Komponenten injiziert.

```bash
ng generate service services/benutzer
```

```typescript
// services/benutzer.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Benutzer {
  id: number;
  name: string;
  email: string;
  aktiv: boolean;
  rolle: string;
}

@Injectable({
  providedIn: 'root'   // Singleton – eine Instanz für die ganze App
})
export class BenutzerService {
  private readonly apiUrl = 'http://localhost:3000/api/benutzer';

  constructor(private http: HttpClient) {}

  alleBenutzer(): Observable<Benutzer[]> {
    return this.http.get<Benutzer[]>(this.apiUrl);
  }

  benutzerById(id: number): Observable<Benutzer> {
    return this.http.get<Benutzer>(`${this.apiUrl}/${id}`);
  }

  benutzerErstellen(benutzer: Omit<Benutzer, 'id'>): Observable<Benutzer> {
    return this.http.post<Benutzer>(this.apiUrl, benutzer);
  }

  benutzerAktualisieren(id: number, benutzer: Partial<Benutzer>): Observable<Benutzer> {
    return this.http.put<Benutzer>(`${this.apiUrl}/${id}`, benutzer);
  }

  benutzerLoeschen(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

---

## 5. Components erstellen

```bash
ng generate component components/benutzer-liste
ng generate component components/benutzer-detail
ng generate component components/benutzer-formular
```

**Struktur einer Standalone-Komponente:**

```typescript
// components/benutzer-liste/benutzer-liste.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';
import { MatTableModule } from '@angular/material/table';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { BenutzerService, Benutzer } from '../../services/benutzer.service';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-benutzer-liste',
  standalone: true,
  imports: [CommonModule, RouterLink, MatTableModule, MatButtonModule, MatIconModule],
  templateUrl: './benutzer-liste.component.html',
  styleUrl: './benutzer-liste.component.scss'
})
export class BenutzerListeComponent implements OnInit {
  benutzer: Benutzer[] = [];
  spalten: string[] = ['id', 'name', 'email', 'aktiv', 'aktionen'];

  constructor(
    private benutzerService: BenutzerService,
    public authService: AuthService
  ) {}

  ngOnInit(): void {
    this.laden();
  }

  laden(): void {
    this.benutzerService.alleBenutzer().subscribe(daten => {
      this.benutzer = daten;
    });
  }
}
```

---

## 6. REST-API Zugriff aus dem Service

Der gesamte HTTP-Zugriff findet **ausschliesslich im Service** statt, nie direkt in der Komponente.

```typescript
// services/produkt.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

export interface Produkt {
  id: number;
  bezeichnung: string;
  preis: number;
  aktiv: boolean;
}

@Injectable({ providedIn: 'root' })
export class ProduktService {
  private readonly apiUrl = 'http://localhost:8080/api/produkte';

  constructor(private http: HttpClient) {}

  // GET – Liste
  alle(): Observable<Produkt[]> {
    return this.http.get<Produkt[]>(this.apiUrl).pipe(
      catchError(this.fehlerBehandeln)
    );
  }

  // GET – Einzeln
  byId(id: number): Observable<Produkt> {
    return this.http.get<Produkt>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.fehlerBehandeln)
    );
  }

  // POST – Erstellen
  erstellen(produkt: Omit<Produkt, 'id'>): Observable<Produkt> {
    return this.http.post<Produkt>(this.apiUrl, produkt).pipe(
      catchError(this.fehlerBehandeln)
    );
  }

  // PUT – Aktualisieren
  aktualisieren(id: number, produkt: Produkt): Observable<Produkt> {
    return this.http.put<Produkt>(`${this.apiUrl}/${id}`, produkt).pipe(
      catchError(this.fehlerBehandeln)
    );
  }

  // DELETE – Löschen
  loeschen(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.fehlerBehandeln)
    );
  }

  private fehlerBehandeln(error: HttpErrorResponse): Observable<never> {
    console.error('HTTP Fehler:', error.status, error.message);
    return throwError(() => new Error(`Fehler ${error.status}: ${error.message}`));
  }
}
```

---

## 7. JWT-Token im HTTP-Request (Interceptor)

Ein **HTTP Interceptor** fügt automatisch den JWT-Token zu jeder ausgehenden Anfrage hinzu.

```bash
ng generate interceptor interceptors/auth
```

```typescript
// interceptors/auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    // Anfrage klonen und Authorization-Header hinzufügen
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
    return next(authReq);
  }

  return next(req);
};
```

```typescript
// app.config.ts – Interceptor registrieren
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor])),
    // ...
  ]
};
```

**Alternative: Token manuell pro Anfrage setzen (ohne Interceptor):**

```typescript
import { HttpHeaders } from '@angular/common/http';

alleBenutzer(): Observable<Benutzer[]> {
  const token = this.authService.getToken();
  const headers = new HttpHeaders({
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  });
  return this.http.get<Benutzer[]>(this.apiUrl, { headers });
}
```

---

## 8. Keycloak Login & Logout

### Installation

```bash
npm install keycloak-js
```

### Auth Service mit Keycloak

```typescript
// services/auth.service.ts
import { Injectable } from '@angular/core';
import Keycloak from 'keycloak-js';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private keycloak: Keycloak;
  private token: string | null = null;

  constructor() {
    this.keycloak = new Keycloak({
      url: 'http://localhost:8180',       // Keycloak-URL
      realm: 'mein-realm',               // Realm-Name
      clientId: 'mein-client'            // Client-ID
    });
  }

  // App-Start: Keycloak initialisieren
  async init(): Promise<boolean> {
    const authentifiziert = await this.keycloak.init({
      onLoad: 'check-sso',               // Prüfe ob Session vorhanden
      silentCheckSsoRedirectUri:
        window.location.origin + '/assets/silent-check-sso.html'
    });

    if (authentifiziert && this.keycloak.token) {
      this.token = this.keycloak.token;
      this.tokenZwischenspeichern(this.token);
    }

    // Token automatisch erneuern vor Ablauf
    this.keycloak.onTokenExpired = () => {
      this.keycloak.updateToken(60).then(erneuert => {
        if (erneuert && this.keycloak.token) {
          this.token = this.keycloak.token;
          this.tokenZwischenspeichern(this.token);
        }
      });
    };

    return authentifiziert;
  }

  // Login – leitet zu Keycloak weiter
  login(): void {
    this.keycloak.login({
      redirectUri: window.location.origin + '/dashboard'
    });
  }

  // Logout – leitet zu Keycloak-Logout weiter
  logout(): void {
    this.token = null;
    localStorage.removeItem('jwt_token');
    this.keycloak.logout({
      redirectUri: window.location.origin + '/login'
    });
  }

  // Eingeloggt?
  isLoggedIn(): boolean {
    return this.keycloak.authenticated === true;
  }

  // Token zurückgeben
  getToken(): string | null {
    return this.token ?? localStorage.getItem('jwt_token');
  }

  // Token im LocalStorage zwischenspeichern
  private tokenZwischenspeichern(token: string): void {
    localStorage.setItem('jwt_token', token);
  }
}
```

### App initialisieren mit Keycloak (APP_INITIALIZER)

```typescript
// app.config.ts
import { ApplicationConfig, APP_INITIALIZER } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { routes } from './app.routes';
import { AuthService } from './services/auth.service';
import { authInterceptor } from './interceptors/auth.interceptor';

function keycloakInitialisieren(authService: AuthService): () => Promise<boolean> {
  return () => authService.init();
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimationsAsync(),
    {
      provide: APP_INITIALIZER,
      useFactory: keycloakInitialisieren,
      deps: [AuthService],
      multi: true
    }
  ]
};
```

### Login-Komponente

```typescript
// components/login/login.component.ts
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatIconModule } from '@angular/material/icon';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [MatButtonModule, MatCardModule, MatIconModule],
  template: `
    <div class="login-container">
      <mat-card>
        <mat-card-header>
          <mat-card-title>Anmelden</mat-card-title>
        </mat-card-header>
        <mat-card-content>
          <p>Bitte melden Sie sich mit Ihrem Konto an.</p>
        </mat-card-content>
        <mat-card-actions>
          <button mat-raised-button color="primary" (click)="login()">
            <mat-icon>login</mat-icon>
            Mit Keycloak anmelden
          </button>
        </mat-card-actions>
      </mat-card>
    </div>
  `,
  styleUrl: './login.component.scss'
})
export class LoginComponent {
  constructor(private authService: AuthService) {}

  login(): void {
    this.authService.login();
  }
}
```

---

## 9. Routes mit Guards schützen

Guards prüfen vor dem Aufrufen einer Route, ob der Benutzer eingeloggt ist.

```bash
ng generate guard guards/auth
```

```typescript
// guards/auth.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true;  // Route freigeben
  }

  // Nicht eingeloggt → zum Login weiterleiten
  router.navigate(['/login'], {
    queryParams: { returnUrl: state.url }
  });
  return false;
};
```

```typescript
// guards/admin.guard.ts – Rollenbasierter Guard
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const adminGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.hatRolle('admin')) {
    return true;
  }

  router.navigate(['/nicht-autorisiert']);
  return false;
};
```

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { authGuard } from './guards/auth.guard';
import { adminGuard } from './guards/admin.guard';

export const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'login', component: LoginComponent },
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [authGuard]               // Nur eingeloggte Benutzer
  },
  {
    path: 'benutzer',
    component: BenutzerListeComponent,
    canActivate: [authGuard]
  },
  {
    path: 'benutzer/:id',
    component: BenutzerDetailComponent,
    canActivate: [authGuard]
  },
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [authGuard, adminGuard]   // Nur Admins
  },
  { path: '**', component: NichtGefundenComponent } // Catch-All
];
```

---

## 10. Rollen-basierte UI (Buttons ein-/ausblenden)

Buttons werden nur angezeigt, wenn die entsprechende Rolle vorhanden ist.

### AuthService – Rollen aus JWT lesen

```typescript
// services/auth.service.ts (Ergänzung)
import { Injectable } from '@angular/core';
import Keycloak from 'keycloak-js';

@Injectable({ providedIn: 'root' })
export class AuthService {
  // ...

  // Einzelne Rolle prüfen
  hatRolle(rolle: string): boolean {
    return this.keycloak.hasRealmRole(rolle) ||
           this.keycloak.hasResourceRole(rolle);
  }

  // Alle Rollen zurückgeben
  getRollen(): string[] {
    return this.keycloak.realmAccess?.roles ?? [];
  }

  // Benutzername
  getBenutzername(): string {
    return (this.keycloak.tokenParsed as any)?.preferred_username ?? '';
  }
}
```

### Template – Buttons per Rolle steuern

```typescript
// benutzer-liste.component.html
```

```html
<!-- Tabellen-Aktionen pro Zeile -->
<td mat-cell *matCellDef="let row">
  <!-- Bearbeiten: für alle eingeloggten Benutzer -->
  <a mat-icon-button [routerLink]="['/benutzer', row.id, 'bearbeiten']"
     color="primary"
     *ngIf="authService.isLoggedIn()">
    <mat-icon>edit</mat-icon>
  </a>

  <!-- Löschen: NUR für Admin-Rolle -->
  <button mat-icon-button color="warn"
          (click)="loeschen(row.id)"
          *ngIf="authService.hatRolle('admin')">
    <mat-icon>delete</mat-icon>
  </button>
</td>
```

```typescript
// In der Komponente – authService public machen für Template-Zugriff
export class BenutzerListeComponent {
  constructor(
    private benutzerService: BenutzerService,
    public authService: AuthService   // public – damit Template Zugriff hat
  ) {}
}
```

---

## 11. JWT-Token zwischenspeichern & Rollen auslesen

### JWT-Token Struktur

Ein JWT besteht aus 3 Base64-kodierten Teilen: `header.payload.signature`

```
eyJhbGciOiJSUzI1NiJ9                   ← Header
.eyJzdWIiOiJ1c2VyMSIsInJvbGVzIjpbImFkbWluIl19  ← Payload
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signatur
```

### Token manuell dekodieren (ohne Library)

```typescript
// services/jwt.service.ts
import { Injectable } from '@angular/core';

interface JwtPayload {
  sub: string;
  preferred_username: string;
  email: string;
  realm_access: { roles: string[] };
  resource_access: { [client: string]: { roles: string[] } };
  exp: number;
  iat: number;
}

@Injectable({ providedIn: 'root' })
export class JwtService {
  // Token im LocalStorage speichern
  speichern(token: string): void {
    localStorage.setItem('jwt_token', token);
  }

  // Token abrufen
  getToken(): string | null {
    return localStorage.getItem('jwt_token');
  }

  // Token löschen
  loeschen(): void {
    localStorage.removeItem('jwt_token');
  }

  // Payload dekodieren
  getPayload(): JwtPayload | null {
    const token = this.getToken();
    if (!token) return null;

    try {
      const parts = token.split('.');
      const payload = parts[1];
      // Base64 URL → Base64 → JSON
      const decoded = atob(payload.replace(/-/g, '+').replace(/_/g, '/'));
      return JSON.parse(decoded) as JwtPayload;
    } catch {
      return null;
    }
  }

  // Rollen aus dem Token lesen
  getRollen(): string[] {
    const payload = this.getPayload();
    return payload?.realm_access?.roles ?? [];
  }

  // Prüfen ob Rolle vorhanden
  hatRolle(rolle: string): boolean {
    return this.getRollen().includes(rolle);
  }

  // Prüfen ob Token abgelaufen
  istAbgelaufen(): boolean {
    const payload = this.getPayload();
    if (!payload) return true;
    const jetzt = Math.floor(Date.now() / 1000);
    return payload.exp < jetzt;
  }
}
```

---

## 12. Client Routing (Redirect & Catch-All)

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { LoginComponent } from './components/login/login.component';
import { DashboardComponent } from './components/dashboard/dashboard.component';
import { BenutzerListeComponent } from './components/benutzer-liste/benutzer-liste.component';
import { BenutzerDetailComponent } from './components/benutzer-detail/benutzer-detail.component';
import { BenutzerFormularComponent } from './components/benutzer-formular/benutzer-formular.component';
import { NichtGefundenComponent } from './components/nicht-gefunden/nicht-gefunden.component';
import { NichtAutorisiertComponent } from './components/nicht-autorisiert/nicht-autorisiert.component';
import { authGuard } from './guards/auth.guard';

export const routes: Routes = [
  // Weiterleitung: / → /dashboard
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },

  // Öffentliche Routen
  { path: 'login', component: LoginComponent },
  { path: 'nicht-autorisiert', component: NichtAutorisiertComponent },

  // Geschützte Routen
  { path: 'dashboard', component: DashboardComponent, canActivate: [authGuard] },
  { path: 'benutzer', component: BenutzerListeComponent, canActivate: [authGuard] },
  { path: 'benutzer/neu', component: BenutzerFormularComponent, canActivate: [authGuard] },
  { path: 'benutzer/:id', component: BenutzerDetailComponent, canActivate: [authGuard] },
  { path: 'benutzer/:id/bearbeiten', component: BenutzerFormularComponent, canActivate: [authGuard] },

  // Catch-All: alles andere → 404
  { path: '**', component: NichtGefundenComponent }
];
```

```typescript
// app.config.ts – Router konfigurieren
import { provideRouter, withComponentInputBinding } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding())  // Ermöglicht Input-Binding für Route-Parameter
  ]
};
```

---

## 13. Menübar erstellen

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive, Router } from '@angular/router';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatMenuModule } from '@angular/material/menu';
import { CommonModule } from '@angular/common';
import { AuthService } from './services/auth.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    RouterOutlet, RouterLink, RouterLinkActive, CommonModule,
    MatToolbarModule, MatButtonModule, MatIconModule, MatMenuModule
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  constructor(
    public authService: AuthService,
    private router: Router
  ) {}

  logout(): void {
    this.authService.logout();
  }
}
```

```html
<!-- app.component.html -->
<mat-toolbar color="primary" class="toolbar">
  <!-- Logo / App-Name -->
  <a routerLink="/dashboard" class="toolbar-logo">
    <mat-icon>apps</mat-icon>
    <span class="toolbar-titel">Meine App</span>
  </a>

  <span class="spacer"></span>

  <!-- Navigation (nur wenn eingeloggt) -->
  <ng-container *ngIf="authService.isLoggedIn()">
    <a mat-button routerLink="/dashboard" routerLinkActive="aktiv-link">
      <mat-icon>dashboard</mat-icon>
      Dashboard
    </a>
    <a mat-button routerLink="/benutzer" routerLinkActive="aktiv-link">
      <mat-icon>people</mat-icon>
      Benutzer
    </a>
  </ng-container>

  <!-- Benutzer-Menü -->
  <ng-container *ngIf="authService.isLoggedIn(); else loginBtn">
    <button mat-icon-button [matMenuTriggerFor]="benutzerMenu">
      <mat-icon>account_circle</mat-icon>
    </button>
    <mat-menu #benutzerMenu="matMenu">
      <button mat-menu-item disabled>
        <mat-icon>person</mat-icon>
        {{ authService.getBenutzername() }}
      </button>
      <mat-divider></mat-divider>
      <button mat-menu-item (click)="logout()">
        <mat-icon>logout</mat-icon>
        Abmelden
      </button>
    </mat-menu>
  </ng-container>

  <ng-template #loginBtn>
    <a mat-button routerLink="/login">
      <mat-icon>login</mat-icon>
      Anmelden
    </a>
  </ng-template>
</mat-toolbar>

<!-- Hauptinhalt -->
<main class="main-inhalt">
  <router-outlet></router-outlet>
</main>
```

```scss
/* app.component.scss */
.toolbar {
  position: sticky;
  top: 0;
  z-index: 100;
}

.toolbar-logo {
  display: flex;
  align-items: center;
  gap: 8px;
  text-decoration: none;
  color: inherit;
}

.toolbar-titel {
  font-size: 1.2rem;
  font-weight: 500;
}

.spacer {
  flex: 1 1 auto;
}

.aktiv-link {
  background: rgba(255, 255, 255, 0.15);
  border-radius: 4px;
}

.main-inhalt {
  padding: 24px;
  max-width: 1200px;
  margin: 0 auto;
}
```

---

## 14. Tabellenartige Liste mit Edit & Delete

### Komponenten-Template

```html
<!-- benutzer-liste.component.html -->
<div class="liste-header">
  <h1>Benutzer</h1>
  <a mat-raised-button color="primary"
     routerLink="/benutzer/neu"
     *ngIf="authService.isLoggedIn()">
    <mat-icon>add</mat-icon>
    Neuer Benutzer
  </a>
</div>

<!-- Lade-Anzeige -->
<div *ngIf="isLoading" class="laden">
  <mat-progress-spinner mode="indeterminate"></mat-progress-spinner>
</div>

<!-- Fehlermeldung -->
<div *ngIf="fehler" class="fehler-meldung">
  <mat-icon>error</mat-icon>
  {{ fehler }}
</div>

<!-- Material Data Table -->
<table mat-table [dataSource]="benutzer" class="benutzer-tabelle" *ngIf="!isLoading">

  <!-- ID -->
  <ng-container matColumnDef="id">
    <th mat-header-cell *matHeaderCellDef>ID</th>
    <td mat-cell *matCellDef="let row">{{ row.id }}</td>
  </ng-container>

  <!-- Name -->
  <ng-container matColumnDef="name">
    <th mat-header-cell *matHeaderCellDef>Name</th>
    <td mat-cell *matCellDef="let row">{{ row.name }}</td>
  </ng-container>

  <!-- E-Mail -->
  <ng-container matColumnDef="email">
    <th mat-header-cell *matHeaderCellDef>E-Mail</th>
    <td mat-cell *matCellDef="let row">{{ row.email }}</td>
  </ng-container>

  <!-- Aktiv (Boolean als Checkbox) -->
  <ng-container matColumnDef="aktiv">
    <th mat-header-cell *matHeaderCellDef>Aktiv</th>
    <td mat-cell *matCellDef="let row">
      <mat-checkbox [checked]="row.aktiv" [disabled]="true"></mat-checkbox>
    </td>
  </ng-container>

  <!-- Aktionen -->
  <ng-container matColumnDef="aktionen">
    <th mat-header-cell *matHeaderCellDef>Aktionen</th>
    <td mat-cell *matCellDef="let row">
      <!-- Detail anzeigen -->
      <a mat-icon-button color="primary"
         [routerLink]="['/benutzer', row.id]"
         matTooltip="Details">
        <mat-icon>visibility</mat-icon>
      </a>

      <!-- Bearbeiten: alle eingeloggten Benutzer -->
      <a mat-icon-button color="accent"
         [routerLink]="['/benutzer', row.id, 'bearbeiten']"
         matTooltip="Bearbeiten"
         *ngIf="authService.isLoggedIn()">
        <mat-icon>edit</mat-icon>
      </a>

      <!-- Löschen: NUR Admin -->
      <button mat-icon-button color="warn"
              (click)="loeschenBestaetigen(row)"
              matTooltip="Löschen"
              *ngIf="authService.hatRolle('admin')">
        <mat-icon>delete</mat-icon>
      </button>
    </td>
  </ng-container>

  <tr mat-header-row *matHeaderRowDef="spalten"></tr>
  <tr mat-row *matRowDef="let row; columns: spalten;"
      class="tabellen-zeile"
      [routerLink]="['/benutzer', row.id]"></tr>
</table>
```

### Komponenten-Klasse

```typescript
// benutzer-liste.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';
import { MatTableModule } from '@angular/material/table';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';
import { MatTooltipModule } from '@angular/material/tooltip';
import { MatDialog } from '@angular/material/dialog';
import { BenutzerService, Benutzer } from '../../services/benutzer.service';
import { AuthService } from '../../services/auth.service';
import { BestaetigungsDialogComponent } from '../bestaetigung-dialog/bestaetigung-dialog.component';

@Component({
  selector: 'app-benutzer-liste',
  standalone: true,
  imports: [
    CommonModule, RouterLink,
    MatTableModule, MatButtonModule, MatIconModule,
    MatCheckboxModule, MatProgressSpinnerModule, MatTooltipModule
  ],
  templateUrl: './benutzer-liste.component.html',
  styleUrl: './benutzer-liste.component.scss'
})
export class BenutzerListeComponent implements OnInit {
  benutzer: Benutzer[] = [];
  spalten: string[] = ['id', 'name', 'email', 'aktiv', 'aktionen'];
  isLoading = false;
  fehler = '';

  constructor(
    private benutzerService: BenutzerService,
    public authService: AuthService,
    private dialog: MatDialog
  ) {}

  ngOnInit(): void {
    this.laden();
  }

  laden(): void {
    this.isLoading = true;
    this.fehler = '';

    this.benutzerService.alleBenutzer().subscribe({
      next: (daten) => {
        this.benutzer = daten;
        this.isLoading = false;
      },
      error: (err) => {
        this.fehler = 'Fehler beim Laden: ' + err.message;
        this.isLoading = false;
      }
    });
  }

  loeschenBestaetigen(benutzer: Benutzer): void {
    const dialogRef = this.dialog.open(BestaetigungsDialogComponent, {
      width: '400px',
      data: {
        titel: 'Benutzer löschen',
        nachricht: `Möchten Sie "${benutzer.name}" wirklich löschen?`
      }
    });

    dialogRef.afterClosed().subscribe(bestaetigt => {
      if (bestaetigt) {
        this.loeschen(benutzer.id);
      }
    });
  }

  private loeschen(id: number): void {
    this.benutzerService.benutzerLoeschen(id).subscribe({
      next: () => {
        // Aus der lokalen Liste entfernen (kein neues HTTP-Request nötig)
        this.benutzer = this.benutzer.filter(b => b.id !== id);
      },
      error: (err) => {
        this.fehler = 'Fehler beim Löschen: ' + err.message;
      }
    });
  }
}
```

### Styling der Tabelle

```scss
/* benutzer-liste.component.scss */
.liste-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.benutzer-tabelle {
  width: 100%;
  background-color: #ffffff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.tabellen-zeile {
  cursor: pointer;
  transition: background-color 0.2s;

  &:hover {
    background-color: #e3f2fd;
  }
}

.fehler-meldung {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #d32f2f;
  background: #ffebee;
  padding: 12px;
  border-radius: 4px;
  margin-bottom: 16px;
}

.laden {
  display: flex;
  justify-content: center;
  padding: 48px;
}
```

---

## 15. Detailseite mit Routing-Daten

### Route-Parameter auslesen

```typescript
// benutzer-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ActivatedRoute, Router, RouterLink } from '@angular/router';
import { MatCardModule } from '@angular/material/card';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatChipsModule } from '@angular/material/chips';
import { BenutzerService, Benutzer } from '../../services/benutzer.service';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-benutzer-detail',
  standalone: true,
  imports: [
    CommonModule, RouterLink,
    MatCardModule, MatButtonModule, MatIconModule, MatCheckboxModule, MatChipsModule
  ],
  templateUrl: './benutzer-detail.component.html',
  styleUrl: './benutzer-detail.component.scss'
})
export class BenutzerDetailComponent implements OnInit {
  benutzer: Benutzer | null = null;
  isLoading = false;
  fehler = '';

  constructor(
    private route: ActivatedRoute,   // Aktuelle Route – für Parameter
    private router: Router,
    private benutzerService: BenutzerService,
    public authService: AuthService
  ) {}

  ngOnInit(): void {
    // Route-Parameter ":id" auslesen
    const id = Number(this.route.snapshot.paramMap.get('id'));
    this.laden(id);
  }

  private laden(id: number): void {
    this.isLoading = true;

    this.benutzerService.benutzerById(id).subscribe({
      next: (daten) => {
        this.benutzer = daten;
        this.isLoading = false;
      },
      error: () => {
        this.fehler = 'Benutzer nicht gefunden.';
        this.isLoading = false;
      }
    });
  }

  zurueck(): void {
    this.router.navigate(['/benutzer']);
  }

  bearbeiten(): void {
    this.router.navigate(['/benutzer', this.benutzer!.id, 'bearbeiten']);
  }
}
```

```html
<!-- benutzer-detail.component.html -->
<div class="detail-container">
  <div class="detail-header">
    <button mat-button (click)="zurueck()">
      <mat-icon>arrow_back</mat-icon>
      Zurück zur Liste
    </button>
    <h1>Benutzer-Details</h1>
  </div>

  <div *ngIf="isLoading">Lädt...</div>
  <div *ngIf="fehler" class="fehler">{{ fehler }}</div>

  <mat-card *ngIf="benutzer">
    <mat-card-header>
      <mat-card-title>{{ benutzer.name }}</mat-card-title>
      <mat-card-subtitle>
        <mat-chip [color]="benutzer.aktiv ? 'primary' : 'warn'" highlighted>
          {{ benutzer.aktiv ? 'Aktiv' : 'Inaktiv' }}
        </mat-chip>
      </mat-card-subtitle>
    </mat-card-header>

    <mat-card-content class="detail-felder">
      <div class="feld">
        <span class="feld-label">ID</span>
        <span>{{ benutzer.id }}</span>
      </div>
      <div class="feld">
        <span class="feld-label">Name</span>
        <span>{{ benutzer.name }}</span>
      </div>
      <div class="feld">
        <span class="feld-label">E-Mail</span>
        <span>{{ benutzer.email }}</span>
      </div>
      <div class="feld">
        <span class="feld-label">Rolle</span>
        <span>{{ benutzer.rolle }}</span>
      </div>
      <div class="feld">
        <span class="feld-label">Aktiv</span>
        <mat-checkbox [checked]="benutzer.aktiv" [disabled]="true"></mat-checkbox>
      </div>
    </mat-card-content>

    <mat-card-actions *ngIf="authService.isLoggedIn()">
      <button mat-raised-button color="primary" (click)="bearbeiten()">
        <mat-icon>edit</mat-icon>
        Bearbeiten
      </button>
    </mat-card-actions>
  </mat-card>
</div>
```

---

## 16. Neu/Bearbeiten-Seite mit Feldvalidierung

Dieselbe Komponente wird für **Neu** und **Bearbeiten** verwendet. Die Route-Parameter entscheiden, ob bearbeitet oder neu angelegt wird.

```typescript
// benutzer-formular.component.ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ActivatedRoute, Router } from '@angular/router';
import { ReactiveFormsModule, FormBuilder, FormGroup, Validators } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatButtonModule } from '@angular/material/button';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatSelectModule } from '@angular/material/select';
import { MatIconModule } from '@angular/material/icon';
import { BenutzerService, Benutzer } from '../../services/benutzer.service';

@Component({
  selector: 'app-benutzer-formular',
  standalone: true,
  imports: [
    CommonModule, ReactiveFormsModule,
    MatFormFieldModule, MatInputModule, MatButtonModule,
    MatCheckboxModule, MatSelectModule, MatIconModule
  ],
  templateUrl: './benutzer-formular.component.html',
  styleUrl: './benutzer-formular.component.scss'
})
export class BenutzerFormularComponent implements OnInit {
  form!: FormGroup;
  istBearbeiten = false;
  benutzerID: number | null = null;
  isLoading = false;
  fehler = '';

  constructor(
    private fb: FormBuilder,
    private route: ActivatedRoute,
    private router: Router,
    private benutzerService: BenutzerService
  ) {}

  ngOnInit(): void {
    // Formular aufbauen mit Validierung
    this.form = this.fb.group({
      name: ['', [
        Validators.required,             // Pflichtfeld
        Validators.minLength(2),         // Mindestlänge
        Validators.maxLength(100)        // Maximallänge
      ]],
      email: ['', [
        Validators.required,
        Validators.email                 // E-Mail-Format prüfen
      ]],
      rolle: ['user', Validators.required],
      aktiv: [true]
    });

    // Route-Parameter prüfen: Bearbeiten oder Neu?
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.istBearbeiten = true;
      this.benutzerID = Number(id);
      this.laden(this.benutzerID);
    }
  }

  private laden(id: number): void {
    this.isLoading = true;
    this.benutzerService.benutzerById(id).subscribe({
      next: (benutzer) => {
        // Formular mit bestehenden Daten füllen
        this.form.patchValue({
          name: benutzer.name,
          email: benutzer.email,
          rolle: benutzer.rolle,
          aktiv: benutzer.aktiv
        });
        this.isLoading = false;
      },
      error: () => {
        this.fehler = 'Benutzer nicht gefunden.';
        this.isLoading = false;
      }
    });
  }

  speichern(): void {
    if (this.form.invalid) {
      // Alle Felder als "berührt" markieren, damit Fehler sichtbar werden
      this.form.markAllAsTouched();
      return;
    }

    this.isLoading = true;
    const daten = this.form.value;

    if (this.istBearbeiten && this.benutzerID) {
      // Bearbeiten
      this.benutzerService.benutzerAktualisieren(this.benutzerID, daten).subscribe({
        next: () => this.router.navigate(['/benutzer', this.benutzerID]),
        error: (err) => {
          this.fehler = 'Fehler beim Speichern: ' + err.message;
          this.isLoading = false;
        }
      });
    } else {
      // Neu erstellen
      this.benutzerService.benutzerErstellen(daten).subscribe({
        next: (neu) => this.router.navigate(['/benutzer', neu.id]),
        error: (err) => {
          this.fehler = 'Fehler beim Erstellen: ' + err.message;
          this.isLoading = false;
        }
      });
    }
  }

  abbrechen(): void {
    if (this.istBearbeiten && this.benutzerID) {
      this.router.navigate(['/benutzer', this.benutzerID]);
    } else {
      this.router.navigate(['/benutzer']);
    }
  }

  // Hilfsmethoden für Template-Fehleranzeige
  istFeldUngueltig(feld: string): boolean {
    const control = this.form.get(feld);
    return !!(control && control.invalid && control.touched);
  }

  getFehler(feld: string): string {
    const control = this.form.get(feld);
    if (!control) return '';

    if (control.hasError('required')) return 'Dieses Feld ist erforderlich.';
    if (control.hasError('email')) return 'Bitte geben Sie eine gültige E-Mail-Adresse ein.';
    if (control.hasError('minlength')) {
      const min = control.errors?.['minlength'].requiredLength;
      return `Mindestens ${min} Zeichen erforderlich.`;
    }
    if (control.hasError('maxlength')) {
      const max = control.errors?.['maxlength'].requiredLength;
      return `Maximal ${max} Zeichen erlaubt.`;
    }
    return 'Ungültige Eingabe.';
  }
}
```

```html
<!-- benutzer-formular.component.html -->
<div class="formular-container">
  <h1>{{ istBearbeiten ? 'Benutzer bearbeiten' : 'Neuer Benutzer' }}</h1>

  <div *ngIf="fehler" class="fehler-meldung">{{ fehler }}</div>

  <form [formGroup]="form" (ngSubmit)="speichern()" class="formular">

    <!-- Name -->
    <mat-form-field appearance="outline">
      <mat-label>Name</mat-label>
      <input matInput formControlName="name" placeholder="Vor- und Nachname">
      <mat-error *ngIf="istFeldUngueltig('name')">
        {{ getFehler('name') }}
      </mat-error>
    </mat-form-field>

    <!-- E-Mail -->
    <mat-form-field appearance="outline">
      <mat-label>E-Mail</mat-label>
      <input matInput type="email" formControlName="email" placeholder="name@beispiel.ch">
      <mat-error *ngIf="istFeldUngueltig('email')">
        {{ getFehler('email') }}
      </mat-error>
    </mat-form-field>

    <!-- Rolle (Dropdown) -->
    <mat-form-field appearance="outline">
      <mat-label>Rolle</mat-label>
      <mat-select formControlName="rolle">
        <mat-option value="user">Benutzer</mat-option>
        <mat-option value="admin">Administrator</mat-option>
        <mat-option value="moderator">Moderator</mat-option>
      </mat-select>
      <mat-error *ngIf="istFeldUngueltig('rolle')">Bitte eine Rolle wählen.</mat-error>
    </mat-form-field>

    <!-- Aktiv (Checkbox) -->
    <div class="checkbox-zeile">
      <mat-checkbox formControlName="aktiv">Benutzer ist aktiv</mat-checkbox>
    </div>

    <!-- Aktionen -->
    <div class="aktionen">
      <button mat-button type="button" (click)="abbrechen()">
        <mat-icon>close</mat-icon>
        Abbrechen
      </button>
      <button mat-raised-button color="primary" type="submit" [disabled]="isLoading">
        <mat-icon>save</mat-icon>
        {{ istBearbeiten ? 'Speichern' : 'Erstellen' }}
      </button>
    </div>

  </form>
</div>
```

```scss
/* benutzer-formular.component.scss */
.formular-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 24px;
}

.formular {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.checkbox-zeile {
  padding: 8px 0;
}

.aktionen {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding-top: 8px;
}

.fehler-meldung {
  background: #ffebee;
  color: #d32f2f;
  padding: 12px;
  border-radius: 4px;
  margin-bottom: 16px;
}
```

---

## 17. Lösch-Bestätigungsdialog

```bash
ng generate component components/bestaetigung-dialog
```

```typescript
// components/bestaetigung-dialog/bestaetigung-dialog.component.ts
import { Component, Inject } from '@angular/core';
import { MatDialogModule, MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';

export interface DialogDaten {
  titel: string;
  nachricht: string;
}

@Component({
  selector: 'app-bestaetigung-dialog',
  standalone: true,
  imports: [MatDialogModule, MatButtonModule, MatIconModule],
  template: `
    <h2 mat-dialog-title>
      <mat-icon color="warn">warning</mat-icon>
      {{ data.titel }}
    </h2>
    <mat-dialog-content>
      <p>{{ data.nachricht }}</p>
    </mat-dialog-content>
    <mat-dialog-actions align="end">
      <button mat-button (click)="abbrechen()">
        Abbrechen
      </button>
      <button mat-raised-button color="warn" (click)="bestaetigen()">
        <mat-icon>delete</mat-icon>
        Löschen
      </button>
    </mat-dialog-actions>
  `
})
export class BestaetigungsDialogComponent {
  constructor(
    private dialogRef: MatDialogRef<BestaetigungsDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public data: DialogDaten
  ) {}

  abbrechen(): void {
    this.dialogRef.close(false);  // false = nicht bestätigt
  }

  bestaetigen(): void {
    this.dialogRef.close(true);   // true = bestätigt
  }
}
```

**Dialog in einer Komponente öffnen:**

```typescript
// In der aufrufenden Komponente:
import { MatDialog } from '@angular/material/dialog';
import { BestaetigungsDialogComponent } from '../bestaetigung-dialog/bestaetigung-dialog.component';

export class BenutzerListeComponent {
  constructor(private dialog: MatDialog) {}

  loeschenBestaetigen(benutzer: Benutzer): void {
    const dialogRef = this.dialog.open(BestaetigungsDialogComponent, {
      width: '400px',
      data: {
        titel: 'Benutzer löschen',
        nachricht: `Möchten Sie "${benutzer.name}" wirklich löschen? Diese Aktion kann nicht rückgängig gemacht werden.`
      }
    });

    dialogRef.afterClosed().subscribe(bestaetigt => {
      if (bestaetigt === true) {
        this.loeschen(benutzer.id);
      }
    });
  }
}
```

---

## 18. Checkbox für Boolean-Werte

```typescript
import { MatCheckboxModule } from '@angular/material/checkbox';
import { ReactiveFormsModule, FormControl } from '@angular/forms';
```

### In Formularen (Reactive Forms)

```html
<!-- Editierbare Checkbox im Formular -->
<mat-checkbox formControlName="aktiv">Benutzer ist aktiv</mat-checkbox>

<!-- Deaktivierte Checkbox (nur lesen) -->
<mat-checkbox [checked]="benutzer.aktiv" [disabled]="true">Aktiv</mat-checkbox>
```

### In Tabellen (nur Anzeige)

```html
<!-- Spalte in der Tabelle -->
<ng-container matColumnDef="aktiv">
  <th mat-header-cell *matHeaderCellDef>Aktiv</th>
  <td mat-cell *matCellDef="let row">
    <mat-checkbox
      [checked]="row.aktiv"
      [disabled]="true"
      [color]="row.aktiv ? 'primary' : 'warn'">
    </mat-checkbox>
  </td>
</ng-container>
```

### Alternativ mit Icon

```html
<!-- Mit Material-Icons (visuell deutlicher) -->
<td mat-cell *matCellDef="let row">
  <mat-icon [color]="row.aktiv ? 'primary' : 'warn'">
    {{ row.aktiv ? 'check_circle' : 'cancel' }}
  </mat-icon>
</td>
```

---

## 19. Redirect nach Aktionen

Nach dem Speichern, Löschen oder Abbrechen wird der Benutzer auf eine passende Seite weitergeleitet.

```typescript
import { Router } from '@angular/router';

export class BenutzerFormularComponent {
  constructor(private router: Router) {}

  // Nach Erstellen → zur Detailseite des neuen Eintrags
  nachErstellen(neuerBenutzer: Benutzer): void {
    this.router.navigate(['/benutzer', neuerBenutzer.id]);
  }

  // Nach Bearbeiten → zur Detailseite
  nachBearbeiten(id: number): void {
    this.router.navigate(['/benutzer', id]);
  }

  // Nach Löschen → zurück zur Liste
  nachLoeschen(): void {
    this.router.navigate(['/benutzer']);
  }

  // Abbrechen → zurück zur vorherigen Seite
  abbrechen(): void {
    // Option 1: Zurück in der Browser-History
    window.history.back();

    // Option 2: Explizit zur Liste navigieren
    this.router.navigate(['/benutzer']);
  }
}
```

**Mit returnUrl nach Login weiterleiten:**

```typescript
// login.component.ts
import { ActivatedRoute, Router } from '@angular/router';

export class LoginComponent {
  constructor(
    private authService: AuthService,
    private router: Router,
    private route: ActivatedRoute
  ) {}

  nachLogin(): void {
    // returnUrl aus Query-Parameter lesen
    const returnUrl = this.route.snapshot.queryParams['returnUrl'] || '/dashboard';
    this.router.navigateByUrl(returnUrl);
  }
}
```

---

## 20. Styling in Component.scss

Jede Komponente hat ihre eigene `.scss`-Datei. Angular kapselt die Styles per Default (ViewEncapsulation).

```scss
/* benutzer-liste.component.scss */

/* Variablen (lokal oder in styles.scss global) */
$haupt-farbe: #1976d2;
$warn-farbe: #d32f2f;
$hintergrund: #f5f5f5;
$tabellen-hover: #e3f2fd;

/* Container */
.liste-container {
  padding: 24px;
  background-color: $hintergrund;
  min-height: calc(100vh - 64px);  // Toolbar-Höhe abziehen
}

/* Tabellen-Styling */
.mat-mdc-table {
  width: 100%;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

  // Header-Zeile
  .mat-mdc-header-row {
    background-color: $haupt-farbe;

    .mat-mdc-header-cell {
      color: white;
      font-weight: 600;
    }
  }

  // Daten-Zeilen
  .mat-mdc-row {
    cursor: pointer;
    transition: background-color 0.2s ease;

    &:hover {
      background-color: $tabellen-hover;
    }
  }
}

/* Responsive: auf Mobilgeräten Tabelle scrollbar */
@media (max-width: 600px) {
  .tabellen-wrapper {
    overflow-x: auto;
  }
}
```

```scss
/* styles.scss – globale Styles */
@use '@angular/material' as mat;

// Material-Theme
$app-theme: mat.define-theme((
  color: (
    theme-type: light,
    primary: mat.$azure-palette,
  )
));

:root {
  @include mat.all-component-themes($app-theme);
}

body {
  margin: 0;
  font-family: Roboto, sans-serif;
  background-color: #fafafa;
}

// Globale Utility-Klassen
.spacer { flex: 1 1 auto; }
.text-center { text-align: center; }
.mt-16 { margin-top: 16px; }
.mb-16 { margin-bottom: 16px; }
```

---

## 21. Vollständiges App-Beispiel (Zusammenfassung)

### Projektstruktur

```
src/
├── app/
│   ├── components/
│   │   ├── login/
│   │   │   ├── login.component.ts
│   │   │   └── login.component.scss
│   │   ├── benutzer-liste/
│   │   │   ├── benutzer-liste.component.ts
│   │   │   ├── benutzer-liste.component.html
│   │   │   └── benutzer-liste.component.scss
│   │   ├── benutzer-detail/
│   │   │   ├── benutzer-detail.component.ts
│   │   │   ├── benutzer-detail.component.html
│   │   │   └── benutzer-detail.component.scss
│   │   ├── benutzer-formular/
│   │   │   ├── benutzer-formular.component.ts
│   │   │   ├── benutzer-formular.component.html
│   │   │   └── benutzer-formular.component.scss
│   │   └── bestaetigung-dialog/
│   │       └── bestaetigung-dialog.component.ts
│   ├── guards/
│   │   ├── auth.guard.ts
│   │   └── admin.guard.ts
│   ├── interceptors/
│   │   └── auth.interceptor.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── benutzer.service.ts
│   │   └── jwt.service.ts
│   ├── app.component.ts
│   ├── app.component.html
│   ├── app.component.scss
│   ├── app.config.ts
│   └── app.routes.ts
├── assets/
│   └── silent-check-sso.html        ← Für Keycloak Silent SSO
└── styles.scss
```

### silent-check-sso.html (für Keycloak)

```html
<!-- src/assets/silent-check-sso.html -->
<!doctype html>
<html>
<body>
  <script>
    parent.postMessage(location.href, location.origin);
  </script>
</body>
</html>
```

### Schnell-Checkliste für die Prüfung

| ✅ | Aufgabe |
|---|---|
| ☐ | `ng new` Projekt mit `--style=scss` erstellt |
| ☐ | `ng add @angular/material` ausgeführt |
| ☐ | `npm install keycloak-js` |
| ☐ | `AuthService` mit `keycloak.init()`, `login()`, `logout()`, `isLoggedIn()`, `hatRolle()` |
| ☐ | `JwtService` oder Token-Handling direkt im `AuthService` |
| ☐ | `authInterceptor` fügt Bearer-Token zu HTTP-Requests hinzu |
| ☐ | `authGuard` schützt alle Routen ausser Login |
| ☐ | `app.routes.ts` mit Redirect (`/` → `/dashboard`), geschützten Routen, Catch-All (`**`) |
| ☐ | Toolbar mit Menü-Links und Logout-Button |
| ☐ | Liste mit `mat-table`, Spalten inkl. Boolean-Checkbox, Edit/Delete-Aktionen pro Zeile |
| ☐ | Delete nur sichtbar für `admin`-Rolle (`*ngIf="authService.hatRolle('admin')"`) |
| ☐ | Lösch-Bestätigung mit `MatDialog` |
| ☐ | Detailseite liest Route-Parameter mit `ActivatedRoute` |
| ☐ | Formular-Komponente für Neu+Bearbeiten (erkennt `:id` in Route) |
| ☐ | Feldvalidierung mit `Validators.required`, `Validators.email` usw. |
| ☐ | `router.navigate()` nach Aktionen (Redirect) |
| ☐ | Styling in `.scss`-Dateien pro Komponente |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Routing & Guards](https://angular.dev/guide/routing)
- 📖 [Angular Docs – HTTP Client & Interceptors](https://angular.dev/guide/http)
- 📖 [Angular Docs – Reactive Forms & Validation](https://angular.dev/guide/forms/reactive-forms)
- 📖 [Angular Material Dokumentation](https://material.angular.io/components)
- 📖 [Keycloak JavaScript Adapter](https://www.keycloak.org/docs/latest/securing_apps/#_javascript_adapter)
- 📖 [GitFlow Cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/)
