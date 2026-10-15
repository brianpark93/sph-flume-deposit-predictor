# Draw a sand slide: SPH flume deposit predictor

**▶ Live page: <https://brianpark93.github.io/sph-flume-deposit-predictor/>**

Set the slope angle of a laboratory sand flume, draw a sand mass on the slope,
and a neural-network surrogate predicts the shape of the deposit it leaves
once it comes to rest. Inference runs entirely in the browser.

Two modes, switched at the top of the page:

- **Forward** — slope angle + a drawn sand shape → predicted deposit.
- **Inverse** — a drawn deposit (or one of the seven digitised experiments)
  → slope angle and rectangular sand block that would leave it, found by
  searching through the forward model in the browser. Distinct candidate
  sources are listed because the answer is not unique.

## Accuracy (150 held-out random shapes, different seed)

| | |
|---|---|
| Deposit surface, mean RMS error | 0.38 cm (90th percentile 0.54 cm) |
| Runout front position, mean abs. error | 1.4 cm (R² 0.987) |
| Maximum deposit height, mean abs. error | 0.67 cm (R² 0.98) |
| Baselines: 5-nearest-neighbour / ridge | 1.31 / 1.35 cm |

Two of the 3,350 shape runs ended abnormally in LS-DYNA and are excluded
(one in the test set, which is therefore 149 shapes).

Doubling the training set from 1,776 to 3,775 simulations cut the surface
error from 0.57 to 0.38 cm and the front error from 2.5 to 1.4 cm; half of
the added shapes were picked where the earlier ensemble disagreed most
(`select_shapes.py`), half at random.

## What is modelled

A quasi-2D dry-sand flume: a 1 cm slice of sand (1276.3 kg/m³) released at
rest on a rigid slope that pivots about a fixed point above a horizontal
floor. Each training case is an LS-DYNA SPH simulation (1 cm particles) run
until the mass settles, using the calibrated *μ(I)* + power-law Perzyna
viscoplastic rheology (static/dynamic friction 17.4°/67.4°, I₀ = 0.432,
τ = 3.5×10⁻⁴ s, N = 3.71). That rheology reproduces digitised experimental
deposits at seven slope angles (30–60°, 5° steps) with one parameter set.

## Training data

| Set | Cases | What varies |
|---|---|---|
| Random shapes | 1,200 + 1,999 | slope 30–60°; shape from five families (rectangle, wedge, triangle, mound, random polyline); length 0.1–1.3 m, thickness ≤ 0.36 m, front 0.8–2.3 m up the slope, 100–3,000 particles. The second batch: 1,000 picked from 20,000 candidates where the previous model's ensemble disagreed most, 1,000 at random |
| Rectangular blocks | 576 | slope 30–60° × volume 0.25–4× the experimental block, two placements (front fixed / centre of mass fixed) |
| Held-out test shapes | 150 (149 solved) | as the random shapes, different seed; never used for training or model selection |

## Model

- **Input:** slope angle + the sand's thickness normal to the slope, averaged
  over 5 cm bins along it (67 bins).
- **Output:** height of the deposit surface above the slope/floor beneath it,
  on a 2 cm horizontal grid (251 points), represented by its principal
  components.
- **Network:** an ensemble of five multilayer perceptrons (64–64 tanh units,
  64 principal components). The band
  on the page is the members' spread — their disagreement, not a calibrated
  probability.
- `model.bin` holds the float32 weights and the PCA basis; `model.json` holds
  the layout, scalings, geometry constants, measured skill and three
  round-trip fixtures. Open the page with `?selftest` to check the browser
  reproduces the Python predictions.

## Limits

- Only the final deposit is predicted, not the flow.
- The rheology was calibrated against one block size at seven angles. Other
  sizes and shapes are model predictions, not experimentally validated.
- Below about 35° the calibrated rheology keeps creeping slowly after the
  main flow stops, so the deposit there depends on when the simulation ends.
- Outside the trained ranges the page flags the prediction as an
  extrapolation. Sand in several separate pieces gets no prediction; a single
  piece with more humps or a more jagged outline than any training shape is
  drawn grey and marked unreliable.

## Related

- Mohr–Coulomb parameter-grid explorer for the same flume:
  <https://brianpark93.github.io/mohr-coulomb-sph-flume-grid/>
