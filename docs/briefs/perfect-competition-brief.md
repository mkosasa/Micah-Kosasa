---
type: brief
engagement: perfect-competition
capability: marginal-analysis
date: 2026-09-01
status: committed
"10 tomato / 30 mesclun / 20 carrot beds maximizes profit: tomato's 2.50 hrs/wk/bed labor requirement compounds at 10%/bed and outpaces its $8,800 price well before the cap, while mesclun's (1.25 hrs/wk/bed) and carrots' (0.833 hrs/wk/bed) lower labor requirements and shallower compounding (1.25%/bed and 2.5%/bed) let their marginal beds keep paying off out to their caps"
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

Tomatoes' 10.00%/bed diminishing-return rate compounds with each additional bed, so the cost of the marginal bed grows geometrically rather than staying flat. By the 20th bed, that compounding puts the labor cost at roughly 1.10²⁰ — about 6.7 times the labor cost of the first bed. Tomatoes' $8,800 price per bed is the highest of the three crops, but a price fixed by assumption cannot absorb an unbounded cost multiplier: at some bed count before the cap of 20, the marginal tomato bed's inflated labor cost (on top of its already-highest base labor requirement of 2.50 hrs/wk/bed) will exceed what that fixed $8,800 covers. Where exactly that crossover falls is what the model needs to solve for — it isn't something a ranking by headline price can answer.

Carrots and mesclun face the same compounding mechanism but at much shallower rates (2.5%/bed and 1.25%/bed respectively), so their marginal bed cost grows far more slowly — over a comparable 20-bed span, carrots' multiplier would be roughly 1.025²⁰ ≈ 1.64x, versus tomatoes' ~6.7x. This is the mechanical reason low diminishing-return rates let a crop absorb higher volume before the marginal bed stops paying for itself, not simply that "less affected crops should be planted more."

On fixed per-bed costs: carrots have the lowest fertilizer cost at $440/bed. Mesclun's fertilizer cost is $880/bed — the same as tomatoes, not lower — so mesclun's cost advantage over tomatoes is entirely in labor (1.25 hrs/wk/bed vs. 2.50 hrs/wk/bed) and in its much gentler diminishing-return rate, not in fertilizer. Carrots hold the lowest cost on both labor (0.833 hrs/wk/bed) and fertilizer, which is why their marginal bed stays cheap to add even as their diminishing-return rate (2.5%/bed) is higher than mesclun's.

## Hypothesis

I expect 10 tomato beds, 30 mesclun beds, and 20 carrot beds to be the profit-maximizing mix, but I hold this loosely because it is not yet backed by the marginal calculation. The reasoning is that tomatoes' compounding 10%/bed rate makes the marginal bed's labor cost roughly 6.7 times the first bed's by bed 20, and I expect that growth to outpace the fixed $8,800 price well before the cap — which is why I don't expect tomatoes to be planted to their full cap of 20. Carrots and mesclun, with much shallower compounding (2.5%/bed and 1.25%/bed) and lower fixed costs (carrots on fertilizer and labor; mesclun on labor only, since its fertilizer cost matches tomatoes' at $880/bed), should be able to absorb more beds before their marginal bed stops paying for itself, which is why I expect them to be planted more heavily. This guess is not derived from marginal analysis or a solver — it's my starting intuition ahead of running the model, and the actual crossover points for each crop are what the model needs to determine.

## How I would know I was wrong

If the model showed tomatoes planted at or near their cap of 20, that would mean the compounding 10%/bed labor penalty does not actually outpace the $8,800 price the way I've argued — i.e., even at ~6.7x the marginal labor cost of the first bed, the fixed price still covers it. That would falsify the specific mechanism I'm relying on, not just the headline allocation, and would mean tomatoes' high price per bed dominates the compounding cost penalty across the full range rather than only at low volume.

If carrots did not reach their max cap while tomatoes fell short of theirs, that would undercut my claim that carrots' shallow 2.5%/bed compounding and low fixed costs (lowest labor and fertilizer of the three crops) let their marginal bed keep paying for itself out to the cap. It would mean some other cost or constraint — not the diminishing-returns rate I've been treating as central — is driving carrots' allocation.

If mesclun could not be planted at its cap at all, that would mean I misunderstood the labor limits on the plantings, independent of the diminishing-returns argument.

I would not consider the hypothesis wrong if the resulting bed counts were similar to my guess, with tomato, mesclun, and carrot counts each shifting by 3 or fewer in either direction — that range would confirm the compounding-cost mechanism is doing the work I think it's doing, while letting the model pin down the exact crossover points more precisely than my estimate could.
