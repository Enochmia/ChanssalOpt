# ChanssalOpt (Parametric-Hanok)

**Parametric-Hanok: Architectural Optimization System** — A closed-loop computational building-physics and CAD synthesis engine for traditional Korean Hanok architecture.

- **Implementation & Engineering Blueprint:** [`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md)
- **License:** [MIT License](./LICENSE)

## Overview

`ChanssalOpt` couples macro-scale annual solar eaves optimization with micro-scale traditional Korean window lattice (*Changsal*) symmetry generation, EN ISO 10077-1 thermal-bridge modeling, and laser/CNC-ready CAD export:

1. **Annual Hourly VSA & 3D Finite-Width Solar Shading (`sun_eaves.py`):** Uses `pvlib` (NREL SPA + Ineichen clear-sky) across 12 monthly representative days (09:00–15:00) to capture Korea's August thermal lag and side-spilling geometry.
2. **6 Traditional Korean Lattice Wallpaper Groups (`lattice_symmetry.py`):** Formalizes `pmm`, `cmm`, `p4`, `p4m`, `p6m`, and `p2` with duplicate-edge removal, frame clipping, connectivity verification, and Shapely `unary_union` area/perimeter calculation.
3. **Closed-Loop Thermal & Hybrid Pareto Optimization (`thermal_pareto.py`):** Computes EN ISO 10077-1 $U_w = (A_g U_g + A_f U_f + l_g \psi) / (A_g + A_f)$ across 3 glazing presets (1-ply Hanji, 2-ply Hanji, Low-E double glazing), models `ACH50` infiltration sensitivity, and generates discrete–continuous Pareto fronts.
4. **Fabrication-Ready CAD Export (`exporter.py`):** Outputs true-millimeter `viewBox` SVGs (`drawsvg`) and multi-layered DXFs (`ezdxf`: `FRAME`, `LATTICE_CUT`, `CENTERLINE`).

## License & Third-Party Notices

This repository is licensed under the **MIT License** ([`LICENSE`](./LICENSE)).

External dependencies are used under permissive or weak-copyleft licenses without bundling GPL/AGPL code (`pysolar`, `ladybug-*`, and `create_symmetry` are explicitly excluded):

| Package | License | Attribution / Redistribution Notice |
| :--- | :--- | :--- |
| `pvlib`, `Shapely`, `NumPy`, `SciPy` | **BSD-3-Clause** | Upstream copyright notices and disclaimers apply. Per the BSD-3-Clause non-endorsement clause, upstream project/contributor names are not used to endorse derived products without prior written permission. |
| `GEOS` *(via `Shapely` wheels)* | **LGPL-2.1** | `Shapely` dynamically links against **GEOS** (LGPL-2.1). If redistributing compiled/bundled binaries, retain GEOS LGPL-2.1 notices and relinking compatibility. |
| `Streamlit` | **Apache-2.0** | Upstream copyright and `NOTICE` file attributions apply upon redistribution. |
| `drawsvg`, `ezdxf`, `Plotly`, `pytest`, `trimesh` | **MIT** | Upstream MIT copyright and permission notices apply. |
