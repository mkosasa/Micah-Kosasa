# Farm Model Analysis

## Result

The profit-maximizing mix is 10 tomato beds, 20 carrot beds and 30 mesclun beds (`Optimization!B4:B6`). That uses 60 of the farm's 64 beds and earns a season profit of $42,761.66 after $20,000 in fixed costs (`CostStructure!B21`). The carrot and mesclun max-bed caps are the only binding limits; tomatoes, the 64-bed total and labor hours all have room to spare. Removing every limit adds only $2,249.13, because diminishing returns shrink each extra bed's contribution quickly.

## Why tomatoes stop at 10 beds

Tomatoes earn the most per bed, especially on the first few beds, but their 10% diminishing-returns rate (`Inputs!B18`) drives marginal cost up quickly. The 10th bed costs $8,248.59 (`MCSchedules!H15`), below the $8,800 price (`Inputs!B21`), so it adds $551.41. The 11th bed costs $9,390.72 (`MCSchedules!H16`), $590.72 above the price, so it loses money (Figure 1).

Profit shows the same turn. It rises from $25,621.36 at 9 beds to $26,172.77 at 10, then falls to $25,582.06 at 11 (`MCSchedules!K14:K16`, before fixed costs). Tomatoes therefore stop at 10 beds even though their cap is 20 (`Inputs!B22`), and relaxing the tomato cap is worth $0 (`Summary!K36`).

**Figure 1** — Marginal cost of each bed against price, for each crop on its own. The dotted line marks the beds planted in the solved mix (10 tomato, 20 carrot, 30 mesclun). Tomato marginal cost crosses the $8,800 price between bed 10 and bed 11. Data: `MCSchedules` and `Optimization!B4:B6`.

![Marginal cost vs price for each crop, with the beds planted marked](figures/perfect-competition-mc-vs-price.png)

## The dip in tomato marginal cost at bed 6

Tomato marginal cost rises from $4,317.50 at bed 1 to $7,660.86 at bed 5, then drops to $4,906.28 at bed 6 (`MCSchedules!H6`, `H10`, `H11`; Figure 2). The cause is the labor rate. By bed 5, cumulative labor reaches 724.73 hours (`MCSchedules!B10`), just past my own 720 hours (`Inputs!B7`), so the work shifts from me at $34.72 an hour (`Inputs!B9`) to temporary workers at $17.36 an hour (`Inputs!B12`). Halving the wage resets the marginal-cost curve to a lower starting point.

Without that switch, tomatoes would stop much earlier. If I had more hours of my own, or had to hire a second worker at my rate, every hour would cost $34.72. The 6th bed would then cost about $8,932.55, above the $8,800 price, and tomatoes would stop at 5 beds (derived).

**Figure 2** — Tomato marginal cost (top) and cumulative labor hours (bottom), beds 1 to 12. The dotted line is bed 5, where cumulative hours first pass my 720 hours; marginal cost then falls from $7,660.86 at bed 5 to $4,906.28 at bed 6. Data: `MCSchedules`, `Inputs!B7`.

![Tomato marginal cost and cumulative labor hours by bed](figures/perfect-competition-tomato-mc-and-hours.png)

## Binding constraints and shadow prices

Carrots (20 beds) and mesclun (30 beds) are both planted to their caps (`Inputs!C22`, `Inputs!D22`), and the model names those caps as the binding limit (`Optimization!B29`). The other limits have slack:

- **Beds:** 60 of 64 used, 4 unused (`Optimization!E14`).
- **Labor:** 5,277.22 of 6,480 hours used, 1,202.78 unused (`Optimization!E22`). Temporary labor equals 3.16 of the 4 available workers (`Summary!B18`).

The shadow prices, the value of one more bed under each cap, are $352.49 for carrots and $246.47 for mesclun (`Summary!K37:K38`); tomatoes and the slack limits are worth $0. Only the first extra bed is worth the full shadow price, and each bed after it is worth less (carrots: $352.49, then $298.09, then $241.77; `Unconstrained!I81:I83`).

## Scenario A: all limits removed

The `Unconstrained` tab removes the crop caps, the 64-bed total and the worker limit. It answers the question I set out to test: if carrots and mesclun expand, do I run out of labor? I do not.

- **Beds:** 10 tomato, 26 carrot, 37 mesclun, 73 in total (`Unconstrained!B7:E7`). Tomatoes do not change.
- **Labor:** 6,453.11 of 6,480 hours, 26.89 under the limit (`Unconstrained!D32`), or 3.98 of the 4 temporary workers (`Unconstrained!B34`).
- **Profit:** $45,010.80 (`Unconstrained!E20`), $2,249.13 or about 5.3% more than today (`Unconstrained!E22`).

Labor was never the constraint; the crop caps were.

What this scenario leaves out is the cost of land. The 73 beds are 9 more than the farm has (`Unconstrained!D26`), and the model has no cost for extra land.

## Scenario B: fertilizer bulk discount

The wage change is one input that bends the marginal-cost curve. The `FertScenario` tab tests another: a 30% bulk discount on fertilizer for every bed beyond 40 across the whole farm (`FertScenario!B7:B8`). Each discounted bed saves $264 (30% of $880).

Like the wage change, the discount produces a dip in marginal cost, but a much smaller one: $264, against $2,754.58 for the wage step. Under today's limits, tomato and carrot beds already total 30, so the dip shows up in mesclun. Mesclun marginal cost is $1,862.87 at its 10th bed (the farm's 40th) and falls to $1,622.22 at its 11th, where without the discount it would have risen to $1,886.22 (`FertScenario!J75`, `J76`, `D76`).

With today's limits, the mix stays 10 / 20 / 30 because the caps still bind, and profit rises $5,280 to $48,041.66 (`FertScenario!C16`, `D16`). With the limits also removed, the mix grows to 10 / 30 / 44 and profit to $55,336.48 (`FertScenario!F12:F16`), but that needs 20 more beds than the farm has and 4.81 temporary workers against the 4 allowed (`FertScenario!F18`, `F20`).

## What this means

The constraints do limit profit, but not by much, because diminishing returns shrink each extra bed's contribution quickly (price minus marginal cost, `Unconstrained`):

- **Carrots:** $1,120.15 on the 1st bed, $405.05 on the 20th, $60.77 on the 26th; the 27th loses $3.81.
- **Mesclun:** $1,028.98 on the 1st bed, $279.90 on the 30th, $33.07 on the 37th; the 38th loses $4.73.

Adding beds is therefore a weak way to grow profit, especially once extra land or extra workers carry costs of their own. The stronger option is another crop. A new crop starts at the top of its own diminishing-returns curve, where the gap between price and marginal cost is widest. More crop types, not more beds, is where the larger increase in profit would come from.

## Why grow a crop that loses money on its own?

The $20,000 in fixed costs (`Inputs!B6`) is paid no matter what I plant. On its own, before fixed costs, carrots' best profit is $3,511.08 and mesclun's is $8,077.81 (`MCSchedules!K56`, `K99`). Neither could cover $20,000 alone, while tomatoes could ($26,172.77, `MCSchedules!K15`).

That is the wrong test, though. The question for each bed is not whether its crop covers the fixed costs, but whether the bed's price exceeds its marginal cost. Every bed that clears that bar adds to the margin that pays down the fixed costs, even if the crop as a whole would lose money on its own. The gap shrinks bed by bed, but summed over many beds it adds up. Each crop's variable margin (`CostStructure!B16:B18`) is $33,143.42 for tomatoes, $13,682.27 for carrots and $15,935.98 for mesclun: $62,761.66 together, about 3.1 times the fixed costs (Figure 3). So the best practice is to plant every bed whose marginal cost is below the price.

**Figure 3** — Each crop's variable margin, the $20,000 in fixed costs, and the $42,761.66 season profit that remains at 10 / 20 / 30 beds. Data: `CostStructure!B16:B21`.

![Variable margin by crop, fixed costs and season profit](figures/perfect-competition-margin-vs-fixed-costs.png)

## My Stage One hypothesis

My Stage One hypothesis was 10 tomato, 20 carrot and 30 mesclun beds (`docs/briefs/perfect-competition-brief.md`). The model's result matches it exactly (`Optimization!B4:B6`), which surprised me.

My reasoning was that tomatoes' 10% diminishing-returns rate meant they would not reach their 20-bed cap, so I guessed half: 10 beds. Carrots and mesclun earn less per bed, but their lower rates (2.5% and 1.25%, `Inputs!C18:D18`) meant they would keep paying off at higher volume, so I planted both to their caps. That reasoning held.

To judge whether I was wrong, I allowed a margin of 3 beds in either direction for each crop. That margin separated sound reasoning from imperfect arithmetic, and it did not come into play: the gap was 0 beds for all three crops. I still think the approach was right. A guess of 11 tomato, 19 carrot and 28 mesclun beds would have followed the same logic; landing exactly on the answer came partly from choosing round numbers.
