# 02 – TypeScript Grundlagen

## Lernziele
- Du weisst, was TypeScript ist und warum es für Angular verwendet wird
- Du kennst die wichtigsten TypeScript-Features (Typen, Interfaces, Klassen, Decorators)
- Du kannst TypeScript-Code lesen und schreiben

---

## 1. Was ist TypeScript?

**TypeScript** ist eine von Microsoft entwickelte Programmiersprache, die eine **typsichere Obermenge von JavaScript** darstellt. Das bedeutet: Jeder gültige JavaScript-Code ist auch gültiger TypeScript-Code – TypeScript erweitert JavaScript jedoch um **statische Typen**.

```
JavaScript → Dynamische Typen (Fehler erst zur Laufzeit)
TypeScript → Statische Typen (Fehler bereits beim Kompilieren / im Editor)
```

### Warum TypeScript für Angular?

- Früherkennung von Fehlern im Editor
- Bessere Autovervollständigung (IntelliSense)
- Klarere Code-Struktur durch Typen und Interfaces
- Objektorientierte Features (Klassen, Interfaces, Generics)

---

## 2. Typen (Types)

### Grundlegende Typen

```typescript
// Zahlen
let alter: number = 25;
let preis: number = 9.99;

// Zeichenketten
let name: string = "Max Muster";
let begruessung: string = `Hallo, ${name}!`; // Template String

// Wahrheitswert
let istAktiv: boolean = true;

// Array
let zahlen: number[] = [1, 2, 3, 4, 5];
let namen: string[] = ["Anna", "Bob", "Clara"];

// Any (Typ unbekannt – sollte vermieden werden!)
let irgendwas: any = "könnte alles sein";

// Null / Undefined
let leer: null = null;
let nichtDefiniert: undefined = undefined;

// Void (kein Rückgabewert)
function logNachricht(): void {
  console.log("Kein Rückgabewert");
}
```

### Union Types (Mehrere mögliche Typen)

```typescript
// Eine Variable kann string ODER number sein
let id: string | number;
id = 123;
id = "abc";

// Funktion akzeptiert string oder number
function zeigeId(id: string | number): void {
  console.log(`ID: ${id}`);
}
```

### Typ-Inferenz (TypeScript erkennt Typen automatisch)

```typescript
// TypeScript erkennt, dass 'zahl' eine number ist
let zahl = 42; // Typ: number (automatisch erkannt)
// zahl = "text"; // ❌ Fehler! Kann nicht string sein
```

---

## 3. Interfaces

Ein **Interface** definiert die **Struktur eines Objekts** – es beschreibt welche Eigenschaften und Typen ein Objekt haben muss.

```typescript
// Interface definieren
interface Benutzer {
  id: number;
  name: string;
  email: string;
  alter?: number; // ? = optionale Eigenschaft
}

// Interface verwenden
const benutzer: Benutzer = {
  id: 1,
  name: "Max Muster",
  email: "max@example.com"
  // alter ist optional, muss nicht angegeben werden
};

// Funktion mit Interface als Parameter
function zeigeBenutzer(b: Benutzer): void {
  console.log(`${b.name} (${b.email})`);
}

zeigeBenutzer(benutzer);
```

### Interface für Arrays / Listen

```typescript
interface Produkt {
  id: number;
  name: string;
  preis: number;
  verfuegbar: boolean;
}

const produkte: Produkt[] = [
  { id: 1, name: "Laptop", preis: 999.99, verfuegbar: true },
  { id: 2, name: "Maus", preis: 29.99, verfuegbar: false },
];
```

---

## 4. Klassen (Classes)

Klassen sind Baupläne für Objekte. In Angular wird fast alles mit Klassen umgesetzt.

```typescript
class Tier {
  // Eigenschaften
  name: string;
  alter: number;

  // Konstruktor (wird beim Erstellen eines Objekts aufgerufen)
  constructor(name: string, alter: number) {
    this.name = name;
    this.alter = alter;
  }

  // Methode
  vorstellen(): string {
    return `Ich bin ${this.name} und bin ${this.alter} Jahre alt.`;
  }
}

// Objekt erstellen
const hund = new Tier("Bello", 3);
console.log(hund.vorstellen()); // "Ich bin Bello und bin 3 Jahre alt."
```

### Vererbung (Inheritance)

```typescript
class Hund extends Tier {
  rasse: string;

  constructor(name: string, alter: number, rasse: string) {
    super(name, alter); // Eltern-Konstruktor aufrufen
    this.rasse = rasse;
  }

  bellen(): void {
    console.log("Wuff!");
  }

  // Methode überschreiben
  vorstellen(): string {
    return `${super.vorstellen()} Ich bin ein ${this.rasse}.`;
  }
}

const meinHund = new Hund("Rex", 5, "Labrador");
console.log(meinHund.vorstellen());
meinHund.bellen();
```

### Access Modifier (Sichtbarkeit)

```typescript
class BankKonto {
  public kontonummer: string;  // von überall zugreifbar
  private guthaben: number;    // nur innerhalb der Klasse
  protected besitzer: string;  // Klasse + Unterklassen

  constructor(kontonummer: string, guthaben: number, besitzer: string) {
    this.kontonummer = kontonummer;
    this.guthaben = guthaben;
    this.besitzer = besitzer;
  }

  public getGuthaben(): number {
    return this.guthaben; // private, nur über Methode zugänglich
  }
}

const konto = new BankKonto("CH93-0076", 1000, "Max");
console.log(konto.kontonummer);  // ✅ OK
// console.log(konto.guthaben);  // ❌ Fehler! Private
console.log(konto.getGuthaben()); // ✅ OK
```

### Kurzschreibweise im Konstruktor

```typescript
// Lange Schreibweise:
class Person {
  name: string;
  alter: number;
  constructor(name: string, alter: number) {
    this.name = name;
    this.alter = alter;
  }
}

// Kurze Schreibweise (TypeScript-spezifisch):
class Person {
  constructor(public name: string, public alter: number) {}
}
```

---

## 5. Generics

Generics ermöglichen es, wiederverwendbare Funktionen/Klassen zu schreiben, die mit verschiedenen Typen funktionieren.

```typescript
// Ohne Generics – nur für eine bestimmten Typ
function ersteZahl(arr: number[]): number {
  return arr[0];
}

// Mit Generics – funktioniert mit jedem Typ
function erstesElement<T>(arr: T[]): T {
  return arr[0];
}

const ersteZahl2 = erstesElement<number>([1, 2, 3]);  // 1
const ersterName = erstesElement<string>(["Anna", "Bob"]); // "Anna"
```

---

## 6. Decorators

**Decorators** sind spezielle TypeScript-Annotationen, die Metadaten zu Klassen, Methoden oder Eigenschaften hinzufügen. In Angular werden Decorators intensiv eingesetzt.

```typescript
// Beispiel eines einfachen Decorators
function Loggable(constructor: Function) {
  console.log(`Klasse erstellt: ${constructor.name}`);
}

@Loggable
class MeineKlasse {
  // ...
}
```

In Angular sehen Decorators so aus:

```typescript
// Komponenten-Decorator
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'meine-app';
}

// Injectable-Decorator (für Services)
@Injectable({
  providedIn: 'root'
})
export class DatenService {
  // ...
}
```

---

## 7. Nützliche TypeScript-Features für Angular

### Readonly

```typescript
interface Konfiguration {
  readonly apiUrl: string; // Kann nach Erstellung nicht mehr geändert werden
  timeout: number;
}

const config: Konfiguration = { apiUrl: "https://api.example.com", timeout: 5000 };
// config.apiUrl = "andere-url"; // ❌ Fehler!
```

### Enums

```typescript
enum Status {
  Aktiv = 'AKTIV',
  Inaktiv = 'INAKTIV',
  Ausstehend = 'AUSSTEHEND'
}

let benutzerStatus: Status = Status.Aktiv;
console.log(benutzerStatus); // "AKTIV"
```

### Type Assertion (Typ-Umwandlung)

```typescript
let wert: any = "Hallo TypeScript";
let laenge: number = (wert as string).length;
// oder:
let laenge2: number = (<string>wert).length;
```

---

## 8. TypeScript kompilieren

TypeScript wird zu JavaScript **kompiliert** (transpiliert). Angular macht das automatisch.

```bash
# TypeScript global installieren
npm install -g typescript

# TypeScript-Datei kompilieren
tsc mein-skript.ts

# Output: mein-skript.js
```

---

## Zusammenfassung

| Feature | Beschreibung |
|---|---|
| **Typen** | Definiert welchen Typ eine Variable haben darf |
| **Interface** | Definiert die Struktur eines Objekts |
| **Klasse** | Bauplan für Objekte mit Eigenschaften und Methoden |
| **Vererbung** | Eine Klasse erbt von einer anderen |
| **Generics** | Wiederverwendbarer Code für verschiedene Typen |
| **Decorators** | Metadaten-Annotationen (intensiv in Angular genutzt) |

---

## Weiterführende Ressourcen

- 📖 [TypeScript Offizielle Dokumentation](https://www.typescriptlang.org/docs)
- 📖 [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- 🎥 [Angular University – TypeScript for Angular](https://angular-university.io/course/typescript-bootcamp-course)
- 🛠️ [TypeScript Playground (online ausprobieren)](https://www.typescriptlang.org/play)
