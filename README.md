# Valentine Project — Technical Architecture

## File Structure
```
Valentine/
├── index.html          # Einzige Datei — alles inline (CSS + JS)
├── images/
│   ├── valentine1.jpg
│   ├── valentine2.jpg
│   ├── valentine3.jpeg
│   ├── valentine4.jpeg
│   ├── valentine5.jpg
│   ├── valentine6.jpg
│   ├── valentine7.jpg
│   ├── valentine8.jpg
│   ├── valentine9.jpg
│   └── valentine-bg.jpeg   # Hintergrundbild für Celebration-Screen
```

## External Dependencies (CDN)
- **Google Fonts:** Roboto (UI), Dancing Script (Envelope/Cards), Great Vibes (Celebration Title)
- **canvas-confetti v1.9.3:** Herz-Konfetti auf dem Celebration-Screen

## Screen-Flow (5 Screens)

### Screen 0 — Envelope (`#screen0`)
- Rosa Hintergrund mit animiertem Briefumschlag
- Empfängerin: "Anastasia Otto" / "c/o wifey"
- Tap öffnet den Umschlag → Brief erscheint mit "Will you be my Valentine?"
- "Yes! 💕" Button → weiter zu Screen 1
- Kein "No" Button (Spoiler-Text darunter)

### Screen 1 — Fake reCAPTCHA Checkbox (`#screen1`)
- Sieht aus wie echtes Google reCAPTCHA
- Klick auf Checkbox → Spinner dreht sich (1.2s) → Checkmark
- Automatischer Übergang zu Screen 2 nach 400ms

### Screen 2 — Image Grid (`#screen2`)
- Fake CAPTCHA: "Select all images with **your Valentine**"
- 3x3 Grid mit 9 zufällig angeordneten Bildern (Fisher-Yates Shuffle)
- Bilder klicken → rosa Overlay mit Herz-Animation (heartPop)
- **Alle 9 müssen ausgewählt werden** → "Verify" Button
- Falsche Auswahl → Shake-Animation + "Please try again 💕"
- Richtige Auswahl → grünes Checkmark-Overlay → weiter zu Screen 3

### Screen 3 — Valentine Celebration (`#screen3`)
- Vollbild mit Hintergrundbild (`valentine-bg.jpeg`) + dunklem Overlay (45% schwarz)
- Titel: "Happy Valentine's Day Hasi! 💕" (Great Vibes Font)
- Nachricht: "Verification complete: Turns out every picture of me is a picture of someone who's crazy about you."
- Herz-Konfetti läuft durchgehend (von links und rechts, `requestAnimationFrame` Loop)
- "Pick our date spot 💌" Button → stoppt Konfetti → weiter zu Screen 4

### Screen 4 — Restaurant Selection (`#screen4`)
- Rosa Gradient-Hintergrund
- 3 Restaurant-Karten mit animiertem Einblenden (gestaffelt 0.2s/0.4s/0.6s):
  1. **Le Chaudron** — Google Maps Link
  2. **Le Cafe Brun** — Google Maps Link
  3. **Brulot** — Google Maps Link
- "Choose 💕" Button öffnet `mailto:leon.brunner@patterno.de` mit Restaurant-Name
- Bestätigungstext erscheint: "[Name] — great choice! 💕"
- Kein Konfetti auf diesem Screen

## JavaScript-Architektur

Alles in einer IIFE `(function() { ... })()` gekapselt.

### Konfiguration
```js
const TOTAL_IMAGES = 9;
const SPINNER_DELAY = 1200;      // Checkbox-Spinner Dauer
const SUCCESS_DELAY = 1500;      // Grünes Checkmark Dauer
const IMAGE_EXT = {1:'jpg',2:'jpg',3:'jpeg',4:'jpeg',5:'jpg',6:'jpg',7:'jpg',8:'jpg',9:'jpg'};
```

### Wichtige Funktionen
| Funktion | Beschreibung |
|---|---|
| `handleCheckboxClick()` | Spinner → Checkmark → Screen 2 |
| `buildGrid()` | Erstellt 3x3 Grid mit zufälliger Bildanordnung |
| `fisherYatesShuffle(arr)` | Zufällige Array-Permutation |
| `toggleCell(cell, idx)` | Bild auswählen/abwählen |
| `launchConfetti()` | Startet Herz-Konfetti Loop (`confettiRunning = true`) |
| `stopConfetti()` | Stoppt Loop + `confetti.reset()` räumt Canvas auf |
| `showScreen(target)` | Entfernt `.active` von allen Screens, setzt auf target |

### Confetti-Mechanismus
- `confettiRunning` Flag steuert die `requestAnimationFrame` Loop
- Zwei Emitter: links (angle 60°, drift +0.5) und rechts (angle 120°, drift -0.5)
- Herz-Shape via `confetti.shapeFromText({ text: '❤️', scalar: 2 })`
- Farben: `#e91e90, #ff6b9d, #ff1744, #f50057, #ff4081`
- `stopConfetti()` setzt Flag auf false + `confetti.reset()` entfernt alle Partikel sofort

## CSS-Highlights
- Alle Screens sind `.screen` Divs, nur `.screen.active` wird angezeigt (`display: flex`)
- Mobile-first: `min-height: 100dvh` für korrekte Mobile-Viewport-Höhe
- `clamp()` für responsive Font-Größen (Titel, Nachricht)
- Animationen: `fadeInUp`, `shake`, `heartPop`, `scaleIn`, `pulse`, `spin`
- Touch-optimiert: `min-height: 44px` auf allen Buttons, `-webkit-tap-highlight-color: transparent`

## Anpassbare Stellen
| Was | Wo (ca. Zeile) |
|---|---|
| Empfängerin Name | `Anastasia Otto` |
| Empfängerin Untertitel | `c/o wifey` |
| Valentine-Frage | `Will you be my Valentine?` |
| Celebration Titel | `Happy Valentine's Day Hasi!` |
| Celebration Nachricht | Screen 3 `.valentine-message` |
| Restaurant-Namen & Links | Screen 4 `.restaurant-card` Blöcke |
| Email-Empfänger | `mailto:leon.brunner@patterno.de` |
| Bild-Dateierweiterungen | `IMAGE_EXT` Objekt im JS |
| Hintergrundbild | `images/valentine-bg.jpeg` |

## Deployment
- **Live:** https://valentine-patterno.vercel.app
- **GitHub:** https://github.com/lmb-patt/Valentine
- Deploy: `npx vercel --prod` (nicht mit Git verbunden)
