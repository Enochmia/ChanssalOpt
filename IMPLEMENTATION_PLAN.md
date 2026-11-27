# ChanssalOpt — Parametric-Hanok: Architectural Optimization System

**Project Execution & Engineering Blueprint (v2 — Audit-Revised Baseline)**

> **License:** MIT License ([`LICENSE`](./LICENSE))  
> **Third-Party Notices:** See Section 6 (BSD-3-Clause, Apache-2.0, and GEOS LGPL-2.1 attribution)

---

## 1. Overview and Background

### 1.1 Historical Motif: Hyeeumwonji (혜음원지)

The **Parametric-Hanok (`ChanssalOpt`)** project bridges traditional Korean timber architecture with verifiable computational building physics. Its architectural motif draws from **Hyeeumwonji (혜음원지)**, a Goryeo Dynasty royal temporary palace and traveler's inn in Paju, South Korea, renowned for terrain-adaptive courtyard planning and passive solar eaves proportions (*Cheoma-gil-i*, 처마 내밀기).

Traditional Hanok structures regulate seasonal solar gain through calibrated eaves geometry—blocking high summer sun (including the August thermal-lag heatwave) while admitting low winter sun deep into living quarters (*Anbang*)—paired with wooden window lattices (*Changsal*) covered with *Hanji* or modern Low-E glazing. **ChanssalOpt** formalizes this vernacular intelligence into a **single closed-loop computational engine** coupling macro-scale annual solar shading with micro-scale lattice symmetry geometry, EN ISO 10077-1 thermal transmittance, and laser/CNC-ready CAD export.

### 1.2 Core Vision & Audit-Aligned Project Goals

1. **Annual Hourly VSA & 3D Finite-Width Solar Optimization (Macro):** Replace naive 2-day solstice-noon closed-form geometry and degenerate 2D roof-curvature optimization with `pvlib`-based 12-month hourly (09:00–15:00) Vertical Shadow Angle (VSA) and 3D finite-width side-spilling irradiance integration that accounts for Korea's August thermal lag.
2. **Formalized 6 Traditional Korean Lattice Symmetry Groups (Micro Geometry):** Focus on the 6 structurally constructible wallpaper groups used in Korean joinery (`pmm`, `cmm`, `p4`, `p4m`, `p6m`, `p2`) rather than over-generalizing all 17 wallpaper groups. Enforce zero floating segments, duplicate coincident edge removal, Shapely rectangular frame clipping, and `unary_union` boolean synthesis for exact area and perimeter metrics.
3. **Closed-Loop Macro–Micro Thermal & Daylight Physics (EN ISO 10077-1):** Couple hourly eaves shading factors directly into window incident solar gain. Compute window transmittance $U_w$ including linear thermal bridge perimeter ($l_g \cdot \psi$), support 3 glazing presets (1-ply Hanji, 2-ply Hanji, Modern Low-E double glazing capturing thermal-inversion physics), and include explicit infiltration (`ACH50`) sensitivity analysis.
4. **Hybrid Discrete–Continuous Pareto Optimization:** Eliminate linear Bang-Bang extreme solutions ($f = 0$ or $1$) and gradient-based `SLSQP` silent failures on discrete step functions by enumerating discrete cell divisions $(n_x, n_y)$ combined with continuous lattice bar width $w \in [w_{\min}, w_{\max}]$ (or `differential_evolution(..., integrality=...)`) to generate multi-objective Pareto fronts across patterns.
5. **Strictly Permissive Open-Source Stack & Production CAD Export:** Exclude GPL/AGPL and unmaintained packages (`pysolar`, `ladybug-*`, `svgwrite`, `create_symmetry`). Export true-millimeter `viewBox` laser-cutting SVGs via `drawsvg` and multi-layered DXFs (`FRAME`, `LATTICE_CUT`, `CENTERLINE`) via `ezdxf`.

---

## 2. System Architecture & Tech Stack

### 2.1 Software Architecture Pipeline (Pure Domain Engine / Thin UI)

Following the **Pure Generator, Thin UI** discipline, all domain physics, geometry, optimization, and CAD export logic lives under `src/parametric_hanok/` with zero Streamlit dependencies, enabling deterministic `pytest` verification and one-command CLI benchmarking.

```text
[ Input Parameters: Latitude, Window Bounds, Glazing Preset, ACH50 ]
                               │
                               ▼
    ┌────────────────────────────────────────────────────────┐
    │  src/parametric_hanok/sun_eaves.py                     │
    │  • pvlib NREL SPA + Ineichen Clear-Sky (12 mo × 09–15h)│
    │  • VSA + 3D Finite-Width Side-Spilling Shading Engine  │
    │  • Decision Vars: Eaves Projection L_e & Clearance g   │
    │  • Output: Hourly Shading Factor S(t) & Optimal (L_e,g)│
    └──────────────────────────┬─────────────────────────────┘
                               │  Hourly Shading Factor S(t)
                               ▼
    ┌────────────────────────────────────────────────────────┐
    │  src/parametric_hanok/lattice_symmetry.py              │
    │  • 6 Wallpaper Groups (pmm, cmm, p4, p4m, p6m, p2)     │
    │  • Deduplicate Edges → Frame Clip → unary_union(w)     │
    │  • Output: Exact A_f, A_g, Boundary Perimeter l_g, Poly│
    └──────────────────────────┬─────────────────────────────┘
                               │  (A_f, A_g, l_g) + S(t)
                               ▼
    ┌────────────────────────────────────────────────────────┐
    │  src/parametric_hanok/thermal_pareto.py                │
    │  • EN ISO 10077-1: U_w = (A_g·U_g + A_f·U_f + l_g·ψ)/A │
    │  • 3 Glazing Presets + ACH50 Infiltration Sensitivity  │
    │  • Hybrid Discrete–Continuous Pareto Front (U_w vs VLT)│
    └──────────────┬───────────────────────────┬─────────────┘
                   │                           │
                   ▼                           ▼
    ┌────────────────────────────┐  ┌─────────────────────────────┐
    │  exporter.py               │  │  app.py (Thin Streamlit UI) │
    │  • drawsvg: True-mm SVG    │  │  • @st.cache_data           │
    │  • ezdxf: Layered DXF      │  │  • @st.fragment isolation   │
    │  • (Opt. Phase 2: 3D OBJ)  │  │  • st.form explicit solver  │
    └────────────────────────────┘  └─────────────────────────────┘
```

### 2.2 Tech Stack & Licensing Audit (Verified Nov 2026)

To ensure clean MIT distribution and avoid copyleft license contamination or abandoned libraries, all dependencies are restricted to actively maintained, permissive packages.

| Category | Library / Tool | Target Version | License | Primary Responsibility & Audit Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Solar Dynamics** | `pvlib` | `>=0.16.1` | BSD-3-Clause | NREL SPA solar position & Ineichen clear-sky irradiance (replaces GPL-3.0 `pysolar`) |
| **2D Geometry** | `Shapely` | `>=2.2.0` | BSD-3-Clause *(GEOS: LGPL-2.1)* | Polygon clipping, `unary_union`, exact $A_f, A_g, l_g$ (dynamic link notice for GEOS) |
| **Numerical & Opt.** | `NumPy` / `SciPy` | `>=2.5` / `>=1.18.1` | BSD-3-Clause | Affine transforms, continuous eaves optimization, `differential_evolution(integrality=...)` |
| **SVG Export** | `drawsvg` | `>=2.5.1` | MIT | True-mm `viewBox` SVG export (replaces inactive `svgwrite` & avoids `ezdxf` SVG coordinate compression) |
| **CAD Export** | `ezdxf` | `>=1.4.5` | MIT | Multi-layered DXF (`FRAME`, `LATTICE_CUT`, `CENTERLINE`) compatible with AutoCAD/LibreCAD |
| **Visualization** | `Plotly` | `>=7.2.0` | MIT | Interactive 2D/3D ray-shading cross-sections, annual solar charts, Pareto front plots |
| **Web Dashboard** | `Streamlit` | `>=1.67.0` | Apache-2.0 | Thin reactive dashboard with `@st.cache_data`, `@st.fragment`, and `st.form` |
| **Testing** | `pytest` | `>=8.0` | MIT | Automated physics, geometry, topology, and CAD round-trip verification |
| **Optional (Phase 2)** | `trimesh` | `>=4.0.0` | MIT | Optional 3D framing/lattice `.obj`/`.stl` mesh export after Dec 1 MVP freeze *(deferred from core MVP)* |

**Explicitly Excluded Libraries:**
- `pysolar` (GPL-3.0 — license contamination risk)
- `ladybug-core` / `ladybug-geometry` / `honeybee` (AGPL-3.0 / GPL-3.0 — network copyleft propagation)
- `phyver/create_symmetry` (GPL-3.0 — raster-only; never referenced or copied)
- `svgwrite` (PyPI status `7 - Inactive` since 2022-07; replaced by `drawsvg`)

---

## 3. Detailed Module Specifications

### 3.1 `src/parametric_hanok/sun_eaves.py` (Macro Annual Solar & Eaves Engine)

Computes hourly solar vectors across representative days of all 12 months and evaluates 3D finite-width window shading.

- **Architectural & Mathematical Corrections Applied:**
  - **Eliminates degenerate 2D curvature optimization:** For any monotonically descending roof cross-section (traditional Hanok *mulmae* pitch 25°–35°, average 29°), $y(x) - x\tan\alpha$ is strictly decreasing on $[0, L_e]$, so shadow depth depends solely on the outermost eaves tip $(L_e, y_e)$ with $\partial J / \partial \kappa \equiv 0$ (0.000 m difference). Therefore, catenary/Bézier roof curves and 3D *Anggok* (앙곡, front elevation corner uplift) / *Anheorigok* (안허리곡, plan-view corner outward flare) are rendered strictly as **architectural reference geometry & drainage indicators**, not optimization variables.
  - **Vertical Shadow Angle (VSA) & 3D Finite-Width Side-Spilling:** For solar altitude $\alpha > 0$ and relative wall azimuth $\gamma$ with $|\gamma| < 90^\circ$ (guarding against $\gamma = \pm 90^\circ$ singularities and back-of-facade sun), the vertical projection angle is:
    - $\tan(\gamma_p) = \tan(\alpha) / \cos(\gamma)$
    - Vertical shadow line on the facade from eaves tip $(L_e, y_e)$: $y_{\text{shadow}} = y_e - L_e \cdot \tan(\gamma_p)$
    - Horizontal lateral shadow shift due to finite window width $W_{\text{win}}$ and side eaves overhangs $(e_{\text{left}}, e_{\text{right}})$: $\Delta x = L_e \cdot \tan(\gamma)$
    - Unshaded window area fraction is computed by exact 2D polygon intersection of the projected eaves shadow polygon against the window aperture rectangle $[0, W_{\text{win}}] \times [y_{\text{sill}}, y_{\text{head}}]$.
- **Optimization Formulation (Why `scipy.optimize` is Non-Trivial):**
  - **Decision Variables:** Eaves projection $L_e \in [0.30, 2.50]\text{ m}$ and vertical eaves clearance $g = y_e - y_{\text{head}} \in [0.10, 0.80]\text{ m}$ (optionally lateral overhang margin $e_{\text{side}}$).
  - **Objective Function:** Weighted annual thermal-load index over 12 monthly representative days (09:00–15:00 hourly, using `pvlib` Ineichen clear-sky POA irradiance):
    - Minimize $J(L_e, g) = w_{\text{cool}} \cdot \sum_{\text{Jun–Sep}} I_{\text{unshaded}}(t) - w_{\text{heat}} \cdot \sum_{\text{Nov–Feb}} I_{\text{unshaded}}(t)$
  - Unlike the 2-day solstice-noon closed-form solution ($L_e \approx 0.478\text{ m}$, which leaves 51.0% of the window exposed at noon on Aug 15), this annual integral has no closed-form solution and yields realistic residential Hanok projections ($L_e \approx 0.90\text{–}1.20\text{ m}$ for Seoul 37.57°N).
- **Core Functions & Signatures:**
  - `get_annual_solar_table(lat: float, lon: float, year: int = 2026) -> pd.DataFrame`
  - `compute_3d_shading_fraction(eaves: EavesGeom, window: WindowGeom, altitude_deg: float, rel_azimuth_deg: float) -> float`
  - `evaluate_annual_solar_load(eaves: EavesGeom, window: WindowGeom, solar_df: pd.DataFrame) -> AnnualSolarResult`
  - `optimize_eaves(window: WindowGeom, solar_df: pd.DataFrame, bounds: EavesBounds) -> EavesOptResult`

### 3.2 `src/parametric_hanok/lattice_symmetry.py` (6 Korean Wallpaper Groups & Computational Geometry)

Generates structurally constructible traditional Korean window lattices (*Changsal*) using affine symmetry matrices and exact 2D boolean geometry.

- **Supported 6 Traditional Wallpaper Groups (Corrected Mapping):**

| Traditional Pattern | Wallpaper Group | Geometric & Joinery Characteristics |
| :--- | :--- | :--- |
| **Tti-sal / A-ja-sal** (띠살 · 아자살) | `pmm` | Orthogonal anisotropic rhythm; full-span vertical/horizontal bars to frame |
| **Yong-ja-sal** (용자살) | `cmm` | Centered rectangular orthogonal reflection & centered sub-bars |
| **Wan-ja-sal** (완자살 · 卍) | `p4` | 90° rotational symmetry without mirror axes; continuous interlocking L-bars |
| **Jeong-ja-sal / 45° Bit-sal** (정자살 · 빗살) | `p4m` | Square orthogonal grid or 45° diagonal lattice with 4-fold dihedral symmetry |
| **Gwigap-sal / Soseul-bit-sal** (귀갑살 · 소슬빗살) | `p6m` | Hexagonal/triangular 60° 3-way intersections (high perimeter $l_g$) |
| **Modified Rotational Lattice** (변형 회전살) | `p2` | 180° rotational periodic bars connected continuously to outer frame |

- **4-Stage Computational Geometry Pipeline:**
  1. **Affine Group Generation:** Generate fundamental domain centerlines and tile via exact 2D rotation/reflection/translation matrices over integer cell counts $(n_x, n_y)$.
  2. **Duplicate Coincident Edge Removal:** Quantize endpoints to $10^{-4}\text{ mm}$ tolerance, canonicalize edge ordering (`p0 < p1`), and deduplicate so laser cutters never burn the same path twice.
  3. **Exact Rectangular Frame Clipping & Connectivity Verification:** Intersect centerlines with the inner frame box (`outer_frame` minus outer *Ulgeomi* stile width) via Shapely `.intersection()`, then verify graph connectivity so **zero floating wood pieces** exist.
  4. **Boolean Union (`unary_union`) & Physical Metrics:** Buffer deduplicated centerlines by half bar width ($w/2$) with flat/miter joins, union with the outer frame polygon via `shapely.unary_union`, and subtract from total window area $A_{\text{tot}}$ to eliminate double-counted $w^2$ intersection areas. Return exact wood area $A_f$, glazing aperture area $A_g$ (verified $|A_f + A_g - A_{\text{tot}}| < 10^{-6}\text{ m}^2$), and exact wood–glazing **boundary perimeter $l_g$** (sum of interior hole perimeters).
- **Core Functions & Signatures:**
  - `generate_lattice_centerlines(pattern: PatternType, inner_box: tuple[float, float], nx: int, ny: int) -> list[LineString]`
  - `deduplicate_edges(lines: list[LineString], tol_mm: float = 1e-4) -> list[LineString]`
  - `build_lattice_geometry(pattern: PatternType, window_w: float, window_h: float, frame_w: float, nx: int, ny: int, bar_w: float) -> LatticeGeometryResult` (contains `wood_polygon`, `glazing_holes`, `centerlines`, `A_f`, `A_g`, `l_g`, `is_connected`)

### 3.3 `src/parametric_hanok/thermal_pareto.py` (EN ISO 10077-1, Infiltration & Hybrid Pareto Engine)

Couples macro eaves shading with micro lattice geometry under international fenestration standards.

- **EN ISO 10077-1 Window Thermal Transmittance Formula:**
  $$U_w = \frac{A_g U_g + A_f U_f + l_g \psi}{A_g + A_f}$$
  Including $l_g \cdot \psi$ directly couples pattern geometry to thermal physics: at an identical 35% wood area ratio, *Gwigap-sal* (`p6m`) has ~39.9% longer perimeter $l_g$ than *Tti-sal* (`pmm`), creating measurable thermal differentiation across patterns.
- **3 Verified Glazing & Material Presets (Capturing Thermal Inversion):**
  1. **Traditional 1-ply Hanji (전통 1겹 한지):** Pine frame $U_f = 2.82\text{ W/(m}^2\cdot\text{K)}$ ($\lambda = 0.13\text{ W/(m}\cdot\text{K)}$, 24 mm, $R_{si} = 0.13$, $R_{se} = 0.04$ per EN ISO 10077-1 & UK BR 443); Hanji $U_g = 5.77\text{ W/(m}^2\cdot\text{K)}$, $\text{SHGC} = 0.29$, $\text{VLT} \approx 0.50$ (wood insulates better than Hanji).
  2. **Traditional 2-ply Hanji Double Window (전통 2겹 한지 이중창, 20 mm air gap):** $U_g = 2.83\text{ W/(m}^2\cdot\text{K)}$, $\text{SHGC} \approx 0.22$, $\text{VLT} \approx 0.35$ ($U_g \approx U_f$, demonstrating insulation neutrality).
  3. **Modern Shin-Hanok Low-E Double Glazing (신한옥 로이 복층 시스템창):** $U_g = 1.10\text{ W/(m}^2\cdot\text{K)}$, $\text{SHGC} = 0.56$, $\text{VLT} = 0.72$, $\psi = 0.04\text{ W/(m}\cdot\text{K)}$, KS F 2292 Grade 1 airtightness (**thermal inversion**: glazing outperforms wood frame).
- **Infiltration (`ACH50`) Sensitivity Module:**
  - Models conductive window heat loss vs. infiltration heat loss ($Q_{\text{inf}} = 0.33 \cdot V_{\text{room}} \cdot (\text{ACH50} / 20) \cdot \Delta T$, per KIEAE Shin-Hanok blower-door field measurements of $\text{ACH50} = 8.42\text{–}78.38\text{ h}^{-1}$). Clearly communicates that lattice thermal optimization assumes airtight system framing (KS F 2292 Grade 1) and quantifies relative impact via an interactive slider.
- **Macro–Micro Closed-Loop & Hybrid Discrete–Continuous Pareto Optimization:**
  - Receives hourly unshaded fractions $S(t)$ from `sun_eaves.py` to compute effective transmitted winter solar gain and daylight availability ($\tau_{\text{eff}} = (A_g / A_{\text{tot}}) \cdot \text{VLT} \cdot S_{\text{avg}}$).
  - **Solves Bang-Bang & `SLSQP` Silent Failure:** Never uses `SLSQP` or `Nelder-Mead` on integer step functions. Instead:
    - Enumerates feasible discrete grid divisions $(n_x, n_y) \in [2, 12] \times [2, 16]$ subject to structural stiffness minimum bar width $w \ge w_{\min}$ (e.g., 12 mm) and maximum $w \le w_{\max}$ (e.g., 35 mm), paired with continuous bar-width sweeps or `scipy.optimize.differential_evolution(..., integrality=[True, True, False])`.
    - Computes the non-dominated **Pareto Front** of:
      - Objective 1 ($f_1$): Minimize window thermal transmittance $U_w\ [\text{W/(m}^2\cdot\text{K)}]$ (or winter net thermal loss including eaves-coupled solar gain).
      - Objective 2 ($f_2$): Maximize effective daylight/aperture transmittance $\tau_{\text{eff}}$ (minimizing $-\tau_{\text{eff}}$, with optional glare penalty above threshold).
- **Core Functions & Signatures:**
  - `calculate_iso10077_uw(geom: LatticeGeometryResult, preset: GlazingPreset) -> ThermalMetrics`
  - `compute_infiltration_comparison(uw: float, window_area: float, room_vol: float, delta_t: float, ach50_values: list[float]) -> pd.DataFrame`
  - `compute_pattern_pareto_front(pattern: PatternType, window: WindowGeom, preset: GlazingPreset, shading: AnnualSolarResult) -> ParetoFrontResult`

### 3.4 `src/parametric_hanok/exporter.py` (Laser/CNC-Ready SVG & Multi-Layer DXF)

Translates verified Shapely geometries into fabrication-ready CAD files without coordinate distortion.

- **True-Millimeter SVG (`drawsvg`):** Writes outer frame boundaries and glazing hole cut paths directly from Shapely polygons with explicit `viewBox` and physical `width`/`height` in `mm` (avoiding `ezdxf`'s `SVGBackend` integer coordinate compression).
- **Multi-Layer DXF (`ezdxf`):** Produces DXF R2018 drawings with strict layer separation:
  - `FRAME` (outer *Ulgeomi* boundary)
  - `LATTICE_CUT` (boolean-unioned wood cut outlines for CNC/laser)
  - `CENTERLINE` (deduplicated joinery reference axes)
- **Phase 2 Optional Extension:** 3D extruded `.obj`/`.stl` lattice & eaves framing mesh via `trimesh` (only implemented during the Dec 2–4 buffer after Dec 1 MVP freeze).
- **Core Functions & Signatures:**
  - `export_svg(geom: LatticeGeometryResult, filepath: str) -> str`
  - `export_dxf(geom: LatticeGeometryResult, filepath: str) -> None`

### 3.5 `app.py` & CLI Benchmark (Thin Streamlit Dashboard + Local Reproducibility)

- **Performance Discipline (Preventing Full-Rerun Freezing):**
  - Wrap pure domain functions (`get_annual_solar_table`, `build_lattice_geometry`, `compute_pattern_pareto_front`) in `@st.cache_data`.
  - Isolate the **Macro Eaves Panel** and **Micro Lattice/Pareto Panel** inside separate `@st.fragment` blocks so moving a lattice slider never recomputes annual solar integrals.
  - Place multi-pattern Pareto optimization behind an explicit `st.form` submit button ("Run Pareto Optimization") rather than firing on every slider drag.
- **Dashboard Layout (4 Synchronized Views):**
  1. **Eaves Cross-Section & Annual Solar Chart:** Compares 2-day solstice baseline (0.48 m) vs. annual VSA/3D optimal eaves (~1.05 m), with catenary/Bézier drainage curve overlay and 3D *Anggok*/*Anheorigok* conceptual reference card.
  2. **6-Pattern Korean Lattice SVG Viewer & CAD Download:** Real-time true-scale SVG preview, exact $A_f$ / $A_g$ / $l_g$ readout, and one-click `.svg` + `.dxf` downloads.
  3. **Multi-Pattern Pareto Front (Plotly):** Overlays Pareto curves across all 6 patterns under the selected glazing preset, highlighting how $l_g \cdot \psi$ separates pattern dominance regions.
  4. **Infiltration (`ACH50`) Sensitivity Chart:** Side-by-side bar comparison of conductive window heat loss vs. airtightness leakage loss.
- **Reproducibility & Cloud Sleep Mitigation:** Because Streamlit Community Cloud free apps sleep after 12 hours of inactivity, primary grading reproducibility is guaranteed via local one-command execution (`streamlit run app.py`, `python -m parametric_hanok benchmark`, `pytest`) plus quantitative benchmark tables and demo GIFs embedded in `README.md`.

---

## 4. Revised Daily Development Milestones (Nov 27 – Dec 6, 2026)

Structured around a **hard Dec 1 MVP Freeze** (100% functional, zero-penalty deliverable) followed by a **Dec 2–4 Enhancement Buffer** and **Dec 5–6 Clean-Environment Audit**.

```text
Phase 1: Core & MVP ──► Phase 2: Buffer ──► Phase 3: Audit
  (11/27 – 12/01)       (12/02 – 12/04)     (12/05 – 12/06)
```

| Date | Phase | Task Description | Deliverables | Verification Gate (`pytest` / Metric) |
| :--- | :--- | :--- | :--- | :--- |
| **11/27 (Fri)** | **Core Macro** | Init MIT repo & `pyproject.toml`; implement `sun_eaves.py` (`pvlib` 12-mo hourly VSA + 3D finite-width shading + continuous eaves optimizer). | `sun_eaves.py`, `tests/test_sun_eaves.py` | Seoul 37.57°N: Solstice baseline ≈ 0.48 m vs. annual optimal ≈ 0.90–1.20 m; zero division-by-zero at $\gamma = \pm 90^\circ$. |
| **11/28 (Sat)** | **Core Micro** | Implement 6 Korean wallpaper groups (`pmm`, `cmm`, `p4`, `p4m`, `p6m`, `p2`), edge deduplication, frame clipping, connectivity check, and `unary_union`. | `lattice_symmetry.py`, `tests/test_lattice.py` | All 6 patterns: 0 floating segments, 0 duplicate edges, $\|A_f + A_g - A_{\text{tot}}\| < 10^{-6}\text{ m}^2$. |
| **11/29 (Sun)** | **Physics & CAD** | Implement `thermal_pareto.py` (EN ISO 10077-1 $l_g\psi$, 3 glazing presets, `ACH50` sensitivity, eaves-coupled hybrid Pareto) & `exporter.py` (`drawsvg` SVG + `ezdxf` DXF). | `thermal_pareto.py`, `exporter.py`, unit tests | Same 35% wood area across 4 patterns shows distinct $U_w$ via $l_g\psi$; DXF re-parses cleanly via `ezdxf.readfile`. |
| **11/30 (Mon)** | **Thin UI & CLI** | Build `app.py` (`@st.cache_data`, `@st.fragment`, `st.form`) with 4 interactive Plotly/SVG views + `python -m parametric_hanok benchmark` CLI. | `app.py`, `cli.py`, integration tests | Slider response < 0.5 s; downloaded SVG & DXF open accurately in external viewers; 100% `pytest` pass. |
| **12/01 (Tue)** | **MVP Freeze** | **Freeze 1st complete submission-ready MVP**; author `README.md` with demo GIF, quantitative tables, implementation scope boundary, licenses, and references. | Frozen MVP tag (`v1.0-mvp`), `README.md` | **Submission-Ready Gate:** Zero crash, zero missing citation/license, reproducible via local one-command run. |
| **12/02–12/04 (Wed–Fri)** | **Buffer & Polish** | Add multi-latitude presets (Jeju 33.5°, Seoul 37.6°, Pyongyang 39.0°, Helsinki 60.2°), 3D *Anggok*/*Anheorigok* reference card, optional `trimesh` OBJ export, and typing/lint cleanup. | Extended comparison presets, polished code | All extended scenarios pass `pytest`; no regressions against `v1.0-mvp`. |
| **12/05–12/06 (Sat–Sun)** | **Final Audit & Submit** | Extract final `.zip` into a **brand-new clean `venv`**, run `pip install`, `pytest`, `benchmark` CLI, and `streamlit run app.py`; verify license & citations before Dec 6 23:59 deadline. | Final archive & Defense Deck | Clean `venv` 100% pass; zero risk of non-execution (−10) or incomplete (−5) penalties. |

---

## 5. Evaluation Defense, Implementation Scope & Citation Protocol

### 5.1 Technical Complexity Matrix

```text
                          COMPLEXITY MATRIX
   High │                                  [Annual VSA + 3D Finite-Width Shading]
        │                                  [ISO 10077-1 Hybrid Pareto Solver]
        │
DIFFICULTY│                 [6-Group Affine + Shapely Boolean Union]
        │                   [True-mm SVG & Multi-Layer DXF Pipeline]
   Low  │  [Thin Streamlit UI (@st.fragment)]
        └─────────────────────────────────────────────────────────────────
           Low                                                      High
                                   CREATIVITY
```

### 5.2 Key Technical Defense Arguments

1. **Closed-Loop Building Physics over Disconnected Toys:** Rather than presenting isolated solar calculators and decorative SVG generators, `sun_eaves.py` feeds hourly 3D shading fractions directly into `thermal_pareto.py`, while `lattice_symmetry.py` supplies exact boolean-unioned areas and thermal-bridge perimeters ($l_g$) that physically differentiate traditional patterns.
2. **Elimination of Common Mathematical & Solver Pitfalls:** Explicitly avoids 2D roof-curvature degeneracy ($\partial J / \partial \kappa \equiv 0$), 2-day solstice over-heating during Korea's August thermal lag, linear Bang-Bang extreme solutions, and `SLSQP` silent failures on discrete step-function tilings.
3. **Honest Modeling Boundaries (Infiltration & Thermal Inversion):** Explicitly accounts for thermal inversion in modern Low-E Shin-Hanok glazing ($U_g = 1.10 < U_f = 2.82$) and quantifies blower-door infiltration (`ACH50`) sensitivity rather than overclaiming conductive savings.
4. **Clean Permissive Stack & Zero License Contamination:** Every dependency is MIT, BSD-3-Clause, or Apache-2.0 (with explicit LGPL-2.1 dynamic-link attribution for Shapely's GEOS backend), cleanly excluding GPL/AGPL libraries (`pysolar`, `ladybug-*`, `create_symmetry`).

### 5.3 Implementation Scope Boundary

- **Directly Designed & Implemented Modules:**
  - `src/parametric_hanok/sun_eaves.py`: VSA + 3D finite-width side-spilling polygon shading integration and annual cooling/heating load optimizer.
  - `src/parametric_hanok/lattice_symmetry.py`: 6 Korean wallpaper-group affine generators, endpoint-quantized edge deduplication, connectivity validator, and boolean-union metric extractor.
  - `src/parametric_hanok/thermal_pareto.py`: EN ISO 10077-1 $l_g \cdot \psi$ transmittance calculator, 3-preset thermal-inversion model, `ACH50` sensitivity engine, and hybrid discrete–continuous Pareto solver.
  - `src/parametric_hanok/exporter.py`: True-mm `drawsvg` serializer and multi-layer `ezdxf` generator.
- **Delegated External Libraries:** Solar ephemeris & clear-sky irradiance (`pvlib`), 2D polygon boolean kernels (`Shapely`), numerical optimization primitives (`SciPy`), DXF/SVG serialization backends (`ezdxf`, `drawsvg`), visualization & UI (`Plotly`, `Streamlit`).

### 5.4 Prior-Art Differentiation & Academic References

- **Prior Open-Source Differentiation:**
  - *Lattice Generators:* `evoluteur/kumiko-maker` (MIT) and `majjacz/kumiko-designer` (Unlicense) target Japanese Kumiko joinery/CNC without building-physics optimization; **ChanssalOpt** formalizes Korean *Changsal* wallpaper groups and couples exact perimeter $l_g$ to EN ISO 10077-1 thermal transmittance.
  - *Solar / Hanok Tools:* `idfkit/idfkit-shoebox` (MIT) simulates generic rectangular boxes, and `midagedev/cheoma` focuses on procedural 3D village massing; **ChanssalOpt** unifies traditional Hanok eaves shading with window-lattice thermal/daylight Pareto optimization in a single Python workflow.
- **Standards & Academic References:**
  1. **EN ISO 10077-1:** Thermal performance of windows, doors and shutters — Calculation of thermal transmittance ($U_w$, $l_g\psi$, $R_{si} = 0.13$, $R_{se} = 0.04\text{ m}^2\cdot\text{K/W}$).
  2. **UK BRE BR 443:** Conventions for U-value calculations (softwood thermal conductivity $\lambda = 0.13\text{ W/(m}\cdot\text{K)}$).
  3. **MDPI *Buildings* (2024):** Quantitative survey of ~50 traditional Hanok eaves projections (avg. 2,577 mm; inner angle avg. 28°; roof pitch 25°–35°, avg. 29°).
  4. **AURI (건축공간연구원):** Modern Shin-Hanok model eaves projection guidelines ($\le 1,200\text{ mm}$).
  5. **MDPI *Sustainability* (2018) & *BioResources* (2018):** Traditional Hanok window thermal transmittance ($U = 4.177\text{–}5.879\text{ W/(m}^2\cdot\text{K)}$, $\text{SHGC} = 0.29$) and architectural code benchmarks.
  6. **KIEAE Journal:** Blower-door airtightness measurements of 16 Shin-Hanok houses / 26 rooms ($\text{ACH50} = 8.42\text{–}78.38\text{ h}^{-1}$, 81% ASHRAE Class I–J).
  7. **안은영·김재원 (2012):** CFD analysis of convective heat loss vs. Hanok window lattice aspect ratio and frame depth.
  8. **이순지·김유신·최안섭 (2008):** Optical daylight diffusion and glare-reduction evaluation of traditional Hanji windows.

---

## 6. Licensing & Third-Party Notices

**ChanssalOpt** is distributed under the **MIT License** ([`LICENSE`](./LICENSE)).

This project relies on external open-source packages installed via `pip` / `pyproject.toml` (not vendored in source). In accordance with **BSD-3-Clause**, **Apache-2.0**, and **LGPL-2.1** license terms:

- **BSD-3-Clause Dependencies** (`pvlib`, `Shapely`, `NumPy`, `SciPy`):
  - Copyright notices, license conditions, and warranty disclaimers of each upstream project are preserved in their respective package distributions (`dist-info/METADATA` / `LICENSE`).
  - Per the BSD-3-Clause non-endorsement clause, the names of upstream copyright holders and contributors (`pvlib`, `Shapely`, `NumPy`, `SciPy`) are **not** used to endorse or promote products derived from this software without specific prior written permission.
- **LGPL-2.1 Dynamic-Link Notice (`GEOS` via `Shapely`):**
  - While `Shapely` itself is licensed under BSD-3-Clause, its binary wheels dynamically link against **GEOS** (Geometry Engine - Open Source), which is licensed under **LGPL-2.1**. Users redistributing bundled executables must preserve GEOS LGPL-2.1 notices and allow relinking against modified GEOS shared libraries.
- **Apache-2.0 Dependency (`Streamlit`):**
  - Distributed under Apache License 2.0; upstream copyright and `NOTICE` attributions are retained in packaged distributions.
- **MIT Dependencies** (`drawsvg`, `ezdxf`, `Plotly`, `pytest`, optional `trimesh`):
  - Permissive MIT License; fully compatible with MIT distribution.
