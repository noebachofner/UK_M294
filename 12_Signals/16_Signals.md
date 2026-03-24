# 16 – Angular Signals

## Lernziele
- Du verstehst was Signals sind und warum sie eingeführt wurden
- Du kannst `signal()`, `computed()` und `effect()` verwenden
- Du weisst wie Signals Komponenten reaktiv machen ohne RxJS
- Du kennst den Unterschied zwischen Signals und BehaviorSubject

---

## 1. Was sind Signals?

**Signals** sind ein neues Reaktivitätssystem, das in Angular 16 eingeführt und in Angular 17/18 stabilisiert wurde. Sie sind eine einfachere Alternative zu RxJS für das Verwalten von lokalem Komponentenzustand.

### Das Problem mit "klassischem" Angular

```typescript
// Ohne Signals: Angular weiss nicht welche Daten sich geändert haben
// → Prüft bei jeder Änderung den GESAMTEN Komponentenbaum (teuer!)
export class KlassischComponent {
  name = 'Max';
  // Wenn name sich ändert, muss Angular alles neu prüfen
}
```

### Mit Signals

```typescript
// Mit Signals: Angular weiss GENAU was sich geändert hat
// → Nur die betroffenen Teile werden aktualisiert (effizient!)
export class SignalsComponent {
  name = signal('Max'); // Angular weiss: "name" ist reaktiv
  // Wenn name sich ändert, wird nur der betroffene DOM aktualisiert
}
```

---

## 2. signal() – Einfacher reaktiver Wert

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <!-- Signal-Wert lesen: mit () aufrufen -->
    <p>Name: {{ name() }}</p>
    <p>Zähler: {{ zaehler() }}</p>
    <p>Ist aktiv: {{ istAktiv() }}</p>

    <button (click)="erhoehenZaehler()">+1</button>
    <button (click)="toggleAktiv()">Toggle</button>
    <input [value]="name()" (input)="setName($event)">
  `
})
export class DemoComponent {
  // Signal erstellen (mit Anfangswert)
  name = signal('Max');
  zaehler = signal(0);
  istAktiv = signal(true);

  erhoehenZaehler(): void {
    // set(): Wert direkt setzen
    this.zaehler.set(this.zaehler() + 1);

    // update(): Wert basierend auf aktuellem Wert ändern
    this.zaehler.update(aktuell => aktuell + 1);
  }

  toggleAktiv(): void {
    this.istAktiv.update(aktuell => !aktuell);
  }

  setName(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.name.set(input.value);
  }
}
```

---

## 3. computed() – Abgeleitete Werte

`computed()` erstellt ein Signal, das **automatisch aktualisiert** wird, wenn sich seine Abhängigkeiten ändern.

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <p>Vorname: {{ vorname() }}</p>
    <p>Nachname: {{ nachname() }}</p>
    <p>Vollständiger Name: {{ vollstaendigerName() }}</p>
    <!-- Wird automatisch aktualisiert wenn vorname oder nachname ändert -->

    <p>Warenkorb-Summe: {{ warenkorbSumme() }} CHF</p>

    <button (click)="vorname.set('Anna')">Vorname ändern</button>
  `
})
export class DemoComponent {
  vorname = signal('Max');
  nachname = signal('Muster');

  // computed: Wird automatisch neu berechnet wenn Abhängigkeiten ändern
  vollstaendigerName = computed(() => `${this.vorname()} ${this.nachname()}`);

  // Komplexeres computed
  warenkorb = signal([
    { name: 'Laptop', preis: 999 },
    { name: 'Maus', preis: 29 },
  ]);

  warenkorbSumme = computed(() =>
    this.warenkorb().reduce((sum, item) => sum + item.preis, 0)
  );

  warenkorbAnzahl = computed(() => this.warenkorb().length);
}
```

---

## 4. effect() – Nebeneffekte bei Signal-Änderungen

`effect()` führt eine Funktion aus, **jedes Mal wenn sich ein Signal ändert**.

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <p>Zaehler: {{ zaehler() }}</p>
    <button (click)="zaehler.update(n => n + 1)">+1</button>
  `
})
export class DemoComponent {
  zaehler = signal(0);
  verdoppelt = computed(() => this.zaehler() * 2);

  constructor() {
    // Effect läuft automatisch wenn zaehler oder verdoppelt sich ändert
    effect(() => {
      console.log(`Zähler: ${this.zaehler()}, Verdoppelt: ${this.verdoppelt()}`);
      // Hier kann man z.B. localStorage aktualisieren, Analytics senden, etc.
    });

    // Praktisches Beispiel: In localStorage speichern
    effect(() => {
      localStorage.setItem('zaehler', String(this.zaehler()));
    });
  }
}
```

---

## 5. Signals mit Objekten und Arrays

```typescript
import { Component, signal, computed } from '@angular/core';

interface Produkt {
  id: number;
  name: string;
  preis: number;
}

@Component({
  selector: 'app-produkte',
  standalone: true,
  template: `
    <p>Anzahl: {{ produkte().length }}</p>
    <ul>
      @for (p of produkte(); track p.id) {
        <li>{{ p.name }} – {{ p.preis }} CHF</li>
      }
    </ul>
    <p>Gesamt: {{ gesamtPreis() }} CHF</p>
    <button (click)="hinzufuegen()">Produkt hinzufügen</button>
  `
})
export class ProdukteComponent {
  produkte = signal<Produkt[]>([
    { id: 1, name: 'Laptop', preis: 999 },
    { id: 2, name: 'Maus', preis: 29 },
  ]);

  gesamtPreis = computed(() =>
    this.produkte().reduce((sum, p) => sum + p.preis, 0)
  );

  hinzufuegen(): void {
    // WICHTIG: Immer neues Array erstellen (Immutability!)
    this.produkte.update(liste => [
      ...liste,
      { id: Date.now(), name: 'Monitor', preis: 299 }
    ]);
  }

  loeschen(id: number): void {
    this.produkte.update(liste => liste.filter(p => p.id !== id));
  }

  preisAendern(id: number, neuerPreis: number): void {
    this.produkte.update(liste =>
      liste.map(p => p.id === id ? { ...p, preis: neuerPreis } : p)
    );
  }
}
```

---

## 6. input() und output() als Signals (Angular 17+)

```typescript
import { Component, input, output, computed } from '@angular/core';

@Component({
  selector: 'app-kind',
  standalone: true,
  template: `
    <p>{{ produktName() }}</p>
    <p>Mit Steuer: {{ preisNettoMitSteuer() }}</p>
    <button (click)="kaufen.emit(produktName())">Kaufen</button>
  `
})
export class KindComponent {
  // input() als Signal (Pflichtfeld)
  produktName = input.required<string>();
  produktPreis = input<number>(0);

  // Abgeleitetes Signal
  preisNettoMitSteuer = computed(() => this.produktPreis() * 1.077);

  // output() ersetzt @Output() + EventEmitter
  kaufen = output<string>();
}
```

```typescript
// Eltern-Komponente
@Component({
  selector: 'app-eltern',
  standalone: true,
  imports: [KindComponent],
  template: `
    <app-kind
      [produktName]="'Laptop'"
      [produktPreis]="999"
      (kaufen)="onKauf($event)">
    </app-kind>
  `
})
export class ElternComponent {
  onKauf(name: string): void {
    console.log('Gekauft:', name);
  }
}
```

---

## 7. toSignal() – Observable zu Signal

```typescript
import { Component, inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <!-- kein | async nötig! -->
    <p>Anzahl Posts: {{ posts()?.length }}</p>
    @for (post of posts() ?? []; track post.id) {
      <p>{{ post.title }}</p>
    }
  `
})
export class DemoComponent {
  private http = inject(HttpClient);

  // Observable zu Signal konvertieren
  posts = toSignal(
    this.http.get<any[]>('https://jsonplaceholder.typicode.com/posts')
  );
}
```

---

## 8. Signals vs. RxJS – Wann was verwenden?

| Situation | Signals | RxJS |
|---|---|---|
| Lokaler Komponenten-State | ✅ Empfohlen | ✅ Möglich |
| Einfache reaktive Werte | ✅ Einfacher | ⚠️ Komplexer |
| HTTP-Anfragen | ⚠️ Mit `toSignal()` | ✅ Empfohlen |
| Komplexe Streams/Transformationen | ❌ | ✅ Empfohlen |
| Zeitbasierte Operationen | ❌ | ✅ |
| Multi-Step-Workflows | ❌ | ✅ |

> 💡 **Empfehlung:** Signals für lokalen State in Komponenten, RxJS für HTTP und komplexe Datenströme.

---

## Zusammenfassung

| API | Beschreibung |
|---|---|
| `signal(wert)` | Reaktiven Wert erstellen |
| `signal()` | Signal-Wert lesen |
| `signal.set(wert)` | Signal-Wert setzen |
| `signal.update(fn)` | Signal-Wert basierend auf aktuellem Wert ändern |
| `computed(() => ...)` | Abgeleiteten Signal-Wert berechnen |
| `effect(() => ...)` | Nebeneffekte bei Signal-Änderungen |
| `input()` | @Input als Signal (Angular 17+) |
| `output()` | @Output als Signal (Angular 17+) |
| `toSignal(obs$)` | Observable in Signal umwandeln |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Signals](https://angular.dev/guide/signals)
- 📖 [Angular Docs – Signal Inputs](https://angular.dev/guide/components/inputs#signal-inputs)
- 🎥 [Angular University – Angular Signals](https://angular-university.io/course/angular-signals-course)
