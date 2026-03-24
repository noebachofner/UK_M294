# 01 – Was ist Angular?

## Lernziele
- Du weisst, was Angular ist und wofür es verwendet wird
- Du kennst den Unterschied zwischen Angular und anderen Frameworks
- Du verstehst die grundlegende Architektur einer Angular-Applikation

---

## 1. Was ist Angular?

**Angular** ist ein Open-Source Front-End Framework von **Google**, das für die Entwicklung von Single-Page Applications (SPA) verwendet wird. Es basiert auf **TypeScript** und bietet eine strukturierte Architektur für die Entwicklung grosser Web-Applikationen.

### Wichtige Fakten

| Eigenschaft | Beschreibung |
|---|---|
| Erstveröffentlichung | 2016 (Angular 2, Neuentwicklung von AngularJS) |
| Entwickler | Google |
| Sprache | TypeScript |
| Architekturmuster | Component-Based Architecture |
| Aktuelle Version | Angular 17+ |

> ⚠️ **Achtung:** AngularJS (Version 1.x) und Angular (Version 2+) sind **komplett unterschiedliche Frameworks**. Wenn von "Angular" gesprochen wird, ist meistens Angular 2+ gemeint.

---

## 2. Was ist eine Single-Page Application (SPA)?

Eine **SPA** lädt nur **einmal** eine HTML-Seite. Danach werden Inhalte **dynamisch** nachgeladen ohne die Seite neu zu laden.

```
Traditionelle Webanwendung:
Browser → HTTP Request → Server → Neue HTML-Seite → Browser zeigt neue Seite

SPA (z.B. Angular):
Browser → HTTP Request → Server → App-Shell laden
Browser → Klick auf Link → Kein HTTP Request → Inhalt wird dynamisch gerendert
```

**Vorteile einer SPA:**
- Schnellere Benutzererfahrung (kein vollständiges Neuladen)
- App-ähnliches Gefühl
- Weniger Serverlast

**Nachteile einer SPA:**
- Längere initiale Ladezeit
- SEO kann schwieriger sein (wird mit Angular SSR gelöst)

---

## 3. Angular Architektur Übersicht

```
Angular Applikation
│
├── AppModule (Root Module)
│   ├── AppComponent (Root Component)
│   │   ├── HeaderComponent
│   │   ├── MainComponent
│   │   │   ├── ProductListComponent
│   │   │   └── ProductDetailComponent
│   │   └── FooterComponent
│   │
│   ├── Services (Business-Logik, HTTP-Calls)
│   └── Router (Navigation zwischen Seiten)
```

### Wichtigste Bausteine

| Baustein | Beschreibung |
|---|---|
| **Component** | Steuert einen Teil der Benutzeroberfläche |
| **Template** | HTML-Datei einer Komponente |
| **Service** | Enthält Business-Logik und wird in Komponenten injiziert |
| **Module** | Gruppiert zusammengehörige Komponenten und Services |
| **Directive** | Erweitert HTML-Elemente mit zusätzlichem Verhalten |
| **Pipe** | Transformiert Daten in Templates |
| **Router** | Verwaltet die Navigation zwischen Ansichten |

---

## 4. Angular vs. andere Frameworks

| Kriterium | Angular | React | Vue.js |
|---|---|---|---|
| Typ | Framework (vollständig) | Library (UI) | Framework (progressiv) |
| Sprache | TypeScript | JavaScript/JSX | JavaScript |
| Lernkurve | Steil | Mittel | Flach |
| Einsatz | Grosse Enterprise-Apps | Flexibel | Klein bis mittel |
| Datenbindung | Two-Way & One-Way | One-Way | Two-Way & One-Way |

---

## 5. Wie Angular funktioniert (vereinfacht)

```
1. Browser lädt index.html
2. index.html lädt den kompilierten Angular-Code (main.js)
3. Angular bootstrapt (startet) die AppComponent
4. Angular rendert den HTML-Inhalt der AppComponent
5. Bei Benutzerinteraktion aktualisiert Angular nur die geänderten DOM-Teile
```

```html
<!-- index.html – Einstiegspunkt der App -->
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>MeineApp</title>
</head>
<body>
  <!-- Hier wird die Angular App eingebettet -->
  <app-root></app-root>
</body>
</html>
```

---

## 6. Angular Versionsverlauf (Kurzübersicht)

| Version | Jahr | Wichtige Neuerungen |
|---|---|---|
| Angular 2 | 2016 | Komplette Neuentwicklung in TypeScript |
| Angular 4 | 2017 | Kleinere Optimierungen (Version 3 wurde übersprungen) |
| Angular 6–8 | 2018–2019 | Angular Elements, Ivy Compiler (preview) |
| Angular 9 | 2020 | Ivy Compiler standardmässig aktiv |
| Angular 14 | 2022 | Standalone Components (Preview) |
| Angular 17 | 2023 | Standalone Components Standard, neue Control Flow Syntax |
| Angular 18+ | 2024 | Zoneless Change Detection, Signals stabil |

---

## Zusammenfassung

- Angular ist ein **Full-Featured Framework** für SPAs
- Basiert auf **TypeScript**
- Besteht aus **Komponenten**, **Services**, **Modulen**, **Direktiven** und **Pipes**
- Entwickelt und gepflegt von **Google**
- Ideal für **grosse, komplexe Web-Applikationen**

---

## Weiterführende Ressourcen

- 📖 [Angular Offizielle Dokumentation](https://angular.dev)
- 🎥 [Angular University – Angular Core Deep Dive](https://angular-university.io/course/angular-core-deep-dive-course)
- 📖 [Angular Dev Tutorial – Erste Schritte](https://angular.dev/tutorials/learn-angular)
