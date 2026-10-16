# Draw a sand slide: SPH flume deposit predictor

**▶ Live page: <https://brianpark93.github.io/sph-flume-deposit-predictor/>**

Set the slope angle of a laboratory sand flume, draw a sand mass on the slope,
and a neural-network surrogate predicts the shape of the deposit it leaves
once it comes to rest. Inference runs entirely in the browser.

Two modes, switched at the top of the page:

- **Forward** — slope angle + a drawn sand shape → predicted deposit.
- **Inverse** — a drawn deposit (or one of the seven digitised experiments)
  → slope angle and rectangular sand block that would leave it, found by
  searching through the forward model in the browser. The angle, release
  position and block proportions can each be fixed when known; with none
  fixed the answer is not unique, and distinct candidates are listed.

## Accuracy (held-out shapes, different random seeds)

| | Original range (149) | Extended range (149) |
|---|---|---|
| Deposit surface, mean RMS error | 0.36 cm | 0.67 cm |
| Runout front position, mean abs. error | 1.5 cm | 2.5 cm |
| Maximum deposit height, mean abs. error | 0.69 cm | 1.14 cm |

Baselines over all 298 test shapes: 5-nearest-neighbour 1.82 cm, ridge 1.76 cm
(MLP ensemble 0.52 cm).

The **extended range** covers everything the page lets you draw except the
slope angle: 32–5,993 particles (0.05–9.3 V₀), up to 0.50 m thick, front
0.05–2.85 m up the slope. Drawing is capped at those limits. Only the slope
angle is flagged when outside 30–60°, the range the rheology was calibrated on.

Of the 5,100 shape runs, 24 ended with LS-DYNA's `URMATHN` access violation in
the user material. 19 had already come to rest (mean speed < 5 cm/s), mostly
large masses on 30–32° slopes creeping slowly afterwards; their last complete
state is used and flagged as such. The other 5 crashed while still flowing and
are excluded.

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
| Random shapes | 1,200 + 1,999 + 1,600 | slope 30–60°; shape from five families (rectangle, wedge, triangle, mound, random polyline); length 0.1–1.3 m, thickness ≤ 0.36 m, front 0.8–2.3 m up the slope, 100–3,000 particles. The second batch: 1,000 picked from 20,000 candidates where the previous model's ensemble disagreed most, 1,000 at random. The third: 1,600 outside the original box (30–6,000 particles, up to 0.5 m thick, front 0.05–2.85 m up the slope, length up to 2.8 m) and 100 tiny ones |
| Rectangular blocks | 576 | slope 30–60° × volume 0.25–4× the experimental block, two placements (front fixed / centre of mass fixed) |
| Held-out test shapes | 149 + 149 | original and extended ranges, different seeds; never used for training or model selection |

## Model

- **Input:** slope angle + the sand's thickness normal to the slope, averaged
  over 5 cm bins along it (67 bins).
- **Output:** height of the deposit surface above the slope/floor beneath it,
  on a 2 cm horizontal grid (251 points), represented by its principal
  components.
- **Network:** an ensemble of five multilayer perceptrons (64–64 tanh units,
  54 principal components). The band
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
- The slope angle outside 30–60° is flagged as an extrapolation. Sand in several separate pieces gets no prediction; a single
  piece with more humps or a more jagged outline than any training shape is
  drawn grey and marked unreliable.

## Related

- Mohr–Coulomb parameter-grid explorer for the same flume:
  <https://brianpark93.github.io/mohr-coulomb-sph-flume-grid/>
