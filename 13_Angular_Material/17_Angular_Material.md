# 17 – Angular Material

## Lernziele
- Du kannst Angular Material in einem Projekt einrichten
- Du kennst die wichtigsten Material-Komponenten
- Du kannst Themes anpassen
- Du weisst wie man ein responsives Layout mit Material erstellt

---

## 1. Was ist Angular Material?

**Angular Material** ist die offizielle UI-Komponenten-Bibliothek für Angular, basierend auf Googles **Material Design**. Sie bietet fertige, zugängliche und konsistente UI-Komponenten.

### Vorteile

- ✅ Fertige, schöne UI-Komponenten
- ✅ Accessibility (a11y) eingebaut
- ✅ Responsive Design
- ✅ Anpassbare Themes
- ✅ Vollständige Angular-Integration

---

## 2. Installation

```bash
ng add @angular/material

# Interaktive Fragen:
# ? Choose a prebuilt theme name:  Indigo/Pink (oder eigenes)
# ? Set up global Angular Material typography styles? Yes
# ? Include the Angular animations module? Include and enable animations
```

Das Kommando installiert automatisch:
- `@angular/material`
- `@angular/cdk`
- `@angular/animations`

---

## 3. Modul-Imports

Jede Material-Komponente hat ein eigenes Modul:

```typescript
// app.config.ts – globale Provider
import { ApplicationConfig } from '@angular/core';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';

export const appConfig: ApplicationConfig = {
  providers: [
    provideAnimationsAsync()  // Animations aktivieren
  ]
};
```

```typescript
// In jeder Komponente die Material nutzt:
import { MatButtonModule } from '@angular/material/button';
import { MatInputModule } from '@angular/material/input';
import { MatCardModule } from '@angular/material/card';

@Component({
  imports: [MatButtonModule, MatInputModule, MatCardModule],
  // ...
})
```

---

## 4. Häufig verwendete Komponenten

### Buttons

```html
<!-- Basic Button -->
<button mat-button>Einfacher Button</button>

<!-- Raised Button (erhöht) -->
<button mat-raised-button>Erhöhter Button</button>

<!-- Filled Button -->
<button mat-flat-button color="primary">Primär</button>

<!-- Stroke Button -->
<button mat-stroked-button color="accent">Umriss</button>

<!-- Icon Button -->
<button mat-icon-button>
  <mat-icon>delete</mat-icon>
</button>

<!-- FAB (Floating Action Button) -->
<button mat-fab color="primary">
  <mat-icon>add</mat-icon>
</button>

<!-- Farben: primary, accent, warn -->
<button mat-raised-button color="primary">Primary</button>
<button mat-raised-button color="accent">Accent</button>
<button mat-raised-button color="warn">Warnung</button>
```

```typescript
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
```

### Input-Felder

```html
<mat-form-field>
  <mat-label>Name</mat-label>
  <input matInput [(ngModel)]="name" placeholder="Name eingeben">
</mat-form-field>

<mat-form-field>
  <mat-label>E-Mail</mat-label>
  <input matInput type="email" [formControl]="emailControl">
  <mat-error *ngIf="emailControl.hasError('email')">
    Ungültige E-Mail
  </mat-error>
</mat-form-field>

<!-- Textarea -->
<mat-form-field>
  <mat-label>Kommentar</mat-label>
  <textarea matInput rows="4" [(ngModel)]="kommentar"></textarea>
</mat-form-field>

<!-- Select / Dropdown -->
<mat-form-field>
  <mat-label>Land</mat-label>
  <mat-select [(ngModel)]="land">
    <mat-option value="CH">Schweiz</mat-option>
    <mat-option value="DE">Deutschland</mat-option>
    <mat-option value="AT">Österreich</mat-option>
  </mat-select>
</mat-form-field>
```

```typescript
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
```

### Cards (Karten)

```html
<mat-card>
  <mat-card-header>
    <mat-card-title>Produkt-Titel</mat-card-title>
    <mat-card-subtitle>Kategorie: Elektronik</mat-card-subtitle>
  </mat-card-header>

  <img mat-card-image src="assets/produkt.jpg" alt="Produkt Bild">

  <mat-card-content>
    <p>Produktbeschreibung geht hier rein...</p>
    <p>Preis: 999 CHF</p>
  </mat-card-content>

  <mat-card-actions>
    <button mat-button>DETAILS</button>
    <button mat-raised-button color="primary">KAUFEN</button>
  </mat-card-actions>
</mat-card>
```

```typescript
import { MatCardModule } from '@angular/material/card';
```

### Toolbar / Navigation

```html
<mat-toolbar color="primary">
  <button mat-icon-button>
    <mat-icon>menu</mat-icon>
  </button>
  <span>Meine App</span>
  <span class="spacer"></span>
  <button mat-icon-button>
    <mat-icon>account_circle</mat-icon>
  </button>
</mat-toolbar>
```

```css
.spacer {
  flex: 1 1 auto; /* Füllt den verfügbaren Platz */
}
```

```typescript
import { MatToolbarModule } from '@angular/material/toolbar';
```

### Tabellen

```typescript
import { Component } from '@angular/core';
import { MatTableModule } from '@angular/material/table';

interface Benutzer {
  id: number;
  name: string;
  email: string;
  rolle: string;
}

@Component({
  selector: 'app-benutzer-tabelle',
  standalone: true,
  imports: [MatTableModule],
  template: `
    <table mat-table [dataSource]="benutzer">

      <!-- ID Spalte -->
      <ng-container matColumnDef="id">
        <th mat-header-cell *matHeaderCellDef>ID</th>
        <td mat-cell *matCellDef="let row">{{ row.id }}</td>
      </ng-container>

      <!-- Name Spalte -->
      <ng-container matColumnDef="name">
        <th mat-header-cell *matHeaderCellDef>Name</th>
        <td mat-cell *matCellDef="let row">{{ row.name }}</td>
      </ng-container>

      <!-- Email Spalte -->
      <ng-container matColumnDef="email">
        <th mat-header-cell *matHeaderCellDef>E-Mail</th>
        <td mat-cell *matCellDef="let row">{{ row.email }}</td>
      </ng-container>

      <tr mat-header-row *matHeaderRowDef="anzeigeSpalteln"></tr>
      <tr mat-row *matRowDef="let row; columns: anzeigeSpalteln;"></tr>
    </table>
  `
})
export class BenutzerTabelleComponent {
  anzeigeSpalteln: string[] = ['id', 'name', 'email'];

  benutzer: Benutzer[] = [
    { id: 1, name: 'Max Muster', email: 'max@example.com', rolle: 'Admin' },
    { id: 2, name: 'Anna Muster', email: 'anna@example.com', rolle: 'User' },
  ];
}
```

### Dialog (Modal)

```typescript
// dialog-inhalt.component.ts
import { Component, Inject } from '@angular/core';
import { MatDialogModule, MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-dialog-inhalt',
  standalone: true,
  imports: [MatDialogModule, MatButtonModule],
  template: `
    <h2 mat-dialog-title>{{ data.titel }}</h2>
    <mat-dialog-content>
      <p>{{ data.nachricht }}</p>
    </mat-dialog-content>
    <mat-dialog-actions>
      <button mat-button (click)="abbrechen()">Abbrechen</button>
      <button mat-raised-button color="warn" (click)="bestaetigen()">Bestätigen</button>
    </mat-dialog-actions>
  `
})
export class DialogInhaltComponent {
  constructor(
    private dialogRef: MatDialogRef<DialogInhaltComponent>,
    @Inject(MAT_DIALOG_DATA) public data: { titel: string; nachricht: string }
  ) {}

  abbrechen(): void {
    this.dialogRef.close(false);
  }

  bestaetigen(): void {
    this.dialogRef.close(true);
  }
}
```

```typescript
// Eltern-Komponente – Dialog öffnen
import { Component } from '@angular/core';
import { MatDialog } from '@angular/material/dialog';
import { MatButtonModule } from '@angular/material/button';
import { DialogInhaltComponent } from './dialog-inhalt.component';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [MatButtonModule],
  template: `
    <button mat-raised-button color="warn" (click)="loeschenBestaetigen()">
      Löschen
    </button>
  `
})
export class DemoComponent {
  constructor(private dialog: MatDialog) {}

  loeschenBestaetigen(): void {
    const dialogRef = this.dialog.open(DialogInhaltComponent, {
      width: '400px',
      data: {
        titel: 'Löschen bestätigen',
        nachricht: 'Möchten Sie diesen Eintrag wirklich löschen?'
      }
    });

    dialogRef.afterClosed().subscribe(ergebnis => {
      if (ergebnis) {
        console.log('Gelöscht!');
      }
    });
  }
}
```

### Snackbar (Toast-Benachrichtigungen)

```typescript
import { Component } from '@angular/core';
import { MatSnackBar } from '@angular/material/snack-bar';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [MatButtonModule],
  template: `
    <button mat-button (click)="zeigeMeldung()">Meldung anzeigen</button>
  `
})
export class DemoComponent {
  constructor(private snackBar: MatSnackBar) {}

  zeigeMeldung(): void {
    this.snackBar.open('Gespeichert! ✓', 'Schliessen', {
      duration: 3000,      // 3 Sekunden
      horizontalPosition: 'center',
      verticalPosition: 'bottom',
      panelClass: ['success-snackbar']
    });
  }
}
```

---

## 5. Responsives Layout mit Angular CDK

```html
<!-- app.component.html -->
<mat-sidenav-container>
  <!-- Seitenmenü -->
  <mat-sidenav #sidenav mode="side" opened>
    <mat-nav-list>
      <a mat-list-item routerLink="/home">
        <mat-icon matListItemIcon>home</mat-icon>
        <span matListItemTitle>Home</span>
      </a>
      <a mat-list-item routerLink="/produkte">
        <mat-icon matListItemIcon>shopping_bag</mat-icon>
        <span matListItemTitle>Produkte</span>
      </a>
    </mat-nav-list>
  </mat-sidenav>

  <!-- Hauptinhalt -->
  <mat-sidenav-content>
    <mat-toolbar color="primary">
      <button mat-icon-button (click)="sidenav.toggle()">
        <mat-icon>menu</mat-icon>
      </button>
      <span>Meine App</span>
    </mat-toolbar>
    <div class="content">
      <router-outlet></router-outlet>
    </div>
  </mat-sidenav-content>
</mat-sidenav-container>
```

---

## 6. Theming (Farben anpassen)

```scss
/* styles.scss */
@use '@angular/material' as mat;

// Eigenes Theme definieren
$meine-app-theme: mat.define-theme((
  color: (
    theme-type: light,
    primary: mat.$azure-palette,
    tertiary: mat.$blue-palette,
  ),
  density: (
    scale: 0,
  )
));

// Theme auf die App anwenden
:root {
  @include mat.all-component-themes($meine-app-theme);
}
```

---

## 7. Vollständiges Beispiel: Produkt-Karte

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { MatCardModule } from '@angular/material/card';
import { MatButtonModule } from '@angular/material/button';
import { MatChipsModule } from '@angular/material/chips';
import { MatIconModule } from '@angular/material/icon';
import { CurrencyPipe } from '@angular/common';

interface Produkt {
  id: number;
  name: string;
  preis: number;
  beschreibung: string;
  verfuegbar: boolean;
  bildUrl: string;
}

@Component({
  selector: 'app-produkt-karte',
  standalone: true,
  imports: [MatCardModule, MatButtonModule, MatChipsModule, MatIconModule, CurrencyPipe],
  template: `
    <mat-card>
      <mat-card-header>
        <mat-card-title>{{ produkt.name }}</mat-card-title>
        <mat-card-subtitle>
          <mat-chip [color]="produkt.verfuegbar ? 'primary' : 'warn'" highlighted>
            {{ produkt.verfuegbar ? 'Auf Lager' : 'Ausverkauft' }}
          </mat-chip>
        </mat-card-subtitle>
      </mat-card-header>

      <img mat-card-image [src]="produkt.bildUrl" [alt]="produkt.name">

      <mat-card-content>
        <p>{{ produkt.beschreibung }}</p>
        <p class="preis">{{ produkt.preis | currency:'CHF':'symbol' }}</p>
      </mat-card-content>

      <mat-card-actions align="end">
        <button mat-button>
          <mat-icon>favorite_border</mat-icon>
          Merken
        </button>
        <button
          mat-raised-button
          color="primary"
          [disabled]="!produkt.verfuegbar"
          (click)="kaufen.emit(produkt)">
          <mat-icon>add_shopping_cart</mat-icon>
          In den Warenkorb
        </button>
      </mat-card-actions>
    </mat-card>
  `,
  styles: [`
    .preis { font-size: 1.5rem; font-weight: bold; color: #1976d2; }
  `]
})
export class ProduktKarteComponent {
  @Input() produkt!: Produkt;
  @Output() kaufen = new EventEmitter<Produkt>();
}
```

---

## Zusammenfassung

| Komponente | Import | Beschreibung |
|---|---|---|
| Button | `MatButtonModule` | Verschiedene Button-Stile |
| Input | `MatInputModule` + `MatFormFieldModule` | Eingabefelder |
| Select | `MatSelectModule` | Dropdown |
| Card | `MatCardModule` | Karten-Container |
| Toolbar | `MatToolbarModule` | Navigationsleiste |
| Table | `MatTableModule` | Datentabellen |
| Dialog | `MatDialogModule` | Modals |
| Snackbar | `MatSnackBar` | Toast-Benachrichtigungen |
| Icon | `MatIconModule` | Material Icons |
| Sidenav | `MatSidenavModule` | Seitenmenü |

---

## Weiterführende Ressourcen

- 📖 [Angular Material Dokumentation](https://material.angular.io)
- 📖 [Angular Material Komponenten](https://material.angular.io/components)
- 📖 [Material Design Guidelines](https://m3.material.io)
- 🎥 [Angular University – Angular Material Course](https://angular-university.io/course/angular-material-course)
