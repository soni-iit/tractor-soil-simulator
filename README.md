# Tractor–Soil Interaction Simulator

An interactive, browser-based teaching model of driven-wheel traction. Starting from soil texture (sand, silt, clay on the USDA triangle), it shows how wheel load, inflation pressure, soil moisture and slip change tractive efficiency, drawbar pull and power, rut depth and subsoil stress.

**Live version:** `https://soni-iit.github.io/tractor-soil-simulator/`

The whole simulator is one file, `index.html`. There is nothing to install: open it in any modern browser, online or offline. (An internet connection is only used to load the fonts. Without it the page falls back to system fonts.)

## What you can do

* Set soil texture by dragging in the USDA triangle, moving the sand, silt and clay sliders, or choosing a typical composition for any of the 12 USDA classes.
* Choose a surface condition (firm, tilled or loose) and the soil moisture (% dry basis).
* Read the Atterberg consistency state (solid, semi-solid, plastic, liquid) at the chosen moisture.
* Optionally enter a measured cone-index profile with depth (0 to 50 cm). Its mean over 0–150 mm replaces the estimated cone index.
* Set wheel load, inflation pressure, slip, theoretical speed and tyre size.
* Read tractive efficiency, net traction ratio, drawbar pull and power, mobility number, rut depth, contact pressure and stress at 0.4 m depth, with warnings for slip, rut depth, compaction and soil consistency.
* Drag on the slip charts to set slip. Optional operating scenarios fill in typical settings while keeping your soil texture.

## Model

### Wheel traction (Brixius wheel-numeric model)

$$B\_n = \\frac{CI, b, d}{W}\\cdot\\frac{1 + 5,\\delta/h}{1 + 3,b/d}$$

$$GTR = 0.88\\left(1 - e^{-0.1 B\_n}\\right)\\left(1 - e^{-7.5, s}\\right) + 0.04$$

$$MRR = \\frac{1.2}{B\_n} + 0.04 + \\frac{0.5, s}{\\sqrt{B\_n}}$$

$$NTR = GTR - MRR, \\qquad TE = \\frac{NTR,(1 - s)}{GTR}$$

Here $CI$ is the cone index (kPa), $b$ the tyre width, $d$ the tyre diameter, $h$ the section height, $\\delta$ the tyre deflection, $W$ the wheel load, and $s$ the slip as a fraction. Drawbar pull is $NTR \\cdot W$, and drawbar power is that pull times the actual travel speed $v\_t(1 - s)$.

### Tyre contact

$$\\frac{\\delta}{h} = 0.15,\\frac{W}{20}\\left(\\frac{100}{p\_i}\\right)^{0.85}\\ \\ (\\text{limited to } 0.04\\text{–}0.35),\\qquad L = 2\\sqrt{d,\\delta},\\qquad A = \\frac{\\pi, b, L}{4},\\qquad p\_c = \\frac{W}{A}$$

with $W$ in kN and $p\_i$ in kPa.

### Sinkage and subsoil stress

$$z = \\left(\\frac{p\_c}{k\_c/b + k\_\\phi}\\right)^{1/n}(1 + 4s)$$

$$\\sigma\_z = p\_c\\left\[1 - \\left(\\frac{1}{1 + a^2/z^2}\\right)^{\\nu/2}\\right], \\qquad a = \\sqrt{A/\\pi}$$

The Bekker sinkage uses $k\_c$ and $k\_\\phi$ scaled by the ratio of the current cone index to the firm-soil reference. The stress concentration factor $\\nu$ is 4 for firm and 5 for tilled and loose surfaces, reduced by $0.08,(w - w\_{FC})$ and limited to 2.5–6. Compaction risk at 0.4 m is flagged as low below 50 kPa, moderate from 50 to 100 kPa and high above 100 kPa. These thresholds are indicative only.

### Soil texture to parameters

All relations below use clay and silt in %, moisture $w$ in % (dry basis), and cone index in kPa.

|Quantity|Relation|
|-|-|
|Firm-surface cone index at reference moisture|$CI\_{ref} = 900 + 15,\\text{clay} + 4,\\text{silt}$|
|Surface condition multiplier|firm 1.0, tilled 0.5, loose 0.28|
|Reference moisture (near field capacity)|$w\_{FC} = 6 + 0.35,\\text{clay} + 0.1,\\text{silt}$|
|Moisture sensitivity|$\\beta = 0.025 + 0.0012,\\text{clay} + 0.0006,\\text{silt}$|
|Bekker exponent|$n = 1.1 - 0.01,\\text{clay} - 0.002,\\text{silt}$ (0.45–1.1)|
|Bekker cohesive modulus|$k\_c = 0.99 + 0.2,\\text{clay} + 0.06,\\text{silt}$ kN/m$^{n+1}$|
|Bekker frictional modulus|$k\_\\phi = \\max(600,\\ 1528 - 14,\\text{clay} - 2,\\text{silt})$ kN/m$^{n+2}$|

### Atterberg limits and cone index

|Quantity|Relation|
|-|-|
|Plasticity index|$PI = \\max(0,\\ 0.65,\\text{clay} + 0.06,\\text{silt} - 2)$; non-plastic if $PI < 4$|
|Plastic limit|$PL = 12 + 0.22,\\text{clay} + 0.10,\\text{silt}$|
|Liquid limit|$LL = PL + PI$|
|Shrinkage limit|$SL = 0.55,PL$|
|Liquidity index|$LI = (w - PL)/PI$|

The estimated cone index is $CI = CI\_{ref}, f\_m, f\_a$, with a minimum of 100 kPa.

$$f\_m = \\operatorname{clamp}!\\left(e^{-\\beta (w - w\_{FC})},\\ 0.4,\\ 2.0\\right)$$

$$f\_a = \\begin{cases} 1 \& LI \\le 0 \\text{ or non-plastic} \\ 1 - 0.7,LI \& 0 < LI \\le 1 \\ 0.3, e^{-3 (LI - 1)} \& LI > 1 \\end{cases}$$

When a measured profile is entered, the mean over 0–150 mm is used as $CI$ for the selected surface condition. The other two conditions are scaled by their multipliers. The modelled profile with depth is $CI(z) = CI\_{eff},(a + b,z)$ with $z$ in metres, $b$ = 4 (firm) or 3 (tilled, loose), and $a = 1 - 0.075,b$.

## Limitations

This is an illustrative teaching model, not a validated design tool.

* The texture-to-parameter and Atterberg relations are simple correlations built from representative terramechanics values. They are not fitted to a measured dataset.
* The Brixius model was developed for pneumatic tyres on agricultural soils. It does not represent tracks, dual wheels, implement draft or load transfer.
* Slip sinkage, the contact-patch geometry and the stress profile are simplified.
* Use measured cone index and Bekker parameters for any design, publication or decision.

## Suggested citation

Soni, P. (2026). *Tractor–Soil Interaction Simulator* \[Software]. GitHub. `https://github.com/soni-iit/tractor-soil-simulator`

## References

* Brixius, W.W. (1987). Traction prediction equations for bias ply tires. ASAE Paper No. 87-1622.
* ASABE D497, Agricultural machinery management data, and ASABE EP496, Agricultural machinery management.
* ASABE S313, Soil cone penetrometer.
* Bekker, M.G. (1969). *Introduction to Terrain-Vehicle Systems*. University of Michigan Press.
* Wong, J.Y. (2010). *Terramechanics and Off-Road Vehicle Engineering*, 2nd ed. Butterworth-Heinemann.
* Söhne, W. (1958). Fundamentals of pressure distribution and soil compaction under tractor tires. *Agricultural Engineering*, 39, 276–281.
* USDA Natural Resources Conservation Service, *Soil Survey Manual*, soil texture classes.

## Licence

MIT. See the `LICENSE` file.

