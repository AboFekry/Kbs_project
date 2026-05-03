# MedExpert — Medical Diagnosis Expert System

> A rule-based clinical triage expert system built with forward chaining inference.  
> **Phase 4 implementation** of the knowledge base design specified in Phase 3.

---

## Team

| Name | ID |
|------|----|
| Hamdy Sameh | 23101383 |
| Adham Ashraf | 23101364 |
| Abdelrahman Ahmed | 23101413 |
| Ahmed Fekry | 23101358 |
| Ahmed Elsayed | 23101350 |
| Fatema Fathy | 22101180 |
| Habiba Hamada | 23101382 |

---

## Overview

MedExpert is a single-page web application that simulates a medical expert system. It collects patient symptoms, vital signs, and background information, then runs a forward-chaining inference engine over 30 production rules to identify up to 3 ranked medical conditions with urgency levels and recommended actions.

The system is a **decision-support tool only** and does not replace professional medical evaluation.

---

## Features

- **30 IF–THEN production rules** covering 6 medical conditions
- **Forward chaining inference engine** with parallel rule firing
- **Priority scoring** (PS = Base Weight + Severity Bonuses)
- **Clinical subsumption** — more specific diagnoses suppress general ones
- **Confidence scoring** per condition (rules fired / total rules × 100%)
- **Explanation facility** — WHY buttons expose fired rules and clinical reasoning
- **Inference trace log** — full step-by-step engine log, collapsible
- **Working memory display** — shows all observed and derived facts used
- **Live vital sign validation** with color-coded clinical hints
- **5-step guided UI** — Symptoms → Vitals → Background → Review → Diagnosis

---

## Getting Started

MedExpert is a self-contained HTML file with no build step or dependencies.

```bash
# Clone or download the project
git clone <repo-url>

# Open directly in a browser
open index2.html
```

No server, no npm install, no compilation required.

---

## How to Use

1. **Step 1 – Symptoms:** Select all symptoms currently present (fever, cough, chest pain, etc.)
2. **Step 2 – Vitals:** Enter numeric vitals — temperature (°C), heart rate (bpm), SpO₂ (%), blood pressure (mmHg). All fields are optional but improve accuracy.
3. **Step 3 – Background:** Enter age, gender (optional), diabetes, hypertension, and smoking status.
4. **Step 4 – Review:** Verify all entered data before running the engine.
5. **Step 5 – Diagnosis:** The inference engine fires, and results are displayed with urgency level, recommended action, and per-condition explanations.

---

## System Architecture

The system implements the classical expert system architecture defined in Phase 3:

```
Patient Input
     │
     ▼
 User Interface  ──────────────────────────────────────────┐
     │                                                      │
     ▼                                                      │
Working Memory  ◄──── Inference Engine ◄──── Knowledge Base│
(state object)        (forwardChain())       (RULES array)  │
     │                      │                              │
     ▼                      ▼                              │
buildFactSet()     Conflict Resolution                      │
                   & Priority Module                        │
                        │                                  │
                        ▼                                  │
               Explanation Facility                         │
               (fired rules, WHY)                          │
                        │                                  │
                        ▼                                  │
              Recommended Action Generator ─────────────────┘
                        │
                        ▼
                  renderResults()
```

See [Architecture Visual](#) for a full interactive diagram mapping each component to its code location.

---

## Knowledge Base

### Conditions & Rules

| Condition | Rules | Urgency Range |
|-----------|-------|---------------|
| General Fatigue | R1–R5 | LOW |
| Common Cold | R6–R10 | LOW |
| Influenza (Flu) | R11–R15 | MEDIUM |
| COVID-like Illness | R16–R20 | MEDIUM–HIGH |
| Respiratory Problem | R21–R25 | MEDIUM–HIGH |
| Possible Cardiac Issue | R26–R30 | MEDIUM–HIGH |

### Input Facts

**Clinical Symptoms (Boolean):** fever, cough, fatigue, headache, shortness of breath, chest pain, sore throat, runny nose, dizziness

**Vital Signs (Numeric):** body temperature (°C), heart rate (bpm), SpO₂ (%), blood pressure (mmHg)

**Background:** age (numeric), gender (optional), diabetes (boolean), hypertension (boolean), smoking status (optional)

### Clinical Thresholds

| Vital | Threshold | Meaning |
|-------|-----------|---------|
| SpO₂ | < 92% | Critical — Emergency risk |
| SpO₂ | < 94% | Low — monitor closely |
| Temperature | ≥ 38°C | Fever |
| Temperature | ≥ 39°C | High fever |
| Heart Rate | > 110 bpm | Tachycardia |
| Blood Pressure | ≥ 140 mmHg systolic | High Stage 2 |

---

## Inference Engine

### Algorithm: Forward Chaining

```
facts = buildFactSet()           // collect all patient data
for each rule in RULES:
    if rule.test(facts) == true:
        fire rule → add condition to PCS
        log firing event

for each condition in PCS:
    PS = URGENCY_WEIGHT[urgency] + Σ(severity bonuses)
    confidence = (rules_fired / total_rules) × 100%

apply subsumption rules
filter confidence < 10%
sort by PS descending
output top 3 conditions
```

### Priority Scoring

```
PS = Base Weight + Σ Severity Bonuses

Base Weights:   CRITICAL=100, HIGH=75, MEDIUM=50, LOW=25
Severity Bonus: +5 each for low SpO₂, high HR, high temp,
                chest pain, shortness of breath
```

### Conflict Resolution (in order)

1. **Urgency Override** — CRITICAL always dominates
2. **Clinical Subsumption** — specific conditions suppress general ones
   - COVID-like Illness subsumes Influenza (Flu)
   - Respiratory Problem subsumes Common Cold
   - Possible Cardiac Issue subsumes General Fatigue
   - COVID-like Illness subsumes Common Cold
3. **Priority Score** — higher PS ranked first
4. **Tie-breaking** — more supporting rules wins; deterministic fallback

### Output Limits

- Maximum **3 conditions** displayed
- Conditions with **confidence < 10%** are hidden

---

## Urgency Levels & Actions

| Urgency | Base Score | Recommended Action |
|---------|------------|--------------------|
| CRITICAL | 100 | Go to emergency immediately |
| HIGH | 75 | Seek urgent medical care |
| MEDIUM | 50 | Visit a doctor soon |
| LOW | 25 | Rest and monitor symptoms |

---

## Code Structure

All logic lives in `index2.html` as a single-file application.

| Code Section | Purpose | Architecture Component |
|---|---|---|
| `const state = { symptoms, vitals, background }` | Patient working memory | Working Memory |
| `const RULES = [...]` (R1–R30) | 30 IF–THEN production rules | Knowledge Base |
| `const SUBSUMPTION_RULES` | Clinical subsumption pairs | Conflict Resolution Module |
| `const URGENCY_WEIGHT`, `ACTION_PRIORITY` | Scoring weights | Priority Scoring Module |
| `buildFactSet()` | Assembles derived facts from state | Input Fact Layer |
| `forwardChain(facts)` | Main inference loop | Inference Engine |
| `renderResults()` | Displays output with fired rules | UI + Explanation Facility |
| `checkVital()` | Live clinical hint validation | User Interface |
| `initSymptoms()` | Renders symptom selection grid | User Interface |
| `CONDITION_WHY` | WHY explanations per condition | Explanation Facility |
| `ACTIONS` | Maps urgency to actionable guidance | Recommended Action Generator |
| `toggleWhy()`, `toggleTrace()` | Explanation panel controls | Explanation Facility |
| `resetSystem()` | Clears working memory and UI | Working Memory Reset |

---

## Disclaimer

This system is a **decision-support tool only** and does not replace professional medical diagnosis. Always consult a qualified healthcare provider for proper evaluation and treatment. In emergencies, call your local emergency services immediately (Egypt: 123).
