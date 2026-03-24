# 21 – Deployment

## Lernziele
- Du kannst eine Angular-App für die Produktion bauen
- Du weisst wie man eine App auf verschiedenen Hosting-Diensten deployt
- Du kennst wichtige Optimierungsoptionen
- Du verstehst Environment-Konfiguration

---

## 1. Production Build

```bash
# Produktions-Build erstellen
ng build

# Oder explizit:
ng build --configuration production
```

Der Build-Prozess optimiert die App:
- ✅ Code-Minifikation (kleiner)
- ✅ Tree-Shaking (unbenutzte Module werden entfernt)
- ✅ AOT-Kompilierung (Ahead-of-Time)
- ✅ Source Maps deaktiviert
- ✅ Lazy Loading optimiert

**Output:** `dist/meine-app/browser/`

---

## 2. Environments (Umgebungen)

```bash
# Environment-Dateien erstellen
ng generate environments
```

```typescript
// src/environments/environment.ts (Development)
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',
  debugModus: true,
};
```

```typescript
// src/environments/environment.production.ts (Produktion)
export const environment = {
  production: true,
  apiUrl: 'https://api.meine-app.ch/api',
  debugModus: false,
};
```

```typescript
// Verwendung im Code:
import { environment } from '../environments/environment';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private apiUrl = environment.apiUrl; // Automatisch richtige URL je nach Build

  constructor(private http: HttpClient) {}

  getDaten() {
    return this.http.get(`${this.apiUrl}/daten`);
  }
}
```

```json
// angular.json – Konfiguration
{
  "configurations": {
    "production": {
      "fileReplacements": [
        {
          "replace": "src/environments/environment.ts",
          "with": "src/environments/environment.production.ts"
        }
      ]
    }
  }
}
```

---

## 3. Hosting-Optionen

### Option A: GitHub Pages (kostenlos, statisch)

```bash
# ng-deploy für GitHub Pages installieren
ng add angular-cli-ghpages

# Deployen
ng deploy --base-href=/repository-name/
```

### Option B: Vercel (kostenlos, empfohlen)

```bash
# Vercel CLI installieren
npm install -g vercel

# Login
vercel login

# Deployen
ng build
vercel dist/meine-app/browser
```

Oder direkt über vercel.com mit GitHub-Integration.

### Option C: Netlify (kostenlos, empfohlen)

```bash
# Netlify CLI installieren
npm install -g netlify-cli

# Login
netlify login

# Deployen
ng build
netlify deploy --prod --dir=dist/meine-app/browser
```

**Netlify-Konfiguration für Angular Router:**

```toml
# netlify.toml – Alle Routen auf index.html leiten
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Option D: Firebase Hosting (Google, kostenlos)

```bash
# Firebase CLI installieren
npm install -g firebase-tools

# Login
firebase login

# Projekt initialisieren
firebase init hosting
# Public directory: dist/meine-app/browser
# Single-page app: Yes
# Automatic builds: Optional

# Deployen
ng build
firebase deploy
```

### Option E: Eigener Server (nginx)

```nginx
# /etc/nginx/sites-available/meine-app
server {
    listen 80;
    server_name meine-app.ch;
    root /var/www/meine-app/browser;
    index index.html;

    # Alle Routen auf index.html leiten (Angular Router)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Caching für statische Dateien
    location ~* \.(js|css|png|jpg|gif|ico|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 4. Wichtige Konfigurationen für Deployment

### Base Href

Wenn die App nicht im Root-Verzeichnis deployed wird:

```bash
ng build --base-href=/meine-app/
```

Oder in `angular.json`:

```json
{
  "configurations": {
    "production": {
      "baseHref": "/meine-app/"
    }
  }
}
```

### Hash-Routing (Alternative zu HTML5-History)

```typescript
// app.config.ts
import { withHashLocation } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withHashLocation())
    // URLs: /#/home, /#/produkte statt /home, /produkte
    // Vorteil: Kein Server-Config nötig
  ]
};
```

---

## 5. Angular SSR (Server-Side Rendering)

Für bessere SEO und Performance kann Angular auch serverseitig gerendert werden:

```bash
# SSR hinzufügen
ng add @angular/ssr

# Build mit SSR
ng build
ng run meine-app:server

# Server starten
node dist/meine-app/server/server.mjs
```

---

## 6. Build-Analyse

```bash
# Bundle-Analyse (zeigt welche Packages wie viel Platz belegen)
ng build --stats-json
npx webpack-bundle-analyzer dist/meine-app/browser/stats.json
```

---

## 7. Deployment-Checkliste

Vor dem Deployment prüfen:

- [ ] `ng build` ohne Fehler
- [ ] Alle API-URLs in environment.production.ts konfiguriert
- [ ] CORS auf dem Backend für die Produktions-URL konfiguriert
- [ ] `netlify.toml` / nginx-Config für SPA-Routing vorhanden
- [ ] HTTPS aktiviert (für HttpsOnly Cookies, PWA, etc.)
- [ ] `ng test` alle Tests bestanden

---

## Zusammenfassung

| Hosting | Kosten | Eignung |
|---|---|---|
| **GitHub Pages** | Kostenlos | Statische Apps ohne Backend |
| **Vercel** | Kostenlos (Basic) | SPA, gute Performance |
| **Netlify** | Kostenlos (Basic) | SPA, einfaches Setup |
| **Firebase** | Kostenlos (Spark) | Google-Ökosystem |
| **Eigener Server** | Variabel | Volle Kontrolle |

**Wichtigste Schritte:**
1. `ng build` → erstellt optimierten Build in `dist/`
2. Environments konfigurieren
3. SPA-Redirect einrichten (alle URLs → index.html)
4. Auf Hosting-Plattform deployen

---

## Weiterführende Ressourcen

- 📖 [Angular Docs – Deployment](https://angular.dev/tools/cli/deployment)
- 📖 [Angular Docs – Build Configuration](https://angular.dev/tools/cli/build)
- 📖 [Angular SSR](https://angular.dev/guide/ssr)
- 📖 [Netlify Docs](https://docs.netlify.com)
- 📖 [Vercel Docs](https://vercel.com/docs)
