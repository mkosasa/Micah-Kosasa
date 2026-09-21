<!-- Micah's draft is unchanged in the commit before this one. Added since: the five bracketed placeholders are filled in [brackets] with cell references, and blocks labelled "Model support" (sheet!cell = value) or "Check" (a number or cell in the draft that did not match the model, or a point to look at) follow the paragraphs they support. All were added by Claude; rephrase or delete them in your own words. Values are from capabilities/marginal-analysis/model.xlsx on main (blob cc3684e). -->

# Farm Model Analysis

**Why tomatoes stop at 10 beds when they're the money crop**

Tomatoes, while they are the most profitable, particularly at the beginning, with the high diminishing percentage of 10%, the marginal costs catch up fairly quickly. At ten tomato beds, the marginal cost is $8,248.59, compared with a price of $8,800. At eleven beds, it jumps up to $9,390.72, which is above that price of $8,800.

> **Model support**
> - Price per bed: tomatoes $8,800 (`Inputs!B21`), carrots $2,094 (`Inputs!C21`), mesclun $2,700 (`Inputs!D21`).
> - Diminishing-returns rate: tomatoes 10% (`Inputs!B18`), carrots 2.5% (`Inputs!C18`), mesclun 1.25% (`Inputs!D18`).

Refer to cell H15 under MC schedules on the model for the ten-bed marginal cost, and cell H16 under MC schedules for the eleven-bed marginal cost.

> **Model support** — your two cell references are correct (on the tomato block of `MCSchedules`, bed number = row − 5):
> - `MCSchedules!H15` = $8,248.59, the marginal cost of the 10th bed. Price: `MCSchedules!J15` = $8,800 (`Inputs!B21`). The 10th bed still adds $551.41 (`Unconstrained!D70`).
> - `MCSchedules!H16` = $9,390.72, the 11th bed: $590.72 above the price (`Unconstrained!D71` = −$590.72, also `Summary!I36`).
> - `MCSchedules!B27` = 10 (best bed count) and `MCSchedules!B28` = 11 (first bed where marginal cost reaches price).

Another place to see this is where the profit is constantly rising — rising at nine, then ten, then eleven — you can see it drop. This can be seen in cells H15, H16, and H17.

> **Check** — H15, H16 and H17 are marginal *costs*, and they keep rising ($8,248.59, $9,390.72, $10,687.59). Profit is column K on the same rows:
> - `MCSchedules!K14` (9 beds) = $25,621.36; `MCSchedules!K15` (10 beds) = $26,172.77; `MCSchedules!K16` (11 beds) = $25,582.06.
> - Profit rises through bed 10 (+$551.41) and drops at bed 11 (−$590.72), so K14:K16 is the range that shows what you describe. These K values are standalone profit before the $20,000 fixed costs.

**Which constraints bind, and what relaxing one is worth**

Tomatoes do not run into this constraint, at least from a bed standpoint — even though it hasn't hit its max of 20, you don't need to, because the profitability goes down.

> **Model support** — tomatoes plant 10 beds (`Optimization!B4`) against a cap of 20 (`Inputs!B22`). Relaxing the tomato cap is worth $0 (`Summary!K36`).

Looking at the constraints, at least from a bed standpoint, we would be able to do four more beds. However, we are limited on our carrots and mesclun, where we have already hit our max beds.

> **Model support**
> - Farm bed total 64 (`Inputs!B5`); beds used 60 (`Optimization!B7`); 4 beds unused (`Optimization!E14`, also `Summary!B7`).
> - Carrots plant 20 beds (`Optimization!B5`), equal to their cap (`Inputs!C22`); mesclun plant 30 (`Optimization!B6`), equal to their cap (`Inputs!D22`).
> - The model names the binding limit: "MAX_BEDS cap: CAR MES" (`Optimization!B29`, also `Summary!B26`).

The constraint on the amount of beds is a limiting factor not for tomatoes, but for carrots and mesclun. [With the caps lifted, the best is 26 carrot beds (`Unconstrained!C7`) and 37 mesclun beds (`Unconstrained!D7`), up from 20 and 30.]

> **Model support** — what relaxing a cap is worth:
> - The first extra carrot bed adds $352.49 (`Summary!K37`); the first extra mesclun bed adds $246.47 (`Summary!K38`); tomatoes $0 (`Summary!K36`). Each is the price minus the marginal cost of the next bed: carrots $2,094 − $1,741.51 (`Summary!G37`), mesclun $2,700 − $2,453.53 (`Summary!G38`).
> - Lifting the caps completely: 26 carrot beds and 37 mesclun beds (`Unconstrained!C7`, `Unconstrained!D7`). The same peaks appear on `MCSchedules` (`E124` = 26, `E147` = 37). The gain is $1,259.79 for carrots (`MCSchedules!E127`) and $989.34 for mesclun (`MCSchedules!E150`), $2,249.13 together.
> - The slack limits are worth $0: the 64-bed total (4 beds unused, `Optimization!E14`) and the temp-labor hours (1,202.78 unused, `Optimization!E21`).

We did not reach the max of 64 total beds, and worker hours was not a limit — that was not a constraint.

> **Model support**
> - Beds: 60 used (`Optimization!B14`) of 64 (`Optimization!D14`).
> - Labor hours: 5,277.22 used (`Optimization!B22`) of 6,480 available (`Optimization!D22`, also `Inputs!B14`), so 1,202.78 hours unused (`Optimization!E22`).
> - Temp hours: 4,557.22 used (`Optimization!B21`) of a 5,760 ceiling (`Optimization!D21`), which is 3.16 of the 4 temp workers (`Summary!B18`).

One thing I want to do a comparison on — and I'll ask for a model update on this — is to see: if I increase the amount of beds planted for carrots and mesclun, will I run out of worker hours? Was that actually a constraint? I want to see, with this constraint, what the max profitability is for this whole situation, and how many workers I need.

> **Model support** — this comparison is now built: the `Unconstrained` sheet (caps, bed total and worker limit all removed).
> - Labor hours needed: 6,453.11 (`Unconstrained!B32`) against 6,480 available (`Unconstrained!C32`), so 26.89 hours **under** (`Unconstrained!D32`; status `Unconstrained!E32` = "under").
> - Workers needed: 3.98 temp workers against the limit of 4 (`Unconstrained!B34`, `Unconstrained!C34`), so no extra worker is needed.
> - Maximum profit with no limits: $45,010.80 (`Unconstrained!E20`), from 10 / 26 / 37 beds = 73 (`Unconstrained!B7:E7`).
> - So worker hours were not what stopped the plan; the crop caps were.

One thing to consider: I've created a scenario, in the fertilizer discount section above, of releasing the limitations/constraints. One of the impacts not being considered there is the cost of increasing the amount of land being used — if that included an additional fixed cost, that may change the situation. We don't have that cost for additional land, but what we saw is that, without the constraints, due to diminishing returns, it actually ends up being a fairly small increased profit — around $2,000. [$2,249.13 (`Unconstrained!E22`)]

> **Check** — releasing the limits is the `Unconstrained` sheet. The fertilizer discount is a separate sheet, `FertScenario`, so "in the fertilizer discount section above" may need to point to `Unconstrained`.
>
> **Model support**
> - Extra profit from releasing the limits: $2,249.13 (`Unconstrained!E22`) = $45,010.80 (`Unconstrained!E20`) − $42,761.66 (`Unconstrained!E21`, the model's `PROFIT`, also `CostStructure!B21`). That is about 5.3% more.
> - Land needed: 73 beds is 9 more than the 64 you have (`Unconstrained!D26`) and 13 more than the 60 planted (`Unconstrained!D27`).
> - The model has no cost for extra land: fixed costs stay $20,000 (`Inputs!B6`).

Therefore, unless the cost of adding land is very small, this probably would not be a useful thing to invest in — the additional land — or if hiring more workers added other overhead costs, there just wouldn't really be an increase. This really points to the fact that while these constraints do put some limitations on profitability, we're seeing a fairly quick drop in profitability as we add more and more beds, due to diminishing returns. Adding more beds is really just not a way to maximize an increase in profits.

> **Check** — "if hiring more workers added other overhead costs": with the limits released and no discount, no extra workers are needed (3.98 needed, limit 4: `Unconstrained!B34`, `Unconstrained!C34`). Extra workers appear only with the discount: 4.81 needed (`FertScenario!F18`), about 0.81 above the limit.
>
> **Model support**
> - Diminishing returns in dollars (price minus marginal cost, `Unconstrained`): the first carrot bed adds $1,120.15 (`I61`), the 20th $405.05 (`I80`), the 26th $60.77 (`I86`); the 27th would lose $3.81 (`I87`). Mesclun: $1,028.98 for the first bed (`N61`), $279.90 at bed 30 (`N90`), $33.07 at bed 37 (`N97`), −$4.73 at bed 38 (`N98`).
> - The most extra land could cost: $2,249.13 ÷ 9 extra beds ≈ $250 per bed per season, before any other overhead. (Derived; not a workbook cell.)
> - The extra labor is already counted: labor cost is $124,533.20 released (`Unconstrained!E18`) against $104,118.34 today (`CostStructure!B9`), $20,414.87 more, and the $2,249.13 is after that.

The biggest thing would be looking at possibly another crop — one that lets you start off with a higher profit and start that diminishing-return curve fresh. It's by adding more crop types that you're going to see an increase in profitability, not by increasing the amount of beds.

**The tomato marginal cost dip at six beds**

The marginal costs are going down, then suddenly dip at six, because of the change in labor costs — coming from the more expensive farmer at [$34.72 per hour (`Inputs!B9`)] to the temporary workers at [$17.36 per hour (`Inputs!B12`)] — resulting in a significant dip in the marginal costs at six. In some ways this resets the profitability concern: if there had not been that change in the workers' wages — let's say the farmer had more available hours, or we were required to hire another farmer at the same rate — the max beds would hit far earlier than 10.

> **Check** — marginal cost is *rising* until bed 5, then dips: $4,317.50 at bed 1 (`MCSchedules!H6`), $7,660.86 at bed 5 (`MCSchedules!H10`), then $4,906.28 at bed 6 (`MCSchedules!H11`), a $2,754.58 drop.
>
> **Model support**
> - The model flags it: `MCSchedules!M11` = "DIP" and `MCSchedules!B29` = "yes - non-monotonic".
> - The switch happens at bed 5: cumulative labor there is 724.73 hours (`MCSchedules!B10`), just past the farmer's 720 hours (`Inputs!B7`). The farmer's $34.72 an hour is $25,000 of the $50,000 salary (`Inputs!B8`) over those 720 hours; a temp worker's $17.36 an hour is $25,000 (`Inputs!B10`) over 1,440 hours (`Inputs!B11`).
> - "Far earlier than 10" holds: if every hour cost $34.72, the 6th tomato bed would cost about $8,932.55, above the $8,800 price, so tomatoes would stop at 5 beds (the 5th costs $7,742.97). (Calculated from the `Inputs` values outside the workbook; not a workbook cell.)

That's an example we could actually look at here — a comparison to similar situations, showing how the inputs really affect marginal cost. I've done an example, seen here at this tab, of what if fertilizer costs suddenly reduced by 30% for any beds past forty total beds across the entire farm — the idea being there was a discount for bulk ordering of fertilizer. In this case, just like the wage, this would be another input that, with some change in pricing, would cause this dip. And this dip would happen around — as we're matching the max, once we hit a total of 40 — we would see a dip across the other plant types, the other things being planted.

> **Check** — the fertilizer discount is a $264 step, much smaller than the wage step.
> - Inputs: 40 beds (`FertScenario!B7`) and 30% (`FertScenario!B8`). A discounted bed saves 30% × $880 = $264 (`Inputs!B19`, `Inputs!D19`).
> - The step at 40 total beds: the discount is $0 at 40 beds (`FertEnum!H20684`), $264 at 41 (`FertEnum!H20685`), $528 at 42 (`FertEnum!H20686`).
> - You can see the dip for mesclun under today's limits (`FertScenario`, tomatoes 10 + carrots 20 = 30 beds already planted): at bed 10 (40 beds on the farm) the marginal cost is $1,862.87 (`J75`); at bed 11 (41 beds) it falls to $1,622.22 (`J76`), where without the discount it would have risen to $1,886.22 (`D76`).
> - Every crop's marginal cost drops by the same $264, including carrots: last-bed tomatoes $8,248.59 → $7,984.59 (`FertScenario!C27` → `D27`), carrots $1,688.95 → $1,424.95 (`C28` → `D28`), mesclun $2,420.10 → $2,156.10 (`C29` → `D29`). For comparison, the wage step is $2,754.58 (`MCSchedules!H10` → `H11`).

Looking at the new fertilizer scenario tab: with the discount but with the limitations still in place, it makes no difference in the amount of beds — that limitation is still a restriction — however, it does increase the profitability with a fertilizer discount going in.

> **Model support** — today's limits (`FertScenario`, columns B to D): beds stay 10 / 20 / 30 (`B12:C14`); profit rises from $42,761.66 (`B16`) to $48,041.66 (`C16`), up $5,280 (`D16`), which equals the discount applied (`C21`).

However, if no limitations — if these limitations on labor and bed limits were removed — with the discount, you would see no change in tomatoes. As stated before, tomatoes just do not go past ten; the marginal cost is just above the price no matter what past ten, so there's no change there. What you do see is an increase in the amount of carrots from 20 to 26 with no discount, and an increase in mesclun beds from 30 to 37 with no discount, for a total of 73 beds. You do have increased labor costs, but you're still under the total labor limits.

> **Model support** — limits released, no discount (`FertScenario` column E, the same figures as `Unconstrained`):
> - Beds 10 / 26 / 37 = 73 (`E12:E15`; also `Unconstrained!B7:E7`).
> - Labor hours 6,453.11 (`E17`), which is 26.89 under the labor limit (`E19`).
> - Tomatoes stay at 10 even with the discount: the 11th bed would still cost $9,126.72 with the discount (`FertScenario!J27`), above the $8,800 price (`FertScenario!B27`).

Where this really adds more is when the constraints on beds or labor are not included, and the discount is being applied. Tomatoes are still the same 10 beds. For carrots, that jumps up to 30 beds, and for mesclun, 44 beds, for a total amount of 84. There you end up using an additional 1,162.34 labor hours past the current labor cap, but you do see increased profitability with that — and this is the max profitability with this setup.

> **Model support** — limits released with the discount (`FertScenario` column F):
> - Beds 10 / 30 / 44 = 84 (`F12:F15`); 84 is 20 more than the 64 you have (`F20`).
> - Labor hours 7,642.34 (`F17`), which is 1,162.34 over the 6,480-hour limit (`F19`; limit at `Inputs!B14`). Temp workers needed: 4.81 (`F18`) against 4 (`Inputs!B13`).
> - Profit $55,336.48 (`F16`): $10,325.68 more than released without the discount (`G16`) and $12,574.81 more than today's $42,761.66 (`B16`). It is the best of all the bed mixes tested (`FertEnum!S3`), and the best mix is inside the tested range (`FertEnum!Q15` = PASS).

**Why grow crops that lose money on their own?**

The key thing here is the fixed costs, as seen in cell [`Inputs!B6`], of $20,000. That has to be covered one way or another. This coverage of that fixed cost is lessened with a larger total amount of plants being planted — that makes a big difference in keeping the costs, so they exceed the variable costs. But that fixed cost is paid anyway. So a crop that might lose money on its own, when contributing to paying off that fixed cost, can still be of value to the overall enterprise.

> **Check** — I could not tell what "so they exceed the variable costs" refers to. If you mean revenue exceeds variable costs: revenue is $210,880 (`CostStructure!B15`) against fertilizer $44,000 (`CostStructure!B14`) plus labor $104,118.34 (`CostStructure!B9`) = $148,118.34, leaving $62,761.66 to cover the $20,000 fixed cost (`CostStructure!B19`) and produce $42,761.66 profit (`CostStructure!B21`).
>
> **Model support**
> - Each crop's variable margin (`CostStructure!B16:B18`): tomatoes $33,143.42, carrots $13,682.27, mesclun $15,935.98. Together $62,761.66, about 3.1 times the $20,000 fixed cost.
> - Each crop's best profit on its own, before fixed costs: tomatoes $26,172.77 at 10 beds (`MCSchedules!K15`), carrots $3,511.08 at 20 (`MCSchedules!K56`), mesclun $8,077.81 at 30 (`MCSchedules!K99`). Against one full $20,000, tomatoes would still cover it (+$6,172.77) but carrots (−$16,488.92) and mesclun (−$11,922.19) would not. That is the "loses money on its own" case.

Here's where one of the greatest values in addressing this fixed cost is: to produce as much of the crop as possible, as long as the marginal cost is lower than the price. Even if overall you are losing money, that slight difference between the marginal cost and the price is helping chip away at the total fixed amount. For a single bed, a high difference is good — but even as that difference between the marginal cost and the price gets smaller and smaller due to diminishing returns, the sum of those differences, at a high number of beds, can take a big chunk out of that fixed cost. That is where it has value and is worth pursuing.

> **Model support** — the shrinking gap between price and marginal cost, bed by bed (`Unconstrained`, "PRICE - MC" columns): carrots $1,120.15 at bed 1 (`I61`), $405.05 at bed 20 (`I80`), $60.77 at bed 26 (`I86`); mesclun $1,028.98 (`N61`), $279.90 at bed 30 (`N90`), $33.07 at bed 37 (`N97`); tomatoes $6,201.25 at bed 1 (`D61`), $551.41 at bed 10 (`D70`), then −$590.72 at bed 11 (`D71`). Together the three crops' variable margins ($62,761.66, `CostStructure!B16:B18`) are about 3.1 times the $20,000 fixed cost (`Inputs!B6`).

**Regarding my Stage One hypothesis**

I'm somewhat surprised and dumbfounded that my hypothesis was exactly right on to the final result: ten tomatoes, twenty carrots, thirty mesclun.

> **Model support**
> - The Stage 1 brief (`docs/briefs/perfect-competition-brief.md`, `hypothesis:` line): "10 tomato / 30 mesclun / 20 carrot".
> - The model: 10 tomato (`Optimization!B4`), 20 carrot (`Optimization!B5`), 30 mesclun (`Optimization!B6`); also `Summary!B3:B5`, and the enumeration maximum agrees (`Enumeration!S2:S4`).
> - Diminishing-returns rates: carrots 2.5% (`Inputs!C18`) and mesclun 1.25% (`Inputs!D18`), against 10% for tomatoes (`Inputs!B18`).

I did make that decision recognizing that tomatoes, due to their high diminishing return, would not likely be at full capacity, and guessed at half — a nice even number of 10. I did not realize I would be exactly on. I then looked at maxing out the other two, since those seemed to be the ones that, while they did not have the highest price, their shorter diminishing-return rate would, at a higher volume, pay off. It appears that was correct.

> **Check** — "shorter diminishing-return rate" reads as "smaller" or "lower" rate (2.5% and 1.25% against 10%, `Inputs!C18`, `Inputs!D18`, `Inputs!B18`).
>
> **Check for the Stage 3 checklist** — the brief's reasoning was that tomato marginal cost compounds past the $8,800 price by bed 11, and the model agrees (`MCSchedules!H16`). What the brief did not anticipate is the dip at bed 6 caused by the switch from farmer to temp labor (`MCSchedules!H10` → `H11`), and that labor was never the binding limit (`Optimization!E22`, `Unconstrained!D32`). Say so if the honest-reflection paragraph asks what the brief missed.

I had also added an additional element for how I would know I was wrong — giving myself a three-bed margin in either direction — to determine if, while I might have had the idea correct with my hypothesis, calculation-wise I didn't have an exact calculation that would have been right on. So that margin also did not end up mattering here.

> **Model support** — the brief's margin: "tomato, mesclun, and carrot counts each shifting by 3 or fewer" (`docs/briefs/perfect-competition-brief.md`, "How I would know I was wrong"). Your eleven / nineteen / twenty-eight example is within it: 1, 1 and 2 beds from 10 / 20 / 30. The model's gap is 0 for all three (`Optimization!B4:B6`).

I still stand by it — I think that was a good approach, since I could have easily said eleven tomatoes, nineteen carrots, and twenty-eight mesclun, and I still think my guess would have been following the correct line of logic. I just happened to hit it perfectly with these even numbers, so I think that was still a good approach for doing so.
