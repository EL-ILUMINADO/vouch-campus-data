# Vouch — Campus Data Contributions

This folder is the open-source layer of **Vouch**. It contains the trivia questions and GPS geofence data that power the campus verification system. Anyone can add their university here and submit a Pull Request.

The live UNIBEN data in [`/schools/uniben`](./schools/uniben) serves as the reference implementation.

---

## Folder Structure

```
campus-data-export/
├── template/               ← Copy this folder to add a new school
│   ├── metadata.json       ← School identity fields
│   ├── trivia.json         ← Array of multiple-choice trivia questions
│   └── geofence.json       ← GPS zones used for campus proximity checks
│
└── schools/
    └── uniben/             ← Live example (University of Benin)
        ├── metadata.json
        ├── trivia.json
        └── geofence.json
```

---

## How to Contribute Your School

### Step 1 — Fork and clone

Fork this repository, then clone your fork locally.

### Step 2 — Duplicate the template

Copy the entire `/template` folder and rename it to your school's lowercase abbreviation:

```
campus-data-export/schools/unilag/
campus-data-export/schools/oau/
campus-data-export/schools/abuad/
```

### Step 3 — Fill in `metadata.json`

```json
{
  "school_name": "University of Lagos",
  "abbreviation": "UNILAG",
  "domain": "unilag.edu.ng",
  "is_active": false
}
```

| Field | Type | Description |
|---|---|---|
| `school_name` | string | Full official name of the university |
| `abbreviation` | string | Common short form used by students (e.g. `UNILAG`) |
| `domain` | string | Official student email domain |
| `is_active` | boolean | Leave as `false` — the Vouch team sets this to `true` after review |

### Step 4 — Fill in `trivia.json`

Write **15–20 questions** that only a genuine student of that campus would know. Think hyper-local: gate names, obscure hall nicknames, faculty locations, founding history, campus slang, notable landmarks.

```json
[
  {
    "id": 1,
    "question": "Which UNILAG hall is closest to the Lagoon?",
    "options": ["Fabian Hall", "Moremi Hall", "Jaja Hall", "Angola Hall"],
    "answer": "Jaja Hall"
  }
]
```

**Rules for trivia:**

- Every question must have **exactly 4 options**.
- The `answer` value must be **an exact string match** to one of the entries in `options`.
- `id` values must be **sequential integers** starting at `1`.
- Questions must be **verifiable facts**, not opinions.
- Avoid questions whose answers change over time (e.g. "Who is the current VC?").
- Minimum **15 questions**, maximum **50 questions** per school.

### Step 5 — Fill in `geofence.json`

Define the GPS zones that cover your campus. A student passes verification if they are physically inside **any one** of the listed zones. Add a separate zone for each discrete physical site (e.g. main campus, college of medicine, satellite campus).

```json
{
  "zones": [
    {
      "label": "Main Campus (Akoka)",
      "lat": 6.5157,
      "lng": 3.3897,
      "radiusKm": 2.0
    },
    {
      "label": "College of Medicine (Idi-Araba)",
      "lat": 6.5069,
      "lng": 3.3583,
      "radiusKm": 0.8
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `label` | string | Human-readable zone name shown in verification error messages |
| `lat` | number | Latitude of the zone centre (decimal degrees, 4+ decimal places) |
| `lng` | number | Longitude of the zone centre (decimal degrees, 4+ decimal places) |
| `radiusKm` | number | Radius of the zone in kilometres. Stand at the furthest legitimate point on campus and measure the distance to the centre — use that value. The Vouch server automatically adds the device's reported GPS accuracy on top. |

**Tips for accurate coordinates:**

1. Open Google Maps, navigate to the geographic centre of the campus.
2. Long-press to drop a pin and copy the coordinates.
3. Alternatively, stand on campus with a clear sky, open your phone's GPS app, and record a reading.
4. Add ~0.2 km margin to `radiusKm` to account for GPS noise on clear-sky days.

### Step 6 — Submit a Pull Request

Push your branch and open a PR targeting `main`. Use this title format:

```
feat(campus-data): add [SCHOOL ABBREVIATION] — [Full School Name]
```

Example:
```
feat(campus-data): add UNILAG — University of Lagos
```

In the PR description, briefly mention how you verified the GPS coordinates and how many trivia questions you included.

---

## JSON Schema Reference

These interfaces are the source of truth. Your JSON must be parseable into them without modification.

```typescript
// metadata.json
interface SchoolMetadata {
  school_name: string;
  abbreviation: string;
  domain: string;
  is_active: boolean;
}

// trivia.json — array of TriviaQuestion
interface TriviaQuestion {
  id: number;
  question: string;
  options: string[];   // exactly 4 items
  answer: string;      // must match one of options exactly
}

// geofence.json
interface CampusGeofence {
  zones: CampusZone[];
}

interface CampusZone {
  label: string;
  lat: number;
  lng: number;
  radiusKm: number;
}
```

---

## Questions?

Open a GitHub Issue or reach out via the Vouch repository discussions.
