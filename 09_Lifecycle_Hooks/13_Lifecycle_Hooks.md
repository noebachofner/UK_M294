# 13 – Lifecycle Hooks

## Lernziele
- Du kennst alle wichtigen Lifecycle Hooks einer Angular-Komponente
- Du weisst wann welcher Hook aufgerufen wird
- Du kannst Hooks korrekt für Initialisierung und Aufräumen verwenden

---

## 1. Was sind Lifecycle Hooks?

Angular-Komponenten durchlaufen einen **Lebenszyklus** von der Erstellung bis zur Zerstörung. Mit **Lifecycle Hooks** kann man in diese Phasen eingreifen.

### Lifecycle einer Komponente

```
1. constructor()          → Klasse wird instanziiert
2. ngOnChanges()          → Input-Werte ändern sich
3. ngOnInit()             → Komponente initialisiert
4. ngDoCheck()            → Eigene Change Detection
5. ngAfterContentInit()   → ng-content projiziert
6. ngAfterContentChecked()→ ng-content geprüft
7. ngAfterViewInit()      → View initialisiert
8. ngAfterViewChecked()   → View geprüft
   (2–8 wiederholen sich bei Änderungen)
9. ngOnDestroy()          → Komponente wird zerstört
```

---

## 2. Wichtigste Lifecycle Hooks

### ngOnInit – Initialisierung (am häufigsten genutzt)

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `<p>{{ daten }}</p>`
})
export class DemoComponent implements OnInit {
  daten = '';

  constructor(private service: DatenService) {
    // constructor: Nur für DI und einfache Initialisierung
    // KEIN HTTP-Calls oder komplexe Logik hier!
    console.log('constructor aufgerufen');
  }

  ngOnInit(): void {
    // ngOnInit: Hier Daten laden und Logik ausführen
    console.log('ngOnInit aufgerufen');
    this.daten = this.service.getDaten();
  }
}
```

> 💡 **Faustregel:** `constructor` nur für Dependency Injection. Alles andere in `ngOnInit`.

### ngOnChanges – Reagiert auf Input-Änderungen

```typescript
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-kind',
  standalone: true,
  template: `<p>{{ wert }}</p>`
})
export class KindComponent implements OnChanges {
  @Input() wert: string = '';

  ngOnChanges(changes: SimpleChanges): void {
    // Wird aufgerufen wenn sich ein @Input() Wert ändert
    console.log('Änderungen:', changes);

    // Prüfen welche Input-Werte sich geändert haben
    if (changes['wert']) {
      console.log('Alter Wert:', changes['wert'].previousValue);
      console.log('Neuer Wert:', changes['wert'].currentValue);
      console.log('Erstes Mal:', changes['wert'].firstChange);
    }
  }
}
```

### ngOnDestroy – Aufräumen (wichtig!)

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription, interval } from 'rxjs';

@Component({
  selector: 'app-timer',
  standalone: true,
  template: `<p>Zeit: {{ sekunden }}s</p>`
})
export class TimerComponent implements OnInit, OnDestroy {
  sekunden = 0;
  private subscription!: Subscription;
  private intervalId!: ReturnType<typeof setInterval>;

  ngOnInit(): void {
    // Observable abonnieren
    this.subscription = interval(1000).subscribe(() => {
      this.sekunden++;
    });

    // Oder mit setInterval
    this.intervalId = setInterval(() => {
      console.log('Ticker...');
    }, 5000);
  }

  ngOnDestroy(): void {
    // WICHTIG: Subscriptions und Timer aufräumen!
    // Verhindert Memory Leaks
    this.subscription?.unsubscribe();
    clearInterval(this.intervalId);
    console.log('TimerComponent zerstört!');
  }
}
```

### ngAfterViewInit – Nach View-Initialisierung

```typescript
import { Component, AfterViewInit, ViewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-demo',
  standalone: true,
  template: `
    <input #meinInput type="text">
    <canvas #meinCanvas></canvas>
  `
})
export class DemoComponent implements AfterViewInit {
  @ViewChild('meinInput') inputElement!: ElementRef;
  @ViewChild('meinCanvas') canvasElement!: ElementRef;

  ngAfterViewInit(): void {
    // View ist jetzt vollständig initialisiert
    // ViewChild-Elemente sind jetzt verfügbar
    this.inputElement.nativeElement.focus(); // Input fokussieren

    // Canvas-Operationen
    const canvas = this.canvasElement.nativeElement as HTMLCanvasElement;
    const ctx = canvas.getContext('2d');
    // ... zeichnen
  }
}
```

---

## 3. Vollständiges Beispiel

```typescript
import {
  Component, Input, OnInit, OnChanges, OnDestroy,
  AfterViewInit, SimpleChanges
} from '@angular/core';
import { Subscription, interval } from 'rxjs';
import { AsyncPipe } from '@angular/common';

@Component({
  selector: 'app-lifecycle-demo',
  standalone: true,
  template: `
    <p>Input: {{ inputWert }}</p>
    <p>Zähler: {{ zaehler }}</p>
  `
})
export class LifecycleDemoComponent implements OnInit, OnChanges, OnDestroy, AfterViewInit {
  @Input() inputWert = '';
  zaehler = 0;
  private sub!: Subscription;

  constructor() {
    // Phase 1: Konstruktor – nur DI
    console.log('1. constructor');
  }

  ngOnChanges(changes: SimpleChanges): void {
    // Phase 2: Input-Werte haben sich geändert
    console.log('2. ngOnChanges', changes);
  }

  ngOnInit(): void {
    // Phase 3: Komponente ist initialisiert
    console.log('3. ngOnInit');
    this.sub = interval(1000).subscribe(() => this.zaehler++);
  }

  ngAfterViewInit(): void {
    // Phase 7: View ist bereit
    console.log('7. ngAfterViewInit');
  }

  ngOnDestroy(): void {
    // Letzte Phase: Aufräumen!
    console.log('9. ngOnDestroy');
    this.sub?.unsubscribe();
  }
}
```

---

## 4. Häufige Fehler und Best Practices

### ❌ HTTP-Call im Constructor

```typescript
export class FalschComponent {
  constructor(private service: ApiService) {
    // ❌ Falsch! HTTP-Calls gehören in ngOnInit
    this.service.getDaten().subscribe(...);
  }
}
```

### ✅ HTTP-Call in ngOnInit

```typescript
export class RichtigComponent implements OnInit {
  constructor(private service: ApiService) {}

  ngOnInit(): void {
    // ✅ Richtig! HTTP-Calls in ngOnInit
    this.service.getDaten().subscribe(...);
  }
}
```

### ❌ ViewChild in ngOnInit verwenden

```typescript
export class FalschComponent implements OnInit {
  @ViewChild('myEl') element!: ElementRef;

  ngOnInit(): void {
    // ❌ Falsch! Element ist in ngOnInit noch nicht verfügbar
    this.element.nativeElement.focus();
  }
}
```

### ✅ ViewChild in ngAfterViewInit verwenden

```typescript
export class RichtigComponent implements AfterViewInit {
  @ViewChild('myEl') element!: ElementRef;

  ngAfterViewInit(): void {
    // ✅ Richtig! Element ist jetzt verfügbar
    this.element.nativeElement.focus();
  }
}
```

### ❌ Subscription nicht aufräumen

```typescript
export class FalschComponent implements OnInit {
  ngOnInit(): void {
    // ❌ Subscription wird nie beendet → Memory Leak!
    interval(1000).subscribe(n => console.log(n));
  }
}
```

### ✅ Subscription in ngOnDestroy aufräumen

```typescript
export class RichtigComponent implements OnInit, OnDestroy {
  private sub!: Subscription;

  ngOnInit(): void {
    this.sub = interval(1000).subscribe(n => console.log(n));
  }

  ngOnDestroy(): void {
    // ✅ Richtig! Subscription aufräumen
    this.sub.unsubscribe();
  }
}
```

---

## 5. Modern: takeUntilDestroyed (Angular 16+)

Ab Angular 16 gibt es eine einfachere Methode, Subscriptions automatisch aufzuräumen:

```typescript
import { Component, OnInit } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { interval } from 'rxjs';

@Component({
  selector: 'app-modern',
  standalone: true,
  template: `<p>{{ zaehler }}</p>`
})
export class ModernComponent implements OnInit {
  zaehler = 0;

  constructor(private destroyRef = inject(DestroyRef)) {}

  ngOnInit(): void {
    // Subscription wird automatisch beim Zerstören der Komponente beendet
    interval(1000)
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(() => this.zaehler++);
  }
}
```

---

## Zusammenfassung

| Hook | Wann? | Typische Verwendung |
|---|---|---|
| `constructor` | Klasse instanziiert | Dependency Injection |
| `ngOnChanges` | @Input ändert sich | Auf Input-Änderungen reagieren |
| `ngOnInit` | Nach Initialisierung | HTTP-Calls, Daten laden |
| `ngAfterViewInit` | View bereit | ViewChild-Zugriff, DOM-Operationen |
| `ngOnDestroy` | Komponente zerstört | Subscriptions aufräumen |

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Lifecycle Hooks](https://angular.dev/guide/components/lifecycle)
