# 15 – Testing

## Lernziele
- Du verstehst die Grundlagen des Unit-Testings in Angular
- Du kannst Komponenten und Services testen
- Du kennst die wichtigsten Jasmine und Angular Testing Utilities
- Du kannst HTTP-Calls in Tests mocken

---

## 1. Grundlagen des Testings

### Testing-Pyramide

```
    /    E2E Tests    \      ← Wenige, langsam, teuer
   /  Integration Tests \   ← Mittel
  /    Unit Tests        \  ← Viele, schnell, günstig
```

### Test-Typen in Angular

| Typ | Tool | Beschreibung |
|---|---|---|
| **Unit Tests** | Jasmine + Karma | Einzelne Klassen isoliert testen |
| **Integration Tests** | TestBed | Komponenten mit Template testen |
| **E2E Tests** | Cypress / Playwright | Vollständige User-Flows testen |

### Tests ausführen

```bash
# Unit Tests (einmalig)
ng test

# Unit Tests (Watch-Modus – läuft kontinuierlich)
ng test --watch

# Mit Code Coverage
ng test --code-coverage
```

---

## 2. Jasmine Grundlagen

```typescript
// Struktur eines Tests
describe('Mein Test Suite', () => {
  // Einmaliges Setup vor ALLEN Tests
  beforeAll(() => {
    console.log('Einmalig vor allen Tests');
  });

  // Setup vor JEDEM Test
  beforeEach(() => {
    console.log('Vor jedem Test');
  });

  // Aufräumen nach JEDEM Test
  afterEach(() => {
    console.log('Nach jedem Test');
  });

  // Einmaliges Aufräumen nach ALLEN Tests
  afterAll(() => {
    console.log('Einmalig nach allen Tests');
  });

  // Einzelner Test
  it('sollte 2 + 2 = 4 sein', () => {
    expect(2 + 2).toBe(4);
  });

  it('sollte wahr sein', () => {
    expect(true).toBeTruthy();
  });
});
```

### Wichtige Matcher (Expects)

```typescript
// Gleichheit
expect(wert).toBe(42);              // Strenge Gleichheit (===)
expect(wert).toEqual({ a: 1 });     // Tiefe Gleichheit

// Wahrheitswerte
expect(wert).toBeTruthy();
expect(wert).toBeFalsy();
expect(wert).toBeNull();
expect(wert).toBeUndefined();
expect(wert).toBeDefined();

// Zahlen
expect(zahl).toBeGreaterThan(5);
expect(zahl).toBeLessThan(10);
expect(zahl).toBeCloseTo(3.14, 2);  // Näherungsweise (2 Dezimalstellen)

// Strings
expect(text).toContain('hallo');
expect(text).toMatch(/^[A-Z]/);     // RegEx

// Arrays
expect(array).toContain('item');
expect(array.length).toBe(3);

// Exceptions
expect(() => fehlerFunktion()).toThrow();
expect(() => fehlerFunktion()).toThrowError('Fehlermeldung');

// Negieren mit .not
expect(wert).not.toBe(0);
expect(array).not.toContain('verboten');
```

---

## 3. Service testen

```typescript
// services/rechner.service.ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class RechnerService {
  addieren(a: number, b: number): number {
    return a + b;
  }

  dividieren(a: number, b: number): number {
    if (b === 0) throw new Error('Division durch Null!');
    return a / b;
  }
}
```

```typescript
// services/rechner.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { RechnerService } from './rechner.service';

describe('RechnerService', () => {
  let service: RechnerService;

  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(RechnerService);
  });

  it('sollte erstellt werden', () => {
    expect(service).toBeTruthy();
  });

  it('sollte 2 + 3 = 5 berechnen', () => {
    const ergebnis = service.addieren(2, 3);
    expect(ergebnis).toBe(5);
  });

  it('sollte negative Zahlen addieren', () => {
    expect(service.addieren(-5, 3)).toBe(-2);
  });

  it('sollte 10 / 2 = 5 berechnen', () => {
    expect(service.dividieren(10, 2)).toBe(5);
  });

  it('sollte Fehler bei Division durch Null werfen', () => {
    expect(() => service.dividieren(10, 0)).toThrowError('Division durch Null!');
  });
});
```

---

## 4. Komponente testen

```typescript
// demo.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <h1>{{ titel }}</h1>
    <button (click)="klick()">Klick mich</button>
    <p *ngIf="geklickt">Button wurde geklickt!</p>
  `
})
export class DemoComponent {
  titel = 'Meine Demo';
  geklickt = false;
  anzahlKlicks = 0;

  klick(): void {
    this.geklickt = true;
    this.anzahlKlicks++;
  }
}
```

```typescript
// demo.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { DemoComponent } from './demo.component';
import { NgIf } from '@angular/common';

describe('DemoComponent', () => {
  let component: DemoComponent;
  let fixture: ComponentFixture<DemoComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [DemoComponent]  // Standalone-Komponente importieren
    }).compileComponents();

    fixture = TestBed.createComponent(DemoComponent);
    component = fixture.componentInstance;
    fixture.detectChanges(); // Initiale Change Detection
  });

  it('sollte erstellt werden', () => {
    expect(component).toBeTruthy();
  });

  it('sollte den Titel anzeigen', () => {
    const h1 = fixture.nativeElement.querySelector('h1');
    expect(h1.textContent).toBe('Meine Demo');
  });

  it('sollte geklickt anfangs false sein', () => {
    expect(component.geklickt).toBeFalse();
  });

  it('sollte geklickt auf true setzen nach Button-Klick', () => {
    const button = fixture.nativeElement.querySelector('button');
    button.click();
    expect(component.geklickt).toBeTrue();
  });

  it('sollte Meldung anzeigen nach Klick', () => {
    // Vor dem Klick: kein p-Element
    expect(fixture.nativeElement.querySelector('p')).toBeNull();

    // Button klicken
    component.klick();
    fixture.detectChanges(); // View aktualisieren

    // Nach dem Klick: p-Element vorhanden
    const paragraph = fixture.nativeElement.querySelector('p');
    expect(paragraph).toBeTruthy();
    expect(paragraph.textContent).toContain('Button wurde geklickt!');
  });

  it('sollte Klick-Anzahl erhöhen', () => {
    component.klick();
    component.klick();
    component.klick();
    expect(component.anzahlKlicks).toBe(3);
  });
});
```

---

## 5. Komponente mit Service testen (Mock)

```typescript
// produkt-liste.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ProduktListeComponent } from './produkt-liste.component';
import { ProduktService } from '../services/produkt.service';

describe('ProduktListeComponent', () => {
  let component: ProduktListeComponent;
  let fixture: ComponentFixture<ProduktListeComponent>;
  let mockProduktService: jasmine.SpyObj<ProduktService>;

  beforeEach(async () => {
    // Mock-Service erstellen
    mockProduktService = jasmine.createSpyObj('ProduktService', [
      'getAlleProdukte',
      'deleteProdukt'
    ]);

    // Mock-Rückgabewert konfigurieren
    mockProduktService.getAlleProdukte.and.returnValue([
      { id: 1, name: 'Laptop', preis: 999 },
      { id: 2, name: 'Maus', preis: 29 },
    ]);

    await TestBed.configureTestingModule({
      imports: [ProduktListeComponent],
      providers: [
        // Echten Service durch Mock ersetzen
        { provide: ProduktService, useValue: mockProduktService }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(ProduktListeComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('sollte Produkte vom Service laden', () => {
    expect(mockProduktService.getAlleProdukte).toHaveBeenCalled();
    expect(component.produkte.length).toBe(2);
  });

  it('sollte Produkte im Template anzeigen', () => {
    const produktElemente = fixture.nativeElement.querySelectorAll('.produkt');
    expect(produktElemente.length).toBe(2);
  });

  it('sollte deleteProdukt aufrufen beim Löschen', () => {
    component.loeschen(1);
    expect(mockProduktService.deleteProdukt).toHaveBeenCalledWith(1);
  });
});
```

---

## 6. HTTP-Service testen

```typescript
// services/api.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { ApiService } from './api.service';
import { Post } from '../models/post.interface';

describe('ApiService', () => {
  let service: ApiService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],  // Echter HTTP wird gemockt!
    });
    service = TestBed.inject(ApiService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify(); // Sicherstellen, dass keine unerwarteten Requests gemacht wurden
  });

  it('sollte alle Posts laden', () => {
    const mockPosts: Post[] = [
      { id: 1, userId: 1, title: 'Test Post', body: 'Inhalt' }
    ];

    // Service-Methode aufrufen
    service.getAllePosts().subscribe(posts => {
      expect(posts.length).toBe(1);
      expect(posts[0].title).toBe('Test Post');
    });

    // HTTP-Request abfangen und Mock-Antwort zurückgeben
    const req = httpMock.expectOne('https://jsonplaceholder.typicode.com/posts');
    expect(req.request.method).toBe('GET');
    req.flush(mockPosts); // Mock-Antwort senden
  });

  it('sollte einen neuen Post erstellen', () => {
    const neuerPost = { userId: 1, title: 'Neuer Post', body: 'Inhalt' };
    const antwortPost = { id: 101, ...neuerPost };

    service.createPost(neuerPost).subscribe(post => {
      expect(post.id).toBe(101);
    });

    const req = httpMock.expectOne('https://jsonplaceholder.typicode.com/posts');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(neuerPost);
    req.flush(antwortPost);
  });

  it('sollte Fehler behandeln', () => {
    service.getAllePosts().subscribe({
      next: () => fail('Sollte einen Fehler werfen'),
      error: (err) => {
        expect(err.message).toBeDefined();
      }
    });

    const req = httpMock.expectOne('https://jsonplaceholder.typicode.com/posts');
    req.flush('Fehler', { status: 500, statusText: 'Server Error' });
  });
});
```

---

## 7. Tipps für gutes Testing

### ARRANGE – ACT – ASSERT Muster

```typescript
it('sollte neuen Todo hinzufügen', () => {
  // ARRANGE – Vorbereitung
  const service = TestBed.inject(TodoService);
  const anfangsAnzahl = service.getAlle().length;

  // ACT – Aktion ausführen
  service.add('Neuer Todo');

  // ASSERT – Ergebnis prüfen
  expect(service.getAlle().length).toBe(anfangsAnzahl + 1);
  expect(service.getAlle()[anfangsAnzahl].text).toBe('Neuer Todo');
});
```

### Was NICHT in Unit Tests gehört:
- Echte HTTP-Anfragen (durch `HttpClientTestingModule` ersetzen)
- Echte Datenbank-Verbindungen
- Externe APIs

### Code Coverage

```bash
ng test --code-coverage
# Erstellt: coverage/meine-app/index.html
# Öffnen für detaillierten Coverage-Report
```

---

## 8. Fixture und DebugElement

```typescript
it('sollte mit DebugElement arbeiten', () => {
  const fixture = TestBed.createComponent(DemoComponent);
  fixture.detectChanges();

  // Elemente über DebugElement finden
  const debugElement = fixture.debugElement;
  const button = debugElement.query(By.css('button'));
  const alle_buttons = debugElement.queryAll(By.css('button'));

  // Klick simulieren
  button.nativeElement.click();
  fixture.detectChanges();

  // Text prüfen
  const paragraph = debugElement.query(By.css('p'));
  expect(paragraph.nativeElement.textContent).toContain('Text');
});
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `TestBed` | Angular Test-Umgebung konfigurieren |
| `ComponentFixture` | Wrapper für die getestete Komponente |
| `fixture.detectChanges()` | Change Detection auslösen |
| `jasmine.createSpyObj()` | Mock-Objekte erstellen |
| `spy.and.returnValue()` | Mock-Rückgabewert setzen |
| `HttpClientTestingModule` | HTTP-Requests mocken |
| `HttpTestingController` | HTTP-Requests abfangen |
| `describe/it` | Test-Suites und Tests definieren |
| `beforeEach/afterEach` | Setup und Teardown |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Testing](https://angular.dev/guide/testing)
- 📖 [Jasmine Dokumentation](https://jasmine.github.io)
- 🎥 [Angular University – Angular Testing](https://angular-university.io/course/angular-testing-course)
