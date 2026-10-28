<!-- This file is compiled into the website through .website_src. -->

# Laminar channel flow with immersed boundaries

This verification case measures the spatial convergence of pressure-driven,
single-phase laminar channel flow. It verifies both a flat channel aligned to
cell faces and a $45^\circ$ slanted channel represented with
`ChannelBuilder` immersed boundaries. Non-immersed-boundary channel-flow
simulations provide a solver reference.

![Flat-channel geometry](figures/flat_geometry_setup.png)

![Slanted-channel geometry](figures/slanted_geometry_setup.png)

## Contents

- [Verification matrix](#verification-matrix)
- [Analytical solution](#analytical-solution)
- [Prerequisites](#prerequisites)
- [Generate solver inputs](#generate-solver-inputs)
- [Run the solver](#run-the-solver)
- [Analyze convergence](#analyze-convergence)
- [Outputs](#outputs)

## Verification matrix

The supported matrix is deliberately small and fixed:

| Series | Geometry | Limiter | $N_x$ |
| --- | --- | --- | --- |
| Flat IB | Cell-face-aligned channel | `default`, `original` | 32, 64, 128, 256, 512, 1024 |
| Slanted IB | $45^\circ$ channel | `default`, `original` | 32, 64, 128, 256, 512, 1024 |
| Flat Reference | No IB | N/A | 32, 64, 128, 256 |

This produces 28 runs: 24 IB cases and four references. The
`default` configuration utilizes the current `DragForcing` implementation. `original` sets `DragForcing.terrain_use_original_implementation = true`.

Shared solver settings are provided in `base.inp`, and geometry-specific
settings are in `flat-ib.inp`, `flat-no-ib.inp`, and `slanted.inp`.
Each generated case explicitly includes the shared settings first and its
geometry-specific settings second. Generated immersed-boundary cases set both
drag-limiter switches explicitly.

## Analytical solution

For channel height $H$, viscosity $\mu$, and axial body-force magnitude
$F$, the fully developed Poiseuille solution is

$$
u(r) = U_{\max}\left(1 - \left(\frac{2r}{H}\right)^2\right),
\qquad
U_{\max} = \frac{F H^2}{8\mu}.
$$

For a centerline with unit tangent $\boldsymbol{t}=(t_x,t_z)$, the analyzer
projects numerical velocity onto that tangent and calculates the perpendicular
distance using the in-plane unit normal
 $\boldsymbol{n}=(-t_z,t_x)$. The analytical profile is zero outside the
 physical channel. Flat and no-IB reference errors are evaluated in the
 physical channel, $r \leq H/2$. Slanted-channel errors use
 $r \leq 0.6H$ to include a small surrounding region while excluding the
 periodic companion segments; this measures any velocity leakage there against
 the zero analytical solution. The reported metric is relative $L_\infty$
 velocity error; the analyzer also reports the observed order between adjacent
 resolutions.

## Prerequisites

Use the case-specific `kynema-sgf-env` conda environment. It contains the
Python, NumPy, Matplotlib, and `yt` dependencies required by the generator
and analyzer.

From this directory, create the environment when necessary and activate it:

```bash
conda env create -f python/kynema-sgf-env.yml
conda activate kynema-sgf-env
```

An available Kynema-SGF executable is also required to run the generated
inputs. Solver launch and resource selection are intentionally controlled by
the user or the target HPC system.

## Generate solver inputs

Generate the complete matrix with:

```bash
python python/generate_cases.py
```

The command writes solver directories and a `manifest.json` file directly in
`cases/`. Each directory contains `inputs.inp`, which includes the checked-in
shared and geometry-specific inputs, then overrides `amr.n_cell` and, where
needed, the drag-limiter settings.

## Run the solver

Run the Kynema-SGF executable from every generated case directory with its
`inputs.inp` file. For example, after generating the full matrix:

```bash
cd cases/flat-default-n128
srun -N 1 --ntasks-per-node=50 kynema_sgf inputs.inp
```

The solver must produce AMReX plot-file directories named `plt*` in the same
case directory. The analyzer selects the plot file with the largest numerical
suffix, so each run may retain intermediate outputs and checkpoints.

## Analyze convergence

After all jobs complete, run:

```bash
python python/analyze.py
```

The analyzer reads `cases/manifest.json`, validates each required velocity
component through `yt`, and loads each case's final plot file. Cases without
a plot file are reported and skipped.

## Outputs

Analysis results are written to `cases/analysis/` by default:

- `flat_error_summary.csv` (flat and no-IB reference cases) and
  `slanted_error_summary.csv` contain the case metadata, selected plot file, sampled
  cell count, absolute and relative $L_\infty$ error, and observed order.
- `flat_convergence.png` compares the two flat immersed-boundary limiters and
  overlays the no-IB reference (with a slope-2 line) when reference results are
  available.
- `slanted_convergence.png` compares the two slanted immersed-boundary limiters.

For a converged steady solution, the reference series should demonstrate the
expected second-order trend. The immersed-boundary series quantify how
alignment, orientation, and limiter selection affect that behavior.