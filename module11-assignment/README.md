# What Drives the Price of a Used Car?

A regression analysis of used car listings to identify the factors that most influence selling price. Built for a used car dealership audience using the CRISP-DM framework.

**Full notebook:** [prompt_II.ipynb](prompt_II.ipynb)

---

## Practical Findings

The selected model is **Lasso + Target Encoding on `model`**, with a test MSE of **0.2583** on log-price (about an 18.7% MSE improvement over the baseline that dropped the `model` column).

### Top price drivers (selected model, ranked by Lasso coefficient magnitude)

| Rank | Original Feature | Effect | Practical Takeaway |
|---|---|---|---|
| 1 | **Vehicle model** (target-encoded) | +0.577 | Largest single effect. Price at the *specific model* level, not just the brand. |
| 2 | **Vehicle age** | −0.393 | Each additional year measurably depresses price. Prioritize newer stock. |
| 3 | **Fuel type — diesel** | +0.30 | Diesel commands a clear premium. Don't underprice diesel trucks and vans. |
| 4 | **Body type** | convertible +0.28, coupe +0.09 / hatchback −0.06 | Body style matters independently of brand. |
| 5 | **Manufacturer** | Porsche +0.24, Lexus +0.11, Toyota +0.07 / Kia −0.13, Hyundai −0.11, Nissan −0.10 | Brand still matters, but the spread is small once model is accounted for. |


### Sourcing recommendations for the dealership
1. **Age and mileage are the primary levers.** Prioritize vehicles under 10 years old and under 80K miles.
2. **Source by body type.** Independent of brand — convertibles and coupes carry a premium, hatchbacks and mini-vans price at a discount. The top body-type coefficient (+0.28) is larger than the top brand coefficient (+0.24), so body type is important.
3. **Source by brand tier.** Luxury (Porsche, Lexus, Audi, BMW) for margin; mainstream reliable (Toyota, Honda, Ford, Chevrolet) for volume; economy (Kia, Hyundai, Nissan) only at a discount.
4. **Diesel premium is real** — price diesel trucks and vans above comparable gas vehicles.
5. **Research at the model level**, not just the brand. Adding the model column alone produced an 18.7% MSE improvement.

### Honest caveats
- Findings are based on **listing prices**, not final sale prices.
- Significant missing data in `condition` (~41%), `drive` (~31%), and `cylinders` (~42%) — findings for those features are directional, not precise.
- A degree-2 polynomial Lasso was tested (MSE 0.2296) — marginal improvement only, with no practical change in prediction precision. 

