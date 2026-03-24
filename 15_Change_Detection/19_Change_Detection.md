# 19 – Change Detection

## Lernziele
- Du verstehst wie Angular Change Detection funktioniert
- Du kennst den Unterschied zwischen Default und OnPush Strategy
- Du kannst Change Detection manuell steuern
- Du weisst wie man Performance-Probleme vermeidet

---

## 1. Was ist Change Detection?

**Change Detection** ist der Mechanismus, mit dem Angular erkennt, ob sich Daten geändert haben und ob das Template neu gerendert werden muss.

### Wie funktioniert es?

```
Benutzer klickt Button
         ↓
Angular löst Change Detection aus
         ↓
Angular prüft alle Komponenten von oben nach unten
         ↓
Wenn sich Daten geändert haben → Template wird aktualisiert
```

---

## 2. Default Change Detection Strategy

Standardmässig prüft Angular bei **jedem Ereignis** (Klick, HTTP-Response, Timer, etc.) **alle Komponenten**:

```typescript
// Standard (Default) – Angular prüft immer alles
@Component({
  selector: 'app-standard',
  template: `<p>{{ berechneterWert() }}</p>`
})
export class StandardComponent {
  daten = { name: 'Max' };

  // Diese Methode wird bei JEDER Change Detection aufgerufen!
  berechneterWert(): string {
    console.log('Wird berechnet!'); // Sehr oft!
    return this.daten.name.toUpperCase();
  }
}
```

### Problem bei grossen Apps

```
AppComponent
├── HeaderComponent (hat sich nicht geändert – aber wird trotzdem geprüft)
├── MainComponent
│   ├── ListeComponent (1000 Elemente – alle werden geprüft)
│   └── DetailComponent (hat sich nicht geändert)
└── FooterComponent (hat sich nicht geändert – aber wird geprüft)
```

---

## 3. OnPush Change Detection Strategy

Mit `ChangeDetectionStrategy.OnPush` wird eine Komponente **nur geprüft wenn**:

1. Ein **@Input()-Wert** sich ändert
2. Ein **Event** in der Komponente ausgelöst wird (z.B. Button-Klick)
3. Ein **Observable mit AsyncPipe** einen neuen Wert emittiert
4. Change Detection **manuell** ausgelöst wird

```typescript
import { Component, ChangeDetectionStrategy, Input } from '@angular/core';

@Component({
  selector: 'app-optimiert',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,  // ← OnPush!
  template: `
    <p>{{ produkt.name }}</p>
    <p>{{ produkt.preis }} CHF</p>
  `
})
export class OptimiertComponent {
  @Input() produkt!: { name: string; preis: number };
}
```

---

## 4. Immutability bei OnPush

**Wichtig:** Mit OnPush muss man immer neue Objekte erstellen (Immutability), sonst erkennt Angular keine Änderungen!

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ benutzer.name }}</p>`
})
export class OnPushComponent {
  @Input() benutzer!: { name: string; alter: number };
}

// Eltern-Komponente:
export class ElternComponent {
  benutzer = { name: 'Max', alter: 25 };

  // ❌ FALSCH: Gleiches Objekt wird modifiziert
  // Angular erkennt keine Änderung → Template wird NICHT aktualisiert
  aendereNameFalsch(): void {
    this.benutzer.name = 'Anna'; // Gleiche Objektreferenz!
  }

  // ✅ RICHTIG: Neues Objekt erstellen
  // Angular erkennt neue Referenz → Template wird aktualisiert
  aendereNameRichtig(): void {
    this.benutzer = { ...this.benutzer, name: 'Anna' }; // Neue Referenz!
  }
}
```

---

## 5. ChangeDetectorRef – Manuelle Steuerung

```typescript
import { Component, ChangeDetectorRef, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-manuell',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ timestamp }}</p>`
})
export class ManuellComponent {
  timestamp = '';

  constructor(private cdr: ChangeDetectorRef) {}

  ngOnInit(): void {
    // Dritter-Partei-Bibliothek (kein Angular-Event)
    setTimeout(() => {
      this.timestamp = new Date().toISOString();
      // Angular weiss nicht von dieser Änderung bei OnPush!
      // Manuell Change Detection auslösen:
      this.cdr.detectChanges();
    }, 1000);
  }

  // Weitere CDR-Methoden:
  pausieren(): void {
    this.cdr.detach(); // Change Detection für diese Komponente pausieren
  }

  wiederaufnehmen(): void {
    this.cdr.reattach(); // Change Detection wieder aktivieren
    this.cdr.detectChanges(); // Sofort prüfen
  }

  markieren(): void {
    this.cdr.markForCheck(); // Für nächsten CD-Zyklus markieren
  }
}
```

---

## 6. Signals und Change Detection

Mit **Signals** (Angular 16+) kann Angular noch präziser erkennen was sich geändert hat:

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-signals',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- Angular weiss genau: Nur dieser Teil muss aktualisiert werden -->
    <p>{{ name() }}</p>
    <p>{{ vollstaendigerName() }}</p>
    <button (click)="name.set('Anna')">Name ändern</button>
  `
})
export class SignalsComponent {
  name = signal('Max');
  nachname = signal('Muster');

  // computed wird automatisch bei Änderungen aktualisiert
  vollstaendigerName = computed(() => `${this.name()} ${this.nachname()}`);
}
```

---

## 7. Zone.js und Zoneless

### Zone.js (Standard)

Angular verwendet standardmässig **Zone.js**, das alle asynchronen Operationen (setTimeout, HTTP, Events) überwacht und Change Detection automatisch auslöst.

```
Benutzer klickt → Zone.js erkennt es → Angular löst CD aus
HTTP-Response  → Zone.js erkennt es → Angular löst CD aus
setTimeout()   → Zone.js erkennt es → Angular löst CD aus
```

### Zoneless (Angular 18+ experimentell)

```typescript
// app.config.ts – Zoneless aktivieren (experimentell)
import { provideExperimentalZonelessChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideExperimentalZonelessChangeDetection()
    // Jetzt muss alles mit Signals oder AsyncPipe reaktiv sein
  ]
};
```

---

## 8. Performance-Tipps

### TrackBy bei *ngFor

```html
<!-- ❌ Ohne trackBy: Alle DOM-Elemente werden bei jeder Änderung neu erstellt -->
<li *ngFor="let item of liste">{{ item.name }}</li>

<!-- ✅ Mit trackBy: Nur geänderte Elemente werden aktualisiert -->
<li *ngFor="let item of liste; trackBy: trackById">{{ item.name }}</li>
```

```typescript
trackById(index: number, item: { id: number }): number {
  return item.id;
}
```

### Pure Pipes statt Methoden im Template

```html
<!-- ❌ Methode wird bei jeder CD aufgerufen -->
<p>{{ berechnePreis(produkt) }}</p>

<!-- ✅ Pure Pipe wird nur bei Eingabe-Änderung aufgerufen -->
<p>{{ produkt | preisFormatierung }}</p>
```

### AsyncPipe mit OnPush

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- AsyncPipe löst automatisch CD aus wenn neuer Wert kommt -->
    <div *ngFor="let post of posts$ | async">{{ post.title }}</div>
  `
})
export class PostsComponent {
  posts$ = this.apiService.getAllePosts();
}
```

---

## 9. Überblick: Wann welche Strategie?

| Situation | Empfehlung |
|---|---|
| Einfache Komponenten / Prototyp | `Default` |
| Performance-kritische Komponenten | `OnPush` |
| Komponenten die viele @Inputs bekommen | `OnPush` |
| Listen mit vielen Elementen | `OnPush` + `trackBy` |
| Neue Angular-Apps (Angular 17+) | Signals + `OnPush` |
| Maximale Performance | Zoneless + Signals |

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `Default` | Prüft alle Komponenten bei jedem Event |
| `OnPush` | Prüft nur bei Input-Änderungen oder Events |
| `Immutability` | Neue Objekte statt Mutation (wichtig bei OnPush) |
| `ChangeDetectorRef` | Manuelle CD-Steuerung |
| `detectChanges()` | CD sofort auslösen |
| `markForCheck()` | Für nächsten CD-Zyklus markieren |
| Signals | Präziseste Change Detection (kein Scanning) |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Change Detection](https://angular.dev/guide/components/change-detection)
- 📖 [Angular Docs – OnPush](https://angular.dev/best-practices/skipping-subtrees)
- 🎥 [Angular University – Change Detection](https://angular-university.io/course/angular-change-detection-course)
