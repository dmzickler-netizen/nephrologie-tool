# CLAUDE.md — Nephro- & Kardiologie-Assistent

## Was dieses Projekt ist

Klinisches Entscheidungstool für Nephrologie und Kardiologie. Zielgruppe: Stationsärzte, die schnell gewichtsadaptierte, eGFR-adjustierte Dosierungsempfehlungen und Risikoscores am Bett brauchen.

**Deployment:** GitHub Pages — `https://dmzickler-netizen.github.io/nephrologie-tool/`
**Repository:** `https://github.com/dmzickler-netizen/nephrologie-tool.git`
**Autor:** KSchlieps / Dzickler

Alle Berechnungen laufen clientseitig — kein Backend, kein Build-Schritt, kein Framework. Die Dateien können direkt im Browser geöffnet werden.

---

## Technologie-Stack

- **Reines HTML5 / CSS3 / Vanilla JavaScript** — keine Abhängigkeiten, kein npm
- **Keine Build-Pipeline** — direktes Bearbeiten der `.html`-Dateien, dann commit + push
- **Kein Framework** (kein React, Vue, jQuery)
- **Lokale Entwicklung:** `python3 -m http.server 8080` im Projektverzeichnis, dann `http://localhost:8080`
- **Deployment:** GitHub Pages, automatisch aus `main`-Branch nach `git push`

---

## Projektstruktur

```
/Users/dzickler/Dropbox/Claude/
│
├── index.html              ← HAUPTDATEI — alle 9 Tabs integriert (~530 KB)
├── lae.html                ← Standalone LAE-Tool (Lungenarterienembolie)
├── ckd-staging.html        ← Standalone CKD-Staging + Arztbrief-Generator
├── antikoag.html           ← Standalone Antikoagulations-Tool (Referenz, nicht live)
│
├── ckd-app/
│   └── index.html          ← Ältere CKD-App (standalone, weitgehend in index.html aufgegangen)
│
├── lipid-tool/
│   └── index.html          ← Älterer standalone Lipid-Assistent
│
├── CLAUDE.md               ← Diese Datei
│
├── AKI/                    ← Referenz-PDFs: Akutes Nierenversagen
├── VHFli/                  ← Referenz-PDFs: Vorhofflimmern
├── Lipide/                 ← Referenz-PDFs: Lipidtherapie
├── LuAE/                   ← Referenz-PDFs: Lungenarterienembolie
├── Thromboagg und AK/      ← Referenz-PDFs: Antiaggregation & Antikoagulation
│
└── KDIGO-*.pdf, Plan 2-*.pdf, *.pdf   ← Leitlinien direkt im Root
```

**Die einzige Datei die im Einsatz ist: `index.html`** (enthält alle Tools). Die anderen `.html`-Dateien sind entweder Standalone-Prototypen oder experimentelle Tools.

---

## Die 9 Tabs in index.html

| data-tab | Div-ID | Emoji | Inhalt |
|---|---|---|---|
| `ckd` | `tab-ckd` | 🫘 | CKD-Staging, eGFR-Berechnung (CKD-EPI 2021), KDIGO Risikomatrix, Medikamenten-Dosisrechner, Elektrolyt-Grenzwerte, Arztbrief-Textbausteine |
| `mbd` | `tab-mbd` | 🩸 | Anämie bei CKD, Calcium-Phosphat-Stoffwechsel, MBD |
| `lipid` | `tab-lipid` | 💊 | 5-Schritt-Wizard: Lipidtherapie nach ACC/AHA 2026 |
| `diabetes` | `tab-diabetes` | 🩺 | Diabetes-Management bei CKD nach KDIGO 2022/2026 |
| `htn` | `tab-htn` | ❤️‍🩹 | Hypertonie bei CKD, KDIGO 2021 BP Guideline |
| `vhf` | `tab-vhf` | 💜 | Vorhofflimmern: CHA₂DS₂-VASc, HAS-BLED, Therapieempfehlung |
| `tah` | `tab-tah` | 🔴 | Antiaggregation: KHK, Stent, ACS, Stroke, pAVK |
| `antikoag` | `tab-antikoag` | 💉 | NMH/UFH Antikoagulation: Prophylaxe, Therapie, UFH-Perfusor, HIT II |
| `aki` | `tab-aki` | 🚨 | AKI-Staging, Ätiologie, Management |

**Reihenfolge in der Navigationsleiste (Zeile ~1344):**
CKD → Anämie/Ca-P → Lipid → Diabetes → Hypertonie → VHF → Antiaggregation → Antikoagulation → AKI

---

## JavaScript-Architektur

### Tab-Switching

```javascript
function switchToolTab(tabName) { ... }   // global, Zeile ~5698
```

Setzt alle Tab-Divs auf `display:none`, aktiviert den gewählten. Ruft beim Öffnen `window.XxxAutoImport()` auf, um Patientendaten (Gewicht, eGFR, Alter) aus anderen Tabs zu übernehmen.

### IIFE-Muster (jedes Tool isoliert)

Jedes Tool-Modul ist in einem IIFE (`(function(){ 'use strict'; ... })()`) gekapselt. Kein globaler Namespace-Konflikt. Benötigte Funktionen werden explizit als `window.xyz = function()` exponiert.

```javascript
(function(){
  'use strict';
  // private Variablen und Hilfsfunktionen
  window.akCalcProphylaxe = function() { ... };  // exponiert für onclick
  window.akAutoImport = function() { ... };       // exponiert für switchToolTab
})();
```

### Auto-Import-Muster

Wenn ein Tab geöffnet wird, wird `akAutoImport()` (oder `diabAutoImport()` etc.) aufgerufen und liest Werte aus anderen Tabs:

```javascript
var ckdW = document.getElementById('weight');       // aus CKD-Tab
var ckdEgfr = document.getElementById('egfr-result-val');
```

### Berechnungen on-the-fly

Alle Tools arbeiten `oninput`-getriggert — keine Submit-Buttons. Eingabe → sofortige Berechnung.

---

## CSS-Architektur

### Globale Shared-Klassen (für alle Tabs)

```css
.shared-header, .nav-tabs-bar, .nav-tab-btn, .nav-tab-btn.active
```

### Tool-spezifische CSS-Präfixe

Jedes Tool hat eigene Präfixe, um Konflikte zu vermeiden:

| Präfix | Tool |
|---|---|
| `diab-` | Diabetes (und als Basis für TAH/HTN wiederverwendet) |
| `ak-` | Antikoagulation (neu, vollständig in `#tab-antikoag` gescopert) |
| `tah-` | Antiaggregation |
| `vhf-` | Vorhofflimmern |
| `aki-` | AKI |
| `htn-` | Hypertonie |
| `mbd-` | Anämie / MBD |

**Wichtig:** Das Antikoagulations-CSS steht innerhalb des `<div id="tab-antikoag">` in einem `<style>`-Block, gescopert mit `#tab-antikoag .ak-*`. Alle anderen Tool-CSS stehen im `<head>`.

### Tab-Sichtbarkeit via CSS + JS

```css
/* Im <head> definiert: */
#tab-ckd { display: block; }   /* Standard: sichtbar */
#tab-lipid { display: none; }  /* alle anderen: versteckt */
```

Tab wird per `element.style.display = 'block'/'none'` in `switchToolTab()` gesteuert.

---

## Antikoagulations-Tab — Detail (zuletzt hinzugefügt)

**Position in index.html:** nach `</div><!-- end tab-tah -->`, vor `</div><!-- end tab-aki -->`
**Sub-Tabs:** Prophylaxe / Therapie NMH / UFH-Perfusor / HIT II / Info
**Datenquelle:** Plan 2 – Version 18.2 (09/2024, KSchlieps)

Sub-Tab-Switching intern über `akShowSub(id, btn)` — unabhängig von `switchToolTab`.

**Funktionen (alle exponiert als window.ak*):**
- `akSelectPInd(i)` — Prophylaxe-Indikation auswählen
- `akCalcProphylaxe()` — Prophylaxe-Empfehlung berechnen
- `akCalcTherapie()` — NMH-Therapiedosis berechnen (Fragmin + Clexane)
- `akCalcUFH()` — UFH-Perfusor-Dosierung + s.c.-Schema
- `akCalcHIT()` — HIT-II-Empfehlung + Argatra-Rechner
- `akAutoImport()` — wird von `switchToolTab('antikoag')` aufgerufen

---

## Konventionen im Code

1. **Deutsche Sprache** — alle Labels, Hinweise, Warnungen auf Deutsch
2. **Kein Framework** — jede Funktion vanilla JS
3. **IDs mit Tab-Präfix** — `ak-p-weight`, `ak-t-egfr`, `tah-age`, `diab-weight`
4. **eGFR-Stufen** — immer: >50 / 30–50 / 15–30 / <15 / HD
5. **Warnboxen** — `ak-warn`-Klasse (orange), `ak-dose-ki`-Klasse (rot) für KI
6. **Tags** — `ak-tag ak-tag-red/orange/green/blue` für inline eGFR-Badges
7. **Auto-calc** — immer `oninput="calcXxx()"` — kein Submit-Button
8. **Quellen** — jede medizinische Empfehlung im Info-Tab mit Leitlinienquelle
9. **Klinische Sicherheit** — Disclaimer-Balken auf jeder Seite, KI-Felder rot markiert

---

## Aktueller Stand

### Fertig und live

- ✅ CKD-Staging + KDIGO Risikomatrix + eGFR-Berechnung
- ✅ Medikamenten-Dosisrechner nach eGFR (CKD-Tab)
- ✅ Anämie / Calcium-Phosphat (MBD-Tab)
- ✅ Lipidtherapie 5-Schritt-Wizard (ACC/AHA 2026)
- ✅ Diabetes-Management (KDIGO 2022/2026)
- ✅ Hypertonie (KDIGO 2021 BP)
- ✅ Vorhofflimmern (CHA₂DS₂-VASc, HAS-BLED)
- ✅ Antiaggregation (KHK, Stent, ACS, Stroke, pAVK)
- ✅ Antikoagulation NMH/UFH (Prophylaxe, Therapie, Perfusor, HIT II)
- ✅ AKI (Staging, Ätiologie, Management)
- ✅ Cross-Tab Auto-Import (Gewicht/eGFR/Alter fließt zwischen Tabs)
- ✅ GitHub Pages Deployment

### Standalone-Dateien (Altbestand, nicht mehr primär)

- `ckd-staging.html` — eigenständig funktionsfähig, veraltet gegenüber index.html
- `lae.html` — LAE-Entscheidungstool, **noch nicht in index.html integriert**
- `antikoag.html` — Vorgänger-Standalone, ersetzt durch Tab in index.html
- `ckd-app/index.html`, `lipid-tool/index.html` — alte Standalone-Versionen

### Noch nicht integriert / offene TODOs

- ⬜ **lae.html** in index.html als neuen Tab integrieren (LAE-Diagnostik, Wells-Score, PESI)
- ⬜ **Glomerulonephritis / ANCA-Vaskulitis** — KDIGO 2021/2024 Leitlinie vorhanden (`KDIGO-2024-ANCA-Vasculitis-Guideline-Update.pdf`)
- ⬜ **Druckansicht** verbessern (aktuell nur CKD-Arztbrief print-optimiert)
- ⬜ Standalone-Dateien auf GitHub Pages sauber verlinken oder entfernen

---

## Nächste sinnvolle Schritte

1. **lae.html als Tab integrieren** — gleiche Methode wie alle bisherigen Tabs: neuen `data-tab="lae"` Button in Nav, `#tab-lae { display:none; }` in CSS, Tab-Content einbauen, in `switchToolTab()` ergänzen
2. **ANCA/Vaskulitis-Tab** — Basis: `KDIGO-2024-ANCA-Vasculitis-Guideline-Update.pdf` im Root
3. **Verlinkung der Standalone-Tools** — z.B. lae.html separat publizieren oder in index.html aufgehen lassen

---

## Wie man einen neuen Tab hinzufügt

**Checkliste — 4 Stellen in index.html anpassen:**

```html
<!-- 1. CSS im <head> -->
#tab-xyz { display: none; }

<!-- 2. Nav-Button (in .nav-tabs-bar, Zeile ~1344) -->
<button class="nav-tab-btn" data-tab="xyz" onclick="switchToolTab('xyz')">🔴 Name</button>

<!-- 3. Tab-Content (nach dem letzten </div><!-- end tab-xyz -->) -->
<div id="tab-xyz">
  <style> /* scoped CSS mit xyz- Präfix */ </style>
  <div class="diab-container">
    <!-- Inhalt mit diab-card, diab-row, diab-field etc. -->
  </div>
  <script>(function(){ 'use strict'; window.xyzAutoImport=function(){...}; })();</script>
</div><!-- end tab-xyz -->

<!-- 4. switchToolTab() Funktion (Zeile ~5698) -->
document.getElementById('tab-xyz').style.display = tabName === 'xyz' ? 'block' : 'none';
if (tabName === 'xyz' && window.xyzAutoImport) { window.xyzAutoImport(); }
```

---

## Medizinische Grundlagen / Leitlinien

Die Empfehlungen basieren auf diesen Quellen (PDFs im Projekt):

| Tool | Quelle |
|---|---|
| CKD-Staging | KDIGO 2021 + CKD-EPI 2021 |
| Blutdruck | KDIGO 2021 BP Guideline |
| Diabetes | KDIGO 2022 + 2026 Update (Draft) |
| Anämie/MBD | KDIGO 2026 Anemia Guideline |
| Lipide | ACC/AHA 2026 (Blumenthal et al.) |
| Antikoagulation | Plan 2 – Version 18.2 (09/2024, KSchlieps) |
| AKI | KDIGO + UpToDate-basierte Artikel |
| VHF | ESC + JAMA Reviews |
| ANCA/Vaskulitis | KDIGO 2024 |

---

## Deployment-Workflow

```bash
# Änderung in index.html machen
git add index.html
git commit -m "Kurze Beschreibung"
git push origin main
# → GitHub Pages publiziert automatisch (ca. 1–2 min)
```

**Kein Build, kein npm install, kein CI** — nur push → live.

---

## Wichtige Element-IDs im CKD-Tab (für Auto-Import)

Andere Tools lesen aus diesen IDs:

| ID | Inhalt |
|---|---|
| `weight` | Körpergewicht (kg) |
| `age` | Alter (Jahre) |
| `egfr-result-val` | Berechneter eGFR-Wert (Text) |
| `sex` | Geschlecht (radio: `male`/`female`) |

Die Antikoag-Funktionen lesen auch aus `tah-weight` (TAH-Tab) als Fallback.
