# 06 – Direktiven (Directives)

## Lernziele
- Du kennst den Unterschied zwischen Struktur- und Attributdirektiven
- Du kannst `*ngIf`, `*ngFor`, `*ngSwitch` und `ngClass` / `ngStyle` anwenden
- Du kannst eigene Attributdirektiven erstellen

---

## 1. Was sind Direktiven?

**Direktiven** sind spezielle Klassen, die das Verhalten oder das Aussehen von HTML-Elementen **erweitern oder verändern**.

Es gibt drei Arten:

| Art | Beschreibung | Beispiele |
|---|---|---|
| **Komponenten** | Direktiven mit eigenem Template | `AppComponent` |
| **Strukturdirektiven** | Verändern die DOM-Struktur | `*ngIf`, `*ngFor`, `*ngSwitch` |
| **Attributdirektiven** | Verändern Aussehen/Verhalten | `ngClass`, `ngStyle` |

---

## 2. Strukturdirektiven

Strukturdirektiven fügen **DOM-Elemente hinzu oder entfernen** sie. Sie beginnen mit einem Sternchen `*`.

### ngIf – Bedingtes Rendering

```typescript
import { Component } from '@angular/core';
import { NgIf } from '@angular/common';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [NgIf],
  template: `
    <button (click)="toggle()">Toggle</button>

    <!-- Element wird gerendert/entfernt -->
    <p *ngIf="zeigeInhalt">Dieser Text ist sichtbar!</p>

    <!-- Mit else-Block -->
    <p *ngIf="istAngemeldet; else nichtAngemeldet">
      Willkommen, {{ benutzername }}!
    </p>
    <ng-template #nichtAngemeldet>
      <p>Bitte melde dich an.</p>
    </ng-template>

    <!-- Mit then und else -->
    <ng-container *ngIf="status === 'laden'; then ladeBlock; else inhaltBlock">
    </ng-container>
    <ng-template #ladeBlock><p>Lädt...</p></ng-template>
    <ng-template #inhaltBlock><p>Inhalt geladen!</p></ng-template>
  `
})
export class DemoComponent {
  zeigeInhalt = true;
  istAngemeldet = false;
  benutzername = 'Max';
  status = 'inhalt';

  toggle(): void {
    this.zeigeInhalt = !this.zeigeInhalt;
  }
}
```

> ⚠️ **Wichtig:** `*ngIf="false"` **entfernt** das Element aus dem DOM. `[hidden]="true"` **versteckt** es nur (CSS `display:none`). Der Unterschied ist wichtig für Performance und Lifecycle Hooks.

### ngFor – Listen Rendering

```typescript
import { Component } from '@angular/core';
import { NgFor } from '@angular/common';

interface Aufgabe {
  id: number;
  text: string;
  erledigt: boolean;
}

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [NgFor],
  template: `
    <ul>
      <li *ngFor="let aufgabe of aufgaben; let i = index; trackBy: trackById">
        {{ i + 1 }}. {{ aufgabe.text }}
        <button (click)="loeschen(aufgabe.id)">Löschen</button>
      </li>
    </ul>

    <!-- Weitere Loop-Variablen -->
    <div *ngFor="let item of items;
                 let i = index;
                 let erster = first;
                 let letzter = last;
                 let gerade = even;
                 let ungerade = odd">
      <span [class.hervorgehoben]="gerade">{{ item }}</span>
    </div>
  `
})
export class DemoComponent {
  aufgaben: Aufgabe[] = [
    { id: 1, text: 'Angular lernen', erledigt: false },
    { id: 2, text: 'Projekt erstellen', erledigt: false },
    { id: 3, text: 'ÜK bestehen', erledigt: false },
  ];

  items = ['Apfel', 'Banane', 'Kirsche'];

  // TrackBy für Performance-Optimierung
  trackById(index: number, aufgabe: Aufgabe): number {
    return aufgabe.id;
  }

  loeschen(id: number): void {
    this.aufgaben = this.aufgaben.filter(a => a.id !== id);
  }
}
```

> 💡 **TrackBy** sollte immer bei `*ngFor` verwendet werden, wenn Listenelemente sich ändern können. Es verhindert unnötiges Re-Rendering.

### ngSwitch – Switch-Anweisung

```typescript
import { Component } from '@angular/core';
import { NgSwitch, NgSwitchCase, NgSwitchDefault } from '@angular/common';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [NgSwitch, NgSwitchCase, NgSwitchDefault],
  template: `
    <div [ngSwitch]="ampelFarbe">
      <p *ngSwitchCase="'rot'">🔴 Stopp!</p>
      <p *ngSwitchCase="'gelb'">🟡 Achtung!</p>
      <p *ngSwitchCase="'gruen'">🟢 Fahren!</p>
      <p *ngSwitchDefault>⚪ Unbekannte Farbe</p>
    </div>

    <button (click)="naechsteFarbe()">Nächste Farbe</button>
  `
})
export class DemoComponent {
  farben = ['rot', 'gelb', 'gruen'];
  index = 0;
  ampelFarbe = this.farben[0];

  naechsteFarbe(): void {
    this.index = (this.index + 1) % this.farben.length;
    this.ampelFarbe = this.farben[this.index];
  }
}
```

---

## 3. Neue Control Flow Syntax (Angular 17+)

Ab Angular 17 gibt es eingebaute Kontrollfluss-Syntax als **Alternative** zu `*ngIf`, `*ngFor`, `*ngSwitch`.

```html
<!-- @if -->
@if (benutzer) {
  <p>Eingeloggt als {{ benutzer.name }}</p>
} @else {
  <p>Nicht eingeloggt</p>
}

<!-- @for mit @empty -->
@for (produkt of produkte; track produkt.id) {
  <app-produkt [produkt]="produkt" />
} @empty {
  <p>Keine Produkte vorhanden.</p>
}

<!-- @switch -->
@switch (rolle) {
  @case ('admin') { <app-admin-panel /> }
  @case ('user') { <app-user-panel /> }
  @default { <p>Unbekannte Rolle</p> }
}
```

---

## 4. Attributdirektiven

### ngClass – CSS-Klassen dynamisch zuweisen

```typescript
import { Component } from '@angular/core';
import { NgClass } from '@angular/common';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [NgClass],
  template: `
    <!-- Boolean-Bedingung -->
    <div [ngClass]="{'aktiv': istAktiv}">Inhalt</div>

    <!-- Mehrere Klassen -->
    <div [ngClass]="{
      'aktiv': istAktiv,
      'hervorgehoben': istHervorgehoben,
      'fehler': hatFehler
    }">Inhalt</div>

    <!-- Array -->
    <div [ngClass]="['klasse1', 'klasse2']">Inhalt</div>

    <!-- Methode -->
    <div [ngClass]="getKlassen()">Inhalt</div>

    <!-- Direktes String-Binding -->
    <div [class]="aktiveKlasse">Inhalt</div>
    <div [class.aktiv]="istAktiv">Inhalt</div>
  `
})
export class DemoComponent {
  istAktiv = true;
  istHervorgehoben = false;
  hatFehler = false;
  aktiveKlasse = 'primary';

  getKlassen(): object {
    return {
      'aktiv': this.istAktiv,
      'fehler': this.hatFehler
    };
  }
}
```

### ngStyle – CSS-Styles dynamisch setzen

```typescript
import { Component } from '@angular/core';
import { NgStyle } from '@angular/common';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [NgStyle],
  template: `
    <!-- Einzelne Style-Eigenschaft -->
    <p [style.color]="textfarbe">Farbiger Text</p>
    <p [style.font-size.px]="schriftgroesse">Grosser Text</p>

    <!-- Mehrere Styles -->
    <div [ngStyle]="{
      'color': textfarbe,
      'font-size': schriftgroesse + 'px',
      'background-color': hintergrund
    }">
      Gestylter Bereich
    </div>

    <button (click)="aendereStyle()">Style ändern</button>
  `
})
export class DemoComponent {
  textfarbe = 'blue';
  schriftgroesse = 16;
  hintergrund = 'lightyellow';

  aendereStyle(): void {
    this.textfarbe = this.textfarbe === 'blue' ? 'red' : 'blue';
    this.schriftgroesse += 2;
  }
}
```

---

## 5. ng-container und ng-template

### ng-container

`ng-container` ist ein Hilfselement, das **keinen DOM-Knoten** erzeugt. Nützlich wenn man mehrere Direktiven gruppieren will.

```html
<!-- Ohne ng-container – fügt ein unnötiges div ein -->
<div *ngIf="liste.length > 0">
  <li *ngFor="let item of liste">{{ item }}</li>
</div>

<!-- Mit ng-container – kein extra DOM-Element -->
<ng-container *ngIf="liste.length > 0">
  <li *ngFor="let item of liste">{{ item }}</li>
</ng-container>

<!-- Mehrere Direktiven kombinieren -->
<ng-container *ngIf="benutzer">
  <ng-container *ngFor="let recht of benutzer.rechte">
    <p>{{ recht }}</p>
  </ng-container>
</ng-container>
```

### ng-template

`ng-template` definiert einen **Template-Block**, der nicht automatisch gerendert wird.

```html
<!-- ng-template wird nur gerendert wenn explizit referenziert -->
<ng-template #ladeAnzeige>
  <p>Lädt...</p>
  <div class="spinner"></div>
</ng-template>

<div *ngIf="!isLoading; else ladeAnzeige">
  <p>Inhalt geladen!</p>
</div>
```

---

## 6. Eigene Attributdirektive erstellen

```bash
ng generate directive directives/hervorheben
```

```typescript
// hervorheben.directive.ts
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHervorheben]',  // Verwendung: <p appHervorheben>
  standalone: true
})
export class HervorhebenDirective {
  @Input() appHervorheben = 'yellow';  // Farbe als Input

  constructor(private el: ElementRef) {}

  @HostListener('mouseenter')
  onMouseEnter(): void {
    this.el.nativeElement.style.backgroundColor = this.appHervorheben;
  }

  @HostListener('mouseleave')
  onMouseLeave(): void {
    this.el.nativeElement.style.backgroundColor = '';
  }
}
```

```typescript
// Verwendung in einer Komponente
import { Component } from '@angular/core';
import { HervorhebenDirective } from '../directives/hervorheben.directive';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [HervorhebenDirective],
  template: `
    <p appHervorheben>Standard gelb beim Hover</p>
    <p [appHervorheben]="'lightblue'">Blau beim Hover</p>
    <p appHervorheben="lightgreen">Grün beim Hover</p>
  `
})
export class DemoComponent {}
```

---

## Zusammenfassung

| Direktive | Typ | Verwendung |
|---|---|---|
| `*ngIf` | Strukturdirektive | Element bedingt anzeigen/entfernen |
| `*ngFor` | Strukturdirektive | Liste von Elementen rendern |
| `*ngSwitch` | Strukturdirektive | Switch-Anweisung im Template |
| `ngClass` | Attributdirektive | CSS-Klassen dynamisch zuweisen |
| `ngStyle` | Attributdirektive | CSS-Styles dynamisch setzen |
| `@if` | Control Flow | Moderner Ersatz für `*ngIf` |
| `@for` | Control Flow | Moderner Ersatz für `*ngFor` |
| `@switch` | Control Flow | Moderner Ersatz für `*ngSwitch` |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Directives](https://angular.dev/guide/directives)
- 📖 [Angular Docs – Attribute Directives](https://angular.dev/guide/directives/attribute-directives)
- 📖 [Angular Docs – Structural Directives](https://angular.dev/guide/directives/structural-directives)
- 🎥 [Angular University – Angular Directives](https://angular-university.io/course/angular-core-deep-dive-course)
