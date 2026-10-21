# Linear Programming Recipe & Cost Optimizer v1.0 — Test Report

**What it is:** A single-file browser app for the Asif SaaS Studio catalogue (R&D & Formulation Calculators) that designs least-cost food formulations. The user enters ingredients with costs, proximate composition and min/max bounds, sets target composition (protein, fat, moisture, carbs, fat:solids ratio, and a "protein from Group A" share rule), and a genuine linear-programming solver — a two-phase simplex with Bland's anti-cycling rule, implemented in the page — returns the cheapest blend meeting every constraint, with a recipe table, cost breakdown, and a compliance panel that is recomputed from the recipe (never copied from solver internals). Target users: food R&D and nutrition formulation teams, students, small producers. By Dr. Asif Ahmad.

## Features

1. **Real LP solver** — two-phase simplex (Bland's rule, tolerance ~1e-9) in a DOM-free PURE CORE. Minimises cost subject to: blend total = 100%, per-nutrient min/max, fat:solids ratio min/max (linear form Σ(fatᵢ − r·solidsᵢ)xᵢ), Group A protein-share rule (Σ_A pᵢxᵢ − (q/100)·Σ pᵢxᵢ ≥ 0), and per-ingredient bounds — bounds are folded in exactly by substitution (x = lb + y, y ≤ ub − lb). Returns optimal / infeasible / unbounded / error status.
2. **Ingredient table** — 2–12 editable rows: name, group (None / A / B), cost per kg (currency-agnostic), protein / fat / moisture / carbs %, computed solids % (100 − moisture), min % and max %.
3. **Targets panel** — protein, fat, moisture and carbohydrate min/max (any bound optional), fat:solids ratio min/max, Group A protein share ≥ q% of total protein, batch size in kg for scaled output.
4. **Results** — status banner; recipe table (% of blend, kg in batch, cost contribution, % of total cost) with totals; achieved composition (protein, fat, moisture, carbs, solids, ratio, Group A share) each shown against its target with a ✓/✗ mark recomputed from the solution; binding-constraint notes identifying exactly which targets/bounds are driving the cost.
5. **Infeasibility help** — exact pre-checks run before solving (ingredient minimums > 100%, maximums < 100%, and each nutrient target against its achievable min/max range given the bounds — computed by an exact greedy fill). Failing checks are named in plain language; if every single check passes but the combination clashes, the solver's infeasible verdict is shown with guidance for finding the conflicting pair.
6. **Presets (illustrative teaching data, labelled as such in-app)** — (a) Beef burger blend: beef 80/20, beef trim 50CL, pork backfat, rusk, ice water, soy protein concentrate, with Group A = meats and an 80% meat-protein share rule; (b) Protein-enriched bread flour: wheat flour, vital wheat gluten, defatted soy flour, protein ≥ 16%; (c) Teaching example: water / sugar / skim milk powder drink mix with protein and moisture targets that both bind.
7. **CSV import** — paste or file upload (.csv/.txt, FileReader, parsed in-page): `name,cost,protein,fat,moisture,carbs,min,max,group`; header optional; comma/semicolon/tab tolerant; unreadable lines reported by line number, never silently dropped; Replace or Append into the ingredient table.
8. **Outputs** — copyable text summary and recipe CSV download. Inline validation throughout (ranges, min ≤ max, protein + fat + moisture + carbs ≤ 100% + 0.5 tolerance per ingredient, named row errors).

## Honest scope

Linear programming assumes cost and proximate composition blend linearly — true for protein, fat, moisture and carbohydrate mass balances, and the solver's answer is **exact for the model and data entered**. It is **not** a sensory or texture model: flavour, bite, colour and processing behaviour enter only through the numeric constraints the user sets (bounds, ratios, group shares). Preset compositions and prices are illustrative teaching values, not lab data or supplier quotes; commercial formulation needs verified lab composition for the actual raw materials and current prices. No backend is needed or faked — everything runs in-page: no server, no storage, no API keys, no data leaves the browser.

## File

- `~/workspace/your_files/lp-recipe-optimizer/index.html` — **49,698 bytes** (limit 150 KB), all CSS/JS inline, no external assets, no localStorage/service workers/permissions/parent-window access. Responsive layout, labelled inputs, inline validation errors.

## How to run

Open `index.html` in any modern browser (double-click, or upload the file's contents into the Wix CMS `appHtml` field — see below). No build step, no server. The app opens with the beef burger preset loaded and already optimised.

## Testing

**Method:** the app's PURE CORE block (simplex solver, constraint builder, achieved-composition computation, achievability pre-checks, CSV parser, presets) was extracted and run in Node against problems solved **by hand** — the expected optima below were derived by algebra before the solver was run, not read off its output. The full inline `<script>` was also extracted and syntax-checked with `node --check` — **OK**.

**Results: 109 / 109 checks passed.** Maximum |Σx − 100| across all optima: **1.42e-14** (spec: 1e-9). Achieved composition and cost were additionally recomputed independently from each solution vector inside the harness and matched to 1e-9.

### Hand derivations (expected answers fixed before solving)

1. **2-ingredient boundary optimum** — A (cost 2, protein 10) + B (cost 5, protein 30), protein ≥ 20. By hand: 10a + 30b ≥ 2000 with a + b = 100 gives b ≥ 50; cost 2a + 5b falls as b falls, so the optimum is the boundary point **(50, 50), 3.50/kg**. Solver matched (x and cost to 1e-6).
2. **3-ingredient vertex** — water-like X (cost 1), Y (cost 4, protein 40), Z (cost 3, protein 10, fat 30); protein ≥ 12, fat ≤ 10. Substituting x = 100 − y − z, cost per 100 = 100 + 3y + 2z and the protein rule becomes 4y + z ≥ 120; y delivers protein at 0.75 cost-units per unit vs z at 2, so the optimum is the vertex **(70, 30, 0) at 1.90/kg**, with achieved protein exactly 12. Solver matched.
3. **Max bound active** — A (cost 1, protein 20, max 40%) + B (cost 4, protein 22), protein ≥ 21. The cap forces b ≥ 60: **(40, 60), 2.80/kg**, x_A exactly at its maximum and the bound note names it. Solver matched.
4. **Fat:solids ratio active** — P (cost 2, fat 40, solids 80) + Q (cost 3, fat 5, solids 90), ratio ≤ 0.25 → 20p − 17.5q ≤ 0 → p ≤ 46.667. Cost falls as p rises, so **(46.667, 53.333), ≈ 2.533/kg**, achieved ratio 0.25 and flagged binding. Solver matched.
5. **Group A share active** — A1 (cost 5, protein 30, group A), A2 (cost 6, protein 10, A), B (cost 1, protein 50, ungrouped); protein ≥ 20, A-share ≥ 60%. By hand a2 = 0 and 6a1 ≥ 15b with a1 + b = 100 gives b ≤ 28.571: **(71.429, 0, 28.571), ≈ 3.857/kg**, share exactly 60%. Solver matched.
6. **Min bound active** — A (cost 9, min 30%) + B (cost 2), no targets: cost pressure pins A at its minimum, **(30, 70), 4.10/kg** — verifies bound folding (x = lb + y) exactly. Solver matched.
7. **Degenerate duplicates** — two identical ingredients (cost 3, protein 20) + B (cost 6, protein 40), protein ≥ 30: hand answer cost **4.50/kg** with the duplicates totalling 50%; solver returned optimal with no cycling (Bland's rule). Matched.

### Results by group

| Group | Checks | Result |
|---|---|---|
| Case 1 — 2-ingredient boundary optimum (status, x, cost) | 4 | ✅ |
| Case 2 — 3-ingredient hand-solved vertex (+ achieved protein exact) | 6 | ✅ |
| Case 3 — max bound active (+ bound note) | 5 | ✅ |
| Case 4 — fat:solids ratio active (+ binding flag) | 6 | ✅ |
| Case 5 — Group A protein share active | 6 | ✅ |
| Case 6 — infeasible nutrient target; pre-check names Protein | 2 | ✅ |
| Case 7 — infeasible bounds Σmax < 100 and Σmin > 100, both named | 4 | ✅ |
| Case 8 — degenerate duplicate ingredients, no cycling | 3 | ✅ |
| Case 9 — min bound pinned via substitution | 3 | ✅ |
| Case 10 — jointly infeasible targets (solver-level, pre-checks empty) | 2 | ✅ |
| Case 11 — raw simplex: unbounded detection; plain equality problem | 4 | ✅ |
| Case 12 — achievableRange greedy vs hand values (open and bounded) | 4 | ✅ |
| Case 13 — CSV parser (header, semicolons, tabs, defaults, bad-line numbers 3 & 4, empty, group B) | 7 | ✅ |
| Case 14 — presets vs hand values (flour cost 0.661, soy 12.2%; teaching cost 1.678, milk 27.8%, water 19.2%, carbs 67.5; burger feasible + share ≥ 80%) | 12 | ✅ |
| Invariants over all 9 optima — Σx = 100, independent recompute of protein & cost to 1e-9, all compliance checks ok | 36 | ✅ |
| Syntax — `node --check` on the full inline script | 1 | ✅ |
| **Total** | **109** | ✅ |

## Known limitations

- At most 12 ingredients and the listed constraint types in this version; no integer/binary variables (e.g. "use either/or" rules), no multi-period or inventory modelling.
- Sensory quality, texture, colour and process behaviour are not modelled — only the numeric constraints the user sets.
- Group B is a label only in v1.0; the protein-share constraint references Group A. (A second share rule is a natural v1.1.)
- Costs are single-period point values; no price-break quantities, MOQs, or supplier minimum-order constraints beyond the % bounds.
- Extremely ill-conditioned hand-entered data (e.g. near-duplicate ingredients with wildly different costs) is handled by Bland's rule and a 20,000-iteration safety cap, which reports an "error" status rather than looping.

## Wix upload steps

1. In the Wix dashboard, open the CMS collection **SaasProducts**.
2. Edit (or add) the item with slug `lp-recipe-optimizer`.
3. Paste the full contents of `index.html` into the **appHtml** field; set Title, Category (Food Science), Version 1.0, Published = on.
4. Save, open the live app page, and run the smoke test: the page loads with the **Beef burger blend** preset already optimised — expect status **OPTIMAL, 4.82 per kg** (total **481.85** for the 100 kg batch) with the recipe **Beef 80/20 37.28%, Beef trim 50CL 28.80%, Pork backfat 0%, Rusk 5.45%, Ice water 25.00%, Soy protein concentrate 3.47%** and achieved protein 14.00%, fat 22.00%, moisture 58.49%, Group A protein share 80.00% (protein, fat and the share rule all binding; ice water at its maximum). Then load the **bread flour** preset → expect **0.66 per kg** with soy flour **12.20%** and gluten 0%. Finally set the burger's protein minimum to 40 → expect **INFEASIBLE** with a pre-check naming Protein above its achievable maximum.

## Listing

- **Title:** Linear Programming Recipe & Cost Optimizer
- **Description:** Find the least-cost food formulation that meets your protein, fat and moisture targets — a real LP simplex solver with bounds, fat:solids and meat-protein share rules, cost breakdown and compliance checks.
- **Category:** Food Science
- **Slug:** `lp-recipe-optimizer`
- **Version:** 1.0
