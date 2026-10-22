# Tractor–Soil Interaction Simulator

An interactive, browser-based teaching model of tractor traction. You choose a drive type (2WD, MFWD or 4WD) and tractor tyres commonly used in India, set the soil from sand, silt and clay on the USDA triangle, and see how moisture, inflation pressure and slip change traction, sinkage and subsoil stress. Results are given in two clearly separated blocks: one **single driving wheel**, and the **whole tractor**.

**Live version:** `https://soni-iit.github.io/tractor-soil-simulator/` (replace with your GitHub Pages address)

The whole simulator is one file, `index.html`. There is nothing to install: open it in any modern browser, online or offline. (An internet connection is only used to load the fonts. Without it the page falls back to system fonts.)

## What you can do

- Select the drive type (2WD, MFWD or 4WD), the rear tyre and the front tyre from typical Indian sizes, and set tractor mass, rear static weight share, wheelbase, drawbar hitch height, tyre inflation pressures and (for MFWD) front-wheel lead.
- Set soil texture by dragging in the USDA triangle, moving the sand, silt and clay sliders, or choosing a typical composition for any of the 12 USDA classes.
- Choose a surface condition (firm, tilled or loose) and the soil moisture (% dry basis).
- Read the Atterberg consistency state (solid, semi-solid, plastic, liquid) at the chosen moisture.
- Optionally enter a measured cone-index profile with depth (0 to 50 cm). Its mean over 0–150 mm replaces the estimated cone index.
- Read **single driving wheel** results: tractive efficiency, net traction ratio, drawbar pull and power for that wheel, mobility number, rut depth, contact pressure and stress at 0.4 m depth, with charts and warnings. You can analyse the rear wheel or, on MFWD and 4WD, a front driving wheel, using the dynamic load from the tractor or a manual load.
- Read **whole tractor** results: drawbar pull and power, tractive efficiency, pull–weight ratio, dynamic axle loads, weight transfer, and rear and front slip, with a chart against rear-wheel slip.
- Drag on the slip charts to set slip. Optional operating scenarios keep your tractor and soil texture, and set surface condition, moisture, slip, speed and rear inflation.

## Tractor tyres

Rear tyres are grouped as in common Indian practice. Overall diameter is estimated from the nominal size as $`d = D_{rim} + 2 \times 0.85 \times w`$ (0.9 for the narrow 2WD front tyres), with section width $`w`$ in inches. The geometry is approximate, not manufacturer data.

| Category | Rear tyre sizes |
|---|---|
| Mini / small tractor | 8.00-18, 8.3-20, 9.5-20 |
| Compact tractor | 8.3-24, 9.5-24, 11.2-24, 12.4-24, 14.9-24 |
| Standard tractor | 11.2-28, 12.4-28, 13.6-28, 14.9-28, 15.9-28, 16.9-28, 17.9-28 |
| Large tractor | 16.9-30, 18.4-30 |
| Specialized agricultural tractor | 8.3-32, 12.4-32, 13.6-38, 15.5-38, 16.9-38 |

Front tyres offered are 5.00-15, 6.00-16, 6.50-16, 7.50-16 and 7.50-18 for 2WD (free-rolling), and 7.50-16, 8.3-20, 8.3-24, 9.5-24, 11.2-24, 12.4-24 and 13.6-24 for MFWD. The 4WD option uses the rear tyre size on both axles.

## Model

### Single driving wheel (Brixius wheel-numeric model)

```math
B_n = \frac{CI \, b \, d}{W} \cdot \frac{1 + 5\,\delta/h}{1 + 3\,b/d}
```

```math
GTR = 0.88\left(1 - e^{-0.1 B_n}\right)\left(1 - e^{-7.5 s}\right) + 0.04
```

```math
MRR = \frac{1.2}{B_n} + 0.04 + \frac{0.5\, s}{\sqrt{B_n}}
```

```math
NTR = GTR - MRR, \qquad TE = \frac{NTR\,(1 - s)}{GTR}
```

Here $`CI`$ is the cone index (kPa), $`b`$ the tyre width, $`d`$ the tyre diameter, $`h`$ the section height, $`\delta`$ the tyre deflection, $`W`$ the wheel load, and $`s`$ the slip as a fraction. Drawbar pull of one wheel is $`NTR \cdot W`$.

### Tyre contact

```math
\frac{\delta}{h} = 0.15\,\frac{W}{20}\left(\frac{100}{p_i}\right)^{0.85} \quad (\text{limited to } 0.04 \text{ to } 0.35)
```

```math
L = 2\sqrt{d\,\delta}, \qquad A = \frac{\pi\, b\, L}{4}, \qquad p_c = \frac{W}{A}
```

with $`W`$ in kN and $`p_i`$ in kPa.

### Sinkage and subsoil stress

```math
z = \left(\frac{p_c}{k_c/b + k_\phi}\right)^{1/n}(1 + 4s)
```

```math
\sigma_z = p_c\left[1 - \left(\frac{1}{1 + a^2/z^2}\right)^{\nu/2}\right], \qquad a = \sqrt{A/\pi}
```

The Bekker sinkage uses $`k_c`$ and $`k_\phi`$ scaled by the ratio of the current cone index to the firm-soil reference. The stress concentration factor $`\nu`$ is 4 for firm and 5 for tilled and loose surfaces, reduced by $`0.08\,(w - w_{FC})`$ and limited to 2.5 to 6. Compaction risk at 0.4 m is flagged as low below 50 kPa, moderate from 50 to 100 kPa and high above 100 kPa. These thresholds are indicative only.

### Whole tractor

Static axle loads come from the tractor mass $`m`$ and the rear weight share. With drawbar pull $`DP`$ at hitch height $`h_d`$ and wheelbase $`L_{wb}`$, load moves to the rear axle:

```math
W_r = W_{r0} + DP\,\frac{h_d}{L_{wb}}, \qquad W_f = W_{f0} - DP\,\frac{h_d}{L_{wb}}
```

The drawbar pull depends on the drive type. Because the axle loads depend on $`DP`$, the model solves them by iteration.

```math
\text{2WD: } DP = W_r\,NTR_r - W_f\,MRR_f(s = 0)
```

```math
\text{MFWD, 4WD: } DP = W_r\,NTR_r + W_f\,NTR_f
```

On MFWD the front wheels have a positive lead, so they slip more than the rear wheels:

```math
s_f = 1 - \frac{1 - s_r}{1 + \text{lead}}
```

The 4WD option has equal tyres on both axles and no lead. Whole-tractor tractive efficiency is the drawbar power divided by the axle input power:

```math
\eta = \frac{DP \, v_a}{\sum GTR\; W\; v_t}, \qquad v_a = v_t\,(1 - s_r)
```

Simplifications: the pull is taken as horizontal, rolling-moment terms are ignored, front wheels are assumed to run in undisturbed soil, and rear wheels running in front-wheel ruts are not modelled. A warning appears when the front axle carries less than 20% of the weight.

### Soil texture to parameters

All relations below use clay and silt in %, moisture $`w`$ in % (dry basis), and cone index in kPa.

| Quantity | Relation |
|---|---|
| Firm-surface cone index at reference moisture | $`CI_{ref} = 900 + 15\,\text{clay} + 4\,\text{silt}`$ |
| Surface condition multiplier | firm 1.0, tilled 0.5, loose 0.28 |
| Reference moisture (near field capacity) | $`w_{FC} = 6 + 0.35\,\text{clay} + 0.1\,\text{silt}`$ |
| Moisture sensitivity | $`\beta = 0.025 + 0.0012\,\text{clay} + 0.0006\,\text{silt}`$ |
| Bekker exponent | $`n = 1.1 - 0.01\,\text{clay} - 0.002\,\text{silt}`$ (limited to 0.45 to 1.1) |
| Bekker cohesive modulus | $`k_c = 0.99 + 0.2\,\text{clay} + 0.06\,\text{silt}`$ kN/m^(n+1) |
| Bekker frictional modulus | $`k_\phi = \max(600,\ 1528 - 14\,\text{clay} - 2\,\text{silt})`$ kN/m^(n+2) |

### Atterberg limits and cone index

| Quantity | Relation |
|---|---|
| Plasticity index | $`PI = \max(0,\ 0.65\,\text{clay} + 0.06\,\text{silt} - 2)`$; non-plastic if PI is below 4 |
| Plastic limit | $`PL = 12 + 0.22\,\text{clay} + 0.10\,\text{silt}`$ |
| Liquid limit | $`LL = PL + PI`$ |
| Shrinkage limit | $`SL = 0.55\,PL`$ |
| Liquidity index | $`LI = (w - PL)/PI`$ |

The estimated cone index is $`CI = CI_{ref}\, f_m\, f_a`$, with a minimum of 100 kPa.

```math
f_m = \mathrm{clamp}\left(e^{-\beta (w - w_{FC})},\ 0.4,\ 2.0\right)
```

```math
f_a =
\begin{cases}
1 & LI \le 0 \text{ or non-plastic} \\
1 - 0.7\,LI & 0 < LI \le 1 \\
0.3\, e^{-3 (LI - 1)} & LI > 1
\end{cases}
```

When a measured profile is entered, the mean over 0–150 mm is used as $`CI`$ for the selected surface condition. The other two conditions are scaled by their multipliers. The modelled profile with depth is:

```math
CI(z) = CI_{eff}\,(a + b\,z), \qquad a = 1 - 0.075\,b
```

with $`z`$ in metres and $`b`$ = 4 (firm) or 3 (tilled, loose).

## Limitations

This is an illustrative teaching model, not a validated design tool.

- The texture-to-parameter and Atterberg relations are simple correlations built from representative terramechanics values. They are not fitted to a measured dataset.
- The Brixius model was developed for pneumatic tyres on agricultural soils. It does not represent tracks, dual wheels or implement draft.
- The whole-tractor block is a simplified estimate. Default tractor masses, weight distributions, front tyre options and tyre geometry are typical values, not data for any specific tractor model.
- Slip sinkage, the contact-patch geometry and the stress profile are simplified.
- Use measured cone index and Bekker parameters for any design, publication or decision.

## Suggested citation

Soni, P. (2026). *Tractor–Soil Interaction Simulator* [Software]. GitHub. `https://github.com/soni-iit/tractor-soil-simulator`

## References

- Brixius, W.W. (1987). Traction prediction equations for bias ply tires. ASAE Paper No. 87-1622.
- ASABE D497, Agricultural machinery management data, and ASABE EP496, Agricultural machinery management.
- ASABE S313, Soil cone penetrometer.
- Bekker, M.G. (1969). *Introduction to Terrain-Vehicle Systems*. University of Michigan Press.
- Wong, J.Y. (2010). *Terramechanics and Off-Road Vehicle Engineering*, 2nd ed. Butterworth-Heinemann.
- Söhne, W. (1958). Fundamentals of pressure distribution and soil compaction under tractor tires. *Agricultural Engineering*, 39, 276–281.
- USDA Natural Resources Conservation Service, *Soil Survey Manual*, soil texture classes.

## Licence

MIT. See the `LICENSE` file.
