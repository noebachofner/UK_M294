# 11 – HTTP Client & RxJS

## Lernziele
- Du kannst HTTP-Anfragen mit dem Angular HttpClient durchführen
- Du verstehst Observables und die Grundlagen von RxJS
- Du kannst mit GET, POST, PUT, DELETE arbeiten
- Du kannst HTTP-Fehler behandeln

---

## 1. HTTP Client Setup

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()  // HttpClient aktivieren!
  ]
};
```

---

## 2. Was sind Observables und RxJS?

**RxJS** (Reactive Extensions for JavaScript) ist eine Library für **reaktive Programmierung** mit Observables.

### Observable vs. Promise

| | Promise | Observable |
|---|---|---|
| **Werte** | Einmal | Mehrmals |
| **Lazy** | Nein | Ja (startet erst bei subscribe) |
| **Abbrechen** | Nicht möglich | Möglich (unsubscribe) |
| **Operatoren** | Wenige | Viele (map, filter, etc.) |
| **Angular HTTP** | Nein | Ja |

```typescript
// Observable – vereinfachtes Konzept
const observable$ = new Observable<number>(subscriber => {
  subscriber.next(1);   // Wert senden
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete(); // Fertig
});

// Abonnieren (subscribe) startet den Datenstrom
observable$.subscribe({
  next: (wert) => console.log(wert),   // 1, 2, 3
  error: (fehler) => console.error(fehler),
  complete: () => console.log('Fertig!')
});
```

---

## 3. Interface für API-Daten definieren

```typescript
// models/post.interface.ts
export interface Post {
  id: number;
  userId: number;
  title: string;
  body: string;
}

export interface Benutzer {
  id: number;
  name: string;
  email: string;
  phone: string;
}
```

---

## 4. HTTP Service erstellen

```typescript
// services/api.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, map } from 'rxjs/operators';
import { Post } from '../models/post.interface';

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private apiUrl = 'https://jsonplaceholder.typicode.com';

  constructor(private http: HttpClient) {}

  // GET – Alle Posts laden
  getAllePosts(): Observable<Post[]> {
    return this.http.get<Post[]>(`${this.apiUrl}/posts`);
  }

  // GET – Einzelnen Post laden
  getPostById(id: number): Observable<Post> {
    return this.http.get<Post>(`${this.apiUrl}/posts/${id}`);
  }

  // POST – Neuen Post erstellen
  createPost(post: Omit<Post, 'id'>): Observable<Post> {
    return this.http.post<Post>(`${this.apiUrl}/posts`, post);
  }

  // PUT – Post vollständig aktualisieren
  updatePost(id: number, post: Post): Observable<Post> {
    return this.http.put<Post>(`${this.apiUrl}/posts/${id}`, post);
  }

  // PATCH – Post teilweise aktualisieren
  patchPost(id: number, aenderungen: Partial<Post>): Observable<Post> {
    return this.http.patch<Post>(`${this.apiUrl}/posts/${id}`, aenderungen);
  }

  // DELETE – Post löschen
  deletePost(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/posts/${id}`);
  }
}
```

---

## 5. HTTP-Aufrufe in Komponenten verwenden

```typescript
// posts.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { NgFor, NgIf } from '@angular/common';
import { Subscription } from 'rxjs';
import { ApiService } from '../services/api.service';
import { Post } from '../models/post.interface';

@Component({
  selector: 'app-posts',
  standalone: true,
  imports: [NgFor, NgIf],
  template: `
    <div *ngIf="isLoading">Lädt...</div>
    <div *ngIf="fehler" class="fehler">{{ fehler }}</div>

    <div *ngIf="!isLoading && !fehler">
      <h1>Posts ({{ posts.length }})</h1>
      <article *ngFor="let post of posts">
        <h3>{{ post.title }}</h3>
        <p>{{ post.body }}</p>
        <button (click)="loeschen(post.id)">Löschen</button>
      </article>
    </div>
  `
})
export class PostsComponent implements OnInit, OnDestroy {
  posts: Post[] = [];
  isLoading = false;
  fehler = '';

  private subscription!: Subscription;

  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.laden();
  }

  laden(): void {
    this.isLoading = true;
    this.fehler = '';

    this.subscription = this.apiService.getAllePosts().subscribe({
      next: (posts) => {
        this.posts = posts;
        this.isLoading = false;
      },
      error: (err) => {
        this.fehler = 'Fehler beim Laden der Posts: ' + err.message;
        this.isLoading = false;
      }
    });
  }

  loeschen(id: number): void {
    this.apiService.deletePost(id).subscribe({
      next: () => {
        this.posts = this.posts.filter(p => p.id !== id);
        console.log('Post gelöscht!');
      },
      error: (err) => console.error('Fehler:', err)
    });
  }

  ngOnDestroy(): void {
    // Wichtig! Subscription aufräumen um Memory Leaks zu vermeiden
    this.subscription?.unsubscribe();
  }
}
```

---

## 6. AsyncPipe – Direktes Verwenden von Observables im Template

Die `AsyncPipe` abonniert automatisch ein Observable und gibt den letzten Wert zurück. Sie kündigt das Abonnement auch automatisch beim Zerstören der Komponente.

```typescript
import { Component } from '@angular/core';
import { AsyncPipe, NgFor, NgIf } from '@angular/common';
import { Observable } from 'rxjs';
import { ApiService } from '../services/api.service';
import { Post } from '../models/post.interface';

@Component({
  selector: 'app-posts-async',
  standalone: true,
  imports: [AsyncPipe, NgFor, NgIf],
  template: `
    <!-- AsyncPipe abonniert das Observable automatisch -->
    <ng-container *ngIf="posts$ | async as posts; else laden">
      <article *ngFor="let post of posts">
        <h3>{{ post.title }}</h3>
      </article>
    </ng-container>

    <ng-template #laden>
      <p>Lädt...</p>
    </ng-template>
  `
})
export class PostsAsyncComponent {
  // Observable – kein subscribe() nötig!
  posts$: Observable<Post[]>;

  constructor(private apiService: ApiService) {
    this.posts$ = this.apiService.getAllePosts();
  }
}
```

> 💡 **Empfehlung:** Die `AsyncPipe` ist oft bevorzugt, weil sie automatisch aufräumt und kein manuelles `unsubscribe()` benötigt.

---

## 7. Wichtige RxJS Operatoren

RxJS-Operatoren werden mit `pipe()` verkettet:

```typescript
import { map, filter, switchMap, catchError, tap, debounceTime, distinctUntilChanged } from 'rxjs/operators';
import { of } from 'rxjs';

// map – Werte transformieren
this.apiService.getAllePosts().pipe(
  map(posts => posts.filter(p => p.userId === 1)), // Nur Posts von User 1
  map(posts => posts.map(p => p.title))            // Nur Titel
).subscribe(titel => console.log(titel));

// filter – Werte filtern
source$.pipe(
  filter(wert => wert > 0)  // Nur positive Zahlen
)

// switchMap – Observable in anderes umwandeln (z.B. bei Route-Parameter)
this.route.paramMap.pipe(
  map(params => Number(params.get('id'))),
  switchMap(id => this.apiService.getPostById(id)) // HTTP-Call pro ID
).subscribe(post => this.post = post);

// catchError – Fehler behandeln
this.apiService.getAllePosts().pipe(
  catchError(err => {
    console.error('Fehler:', err);
    return of([]);  // Leeres Array zurückgeben bei Fehler
  })
)

// tap – Nebeneffekte (z.B. Logging, kein Datenstrom-Eingriff)
this.apiService.getAllePosts().pipe(
  tap(posts => console.log('Geladen:', posts.length)),
  map(posts => posts.slice(0, 10))
)

// debounceTime – Wartet nach letzter Eingabe (für Suche)
searchInput.valueChanges.pipe(
  debounceTime(300),                  // Warte 300ms
  distinctUntilChanged(),             // Nur wenn sich der Wert ändert
  switchMap(term => this.apiService.suchen(term))
)
```

---

## 8. Fehlerbehandlung

```typescript
import { HttpErrorResponse } from '@angular/common/http';
import { catchError, throwError } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}

  getAllePosts(): Observable<Post[]> {
    return this.http.get<Post[]>('/api/posts').pipe(
      catchError(this.handleError)
    );
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    let fehlermeldung = 'Ein Fehler ist aufgetreten.';

    if (error.status === 0) {
      // Client-seitiger Fehler (Netzwerk)
      fehlermeldung = 'Netzwerkfehler. Bitte Internet-Verbindung prüfen.';
    } else if (error.status === 401) {
      fehlermeldung = 'Nicht autorisiert. Bitte anmelden.';
    } else if (error.status === 403) {
      fehlermeldung = 'Zugriff verweigert.';
    } else if (error.status === 404) {
      fehlermeldung = 'Ressource nicht gefunden.';
    } else if (error.status >= 500) {
      fehlermeldung = 'Server-Fehler. Bitte später erneut versuchen.';
    }

    console.error(`HTTP Fehler ${error.status}:`, error.message);
    return throwError(() => new Error(fehlermeldung));
  }
}
```

---

## 9. HTTP Headers & Query-Parameter

```typescript
import { HttpHeaders, HttpParams } from '@angular/common/http';

// Mit Headers (z.B. Auth-Token)
const headers = new HttpHeaders({
  'Authorization': `Bearer ${this.authToken}`,
  'Content-Type': 'application/json'
});

this.http.get<Post[]>('/api/posts', { headers });

// Mit Query-Parametern
// URL wird: /api/posts?page=1&limit=10&sort=date
const params = new HttpParams()
  .set('page', '1')
  .set('limit', '10')
  .set('sort', 'date');

this.http.get<Post[]>('/api/posts', { params });
```

---

## 10. HTTP Interceptors

Interceptors "fangen" alle HTTP-Anfragen ab und können sie modifizieren.

```typescript
// interceptors/auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token');

  if (token) {
    // Auth-Header zu allen Anfragen hinzufügen
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
    provideHttpClient(withInterceptors([authInterceptor]))
  ]
};
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `HttpClient` | Service für HTTP-Anfragen |
| `Observable` | Datenstrom (mehrere Werte über Zeit) |
| `subscribe()` | Observable abonnieren |
| `AsyncPipe` | Observable direkt im Template verwenden |
| `pipe()` | RxJS-Operatoren verketten |
| `map` | Werte transformieren |
| `catchError` | Fehler behandeln |
| `switchMap` | Observable in anderes umwandeln |
| `debounceTime` | Verzögerung bei Eingaben |
| Interceptors | HTTP-Anfragen abfangen und modifizieren |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – HTTP Client](https://angular.dev/guide/http)
- 📖 [RxJS Dokumentation](https://rxjs.dev)
- 🎥 [Angular University – RxJS & Reactive Angular](https://angular-university.io/course/rxjs-course)
- 🛠️ [RxJS Visualizer (interaktiv)](https://rxviz.com)
- 🛠️ [JSONPlaceholder – Kostenlose Test-API](https://jsonplaceholder.typicode.com)
