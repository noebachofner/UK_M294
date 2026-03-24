# 09 – Template-driven Forms

## Lernziele
- Du verstehst den Unterschied zwischen Template-driven und Reactive Forms
- Du kannst einfache Formulare mit Template-driven Forms erstellen
- Du kannst Formulare validieren und Fehler anzeigen

---

## 1. Zwei Arten von Formularen in Angular

| | Template-driven | Reactive Forms |
|---|---|---|
| **Komplexität** | Einfach | Mittel–Komplex |
| **Wo definiert** | Im HTML-Template | In der TypeScript-Klasse |
| **Formular-Kontrolle** | Angular erstellt automatisch | Manuell erstellt |
| **Validierung** | HTML-Attribute | TypeScript-Code |
| **Eignet sich für** | Einfache Formulare | Komplexe Formulare |
| **Modul** | `FormsModule` | `ReactiveFormsModule` |

---

## 2. Grundlegendes Setup

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-formular',
  standalone: true,
  imports: [FormsModule],  // FormsModule muss importiert werden!
  template: `...`
})
export class FormularComponent {}
```

---

## 3. Einfaches Formular

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

interface BenutzerFormular {
  name: string;
  email: string;
  passwort: string;
}

@Component({
  selector: 'app-registrierung',
  standalone: true,
  imports: [FormsModule],
  template: `
    <form #regForm="ngForm" (ngSubmit)="onSubmit(regForm)">
      <h2>Registrierung</h2>

      <!-- Name Feld -->
      <div>
        <label for="name">Name:</label>
        <input
          id="name"
          name="name"
          [(ngModel)]="formular.name"
          required
          minlength="2">
      </div>

      <!-- E-Mail Feld -->
      <div>
        <label for="email">E-Mail:</label>
        <input
          id="email"
          name="email"
          type="email"
          [(ngModel)]="formular.email"
          required
          email>
      </div>

      <!-- Passwort Feld -->
      <div>
        <label for="passwort">Passwort:</label>
        <input
          id="passwort"
          name="passwort"
          type="password"
          [(ngModel)]="formular.passwort"
          required
          minlength="8">
      </div>

      <!-- Submit-Button (deaktiviert wenn Formular ungültig) -->
      <button type="submit" [disabled]="regForm.invalid">
        Registrieren
      </button>

      <!-- Debug-Ausgabe (für Entwicklung) -->
      <pre>{{ regForm.value | json }}</pre>
      <p>Gültig: {{ regForm.valid }}</p>
    </form>
  `
})
export class RegistrierungComponent {
  formular: BenutzerFormular = {
    name: '',
    email: '',
    passwort: ''
  };

  onSubmit(form: any): void {
    if (form.valid) {
      console.log('Formular gesendet:', this.formular);
    }
  }
}
```

---

## 4. Validierung und Fehlermeldungen

```typescript
import { Component } from '@angular/core';
import { FormsModule, NgModel } from '@angular/forms';
import { NgIf } from '@angular/common';

@Component({
  selector: 'app-formular-validierung',
  standalone: true,
  imports: [FormsModule, NgIf],
  template: `
    <form #myForm="ngForm" (ngSubmit)="onSubmit()">

      <!-- Feld mit Template-Referenz für Validierung -->
      <div>
        <label>Name:</label>
        <input
          name="name"
          #nameField="ngModel"
          [(ngModel)]="name"
          required
          minlength="2"
          maxlength="50"
          [class.fehler]="nameField.invalid && nameField.touched">

        <!-- Fehlermeldungen -->
        <div *ngIf="nameField.invalid && nameField.touched">
          <span *ngIf="nameField.errors?.['required']" class="fehler-text">
            Name ist erforderlich.
          </span>
          <span *ngIf="nameField.errors?.['minlength']" class="fehler-text">
            Name muss mindestens 2 Zeichen lang sein.
          </span>
          <span *ngIf="nameField.errors?.['maxlength']" class="fehler-text">
            Name darf maximal 50 Zeichen lang sein.
          </span>
        </div>
      </div>

      <!-- E-Mail -->
      <div>
        <label>E-Mail:</label>
        <input
          name="email"
          type="email"
          #emailField="ngModel"
          [(ngModel)]="email"
          required
          email>

        <div *ngIf="emailField.invalid && emailField.touched">
          <span *ngIf="emailField.errors?.['required']">E-Mail ist erforderlich.</span>
          <span *ngIf="emailField.errors?.['email']">Ungültige E-Mail-Adresse.</span>
        </div>
      </div>

      <!-- Alter -->
      <div>
        <label>Alter:</label>
        <input
          name="alter"
          type="number"
          #alterField="ngModel"
          [(ngModel)]="alter"
          required
          min="18"
          max="120">

        <div *ngIf="alterField.invalid && alterField.touched">
          <span *ngIf="alterField.errors?.['required']">Alter ist erforderlich.</span>
          <span *ngIf="alterField.errors?.['min']">Mindestalter ist 18 Jahre.</span>
          <span *ngIf="alterField.errors?.['max']">Maximales Alter ist 120 Jahre.</span>
        </div>
      </div>

      <!-- Formular Status -->
      <p>Formular-Status: {{ myForm.status }}</p>

      <button type="submit" [disabled]="myForm.invalid">
        Absenden
      </button>
    </form>
  `,
  styles: [`
    .fehler { border-color: red; }
    .fehler-text { color: red; font-size: 12px; }
  `]
})
export class FormularValidierungComponent {
  name = '';
  email = '';
  alter: number | null = null;

  onSubmit(): void {
    console.log({ name: this.name, email: this.email, alter: this.alter });
  }
}
```

---

## 5. CSS-Klassen für Formular-Zustände

Angular fügt automatisch CSS-Klassen zu Formular-Elementen hinzu:

| Klasse | Beschreibung |
|---|---|
| `ng-valid` | Feld ist gültig |
| `ng-invalid` | Feld ist ungültig |
| `ng-pristine` | Feld wurde noch nicht verändert |
| `ng-dirty` | Feld wurde verändert |
| `ng-untouched` | Feld wurde noch nicht fokussiert |
| `ng-touched` | Feld wurde fokussiert und verlassen |

```css
/* Eigene Styles für Formular-Zustände */
input.ng-invalid.ng-touched {
  border-color: red;
  background-color: #fff5f5;
}

input.ng-valid.ng-dirty {
  border-color: green;
}
```

---

## 6. Select, Checkbox und Radio

```typescript
@Component({
  selector: 'app-formular-felder',
  standalone: true,
  imports: [FormsModule, NgFor],
  template: `
    <form #form="ngForm">

      <!-- Dropdown (Select) -->
      <div>
        <label>Land:</label>
        <select name="land" [(ngModel)]="land" required>
          <option value="">-- Bitte wählen --</option>
          <option *ngFor="let l of laender" [value]="l.code">
            {{ l.name }}
          </option>
        </select>
      </div>

      <!-- Checkbox -->
      <div>
        <label>
          <input
            type="checkbox"
            name="agb"
            [(ngModel)]="agbAkzeptiert"
            required>
          Ich akzeptiere die AGB
        </label>
      </div>

      <!-- Radio Buttons -->
      <div>
        <label>Geschlecht:</label>
        <label>
          <input type="radio" name="geschlecht" [(ngModel)]="geschlecht" value="m">
          Männlich
        </label>
        <label>
          <input type="radio" name="geschlecht" [(ngModel)]="geschlecht" value="w">
          Weiblich
        </label>
        <label>
          <input type="radio" name="geschlecht" [(ngModel)]="geschlecht" value="d">
          Divers
        </label>
      </div>

      <!-- Textarea -->
      <div>
        <label>Kommentar:</label>
        <textarea
          name="kommentar"
          [(ngModel)]="kommentar"
          rows="4"
          maxlength="500">
        </textarea>
        <p>{{ kommentar.length }}/500 Zeichen</p>
      </div>

    </form>

    <p>Land: {{ land }}</p>
    <p>AGB akzeptiert: {{ agbAkzeptiert }}</p>
    <p>Geschlecht: {{ geschlecht }}</p>
  `
})
export class FormularFelderComponent {
  land = '';
  agbAkzeptiert = false;
  geschlecht = '';
  kommentar = '';

  laender = [
    { code: 'CH', name: 'Schweiz' },
    { code: 'DE', name: 'Deutschland' },
    { code: 'AT', name: 'Österreich' },
  ];
}
```

---

## 7. Formular zurücksetzen

```typescript
@Component({
  selector: 'app-reset',
  standalone: true,
  imports: [FormsModule],
  template: `
    <form #form="ngForm" (ngSubmit)="onSubmit(form)">
      <input name="name" [(ngModel)]="name" required>
      <button type="submit">Absenden</button>
      <button type="button" (click)="reset(form)">Zurücksetzen</button>
    </form>
  `
})
export class ResetComponent {
  name = '';

  onSubmit(form: any): void {
    console.log('Name:', this.name);
    this.reset(form); // Nach Submit zurücksetzen
  }

  reset(form: any): void {
    form.reset();     // Formular zurücksetzen (Werte + Zustände)
    this.name = '';   // Model zurücksetzen
  }
}
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `FormsModule` | Muss für Template-driven Forms importiert werden |
| `ngForm` | Repräsentiert das gesamte Formular |
| `ngModel` | Zwei-Wege-Datenbindung für Formular-Felder |
| `required`, `email`, `minlength` | HTML-Validierungsattribute |
| `ng-valid/ng-invalid` | CSS-Klassen für Validierungszustände |
| `touched/dirty` | Zustände für bedingte Fehlermeldungen |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Template-driven Forms](https://angular.dev/guide/forms/template-driven-forms)
- 📖 [Angular Docs – Form Validation](https://angular.dev/guide/forms/form-validation)
