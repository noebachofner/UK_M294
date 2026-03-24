# 10 – Reactive Forms

## Lernziele
- Du kannst Reactive Forms erstellen und konfigurieren
- Du kennst `FormControl`, `FormGroup` und `FormArray`
- Du kannst benutzerdefinierte Validatoren schreiben
- Du weisst wann Template-driven vs. Reactive Forms besser geeignet sind

---

## 1. Was sind Reactive Forms?

Reactive Forms definieren die Formular-Struktur vollständig in der **TypeScript-Klasse**, nicht im Template. Dies bietet:

- Bessere **Testbarkeit**
- Mehr **Kontrolle** über den Formular-Zustand
- Einfachere **dynamische Formulare**
- Bessere Integration mit **RxJS**

---

## 2. Grundlegendes Setup

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormControl, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-formular',
  standalone: true,
  imports: [ReactiveFormsModule],  // ReactiveFormsModule importieren!
  template: `...`
})
export class FormularComponent {}
```

---

## 3. FormControl – Einzelnes Feld

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormControl, Validators } from '@angular/forms';
import { NgIf } from '@angular/common';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [ReactiveFormsModule, NgIf],
  template: `
    <!-- formControl-Direktive verbindet das Feld mit dem FormControl -->
    <input [formControl]="nameControl" placeholder="Name eingeben...">

    <!-- Fehler anzeigen -->
    <div *ngIf="nameControl.invalid && nameControl.touched">
      <span *ngIf="nameControl.errors?.['required']">Name ist erforderlich.</span>
      <span *ngIf="nameControl.errors?.['minlength']">Mindestens 2 Zeichen.</span>
    </div>

    <p>Wert: {{ nameControl.value }}</p>
    <p>Gültig: {{ nameControl.valid }}</p>
  `
})
export class DemoComponent {
  nameControl = new FormControl('', [
    Validators.required,
    Validators.minLength(2),
    Validators.maxLength(50)
  ]);
}
```

---

## 4. FormGroup – Gruppe von Feldern

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormGroup, FormControl, Validators } from '@angular/forms';
import { NgIf } from '@angular/common';

@Component({
  selector: 'app-registrierung',
  standalone: true,
  imports: [ReactiveFormsModule, NgIf],
  template: `
    <form [formGroup]="registrierungForm" (ngSubmit)="onSubmit()">
      <h2>Registrierung</h2>

      <!-- formControlName verbindet Feld mit FormGroup -->
      <div>
        <label>Name:</label>
        <input formControlName="name">
        <div *ngIf="name?.invalid && name?.touched">
          <span *ngIf="name?.errors?.['required']">Pflichtfeld</span>
          <span *ngIf="name?.errors?.['minlength']">Mindestens 2 Zeichen</span>
        </div>
      </div>

      <div>
        <label>E-Mail:</label>
        <input formControlName="email" type="email">
        <div *ngIf="email?.invalid && email?.touched">
          <span *ngIf="email?.errors?.['required']">Pflichtfeld</span>
          <span *ngIf="email?.errors?.['email']">Ungültige E-Mail</span>
        </div>
      </div>

      <div>
        <label>Passwort:</label>
        <input formControlName="passwort" type="password">
        <div *ngIf="passwort?.invalid && passwort?.touched">
          <span *ngIf="passwort?.errors?.['required']">Pflichtfeld</span>
          <span *ngIf="passwort?.errors?.['minlength']">Mindestens 8 Zeichen</span>
        </div>
      </div>

      <!-- Formular Status -->
      <button type="submit" [disabled]="registrierungForm.invalid">
        Registrieren
      </button>
    </form>

    <!-- Debug -->
    <pre>{{ registrierungForm.value | json }}</pre>
  `
})
export class RegistrierungComponent {
  registrierungForm = new FormGroup({
    name: new FormControl('', [Validators.required, Validators.minLength(2)]),
    email: new FormControl('', [Validators.required, Validators.email]),
    passwort: new FormControl('', [Validators.required, Validators.minLength(8)]),
  });

  // Kurzwege zu den Controls (bessere Lesbarkeit)
  get name() { return this.registrierungForm.get('name'); }
  get email() { return this.registrierungForm.get('email'); }
  get passwort() { return this.registrierungForm.get('passwort'); }

  onSubmit(): void {
    if (this.registrierungForm.valid) {
      console.log('Formular:', this.registrierungForm.value);
    }
  }
}
```

---

## 5. FormBuilder – Kürzere Schreibweise

`FormBuilder` ist ein Service, der das Erstellen von Formularen vereinfacht.

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, Validators } from '@angular/forms';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="name">
      <input formControlName="email">
      <button type="submit">Senden</button>
    </form>
  `
})
export class DemoComponent {
  // FormBuilder per DI injizieren
  constructor(private fb: FormBuilder) {}

  // Kürzere Schreibweise mit FormBuilder
  form = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    email: ['', [Validators.required, Validators.email]],
    passwort: ['', [Validators.required, Validators.minLength(8)]],
  });

  onSubmit(): void {
    if (this.form.valid) {
      console.log(this.form.value);
      // Typ-sicherer Zugriff:
      const name = this.form.value.name;
      const email = this.form.value.email;
    }
  }
}
```

---

## 6. FormArray – Dynamische Felder

`FormArray` ermöglicht eine **variable Anzahl** von Formular-Feldern.

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, FormArray, Validators } from '@angular/forms';
import { NgFor } from '@angular/common';

@Component({
  selector: 'app-dynamic-form',
  standalone: true,
  imports: [ReactiveFormsModule, NgFor],
  template: `
    <form [formGroup]="form">
      <h3>Zutaten</h3>

      <!-- FormArray itererieren -->
      <div formArrayName="zutaten">
        <div *ngFor="let zutat of zutaten.controls; let i = index">
          <input [formControlName]="i" placeholder="Zutat {{ i + 1 }}">
          <button type="button" (click)="entfernen(i)">-</button>
        </div>
      </div>

      <button type="button" (click)="hinzufuegen()">+ Zutat hinzufügen</button>

      <button type="submit" (click)="onSubmit()">Speichern</button>
    </form>

    <p>Zutaten: {{ form.value.zutaten | json }}</p>
  `
})
export class DynamicFormComponent {
  form = this.fb.group({
    zutaten: this.fb.array([
      this.fb.control('Mehl', Validators.required),
      this.fb.control('Zucker', Validators.required),
    ])
  });

  // Zugriff auf das FormArray
  get zutaten(): FormArray {
    return this.form.get('zutaten') as FormArray;
  }

  constructor(private fb: FormBuilder) {}

  hinzufuegen(): void {
    this.zutaten.push(this.fb.control('', Validators.required));
  }

  entfernen(index: number): void {
    this.zutaten.removeAt(index);
  }

  onSubmit(): void {
    console.log(this.form.value);
  }
}
```

---

## 7. Alle Standard-Validatoren

```typescript
import { Validators } from '@angular/forms';

// Pflichtfeld
Validators.required

// Min/Max Länge
Validators.minLength(2)
Validators.maxLength(100)

// E-Mail Format
Validators.email

// Muster (RegEx)
Validators.pattern(/^[0-9]{4,6}$/)
Validators.pattern('[a-zA-Z ]*')

// Zahlen-Bereich
Validators.min(0)
Validators.max(100)

// Mehrere Validatoren kombinieren
new FormControl('', [
  Validators.required,
  Validators.minLength(2),
  Validators.email
])
```

---

## 8. Eigene Validatoren (Custom Validators)

```typescript
// validators/plz.validator.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Validator-Funktion für Schweizer Postleitzahlen (4 Stellen)
export function plzValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value;

    if (!value) return null; // Wird von required behandelt

    const plzPattern = /^[0-9]{4}$/;
    const gueltig = plzPattern.test(value);

    return gueltig ? null : { plzUngueltig: { value: control.value } };
  };
}

// Passwort-Bestätigung Validator (für FormGroup)
export function passwortBestaetigenValidator(
  passwortKey: string,
  bestaetigenKey: string
): ValidatorFn {
  return (group: AbstractControl): ValidationErrors | null => {
    const passwort = group.get(passwortKey)?.value;
    const bestaetigen = group.get(bestaetigenKey)?.value;

    return passwort === bestaetigen ? null : { passwortStimmeNichtUeberein: true };
  };
}
```

```typescript
// Verwendung:
import { plzValidator, passwortBestaetigenValidator } from './validators/plz.validator';

form = this.fb.group({
  plz: ['', [Validators.required, plzValidator()]],
  passwort: ['', [Validators.required, Validators.minLength(8)]],
  passwortBestaetigen: ['', Validators.required],
}, {
  validators: passwortBestaetigenValidator('passwort', 'passwortBestaetigen')
});
```

```html
<!-- Fehlermeldung für benutzerdefinierten Validator -->
<span *ngIf="form.get('plz')?.errors?.['plzUngueltig']">
  Ungültige Schweizer PLZ (4 Ziffern).
</span>

<span *ngIf="form.errors?.['passwortStimmeNichtUeberein']">
  Passwörter stimmen nicht überein.
</span>
```

---

## 9. Formular Werte setzen & zurücksetzen

```typescript
export class DemoComponent {
  form = this.fb.group({
    name: [''],
    email: [''],
  });

  // Alle Werte setzen (alle Felder müssen angegeben werden)
  setzeAlleWerte(): void {
    this.form.setValue({
      name: 'Max Muster',
      email: 'max@example.com'
    });
  }

  // Nur bestimmte Werte setzen (teilweises Update)
  setzeEinzelneWerte(): void {
    this.form.patchValue({
      name: 'Anna Muster'
      // email muss nicht angegeben werden
    });
  }

  // Formular zurücksetzen
  zuruecksetzen(): void {
    this.form.reset();
    // Oder mit Standardwerten:
    this.form.reset({ name: '', email: '' });
  }
}
```

---

## 10. Auf Wertänderungen reagieren

```typescript
export class DemoComponent implements OnInit, OnDestroy {
  form = this.fb.group({
    suche: [''],
  });

  private subscription!: Subscription;

  ngOnInit(): void {
    // Auf Änderungen im Feld reagieren
    this.subscription = this.form.get('suche')!.valueChanges.subscribe(wert => {
      console.log('Sucheingabe:', wert);
      this.suchen(wert);
    });

    // Mit debounce (wartet 300ms nach letzter Eingabe)
    this.form.get('suche')!.valueChanges
      .pipe(debounceTime(300))
      .subscribe(wert => this.suchen(wert));
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe(); // Wichtig! Memory Leak verhindern
  }

  suchen(suchbegriff: string | null): void {
    if (suchbegriff) console.log('Suche nach:', suchbegriff);
  }
}
```

---

## 11. Typed Forms (Angular 14+)

Ab Angular 14 sind Reactive Forms **streng typisiert**:

```typescript
import { FormControl, FormGroup } from '@angular/forms';

// TypeScript kennt die Typen der Formular-Felder
const form = new FormGroup({
  name: new FormControl<string>(''),        // name ist string
  alter: new FormControl<number | null>(null),  // alter ist number oder null
});

// TypeScript-Fehler wenn falscher Typ
form.value.name; // → string | null (automatisch inferiert)
form.value.alter; // → number | null
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `FormControl` | Einzelnes Formular-Feld |
| `FormGroup` | Gruppe von Feldern |
| `FormArray` | Dynamische Anzahl von Feldern |
| `FormBuilder` | Helper-Service für kürzere Syntax |
| `Validators` | Eingebaute Validatoren |
| `valueChanges` | Observable für Wertänderungen |
| `setValue()` | Alle Werte setzen |
| `patchValue()` | Bestimmte Werte setzen |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Reactive Forms](https://angular.dev/guide/forms/reactive-forms)
- 📖 [Angular Docs – Form Validation](https://angular.dev/guide/forms/form-validation)
- 📖 [Angular Docs – Dynamic Forms](https://angular.dev/guide/forms/dynamic-forms)
- 🎥 [Angular University – Reactive Forms](https://angular-university.io/course/angular-forms-course)
