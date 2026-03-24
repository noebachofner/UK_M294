# 04 – Komponenten (Components)

## Lernziele
- Du weisst, was eine Komponente ist und wie sie aufgebaut ist
- Du kannst Komponenten erstellen und in anderen Komponenten verwenden
- Du verstehst den `@Component`-Decorator und seine wichtigsten Optionen
- Du kennst die Kommunikation zwischen Komponenten (@Input / @Output)

---

## 1. Was ist eine Komponente?

Eine **Komponente** ist der grundlegende Baustein einer Angular-Applikation. Jede Komponente besteht aus:

- **TypeScript-Klasse** → enthält die Logik und Daten
- **HTML-Template** → definiert die Ansicht
- **CSS-Styles** → definiert das Aussehen (optional)

```
Komponente = TypeScript-Klasse + HTML-Template + CSS-Styles
```

Eine Angular-App ist ein **Baum von Komponenten**:

```
AppComponent (Root)
├── HeaderComponent
├── MainContentComponent
│   ├── ProduktListeComponent
│   │   └── ProduktKarteComponent
│   └── WarenkorbComponent
└── FooterComponent
```

---

## 2. Komponente erstellen

### Mit der CLI (empfohlen)

```bash
ng generate component mein-name
# Kurzform:
ng g c mein-name
```

### Aufbau einer Komponente

```typescript
// produkt.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-produkt',           // HTML-Tag der Komponente
  standalone: true,                   // Moderne Standalone-Komponente
  templateUrl: './produkt.component.html',  // Externes Template
  styleUrl: './produkt.component.css'       // Externe Styles
})
export class ProduktComponent {
  // Eigenschaften (Properties)
  name: string = 'Laptop';
  preis: number = 999.99;
  verfuegbar: boolean = true;

  // Methoden (Methods)
  kaufen(): void {
    console.log(`${this.name} wurde gekauft.`);
  }
}
```

```html
<!-- produkt.component.html -->
<div class="produkt">
  <h2>{{ name }}</h2>
  <p>Preis: {{ preis }} CHF</p>
  <p>{{ verfuegbar ? 'Verfügbar' : 'Nicht verfügbar' }}</p>
  <button (click)="kaufen()">Kaufen</button>
</div>
```

```css
/* produkt.component.css */
.produkt {
  border: 1px solid #ccc;
  padding: 16px;
  border-radius: 8px;
}
```

### Inline-Template (alles in einer Datei)

```typescript
@Component({
  selector: 'app-klein',
  standalone: true,
  template: `
    <h1>{{ titel }}</h1>
    <p>Hallo von der kleinen Komponente!</p>
  `,
  styles: [`
    h1 { color: blue; }
  `]
})
export class KleinComponent {
  titel = 'Kleine Komponente';
}
```

---

## 3. Selector-Typen

```typescript
// Standardmässig als HTML-Element (empfohlen)
selector: 'app-meine-komponente'
// Verwendung: <app-meine-komponente></app-meine-komponente>

// Als Attribut
selector: '[app-meine-komponente]'
// Verwendung: <div app-meine-komponente></div>

// Als CSS-Klasse (selten genutzt)
selector: '.app-meine-komponente'
// Verwendung: <div class="app-meine-komponente"></div>
```

---

## 4. Komponente in einer anderen Komponente verwenden

```typescript
// eltern.component.ts
import { Component } from '@angular/core';
import { ProduktComponent } from '../produkt/produkt.component';

@Component({
  selector: 'app-eltern',
  standalone: true,
  imports: [ProduktComponent],  // Muss importiert werden!
  template: `
    <h1>Produktliste</h1>
    <app-produkt></app-produkt>
    <app-produkt></app-produkt>
  `
})
export class ElternComponent {}
```

---

## 5. Input – Daten von Eltern an Kind übergeben

Mit `@Input()` können Daten von einer **Eltern-Komponente** an eine **Kind-Komponente** übergeben werden.

```typescript
// kind.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-kind',
  standalone: true,
  template: `
    <div>
      <h3>{{ produktName }}</h3>
      <p>{{ produktPreis }} CHF</p>
    </div>
  `
})
export class KindComponent {
  @Input() produktName: string = '';
  @Input() produktPreis: number = 0;
}
```

```typescript
// eltern.component.ts
import { Component } from '@angular/core';
import { KindComponent } from '../kind/kind.component';

@Component({
  selector: 'app-eltern',
  standalone: true,
  imports: [KindComponent],
  template: `
    <app-kind
      [produktName]="'Laptop'"
      [produktPreis]="999">
    </app-kind>
    <app-kind
      [produktName]="ausgewaehlterName"
      [produktPreis]="ausgewaehlterPreis">
    </app-kind>
  `
})
export class ElternComponent {
  ausgewaehlterName = 'Maus';
  ausgewaehlterPreis = 29;
}
```

### Input mit Pflichtfeldern (Angular 17+)

```typescript
import { Component, Input, input } from '@angular/core';

@Component({ ... })
export class KindComponent {
  // Neue Signals-basierte Input-Syntax (empfohlen ab Angular 17)
  produktName = input.required<string>();
  produktPreis = input<number>(0); // 0 ist der Standardwert
}
```

---

## 6. Output – Ereignisse von Kind an Eltern senden

Mit `@Output()` und `EventEmitter` kann eine **Kind-Komponente** Ereignisse an die **Eltern-Komponente** senden.

```typescript
// kind.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-produkt-karte',
  standalone: true,
  template: `
    <div>
      <h3>{{ name }}</h3>
      <button (click)="auswaehlen()">Auswählen</button>
    </div>
  `
})
export class ProduktKarteComponent {
  @Input() name: string = '';
  @Output() produktAusgewaehlt = new EventEmitter<string>();

  auswaehlen(): void {
    this.produktAusgewaehlt.emit(this.name); // Ereignis auslösen
  }
}
```

```typescript
// eltern.component.ts
import { Component } from '@angular/core';
import { ProduktKarteComponent } from '../produkt-karte/produkt-karte.component';

@Component({
  selector: 'app-eltern',
  standalone: true,
  imports: [ProduktKarteComponent],
  template: `
    <app-produkt-karte
      [name]="'Laptop'"
      (produktAusgewaehlt)="onProduktAusgewaehlt($event)">
    </app-produkt-karte>
    <p *ngIf="ausgewaehltes">Ausgewählt: {{ ausgewaehltes }}</p>
  `
})
export class ElternComponent {
  ausgewaehltes: string = '';

  onProduktAusgewaehlt(produktName: string): void {
    this.ausgewaehltes = produktName;
  }
}
```

---

## 7. ViewChild – Direkter Zugriff auf Kind-Komponenten

```typescript
import { Component, ViewChild, AfterViewInit } from '@angular/core';
import { KindComponent } from './kind.component';

@Component({
  selector: 'app-eltern',
  standalone: true,
  imports: [KindComponent],
  template: `<app-kind #meinKind></app-kind>`
})
export class ElternComponent implements AfterViewInit {
  @ViewChild('meinKind') kindComponent!: KindComponent;

  ngAfterViewInit(): void {
    // Zugriff auf die Kind-Komponente nach der View-Initialisierung
    console.log(this.kindComponent);
  }
}
```

---

## 8. Vollständiges Beispiel: Produkt-Liste

```typescript
// produkt.interface.ts
export interface Produkt {
  id: number;
  name: string;
  preis: number;
  verfuegbar: boolean;
}
```

```typescript
// produkt-karte.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { Produkt } from '../produkt.interface';

@Component({
  selector: 'app-produkt-karte',
  standalone: true,
  template: `
    <div class="karte">
      <h3>{{ produkt.name }}</h3>
      <p>{{ produkt.preis }} CHF</p>
      <span [class]="produkt.verfuegbar ? 'verfuegbar' : 'nicht-verfuegbar'">
        {{ produkt.verfuegbar ? 'Auf Lager' : 'Ausverkauft' }}
      </span>
      <button (click)="inWarenkorb()" [disabled]="!produkt.verfuegbar">
        In den Warenkorb
      </button>
    </div>
  `,
  styles: [`
    .karte { border: 1px solid #ccc; padding: 16px; margin: 8px; }
    .verfuegbar { color: green; }
    .nicht-verfuegbar { color: red; }
  `]
})
export class ProduktKarteComponent {
  @Input() produkt!: Produkt;
  @Output() zumWarenkorb = new EventEmitter<Produkt>();

  inWarenkorb(): void {
    this.zumWarenkorb.emit(this.produkt);
  }
}
```

```typescript
// produkt-liste.component.ts
import { Component } from '@angular/core';
import { NgFor } from '@angular/common';
import { ProduktKarteComponent } from '../produkt-karte/produkt-karte.component';
import { Produkt } from '../produkt.interface';

@Component({
  selector: 'app-produkt-liste',
  standalone: true,
  imports: [NgFor, ProduktKarteComponent],
  template: `
    <h1>Produkte</h1>
    <div class="liste">
      <app-produkt-karte
        *ngFor="let p of produkte"
        [produkt]="p"
        (zumWarenkorb)="handleWarenkorb($event)">
      </app-produkt-karte>
    </div>
    <div *ngIf="warenkorb.length > 0">
      <h2>Warenkorb ({{ warenkorb.length }} Artikel)</h2>
    </div>
  `
})
export class ProduktListeComponent {
  produkte: Produkt[] = [
    { id: 1, name: 'Laptop', preis: 999, verfuegbar: true },
    { id: 2, name: 'Maus', preis: 29, verfuegbar: true },
    { id: 3, name: 'Tastatur', preis: 79, verfuegbar: false },
  ];

  warenkorb: Produkt[] = [];

  handleWarenkorb(produkt: Produkt): void {
    this.warenkorb.push(produkt);
  }
}
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `@Component` | Decorator, der eine Klasse als Komponente markiert |
| `selector` | HTML-Tag der Komponente |
| `template` / `templateUrl` | HTML-Vorlage der Komponente |
| `@Input()` | Daten von Eltern an Kind übergeben |
| `@Output()` | Ereignisse von Kind an Eltern senden |
| `EventEmitter` | Sendet Ereignisse aus einer Komponente |
| `imports` | Andere Komponenten/Module die verwendet werden |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Components](https://angular.dev/guide/components)
- 📖 [Angular Docs – Inputs](https://angular.dev/guide/components/inputs)
- 📖 [Angular Docs – Outputs](https://angular.dev/guide/components/outputs)
- 🎥 [Angular University – Components and Templates](https://angular-university.io/course/angular-core-deep-dive-course)
