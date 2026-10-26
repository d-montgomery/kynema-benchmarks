<!-- This file is compiled into the website. -->

# Laminar channel flow with immersed boundaries

This verification case measures the spatial convergence of pressure-driven,
single-phase laminar channel flow. It verifies both a flat channel aligned to
cell faces and a $45^\circ$ slanted channel represented with
`ChannelBuilder` immersed boundaries. A non-immersed-boundary channel-flow
series provides a solver reference.

## Verification matrix

| Series | Geometry | Limiter | $N_x$ |
| --- | --- | --- | --- |
| Flat IB | Cell-face-aligned channel | `default`, `original` | 32, 64, 128, 256, 512, 1024 |
| Slanted IB | $45^\circ$ channel | `default`, `original` | 32, 64, 128, 256, 512, 1024 |
| Reference | No immersed boundary | Not applicable | 32, 64, 128, 256 |

The 28-run study contains 24 immersed-boundary cases and four no-IB reference
cases. The current temporal drag limiter is named `default`, replacing the
historical `tf1` label. `original` remains as the comparison configuration.

## Analytical solution

For channel height $H$, viscosity $\mu$, and axial body-force magnitude
$F_\parallel$, the fully developed Poiseuille solution is

$$
u_\parallel(r) = U_{\max}\left(1 - \left(\frac{2r}{H}\right)^2\right),
\qquad
U_{\max} = \frac{F_\parallel H^2}{8\mu}.
$$

The convergence metric is relative $L_\infty$ velocity error over
$r < 0.6H$, with observed orders calculated between adjacent grids.

## Generate solver inputs

Use the case-specific `kynema-sgf-env` conda environment. Generate the
complete case matrix with:

```bash
python python/generate_cases.py
```

The generator writes solver directories and a manifest below `generated_cases/`.
It supports dry runs and narrow geometry, limiter, and resolution selections.

## Run and analyze

Run Kynema-SGF in every generated case directory using its `inputs.inp` file.
After the solver produces `plt*` output directories, analyze the full matrix:

```bash
python python/analyze.py
```

The analyzer validates required velocity fields through `yt`, writes a CSV
summary, reports refinement orders, and creates flat, slanted, and no-IB
reference convergence figures in `generated_cases/analysis/`.