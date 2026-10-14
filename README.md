# Draw a sand slide: SPH flume deposit predictor

**▶ Live page: <https://brianpark93.github.io/sph-flume-deposit-predictor/>**

Set the slope angle of a laboratory sand flume, draw a sand mass on the slope,
and a neural-network surrogate predicts the shape of the deposit it leaves
once it comes to rest. Inference runs entirely in the browser.

> **Status: preliminary model.** The page currently ships a model trained on
> the first part of the simulation set while the remaining runs finish. The
> accuracy numbers on the page are always the ones measured for the model it
> is serving.

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
| Random shapes | 1,200 | slope 30–60°; shape from five families (rectangle, wedge, triangle, mound, random polyline); length 0.1–1.3 m, thickness ≤ 0.36 m, front 0.8–2.3 m up the slope, 100–3,000 particles |
| Rectangular blocks | 576 | slope 30–60° × volume 0.25–4× the experimental block, two placements (front fixed / centre of mass fixed) |
| Held-out test shapes | 150 | as the random shapes, different seed; never used for training or model selection |

## Model

- **Input:** slope angle + the sand's thickness normal to the slope, averaged
  over 5 cm bins along it (67 bins).
- **Output:** height of the deposit surface above the slope/floor beneath it,
  on a 2 cm horizontal grid (251 points), represented by its principal
  components.
- **Network:** an ensemble of five multilayer perceptrons (tanh). The band
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
  extrapolation.

## Related

- Mohr–Coulomb parameter-grid explorer for the same flume:
  <https://brianpark93.github.io/mohr-coulomb-sph-flume-grid/>
