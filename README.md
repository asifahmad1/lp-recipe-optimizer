# Linear Programming Recipe & Cost Optimizer

By **Dr. Asif Ahmad** — Asif SaaS Studio

Category **Food Science** · Version **1.0** · MIT License

Find the least-cost food formulation that meets your protein, fat and moisture targets — a genuine two-phase simplex LP solver with ingredient bounds, fat-to-solids ratio and protein-share rules, cost breakdowns and compliance checks. **100% in your browser**: no uploads, no accounts, and no data ever leaves your device.

**▶ Live on Asif SaaS Studio:** open the tools catalogue and choose *Linear Programming Recipe & Cost Optimizer* (slug `lp-recipe-optimizer`).

## Run it

Open `index.html` in any modern browser — or from this repo via any static host. No build step, no dependencies, no server. The app opens with the beef burger preset loaded and already optimised.

## What it does

1. **Real LP solver** — two-phase simplex (Bland's anti-cycling rule, tolerance ~1e-9) in a DOM-free core. Minimises cost subject to: blend total = 100%, per-nutrient min/max, fat:solids ratio min/max (linear form Σ(fatᵢ − r·solidsᵢ)xᵢ), Group A protein-share rule (Σ_A pᵢxᵢ − (q/100)·Σ pᵢxᵢ ≥ 0), and per-ingredient bounds — bounds folded in exactly by substitution (x = lb + y, y ≤ ub − lb). Returns optimal / infeasible / unbounded / error status.
2. **Ingredient table** — 2–12 editable rows: name, group (None / A / B), cost per kg (currency-agnostic), protein / fat / moisture / carbs %, computed solids % (100 − moisture), min % and max %.
3. **Targets panel** — protein, fat, moisture and carbohydrate min/max (any bound optional), fat:solids ratio min/max, Group A protein share ≥ q% of total protein, batch size in kg for scaled output.
4. **Results** — status banner; recipe table (% of blend, kg in batch, cost contribution, % of total cost) with totals; achieved composition (protein, fat, moisture, carbs, solids, ratio, Group A share) each shown against its target with a ✓/✗ mark **recomputed from the recipe**, never copied from solver internals; binding-constraint notes identifying exactly which targets and bounds are driving the cost.
5. **Infeasibility help** — exact pre-checks run before solving (ingredient minimums > 100%, maximums < 100%, and each nutrient target against its achievable min/max range given the bounds, computed by an exact greedy fill). Failing checks are named in plain language; if every single check passes but the combination clashes, the solver's infeasible verdict is shown with guidance for finding the conflicting pair.
6. **Presets (illustrative teaching data, labelled as such in-app)** — (a) Beef burger blend: beef 80/20, beef trim 50CL, pork backfat, rusk, ice water, soy protein concentrate, with Group A = meats and an 80% meat-protein share rule; (b) Protein-enriched bread flour: wheat flour, vital wheat gluten, defatted soy flour, protein ≥ 16%; (c) Teaching example: water / sugar / skim milk powder drink mix with protein and moisture targets that both bind.
7. **CSV import** — paste or file upload (.csv/.txt, read in-page): `name,cost,protein,fat,moisture,carbs,min,max,group`; header optional; comma/semicolon/tab tolerant; unreadable lines reported by line number, never silently dropped; Replace or Append into the ingredient table.
8. **Outputs** — copyable text summary and recipe CSV download. Inline validation throughout (ranges, min ≤ max, protein + fat + moisture + carbs ≤ 100% + 0.5 tolerance per ingredient, named row errors).

## Honest scope

- Linear programming assumes cost and proximate composition blend linearly — true for protein, fat, moisture and carbohydrate mass balances, and the solver's answer is **exact for the model and data entered**.
- It is **not** a sensory or texture model: flavour, bite, colour and processing behaviour enter only through the numeric constraints the user sets (bounds, ratios, group shares).
- Preset compositions and prices are illustrative teaching values, not lab data or supplier quotes; commercial formulation needs verified lab composition for the actual raw materials and current prices.
- Group B is a label only in v1.0; the protein-share constraint references Group A.
- No backend is needed or faked — everything runs in-page: no server, no storage, no API keys.

## Files

- `index.html` — the complete app, self-contained: all CSS and JavaScript inline, 49,698 bytes. No localStorage, no service workers, no browser permissions, no parent-window access — safe in a sandboxed iframe, and it works fully offline.
- `lp-recipe-optimizer-test-report.md` — the full test report, honest-scope notes and the Wix listing details.
- Documentation (PDF) → `docs/lp-recipe-optimizer-docs.pdf`

## Testing

109 automated checks run against the shipped PURE CORE — all passed. The expected optima were derived **by hand, by algebra, before the solver was run**: a 2-ingredient boundary optimum (50, 50) at 3.50/kg; a 3-ingredient vertex (70, 30, 0) at 1.90/kg with protein exactly 12; max-bound, min-bound, fat:solids-ratio and Group A share cases each landing exactly on their hand-computed vertices; degenerate duplicate ingredients solved without cycling (Bland's rule); infeasible targets named by the pre-checks; CSV parser cases including bad-line numbers; and all three presets matched to hand values. Invariants over all 9 optima: Σx = 100 to 1.42e-14, with achieved composition and cost recomputed independently to 1e-9. Full report: `lp-recipe-optimizer-test-report.md`.

## Documentation

Complete user guide and technical documentation — how LP works here, the solver, the ratio and share-rule algebra, the infeasibility guide, the burger worked example (OPTIMAL at 4.82/kg; protein 14.00%, fat 22.00%, Group A share 80.00%, three constraints binding), honest scope, and the tested core verbatim: `docs/lp-recipe-optimizer-docs.pdf`.

Created by Dr. Asif Ahmad · Asif SaaS Studio · v1.0
