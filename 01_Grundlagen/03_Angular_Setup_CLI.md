# 03 – Angular Setup & CLI

## Lernziele
- Du kannst Angular lokal einrichten und ein neues Projekt erstellen
- Du kennst die wichtigsten Angular CLI Befehle
- Du verstehst die Projektstruktur einer Angular-Applikation

---

## 1. Voraussetzungen

Vor der Installation von Angular müssen folgende Tools installiert sein:

### Node.js & npm

```bash
# Node.js Version prüfen (mindestens v18 empfohlen)
node --version
# v20.x.x

# npm Version prüfen
npm --version
# v10.x.x
```

> 💡 Node.js kann von https://nodejs.org heruntergeladen werden.

---

## 2. Angular CLI installieren

Die **Angular CLI** (Command Line Interface) ist das Hauptwerkzeug für Angular-Projekte. Sie ermöglicht das Erstellen von Projekten, Komponenten, Services und vielem mehr.

```bash
# Angular CLI global installieren
npm install -g @angular/cli

# Installation prüfen
ng version
```

---

## 3. Neues Angular-Projekt erstellen

```bash
# Neues Projekt erstellen
ng new meine-app

# Interaktive Fragen:
# ? Would you like to add Angular routing? Yes
# ? Which stylesheet format would you like to use? CSS (oder SCSS)
```

```bash
# In den Projektordner wechseln
cd meine-app

# Entwicklungsserver starten
ng serve

# App öffnen: http://localhost:4200
```

---

## 4. Projektstruktur

Nach dem Erstellen eines neuen Projekts sieht die Struktur so aus:

```
meine-app/
├── src/
│   ├── app/
│   │   ├── app.component.ts       ← Root-Komponente (TypeScript)
│   │   ├── app.component.html     ← Root-Komponente (Template)
│   │   ├── app.component.css      ← Root-Komponente (Styles)
│   │   ├── app.component.spec.ts  ← Root-Komponente (Tests)
│   │   └── app.module.ts          ← Haupt-Modul (bei älteren Angular-Versionen)
│   │   └── app.config.ts          ← App-Konfiguration (neuere Angular-Versionen)
│   │   └── app.routes.ts          ← Routing-Konfiguration
│   ├── assets/                    ← Bilder, Fonts, etc.
│   ├── index.html                 ← Haupt-HTML-Seite
│   ├── main.ts                    ← Einstiegspunkt der Applikation
│   └── styles.css                 ← Globale Styles
├── angular.json                   ← Angular Projektkonfiguration
├── package.json                   ← npm Abhängigkeiten
├── tsconfig.json                  ← TypeScript Konfiguration
└── README.md
```

### Wichtige Dateien erklärt

#### `main.ts` – Einstiegspunkt
```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

// Startet die Angular-Applikation
bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

#### `app.component.ts` – Root-Komponente
```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',      // HTML-Tag: <app-root>
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'meine-app';
}
```

#### `app.component.html` – Root-Template
```html
<!-- app.component.html -->
<h1>Willkommen zu {{ title }}!</h1>
<router-outlet></router-outlet>
```

#### `app.config.ts` – App-Konfiguration (Standalone)
```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)]
};
```

---

## 5. Wichtige Angular CLI Befehle

### Projekt erstellen

```bash
ng new projekt-name
```

### Entwicklungsserver starten

```bash
ng serve
# oder mit automatischem Browser-Öffnen:
ng serve --open
```

### Komponente erstellen

```bash
ng generate component komponenten-name
# Kurzform:
ng g c komponenten-name

# Beispiel:
ng g c produkt-liste
# Erstellt:
# src/app/produkt-liste/produkt-liste.component.ts
# src/app/produkt-liste/produkt-liste.component.html
# src/app/produkt-liste/produkt-liste.component.css
# src/app/produkt-liste/produkt-liste.component.spec.ts
```

### Service erstellen

```bash
ng generate service services/daten
# Kurzform:
ng g s services/daten
```

### Modul erstellen (ältere Projekte)

```bash
ng generate module modul-name
# Kurzform:
ng g m modul-name
```

### Guard erstellen (für Routing-Schutz)

```bash
ng generate guard guards/auth
```

### Pipe erstellen

```bash
ng generate pipe pipes/formatierung
```

### Interface erstellen

```bash
ng generate interface models/benutzer
```

### App für Produktion bauen

```bash
ng build
# Optimierter Build für Produktion:
ng build --configuration production
# Output: dist/meine-app/
```

### Tests ausführen

```bash
# Unit Tests
ng test

# End-to-End Tests
ng e2e
```

### Alle CLI Optionen anzeigen

```bash
ng help
ng generate --help
```

---

## 6. Standalone Components vs. NgModule

Ab Angular 14/17 gibt es zwei Ansätze:

### Modernes Angular (Standalone – empfohlen seit Angular 17)

```typescript
@Component({
  selector: 'app-beispiel',
  standalone: true,          // Standalone Komponente
  imports: [CommonModule],   // Direkte Imports
  template: `<h1>Hallo!</h1>`
})
export class BeispielComponent {}
```

### Älteres Angular (NgModule-basiert)

```typescript
// app.module.ts
@NgModule({
  declarations: [AppComponent, BeispielComponent],
  imports: [BrowserModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

> 💡 **Empfehlung:** Für neue Projekte immer **Standalone Components** verwenden. Das ist der aktuelle Standard.

---

## 7. angular.json – Projektkonfiguration

Die `angular.json` enthält die wichtigsten Projekt-Einstellungen:

```json
{
  "projects": {
    "meine-app": {
      "architect": {
        "build": {
          "options": {
            "outputPath": "dist/meine-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "styles": ["src/styles.css"],
            "assets": ["src/assets"]
          }
        }
      }
    }
  }
}
```

---

## 8. package.json – Abhängigkeiten

```json
{
  "scripts": {
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test"
  },
  "dependencies": {
    "@angular/core": "^17.0.0",
    "@angular/common": "^17.0.0",
    "@angular/router": "^17.0.0",
    "rxjs": "~7.8.0"
  },
  "devDependencies": {
    "@angular/cli": "^17.0.0",
    "typescript": "~5.2.0"
  }
}
```

---

## Zusammenfassung

| Befehl | Beschreibung |
|---|---|
| `npm install -g @angular/cli` | Angular CLI installieren |
| `ng new projekt` | Neues Projekt erstellen |
| `ng serve` | Entwicklungsserver starten |
| `ng g c name` | Neue Komponente erstellen |
| `ng g s name` | Neuen Service erstellen |
| `ng build` | App für Produktion bauen |
| `ng test` | Unit Tests ausführen |

---

## Weiterführende Ressourcen

- 📖 [Angular CLI Dokumentation](https://angular.dev/tools/cli)
- 📖 [Angular Dev – Erste Schritte](https://angular.dev/tutorials/learn-angular)
- 🎥 [Angular University – Angular CLI](https://angular-university.io)
