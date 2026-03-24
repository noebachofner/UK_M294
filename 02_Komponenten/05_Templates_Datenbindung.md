# 05 – Templates & Datenbindung

## Lernziele
- Du verstehst die verschiedenen Arten der Datenbindung in Angular
- Du kannst Interpolation, Property Binding, Event Binding und Two-Way Binding anwenden
- Du verstehst die neue Control Flow Syntax in Angular 17+

---

## 1. Was ist Datenbindung?

**Datenbindung** (Data Binding) ist der Mechanismus, der die **TypeScript-Klasse** (Komponentenlogik) mit dem **HTML-Template** (Ansicht) verbindet.

```
TypeScript-Klasse  ←→  HTML-Template
     (Logik)              (Ansicht)
```

Angular kennt vier Arten der Datenbindung:

| Art | Syntax | Richtung |
|---|---|---|
| **Interpolation** | `{{ ausdruck }}` | Komponente → View |
| **Property Binding** | `[eigenschaft]="wert"` | Komponente → View |
| **Event Binding** | `(ereignis)="methode()"` | View → Komponente |
| **Two-Way Binding** | `[(ngModel)]="eigenschaft"` | Bidirektional |

---

## 2. Interpolation `{{ }}`

Zeigt Werte aus der Komponente im Template an.

```typescript
@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <h1>{{ titel }}</h1>
    <p>{{ 1 + 2 }}</p>
    <p>{{ name.toUpperCase() }}</p>
    <p>{{ isLoggedIn ? 'Angemeldet' : 'Abgemeldet' }}</p>
  `
})
export class DemoComponent {
  titel = 'Meine Angular App';
  name = 'Max';
  isLoggedIn = true;
}
```

> ⚠️ In Interpolation kann kein vollständiger Code ausgeführt werden – nur **Ausdrücke** (Expressions).

---

## 3. Property Binding `[ ]`

Bindet einen Wert aus der Komponente an eine **HTML-Eigenschaft** (Property).

```typescript
@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <!-- Src eines Bildes dynamisch setzen -->
    <img [src]="bildUrl" [alt]="bildBeschreibung">

    <!-- Button deaktivieren -->
    <button [disabled]="istDeaktiviert">Klick mich</button>

    <!-- CSS-Klasse dynamisch hinzufügen -->
    <div [class]="cssKlasse">Inhalt</div>

    <!-- Style direkt setzen -->
    <p [style.color]="textfarbe">Farbiger Text</p>

    <!-- Attribut setzen -->
    <input [value]="eingabewert">
  `
})
export class DemoComponent {
  bildUrl = 'assets/bild.jpg';
  bildBeschreibung = 'Ein schönes Bild';
  istDeaktiviert = false;
  cssKlasse = 'hervorgehoben';
  textfarbe = 'blue';
  eingabewert = 'Hallo';
}
```

### Unterschied Interpolation vs. Property Binding

```html
<!-- Interpolation – konvertiert immer zu String -->
<img src="{{ bildUrl }}">

<!-- Property Binding – übergibt den tatsächlichen Wert -->
<img [src]="bildUrl">

<!-- Bei Nicht-String-Werten MUSS Property Binding verwendet werden! -->
<app-kind [anzahl]="42"></app-kind>       <!-- ✅ Übergibt Zahl 42 -->
<app-kind anzahl="{{ 42 }}"></app-kind>   <!-- ❌ Übergibt String "42" -->
```

---

## 4. Event Binding `( )`

Reagiert auf Benutzeraktionen (Events) und ruft Methoden in der Komponente auf.

```typescript
@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <!-- Klick-Event -->
    <button (click)="onKlick()">Klick mich!</button>

    <!-- Event mit Daten -->
    <button (click)="onKlickMitDaten('Hallo')">Mit Daten</button>

    <!-- $event – das Event-Objekt -->
    <input (input)="onEingabe($event)">
    <input (keyup.enter)="onEnterDruck()">

    <!-- Maus-Events -->
    <div (mouseover)="onMouseOver()" (mouseout)="onMouseOut()">
      Hover mich!
    </div>

    <!-- Formular Submit -->
    <form (submit)="onSubmit($event)">
      <button type="submit">Absenden</button>
    </form>
  `
})
export class DemoComponent {
  onKlick(): void {
    console.log('Button wurde geklickt!');
  }

  onKlickMitDaten(text: string): void {
    console.log(text);
  }

  onEingabe(event: Event): void {
    const input = event.target as HTMLInputElement;
    console.log(input.value);
  }

  onEnterDruck(): void {
    console.log('Enter wurde gedrückt!');
  }

  onMouseOver(): void { console.log('Maus drüber'); }
  onMouseOut(): void { console.log('Maus weg'); }

  onSubmit(event: Event): void {
    event.preventDefault();
    console.log('Formular abgesendet!');
  }
}
```

---

## 5. Two-Way Binding `[( )]`

Synchronisiert Daten **bidirektional** zwischen Komponente und Template.  
Erfordert das **FormsModule**.

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [FormsModule],
  template: `
    <!-- Änderungen im Input werden sofort in der Komponente gespeichert -->
    <input [(ngModel)]="benutzername">

    <!-- Wird sofort aktualisiert -->
    <p>Hallo, {{ benutzername }}!</p>

    <!-- Gleichwertige Langschreibweise: -->
    <input [ngModel]="benutzername" (ngModelChange)="benutzername = $event">
  `
})
export class DemoComponent {
  benutzername = 'Max';
}
```

---

## 6. Class & Style Binding

### Class Binding

```html
<!-- Einzelne Klasse -->
<div [class.aktiv]="istAktiv">Inhalt</div>
<!-- "aktiv"-Klasse wird hinzugefügt wenn istAktiv = true -->

<!-- Mehrere Klassen mit ngClass -->
<div [ngClass]="{'aktiv': istAktiv, 'hervorgehoben': istHervorgehoben}">
  Inhalt
</div>

<!-- Array von Klassen -->
<div [ngClass]="['klasse1', 'klasse2']">Inhalt</div>
```

### Style Binding

```html
<!-- Einzelne Style-Eigenschaft -->
<p [style.color]="textfarbe">Text</p>
<p [style.font-size.px]="schriftgroesse">Text</p>

<!-- Mehrere Styles mit ngStyle -->
<div [ngStyle]="{'color': textfarbe, 'font-size': schriftgroesse + 'px'}">
  Gestylter Text
</div>
```

---

## 7. Control Flow Syntax (Angular 17+)

Ab Angular 17 gibt es eine neue, eingebaute Control Flow Syntax, die `*ngIf`, `*ngFor` und `*ngSwitch` ersetzt.

### @if – Bedingtes Rendering

```html
<!-- Neu (Angular 17+) -->
@if (benutzer.eingeloggt) {
  <p>Willkommen, {{ benutzer.name }}!</p>
} @else if (benutzer.gesperrt) {
  <p>Ihr Konto ist gesperrt.</p>
} @else {
  <p>Bitte melden Sie sich an.</p>
}

<!-- Alt (immer noch gültig) -->
<p *ngIf="benutzer.eingeloggt; else nichtEingeloggt">
  Willkommen!
</p>
<ng-template #nichtEingeloggt>
  <p>Bitte anmelden.</p>
</ng-template>
```

### @for – Schleifen

```html
<!-- Neu (Angular 17+) -->
@for (produkt of produkte; track produkt.id) {
  <div class="produkt">
    <h3>{{ produkt.name }}</h3>
    <p>{{ produkt.preis }} CHF</p>
  </div>
} @empty {
  <p>Keine Produkte gefunden.</p>
}

<!-- Alt (immer noch gültig) -->
<div *ngFor="let produkt of produkte; trackBy: trackById">
  <h3>{{ produkt.name }}</h3>
</div>
```

### @switch – Switch-Anweisung

```html
<!-- Neu (Angular 17+) -->
@switch (status) {
  @case ('aktiv') {
    <span class="gruen">Aktiv</span>
  }
  @case ('inaktiv') {
    <span class="grau">Inaktiv</span>
  }
  @default {
    <span class="gelb">Unbekannt</span>
  }
}
```

---

## 8. Template Reference Variables

Mit `#name` können Template-Elemente mit einem Namen versehen und direkt im Template referenziert werden.

```html
<!-- Input-Wert direkt im Template lesen -->
<input #meinInput type="text">
<button (click)="zeigWert(meinInput.value)">Zeigen</button>

<!-- Komponenten-Referenz -->
<app-kind #kindRef></app-kind>
<button (click)="kindRef.methode()">Kind-Methode aufrufen</button>
```

---

## 9. Pipes in Templates

Pipes transformieren Daten zur Anzeige.

```html
<p>{{ preis | currency:'CHF':'symbol':'1.2-2' }}</p>
<p>{{ datum | date:'dd.MM.yyyy' }}</p>
<p>{{ name | uppercase }}</p>
<p>{{ name | lowercase }}</p>
<p>{{ langer_text | slice:0:50 }}...</p>
<p>{{ objekt | json }}</p>  <!-- Für Debugging -->
```

---

## 10. Safe Navigation Operator `?.`

Verhindert Fehler bei `null` oder `undefined` Werten.

```typescript
benutzer: Benutzer | null = null;
```

```html
<!-- Ohne Safe Navigation: Fehler wenn benutzer null ist! -->
<p>{{ benutzer.name }}</p>

<!-- Mit Safe Navigation: Zeigt nichts wenn benutzer null ist -->
<p>{{ benutzer?.name }}</p>
<p>{{ benutzer?.adresse?.strasse }}</p>
```

---

## Zusammenfassung

| Syntax | Art | Richtung |
|---|---|---|
| `{{ wert }}` | Interpolation | Komponente → View |
| `[eigenschaft]="wert"` | Property Binding | Komponente → View |
| `(ereignis)="methode()"` | Event Binding | View → Komponente |
| `[(ngModel)]="wert"` | Two-Way Binding | Bidirektional |
| `@if (bed) { }` | Control Flow | - |
| `@for (item of liste) { }` | Control Flow | - |
| `#ref` | Template Reference | - |
| `wert?.eigenschaft` | Safe Navigation | - |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Templates](https://angular.dev/guide/templates)
- 📖 [Angular Docs – Data Binding](https://angular.dev/guide/templates/binding)
- 📖 [Angular Docs – Control Flow](https://angular.dev/guide/templates/control-flow)
- 🎥 [Angular University – Angular Template Syntax](https://angular-university.io)
