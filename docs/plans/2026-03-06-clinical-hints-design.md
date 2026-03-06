# Clinical Hints Feature (Podpowiedzi kliniczne)

## Summary

Add a toggleable clinical hints system that shows scenario-specific guidance (what to examine, diagnose, and do) as a read-only first step when a scenario is started.

## Toggle

- Toggle switch on the welcome modal, between version and author lines
- Label: "Podpowiedzi kliniczne"
- State persisted in `localStorage` (key: `hintsEnabled`), default: ON
- Only accessible via welcome modal (shown on each app launch)

## Hints Step

- When enabled, `start(k)` prepends a read-only hints step (step 0) before SAMPLE
- When disabled, flow unchanged — starts at SAMPLE as today
- Title: scenario-specific, e.g. "Wskazowki: Bol w klatce piersiowej"
- Styled distinctly (info-style background) — clearly not a form step
- Standard "Dalej" button to proceed

### Content Categories

Each hint step displays 3 categories:
- **Badanie** (Examination) — what to check physically
- **Diagnostyka** (Diagnostics) — EKG, glucose, SpO2, etc.
- **Postepowanie** (Management) — key protocols/actions

## Data Structure

Each scenario in `SC` gets a `hints` property:

```javascript
chest_pain: {
  hints: {
    badanie: ["Osluchiwanie klatki piersiowej", "Tetno na obwodzie", ...],
    diagnostyka: ["EKG 12-odprowadzeniowe", "Pomiar troponiny", ...],
    postepowanie: ["Protokol MONA", "ASA 300mg", ...]
  },
  steps: [...],
  gen: ...,
  ...
}
```

Scenarios without hints (`pocus`, `infusion`, `coffee`, `email`) skip the step automatically — no `hints` property needed.

## Affected Scenarios (16)

chest_pain, dyspnea, stroke, cardiac_arrest, hypertension, trauma, traffic, abdominal, hypoglycemia, allergy, pregnancy, fever, alcohol, psych, syncope, general

## Hint Content

Hardcoded per scenario based on medical standards. Content to be verified by the author for clinical accuracy.
