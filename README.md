# Remote NDT Ontology (RNDT)

An ontology — a structured, machine-readable vocabulary — that describes how a **remote
non-destructive testing (NDT) inspection** is carried out: who does what, at which stage, with
what equipment and software, under which procedure, and with what authority to sign off the result.

It is split into a **method-agnostic core** plus **extensions** for specific inspection methods
(Digital Radiography and Ultrasonic Testing so far), and comes with a notebook that loads example
inspection data and checks the ontology can answer a set of **competency questions**.

---

## The big picture

```
                 ┌──────────────────────────┐
                 │  rndt-core.ttl           │  generic classes & properties
                 │  (works for any method)  │  + baseline process taxonomy
                 └────────────┬─────────────┘
                   owl:imports│
             ┌────────────────┴────────────────┐
   ┌─────────▼─────────┐             ┌─────────▼─────────┐
   │ rndt-dr.ttl       │             │ rndt-ut.ttl       │  method-specific
   │ Digital Radiog.   │             │ Ultrasonic Test.  │  vocabulary
   └─────────┬─────────┘             └─────────┬─────────┘
             │                                 │
             │   rndt-data-template.xlsx  (your inspection data)
             │                 │
             │                 ▼
             │   rndt-competency-tests.ipynb  ──► runs 34 SPARQL questions
             │                 │
             ▼                 ▼
   rndt-populated-dr.ttl  rndt-populated-ut.ttl  rndt-populated-shared.ttl
```

1. **Schema** (the `.ttl` ontology files) defines *what kinds of things exist* and how they relate.
2. **Data** (the Excel template) lists *actual inspections* — components, people, images, reports.
3. **Notebook** turns the data into RDF, merges it with the schema, and asks it questions to prove
   the model works.

---

## What's in the repo

### Ontology (schema) — Turtle `.ttl` files

| File | What it is |
|---|---|
| `rndt-core.ttl` | The generic core. Classes such as `RemoteNDTProcess`, `ProcessStage`, `MajorAction`, `ActionArea`, `Activity`, `Capability`, `Participant`, `Role`, `AuthorityClass`, `Procedure`, `Software`, `Facility`, plus the properties linking them. Also holds the baseline taxonomy of stages, actions, activities and authority levels. |
| `rndt-dr.ttl` | Digital Radiography extension. Adds DR-only concepts: exposure set-up, source-to-detector geometry, image quality indicators (`IQIType`), radiographic technique. |
| `rndt-ut.ttl` | Ultrasonic Testing extension. Adds UT-only concepts: probe calibration and positioning, beam angle, scan execution, indication sizing, `UTMethod` (pulse-echo, through-transmission…) and a `Probe` class. |

Both extensions import the core and **never modify it**, so a new method (e.g. eddy current) can be
added the same way.

### How the core is structured

A remote inspection is modelled as a hierarchy:

```
RemoteNDTProcess
  └─ hasProcessStage ─► ProcessStage        (e.g. Data Acquisition, Evaluation, Oversight)
        └─ realisedThrough ─► MajorAction   (Sense / Act / Analyse / Verify / Decide …)
              └─ ActionArea                 (a group of related activities)
                    └─ Activity             (a concrete task)
                          ├─ requiresCapability ─► Capability
                          ├─ performedBy ─► Participant ─ hasRole ─► Role
                          └─ associatedWithAuthority ─► AuthorityClass
```

Around that sit the "things" used during an inspection — physical equipment, information
(images, reports, procedures, specifications, acceptance criteria), software and facilities — and
a per-inspection activity log recording who did what and when.

### Data and test harness

| File | What it is |
|---|---|
| `rndt-data-template-blank.xlsx` | Empty data template. Start here to enter your own inspections. One sheet per kind of thing (Components, Participants, Equipment, Procedures, Inspections, Images, Reports, Indications, ActivityLog…), with a `READ ME` sheet explaining each. |
| `rndt-data-template.xlsx` | The same template filled with a worked example: inspections `INSP_001`–`009` (DR) and `INSP_010` (UT). Method-specific columns live on their own sheets (`ProceduresDR`, `InspectionsUT`, `EquipmentUT`…). |
| `rndt-competency-tests.ipynb` | Jupyter notebook: loads the template, validates it, converts it to RDF, runs the 34 competency questions (23 generic, 6 DR, 5 UT) and shows pass/fail with the answers. It also re-runs the generic questions against the UT example to show they work unchanged across methods. |
| `rndt-populated-dr.ttl` / `-ut.ttl` / `-shared.ttl` | Output of the notebook: the example data as RDF, one file per method, plus a shared file for things both use (people, the governing specification). |
| `RT-UT_Term_Mapping.xlsx` | Reference spreadsheet mapping radiography (RT) terms to their ultrasonic (UT) equivalents. |

### Visualisations

| File | What it is |
|---|---|
| `rndt-core-knowledge-graph.html` | Interactive diagram of the core ontology. Open in a browser. |
| `rndt-dr-knowledge-graph.html` | Same, for the DR extension. |
| `rndt-ut-knowledge-graph.html` | Same, for the UT extension. |
| `rndt-knowledge-graph-template.html` | Shared template the three pages above are generated from (by the last cell of the notebook) — edit this, not the generated pages. |

---

## Getting started

**Just look at it:** open any `*-knowledge-graph.html` file in a web browser.

**Explore the ontology:** open `rndt-core.ttl` (and an extension) in
[Protégé](https://protege.stanford.edu/), or read it as text — every class and property has a
plain-English `rdfs:comment`.

**Run the competency questions:**

```bash
pip install jupyter rdflib openpyxl
jupyter notebook rndt-competency-tests.ipynb
```

Then *Run All*. The only cell you normally edit is the **CONFIGURATION** cell, which says which
data file to load and which schema files to merge.

**Use your own data:** copy `rndt-data-template-blank.xlsx`, fill it in, point `DATA_IN` in the
configuration cell at it, and run the notebook. Missing data doesn't crash anything — you'll just
see which questions can't be answered yet.

---

## Adding a new NDT method

1. Create `rndt-<method>.ttl` that `owl:imports` the core (copy the header of `rndt-ut.ttl`).
2. Add method-specific activities and capabilities as individuals of the core classes; reuse core
   individuals where a generic one already fits.
3. Only add new classes/properties for concepts with no core equivalent (like `Probe` or `IQIType`).
4. Add any method-specific columns to the data template as separate `…<METHOD>` sheets.
5. Add the file to `SCHEMA_FILES` in the notebook and write competency questions for it.

---

## Branches

`main` is the current version. `dr-extension`, `UT` and `combined` are older feature branches that
have all been merged into `main`.
