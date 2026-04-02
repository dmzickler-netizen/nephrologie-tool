# CLAUDE.md — Klinische Entscheidungstools

## Was dieses Projekt ist

Zwei klinische Entscheidungstools für den Stationsalltag. Zielgruppe: Stationsärzte, die schnell eGFR-adjustierte Dosierungsempfehlungen, Risikoscores und klinische Algorithmen am Bett brauchen.

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
├── index.html              ← HAUPTDATEI 1 — Nephrologie/Kardiologie, 9 Tabs (~530 KB)
├── crit-care.html          ← HAUPTDATEI 2 — Intensivmedizin, eigene Tabs (rot/crimson)
│
├── lae.html                ← Standalone LAE-Tool (Altbestand, aufgegangen in crit-care.html)
├── ckd-staging.html        ← Standalone CKD-Staging (Altbestand)
├── antikoag.html           ← Standalone Antikoagulations-Tool (Altbestand, ersetzt durch Tab in index.html)
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

**Zwei aktive Hauptdateien:**
- `index.html` — Nephrologie/Kardiologie (blaues Theme, 9 Tabs)
- `crit-care.html` — Intensivmedizin (rotes/crimson Theme, eigene Tabs)

---

## Tool 1: index.html — Nephrologie & Kardiologie

### Die 9 Tabs

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

**Reihenfolge in der Navigationsleiste:**
CKD → Anämie/Ca-P → Lipid → Diabetes → Hypertonie → VHF → Antiaggregation → Antikoagulation → AKI

### JavaScript-Architektur (index.html)

#### Tab-Switching

```javascript
function switchToolTab(tabName) { ... }   // global, Zeile ~5698
```

Setzt alle Tab-Divs auf `display:none`, aktiviert den gewählten. Ruft beim Öffnen `window.XxxAutoImport()` auf, um Patientendaten (Gewicht, eGFR, Alter) aus anderen Tabs zu übernehmen.

#### IIFE-Muster (jedes Tool isoliert)

Jedes Tool-Modul ist in einem IIFE (`(function(){ 'use strict'; ... })()`) gekapselt. Kein globaler Namespace-Konflikt. Benötigte Funktionen werden explizit als `window.xyz = function()` exponiert.

```javascript
(function(){
  'use strict';
  // private Variablen und Hilfsfunktionen
  window.akCalcProphylaxe = function() { ... };  // exponiert für onclick
  window.akAutoImport = function() { ... };       // exponiert für switchToolTab
})();
```

#### Auto-Import-Muster

Wenn ein Tab geöffnet wird, wird `akAutoImport()` (oder `diabAutoImport()` etc.) aufgerufen und liest Werte aus anderen Tabs:

```javascript
var ckdW = document.getElementById('weight');       // aus CKD-Tab
var ckdEgfr = document.getElementById('egfr-result-val');
```

### CSS-Architektur (index.html)

#### Tool-spezifische CSS-Präfixe

| Präfix | Tool |
|---|---|
| `diab-` | Diabetes (und als Basis für TAH/HTN wiederverwendet) |
| `ak-` | Antikoagulation (gescopert in `#tab-antikoag`) |
| `tah-` | Antiaggregation |
| `vhf-` | Vorhofflimmern |
| `aki-` | AKI |
| `htn-` | Hypertonie |
| `mbd-` | Anämie / MBD |

**Wichtig:** Das Antikoagulations-CSS steht innerhalb des `<div id="tab-antikoag">` in einem `<style>`-Block, gescopert mit `#tab-antikoag .ak-*`. Alle anderen Tool-CSS stehen im `<head>`.

### Wie man einen neuen Tab in index.html hinzufügt

**Checkliste — 4 Stellen anpassen:**

```html
<!-- 1. CSS im <head> -->
#tab-xyz { display: none; }

<!-- 2. Nav-Button (in .nav-tabs-bar) -->
<button class="nav-tab-btn" data-tab="xyz" onclick="switchToolTab('xyz')">🔴 Name</button>

<!-- 3. Tab-Content (nach dem letzten Tab-Div) -->
<div id="tab-xyz">
  <style> /* scoped CSS mit xyz- Präfix */ </style>
  <div class="diab-container">
    <!-- Inhalt -->
  </div>
  <script>(function(){ 'use strict'; window.xyzAutoImport=function(){...}; })();</script>
</div><!-- end tab-xyz -->

<!-- 4. switchToolTab() Funktion -->
document.getElementById('tab-xyz').style.display = tabName === 'xyz' ? 'block' : 'none';
if (tabName === 'xyz' && window.xyzAutoImport) { window.xyzAutoImport(); }
```

### Antikoagulations-Tab — Detail

**Sub-Tabs:** Prophylaxe / Therapie NMH / UFH-Perfusor / HIT II / Info
**Datenquelle:** Plan 2 – Version 18.2 (09/2024, KSchlieps)

**Funktionen (alle exponiert als window.ak*):**
- `akSelectPInd(i)` — Prophylaxe-Indikation auswählen
- `akCalcProphylaxe()` — Prophylaxe-Empfehlung berechnen
- `akCalcTherapie()` — NMH-Therapiedosis berechnen (Fragmin + Clexane)
- `akCalcUFH()` — UFH-Perfusor-Dosierung + s.c.-Schema
- `akCalcHIT()` — HIT-II-Empfehlung + Argatra-Rechner
- `akAutoImport()` — wird von `switchToolTab('antikoag')` aufgerufen

### Wichtige Element-IDs (für Auto-Import aus CKD-Tab)

| ID | Inhalt |
|---|---|
| `weight` | Körpergewicht (kg) |
| `age` | Alter (Jahre) |
| `egfr-result-val` | Berechneter eGFR-Wert (Text) |
| `sex` | Geschlecht (radio: `male`/`female`) |

---

## Tool 2: crit-care.html — Intensivmedizin

### Übersicht

Eigenständiges Tool (keine Verbindung zu index.html). **Rotes/Crimson-Farbschema** (`#b71c1c`). Gleiche Vanilla-JS-Architektur, aber eigenes Tab-Switching-System.

**Aktuell ein Tool aktiv:** 🫁 Lungenarterienembolie (LAE)
**Geplant (auskommentiert):** Sepsis, ARDS

### Tab-Switching (crit-care.html)

```javascript
window.switchTool = function(name) {
  document.querySelectorAll('.nav-tab-btn').forEach(function(b){ b.classList.remove('active'); });
  document.querySelector('.nav-tab-btn[data-tool="'+name+'"]').classList.add('active');
  document.getElementById('tool-lae').style.display = name === 'lae' ? 'block' : 'none';
  // weitere tools: name === 'sepsis' etc.
};
```

Nav-Buttons verwenden `data-tool` (nicht `data-tab` wie in index.html).

### LAE-Tool (crit-care.html)

**Tool-Container-ID:** `tool-lae`
**Sub-Tabs:** 🔍 Diagnose / 📊 Risiko / 💊 Therapie / 🔄 Nachsorge

**Sub-Tab-Switching:**
```javascript
window.laeShowTab = function(id, btn) {
  document.querySelectorAll('.lae-tab').forEach(function(t){ t.classList.remove('active'); });
  document.querySelectorAll('.lae-tab-btn').forEach(function(b){ b.classList.remove('active'); });
  document.getElementById('lae-' + id).classList.add('active');
  btn.classList.add('active');
};
```

**CSS-Präfix:** `lae-` (z.B. `lae-card`, `lae-tab`, `lae-check`, `lae-result`, `lae-r-low/mid/high`)

**JS-Funktionen (alle als `window.lae*` exponiert):**
- `laeShowTab(id, btn)` — Sub-Tab wechseln
- `laeCalcWells()` — Wells-Score für LAE
- `laeCalcPERC()` — PERC-Regel
- `laeCalcDDimer()` — D-Dimer-Schwellenwert-Berechnung
- `laeCalcSPESI()` — sPESI-Score
- `laeCalcRisk()` — ESC-Risikoklassifikation
- `laeCalcHIT()` — 4T-Score (HIT)
- `laeCalcCTEPH()` — CTEPH-Nachsorgerisiko
- `laeCalcBLEED()` — Blutungsrisiko
- `laeCalcNMH()` — Antikoagulationsdosis (NMH/DOAC)
- `laeCalcHESTIA()` — HESTIA-Kriterien (ambulante Therapie)
- `laeHaemoChange()` — Hämodynamik-Auswahl
- `laeRenderScoreForm()` — Score-Formular neu rendern
- `laeToggleAcc(id)` — Accordion ein-/ausklappen

### Wie man ein neues Tool in crit-care.html hinzufügt

```html
<!-- 1. Nav-Button -->
<button class="nav-tab-btn" data-tool="sepsis" onclick="switchTool('sepsis')">🔴 Sepsis</button>

<!-- 2. Tool-Container -->
<div id="tool-sepsis" style="display:none;">
  <style> /* scoped CSS mit sepsis- Präfix */ </style>
  <!-- Inhalt -->
  <script>(function(){ 'use strict'; /* JS mit sepsis* Präfix */ })();</script>
</div>

<!-- 3. switchTool() erweitern -->
document.getElementById('tool-sepsis').style.display = name === 'sepsis' ? 'block' : 'none';
```

---

## Gemeinsame Konventionen (beide Tools)

1. **Deutsche Sprache** — alle Labels, Hinweise, Warnungen auf Deutsch
2. **Kein Framework** — jede Funktion vanilla JS
3. **IDs mit Tool-Präfix** — `ak-p-weight`, `lae-perc-age`, `tah-age`
4. **eGFR-Stufen** — immer: >50 / 30–50 / 15–30 / <15 / HD
5. **Auto-calc** — immer `oninput="calcXxx()"` — kein Submit-Button
6. **Quellen** — jede medizinische Empfehlung im Info-Tab mit Leitlinienquelle
7. **Klinische Sicherheit** — Disclaimer-Balken auf jeder Seite, KI-Felder rot markiert
8. **IIFE-Kapselung** — jedes Tool in eigenem IIFE, keine globalen Variablen außer window.xyz

---

## Aktueller Stand

### Fertig und live (index.html)

- ✅ CKD-Staging + KDIGO Risikomatrix + eGFR-Berechnung
- ✅ Anämie / Calcium-Phosphat (MBD-Tab)
- ✅ Lipidtherapie 5-Schritt-Wizard (ACC/AHA 2026)
- ✅ Diabetes-Management (KDIGO 2022/2026)
- ✅ Hypertonie (KDIGO 2021 BP)
- ✅ Vorhofflimmern (CHA₂DS₂-VASc, HAS-BLED)
- ✅ Antiaggregation (KHK, Stent, ACS, Stroke, pAVK)
- ✅ Antikoagulation NMH/UFH (Prophylaxe, Therapie, Perfusor, HIT II)
- ✅ AKI (Staging, Ätiologie, Management)
- ✅ Cross-Tab Auto-Import (Gewicht/eGFR/Alter fließt zwischen Tabs)

### Fertig und live (crit-care.html)

- ✅ Lungenarterienembolie (LAE) — vollständiger klinischer Algorithmus
  - Wells-Score, PERC-Regel, D-Dimer-Berechnung
  - sPESI, ESC-Risikoklassifikation, Hämodynamik
  - Therapie (NMH/DOAC nach Körpergewicht + eGFR)
  - HESTIA (ambulante Eignung), 4T-Score (HIT)
  - Nachsorge: CTEPH-Risiko, Rezidivprophylaxe, Blutungsrisiko

### Standalone-Altbestand (nicht mehr primär)

- `lae.html` — Vorgänger des LAE-Tools, jetzt in crit-care.html aufgegangen
- `antikoag.html` — Vorgänger, ersetzt durch Tab in index.html
- `ckd-staging.html` — veraltet gegenüber index.html
- `ckd-app/index.html`, `lipid-tool/index.html` — alte Standalone-Versionen

### Offene TODOs

- ⬜ **Sepsis-Tab** in crit-care.html (Platzhalter schon vorbereitet)
- ⬜ **ARDS-Tab** in crit-care.html
- ⬜ **Glomerulonephritis / ANCA-Vaskulitis** — KDIGO 2024 Leitlinie vorhanden
- ⬜ **Druckansicht** verbessern (aktuell nur CKD-Arztbrief print-optimiert)
- ⬜ Navigation/Links zwischen index.html und crit-care.html

---

## Medizinische Grundlagen / Leitlinien

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
| LAE | ESC 2019 LAE-Leitlinie + aktuelle Reviews |

---

## Deployment-Workflow

```bash
# Änderungen machen, dann:
git add index.html crit-care.html   # oder alle relevanten Dateien
git commit -m "Kurze Beschreibung"
git push origin main
# → GitHub Pages publiziert automatisch (ca. 1–2 min)
```

**Kein Build, kein npm install, kein CI** — nur push → live.
