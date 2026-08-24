---
type: brief
engagement: perfect-competition
capability: marginal-analysis
date: 2026-08-24
status: committed
hypothesis: "20 tomato / 30 mesclun / 14 carrot beds maximizes profit: tomato's margin dominates, mesclun fills remaining bed capacity, carrots absorb the labor-cheap leftover"
---

# Perfect competition — engagement brief

## The problem

I am deciding, as the farm operator, how many beds to plant of each of three crops — tomatoes, carrots, mesclun — for the season.

What is fixed (from the marginal-analysis spec): the 36-week season, 64 total beds, $20,000 fixed costs, my own 720 field hours, up to 4 temporary workers at 1,440 hours each, and, per crop, the diminishing-returns rate, fertilizer $/bed, labor hrs/wk/bed, price $/bed, and max-beds cap. Prices and costs are fixed regardless of quantity — I'm a price taker.

What I'm choosing is the number of beds allocated to each crop, subject to the total not exceeding 64 beds and no crop exceeding its max-beds cap.

If I get the mix wrong, I leave profit on the table, or commit labor hours (mine or hired) that don't pay off in revenue.

## What I am assuming

- All values in the marginal-analysis spec (bed count, costs, prices, labor hours, diminishing-returns rates, max-beds caps) are given and fixed for this decision.
- Prices per crop are fixed regardless of quantity sold — the perfect-competition, price-taking assumption.
- Fertilizer and labor costs are fixed per bed/hour regardless of volume.

The one I'd most want to test if I had more time: the diminishing-returns rates per crop (10.00%/bed for tomatoes, 2.50%/bed for carrots, 1.25%/bed for mesclun). These are the least directly observable of the spec inputs and the mix is sensitive to them.

## Hypothesis

I expect 20 tomato beds, 30 mesclun beds, and 14 carrot beds to be the profit-maximizing mix because tomatoes carry the best margin per bed and should be planted up to their cap first; mesclun's lower fertilizer and labor cost per bed lets it fill most of the remaining bed capacity; carrots take the labor-cheap leftover capacity after tomatoes and mesclun are allocated. This guess is not derived from marginal analysis or a solver — it's my starting intuition ahead of running the model.

## How I would know I was wrong

I'd be proved wrong if solving the perfect-competition model with a solver (Excel Solver) returns a different optimal bed allocation than 20 tomato / 30 mesclun / 14 carrot.
