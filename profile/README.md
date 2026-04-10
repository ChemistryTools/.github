<div align="center">
  <img src="https://raw.githubusercontent.com/ChemistryTools/ChemistryLab.jl/main/docs/src/assets/logo.png" width="220" alt="ChemistryTools logo"/>

  # ChemistryTools

  _Julia ecosystem for computational chemistry and thermodynamic equilibrium_

  _Cement hydration · Aqueous geochemistry · Low-carbon materials_
</div>

---

ChemistryTools is an open-source Julia ecosystem designed for researchers, engineers, and developers working on thermodynamic equilibrium calculations in chemistry. Built around Gibbs energy minimisation, the packages handle everything from chemical formula parsing and database interoperability (ThermoFun / Cemdata18) to full speciation and multi-phase equilibrium. The stack is ForwardDiff-compatible, SciML-native, and designed for integration with broader scientific computing workflows in Julia.

---

## Packages

### [ChemistryLab.jl](https://github.com/ChemistryTools/ChemistryLab.jl) — Core toolkit

[![Build Status](https://github.com/ChemistryTools/ChemistryLab.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/ChemistryTools/ChemistryLab.jl/actions/workflows/CI.yml?query=branch%3Amain)
[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://ChemistryTools.github.io/ChemistryLab.jl/stable/)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://ChemistryTools.github.io/ChemistryLab.jl/dev/)
[![DOI](https://zenodo.org/badge/1054296488.svg)](https://doi.org/10.5281/zenodo.17756074)

ChemistryLab.jl is the central package of the ecosystem. It provides chemical formula handling, species management, stoichiometric matrix construction, and database interoperability (ThermoFun and Cemdata18). Its equilibrium engine drives Gibbs energy minimisation through a composable activity model interface and supports both aqueous speciation and multi-phase solid solution assemblages.

| Feature | Details |
|---|---|
| **Chemical formula handling** | Parsing, molar mass, charge, Unicode / Phreeqc notation |
| **Database interoperability** | Import ThermoFun (.json) and Cemdata (.dat); merge datasets |
| **Activity models** | `DiluteSolutionModel`, `HKFActivityModel` (B-dot), `DaviesActivityModel` |
| **Solid solutions** | Ideal (`IdealSolidSolutionModel`) and binary non-ideal (`RedlichKisterModel`) |
| **Chemical equilibrium** | `ChemicalSystem` / `ChemicalState` / `equilibrate` (Gibbs minimisation) |
| **Automatic differentiation** | Full ForwardDiff compatibility throughout |

```julia
julia> import Pkg; Pkg.add("ChemistryLab")
```

[Full documentation and tutorials →](https://ChemistryTools.github.io/ChemistryLab.jl/stable/)

---

### [OptimaSolver.jl](https://github.com/ChemistryTools/OptimaSolver.jl) — Equilibrium solver

[![Build Status](https://github.com/ChemistryTools/OptimaSolver.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/ChemistryTools/OptimaSolver.jl/actions/workflows/CI.yml?query=branch%3Amain)
[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://ChemistryTools.github.io/OptimaSolver.jl/stable/)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://ChemistryTools.github.io/OptimaSolver.jl/dev/)

OptimaSolver.jl is a Julia-native primal-dual interior-point solver for Gibbs energy minimisation in equilibrium chemistry. It is a Julia port of the [Optima](https://github.com/reaktoro/optima) C++ library developed by Allan Leal (ETH Zürich), adapted and extended for seamless SciML integration. `OptimaOptimizer` is a drop-in replacement for `IpoptOptimizer` in ChemistryLab.jl.

| Feature | Details |
|---|---|
| **Algorithm** | Log-barrier interior-point method (primal-dual) |
| **Schur-complement Newton step** | KKT system reduced from $(n_s+m)\times(n_s+m)$ to $m\times m$ |
| **Warm-start** | Consecutive solves reuse previous solution as starting point |
| **Implicit-differentiation sensitivity** | $\partial n^{\ast}/\partial b$ and $\partial n^{\ast}/\partial(\mu^0/RT)$ at marginal cost |
| **Automatic differentiation** | Generic Julia arithmetic — no `Float64` casts |
| **SciML / ChemistryLab integration** | `OptimaOptimizer` implements `SciMLBase.AbstractOptimizationAlgorithm` |

```julia
julia> import Pkg; Pkg.add("OptimaSolver")
```

---

## Quick Start

Calcite–water equilibrium using the Cemdata18 database:

```julia
using ChemistryLab

# Load species from the bundled Cemdata18/ThermoFun database
all_species = build_species("data/cemdata18-thermofun.json")

# Select species relevant to calcite dissolution in water
species_calcite = speciation(all_species, split("Cal H2O@ CO2");
                             aggregate_state = [AS_AQUEOUS],
                             exclude_species = split("H2@ O2@ CH4@"))
dict_calcite = Dict(symbol(s) => s for s in species_calcite)

# Build the chemical system and define an initial state
cs     = ChemicalSystem(collect(values(dict_calcite)))
state0 = ChemicalState(cs; T = 298.15u"K", P = 1u"bar")
set_quantity!(state0, "H2O@", 55.5u"mol")
set_quantity!(state0, "Cal",  1e-3u"mol")
set_quantity!(state0, "H+",   1e-7u"mol")
set_quantity!(state0, "OH-",  1e-7u"mol")

# Compute thermodynamic equilibrium (Gibbs energy minimisation)
state_eq = equilibrate(state0)

# Read results
pH(state_eq)              # equilibrium pH  (~8.3)
moles(state_eq, "Ca+2")   # dissolved calcium
```

Use OptimaSolver.jl as a faster, pure-Julia alternative to Ipopt:

```julia
using OptimaSolver
state_eq = equilibrate(state0; solver = OptimaOptimizer(tol = 1e-10))
```

---

## Authors & Institution

Developed by **Jean-François Barthélémy** and **Anthony Soive**, researchers at [Cerema](https://www.cerema.fr/en) in the [UMR MCD](https://mcd.univ-gustave-eiffel.fr/) research unit.

---

## License

MIT — see [ChemistryLab.jl/LICENSE](https://github.com/ChemistryTools/ChemistryLab.jl/blob/main/LICENSE) · [OptimaSolver.jl/LICENSE](https://github.com/ChemistryTools/OptimaSolver.jl/blob/main/LICENSE)
