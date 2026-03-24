# 18 – Content Projection (ng-content)

## Lernziele
- Du verstehst was Content Projection ist und wofür sie verwendet wird
- Du kannst `ng-content`, `ng-template` und `ng-container` richtig einsetzen
- Du kannst wiederverwendbare "Wrapper-Komponenten" erstellen

---

## 1. Was ist Content Projection?

**Content Projection** ermöglicht es, HTML-Inhalt von einer **Eltern-Komponente** in eine **Kind-Komponente** zu "projizieren" (einzufügen).

### Problem ohne Content Projection

```typescript
// Karte für Produkte
@Component({ template: `<div class="karte"><h3>Produkt</h3></div>` })
export class ProduktKarte {}

// Karte für Benutzer – fast gleich, aber eigene Komponente nötig
@Component({ template: `<div class="karte"><h3>Benutzer</h3></div>` })
export class BenutzerKarte {}
```

### Lösung: Content Projection

```typescript
// Eine generische Karte – der Inhalt kommt von aussen
@Component({ template: `<div class="karte"><ng-content></ng-content></div>` })
export class KarteComponent {}

// Verwendung:
// <app-karte><h3>Produkt</h3></app-karte>
// <app-karte><h3>Benutzer</h3></app-karte>
```

---

## 2. Einfache Content Projection

```typescript
// karte.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-karte',
  standalone: true,
  template: `
    <div class="karte">
      <!-- Hier wird der Inhalt der Eltern-Komponente eingefügt -->
      <ng-content></ng-content>
    </div>
  `,
  styles: [`
    .karte {
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 16px;
      margin: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
  `]
})
export class KarteComponent {}
```

```typescript
// eltern.component.ts
import { Component } from '@angular/core';
import { KarteComponent } from './karte.component';

@Component({
  selector: 'app-eltern',
  standalone: true,
  imports: [KarteComponent],
  template: `
    <app-karte>
      <!-- Dieser Inhalt wird in ng-content eingefügt -->
      <h2>Mein Titel</h2>
      <p>Irgendein Inhalt...</p>
      <button>Klick mich</button>
    </app-karte>

    <app-karte>
      <h2>Anderer Inhalt</h2>
      <p>Verschiedene Karte, gleiche Struktur.</p>
    </app-karte>
  `
})
export class ElternComponent {}
```

---

## 3. Multi-Slot Content Projection

Mehrere `ng-content`-Slots mit CSS-Selektoren:

```typescript
// panel.component.ts
@Component({
  selector: 'app-panel',
  standalone: true,
  template: `
    <div class="panel">
      <div class="panel-header">
        <!-- Nur Elemente mit slot="header" werden hier eingefügt -->
        <ng-content select="[slot=header]"></ng-content>
      </div>

      <div class="panel-body">
        <!-- Alle anderen Elemente kommen hier rein -->
        <ng-content></ng-content>
      </div>

      <div class="panel-footer">
        <!-- Nur mat-card-actions oder .footer-klasse -->
        <ng-content select=".panel-footer-inhalt"></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .panel { border: 1px solid #ddd; border-radius: 8px; overflow: hidden; }
    .panel-header { background: #f5f5f5; padding: 12px 16px; font-weight: bold; }
    .panel-body { padding: 16px; }
    .panel-footer { background: #f5f5f5; padding: 8px 16px; text-align: right; }
  `]
})
export class PanelComponent {}
```

```typescript
// Verwendung:
@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [PanelComponent],
  template: `
    <app-panel>
      <!-- Header-Slot -->
      <span slot="header">📌 Wichtige Meldung</span>

      <!-- Standard-Slot (Panel-Body) -->
      <p>Das ist der Hauptinhalt des Panels.</p>
      <p>Er kann mehrere Elemente enthalten.</p>

      <!-- Footer-Slot -->
      <div class="panel-footer-inhalt">
        <button>OK</button>
        <button>Abbrechen</button>
      </div>
    </app-panel>
  `
})
export class DemoComponent {}
```

---

## 4. ng-template – Templates definieren

`ng-template` definiert ein Template das **nicht sofort gerendert** wird:

```typescript
@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <!-- ng-template wird NICHT gerendert bis es explizit eingebunden wird -->
    <ng-template #ladeAnzeige>
      <div class="spinner">
        <p>Lädt...</p>
      </div>
    </ng-template>

    <ng-template #fehlerAnzeige let-message>
      <div class="fehler">
        <p>❌ {{ message }}</p>
      </div>
    </ng-template>

    <!-- Template bedingt verwenden -->
    <ng-container *ngIf="isLoading; else inhaltOderFehler">
      <ng-container *ngTemplateOutlet="ladeAnzeige"></ng-container>
    </ng-container>

    <ng-template #inhaltOderFehler>
      <ng-container *ngIf="fehler; else inhalt">
        <ng-container *ngTemplateOutlet="fehlerAnzeige; context: { $implicit: fehler }">
        </ng-container>
      </ng-container>
    </ng-template>

    <ng-template #inhalt>
      <p>Inhalt erfolgreich geladen!</p>
    </ng-template>
  `
})
export class DemoComponent {
  isLoading = false;
  fehler = '';
}
```

---

## 5. ng-container – Unsichtbares Wrapper-Element

`ng-container` erzeugt **keinen DOM-Knoten**. Nützlich wenn man strukturelle Direktiven ohne extra HTML-Elemente verwenden will:

```html
<!-- PROBLEM: Unnötiges div im DOM -->
<div *ngIf="benutzer">
  <div *ngFor="let recht of benutzer.rechte">
    {{ recht }}
  </div>
</div>

<!-- LÖSUNG: ng-container – kein extra DOM-Element -->
<ng-container *ngIf="benutzer">
  <ng-container *ngFor="let recht of benutzer.rechte">
    {{ recht }}
  </ng-container>
</ng-container>

<!-- Für Tabellen besonders nützlich: -->
<table>
  <ng-container *ngFor="let gruppe of gruppen">
    <tr class="gruppe-header">
      <td colspan="3">{{ gruppe.name }}</td>
    </tr>
    <tr *ngFor="let item of gruppe.items">
      <td>{{ item.name }}</td>
      <td>{{ item.wert }}</td>
    </tr>
  </ng-container>
</table>
```

---

## 6. ngTemplateOutlet – Templates dynamisch rendern

```typescript
@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <!-- Templates definieren -->
    <ng-template #blauesTemplate>
      <p style="color: blue">Ich bin blau!</p>
    </ng-template>

    <ng-template #rotesTemplate>
      <p style="color: red">Ich bin rot!</p>
    </ng-template>

    <!-- Dynamisch ein Template auswählen -->
    <ng-container *ngTemplateOutlet="aktualesTemplate"></ng-container>

    <button (click)="wechseln()">Template wechseln</button>
  `
})
export class DemoComponent {
  @ViewChild('blauesTemplate') blauesTemplate!: TemplateRef<any>;
  @ViewChild('rotesTemplate') rotesTemplate!: TemplateRef<any>;

  aktualesTemplate!: TemplateRef<any>;
  zeigeBlau = true;

  ngAfterViewInit(): void {
    this.aktualesTemplate = this.blauesTemplate;
  }

  wechseln(): void {
    this.zeigeBlau = !this.zeigeBlau;
    this.aktualesTemplate = this.zeigeBlau ? this.blauesTemplate : this.rotesTemplate;
  }
}
```

---

## 7. Praktisches Beispiel: Generische Tabellen-Komponente

```typescript
// generic-table.component.ts
import { Component, Input, ContentChild, TemplateRef } from '@angular/core';
import { NgFor } from '@angular/common';

@Component({
  selector: 'app-tabelle',
  standalone: true,
  imports: [NgFor],
  template: `
    <table>
      <thead>
        <tr>
          <ng-content select="[tabellenKopf]"></ng-content>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let zeile of daten">
          <ng-container *ngTemplateOutlet="zeilenTemplate; context: { $implicit: zeile }">
          </ng-container>
        </tr>
      </tbody>
    </table>
  `
})
export class TabelleComponent {
  @Input() daten: any[] = [];
  @ContentChild('zeile') zeilenTemplate!: TemplateRef<any>;
}
```

```typescript
// Verwendung
@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [TabelleComponent],
  template: `
    <app-tabelle [daten]="benutzer">
      <!-- Tabellen-Kopf -->
      <th tabellenKopf>Name</th>
      <th tabellenKopf>E-Mail</th>

      <!-- Zeilen-Template (let-item = aktuelle Zeile) -->
      <ng-template #zeile let-item>
        <td>{{ item.name }}</td>
        <td>{{ item.email }}</td>
      </ng-template>
    </app-tabelle>
  `
})
export class DemoComponent {
  benutzer = [
    { name: 'Max', email: 'max@example.com' },
    { name: 'Anna', email: 'anna@example.com' },
  ];
}
```

---

## Zusammenfassung

| Konzept | Beschreibung |
|---|---|
| `<ng-content>` | Inhalt von Eltern-Komponente einfügen |
| `<ng-content select="...">` | Gezielter Slot für bestimmte Inhalte |
| `<ng-template>` | Template definieren ohne zu rendern |
| `<ng-container>` | Strukturelles Wrapper-Element ohne DOM |
| `*ngTemplateOutlet` | Template dynamisch rendern |
| `@ContentChild` | Zugriff auf projizierte Templates |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Content Projection](https://angular.dev/guide/components/content-projection)
- 📖 [Angular Docs – ng-template](https://angular.dev/guide/templates/ng-template)
