# 12 – Pipes

## Lernziele
- Du kennst die wichtigsten eingebauten Pipes in Angular
- Du kannst Pipes in Templates verwenden und kombinieren
- Du kannst eigene Pipes erstellen

---

## 1. Was sind Pipes?

**Pipes** transformieren Daten **zur Anzeige** im Template, ohne die eigentlichen Daten zu verändern.

```
Originaldaten → Pipe → Transformierte Anzeige
1234.5       → currency | 'CHF' → 'CHF 1'234.50'
'hallo'      → uppercase        → 'HALLO'
new Date()   → date | 'dd.MM.yyyy' → '24.03.2024'
```

### Syntax

```html
{{ wert | pipeName }}
{{ wert | pipeName:parameter1:parameter2 }}
{{ wert | pipe1 | pipe2 }}  <!-- Pipes verketten -->
```

---

## 2. Eingebaute Pipes

### DatePipe – Datum formatieren

```html
{{ datum | date }}                     <!-- Mar 24, 2024 -->
{{ datum | date:'dd.MM.yyyy' }}        <!-- 24.03.2024 -->
{{ datum | date:'full' }}              <!-- Sunday, March 24, 2024 -->
{{ datum | date:'HH:mm' }}             <!-- 14:30 -->
{{ datum | date:'dd.MM.yyyy HH:mm' }}  <!-- 24.03.2024 14:30 -->
{{ datum | date:'short' }}             <!-- 3/24/24, 2:30 PM -->
```

```typescript
export class DemoComponent {
  datum = new Date();
  anderesDatum = new Date('2024-12-24');
}
```

### CurrencyPipe – Währung formatieren

```html
{{ preis | currency }}                     <!-- $999.99 (Standard: USD) -->
{{ preis | currency:'CHF' }}               <!-- CHF999.99 -->
{{ preis | currency:'CHF':'symbol' }}      <!-- CHF999.99 -->
{{ preis | currency:'CHF':'symbol':'1.2-2' }}  <!-- CHF999.99 -->
{{ preis | currency:'EUR':'code':'1.0-0' }}    <!-- EUR1000 -->
```

### DecimalPipe – Zahlen formatieren

```html
<!-- Format: 'minGanzzahlen.minNachkomma-maxNachkomma' -->
{{ 1234.567 | number }}              <!-- 1,234.567 -->
{{ 1234.567 | number:'1.2-2' }}      <!-- 1,234.57 -->
{{ 1234 | number:'5.0-0' }}          <!-- 01,234 -->
{{ 0.752 | number:'1.0-0' }}         <!-- 1 (gerundet) -->
```

### PercentPipe – Prozent formatieren

```html
{{ 0.25 | percent }}          <!-- 25% -->
{{ 0.75 | percent:'1.1-2' }}  <!-- 75.0% -->
{{ 1.5 | percent }}           <!-- 150% -->
```

### UpperCasePipe / LowerCasePipe / TitleCasePipe

```html
{{ 'hallo welt' | uppercase }}    <!-- HALLO WELT -->
{{ 'HALLO WELT' | lowercase }}    <!-- hallo welt -->
{{ 'hallo welt' | titlecase }}    <!-- Hallo Welt -->
```

### SlicePipe – Arrays/Strings kürzen

```html
<!-- String kürzen -->
{{ 'Langer Text der gekürzt wird' | slice:0:10 }}  <!-- Langer Tex -->

<!-- Array kürzen -->
{{ [1,2,3,4,5] | slice:1:4 }}  <!-- [2,3,4] -->
{{ [1,2,3,4,5] | slice:-2 }}   <!-- [4,5] (letzte 2) -->
```

### JsonPipe – Objekte als JSON anzeigen (zum Debuggen)

```html
{{ objekt | json }}
<!-- Output: { "name": "Max", "alter": 25 } -->

<pre>{{ komplexesObjekt | json }}</pre>
```

### KeyValuePipe – Objekt-Einträge itererieren

```html
<div *ngFor="let item of objekt | keyvalue">
  {{ item.key }}: {{ item.value }}
</div>
```

```typescript
objekt = { name: 'Max', alter: 25, stadt: 'Zürich' };
// Gibt aus: name: Max, alter: 25, stadt: Zürich
```

### AsyncPipe – Observables/Promises im Template

```html
<!-- Observable direkt im Template verwenden -->
<p>{{ observable$ | async }}</p>

<!-- Mit NgIf kombinieren -->
<div *ngIf="daten$ | async as daten">
  {{ daten | json }}
</div>
```

---

## 3. Pipes kombinieren (Chaining)

```html
<!-- Mehrere Pipes hintereinander -->
{{ 'hallo welt' | titlecase | slice:0:5 }}
<!-- Ergebnis: Hallo -->

{{ datum | date:'dd.MM.yyyy' | uppercase }}
<!-- Ergebnis: 24.03.2024 (uppercase hat hier keinen Effekt) -->

<!-- Mit langen Texten -->
<p>{{ produktBeschreibung | slice:0:100 | lowercase }}</p>
```

---

## 4. Eigene Pipe erstellen

### Simple Pipe

```bash
ng generate pipe pipes/kurzen
# oder:
ng g p pipes/kurzen
```

```typescript
// pipes/kurzen.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'kurzen',
  standalone: true
})
export class KurzenPipe implements PipeTransform {
  transform(wert: string, maxLaenge: number = 100, suffix: string = '...'): string {
    if (!wert) return '';
    if (wert.length <= maxLaenge) return wert;
    return wert.substring(0, maxLaenge).trim() + suffix;
  }
}
```

```typescript
// Verwendung
import { Component } from '@angular/core';
import { KurzenPipe } from '../pipes/kurzen.pipe';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [KurzenPipe],
  template: `
    <p>{{ langer_text | kurzen }}</p>
    <p>{{ langer_text | kurzen:50 }}</p>
    <p>{{ langer_text | kurzen:30:'...' }}</p>
  `
})
export class DemoComponent {
  langer_text = 'Dieser Text ist sehr lang und soll nach einer gewissen Anzahl von Zeichen abgeschnitten werden.';
}
```

### Pipe mit Interface (Komplexerer Beispiel)

```typescript
// pipes/filter.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter',
  standalone: true
})
export class FilterPipe implements PipeTransform {
  transform<T extends Record<string, any>>(
    items: T[],
    suchbegriff: string,
    feld: keyof T
  ): T[] {
    if (!items || !suchbegriff) return items;

    const suchbegriffKlein = suchbegriff.toLowerCase();
    return items.filter(item =>
      String(item[feld]).toLowerCase().includes(suchbegriffKlein)
    );
  }
}
```

```typescript
// Verwendung:
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { NgFor } from '@angular/common';
import { FilterPipe } from '../pipes/filter.pipe';

@Component({
  selector: 'app-suche',
  standalone: true,
  imports: [FormsModule, NgFor, FilterPipe],
  template: `
    <input [(ngModel)]="suchbegriff" placeholder="Suchen...">
    <ul>
      <!-- Filtert die Produkte nach 'name' -->
      <li *ngFor="let p of produkte | filter:suchbegriff:'name'">
        {{ p.name }}
      </li>
    </ul>
  `
})
export class SucheComponent {
  suchbegriff = '';
  produkte = [
    { id: 1, name: 'Laptop' },
    { id: 2, name: 'Maus' },
    { id: 3, name: 'Tastatur' },
    { id: 4, name: 'Monitor' },
  ];
}
```

---

## 5. Pure vs. Impure Pipes

### Pure Pipes (Standard)

Eine **Pure Pipe** wird nur neu berechnet, wenn sich der **Eingabewert** ändert. Das ist performanter.

```typescript
@Pipe({
  name: 'meinePipe',
  pure: true // Standard – muss nicht explizit gesetzt werden
})
```

### Impure Pipes

Eine **Impure Pipe** wird bei **jeder Change Detection** neu berechnet. Vorsicht: Kann Performance-Probleme verursachen!

```typescript
@Pipe({
  name: 'meineImpurePipe',
  pure: false  // Achtung: Langsam!
})
```

> ⚠️ **Empfehlung:** Immer Pure Pipes verwenden, ausser es ist wirklich nötig.

---

## Zusammenfassung

| Pipe | Beschreibung | Beispiel |
|---|---|---|
| `date` | Datum formatieren | `{{ datum \| date:'dd.MM.yyyy' }}` |
| `currency` | Währung formatieren | `{{ preis \| currency:'CHF' }}` |
| `number` | Zahl formatieren | `{{ zahl \| number:'1.2-2' }}` |
| `percent` | Prozent formatieren | `{{ 0.5 \| percent }}` |
| `uppercase` | Grossbuchstaben | `{{ text \| uppercase }}` |
| `lowercase` | Kleinbuchstaben | `{{ text \| lowercase }}` |
| `titlecase` | Erster Buchstabe gross | `{{ text \| titlecase }}` |
| `slice` | Kürzen | `{{ text \| slice:0:10 }}` |
| `json` | JSON-Ausgabe | `{{ objekt \| json }}` |
| `async` | Observable/Promise | `{{ obs$ \| async }}` |
| `keyvalue` | Objekt itererieren | `*ngFor="let e of obj \| keyvalue"` |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Pipes](https://angular.dev/guide/pipes)
- 📖 [Angular Docs – Custom Pipes](https://angular.dev/guide/pipes/custom-pipes)
