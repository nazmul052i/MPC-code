# CLAUDE.md - AI Assistant Guide for MPC-code

## Project Overview

This is a **Model Predictive Control (MPC) framework** written in Python, developed at the University of Pisa (CPCLAB-UNIPI). It provides a flexible, modular implementation of various MPC strategies including Linear MPC (LMPC), Nonlinear MPC (NMPC), and Economic MPC (ENMPC), with multiple state estimation methods.

**License:** GNU LGPL v3
**Authors:** Marco Vaccari, Mirco, Gabriele

## Repository Structure

All source files are in the root directory (no subdirectories for code):

```
MPC-code/
├── MPC_code.py          # Main entry point - orchestrates the full MPC control loop
├── Estimator.py         # State estimators (Kalman, EKF, MHE, Luenberger)
├── Utilities.py         # Utility functions (plant/model defs, integration, plotting)
├── Control_Calc.py      # Dynamic optimization problem builder
├── Target_Calc.py       # Steady-state target optimization problem builder
├── Default_Values.py    # Default configuration flags and parameters
├── SS_JAC_ID.py         # Steady-state Jacobian identification / linearization
├── Ex_LMPC_WB.py        # Example: Linear MPC with white noise
├── Ex_LMPC_CSTR.py      # Example: Linear MPC for CSTR reactor
├── Ex_LMPC_nlplant.py   # Example: Linear MPC with nonlinear plant
├── Ex_LMPCxp_nlplant.py # Example: Linear MPC with noise, nonlinear plant
├── Ex_NMPC.py           # Example: Nonlinear MPC (continuous-time CSTR)
├── Ex_NMPC_dis.py       # Example: Nonlinear MPC (discrete-time)
├── Ex_ENMPC.py          # Example: Economic NMPC
├── User_Guide.pdf       # Comprehensive user documentation
├── LICENSE              # GNU LGPL v3
└── .gitignore
```

### Core Modules

| Module | Purpose |
|--------|---------|
| `MPC_code.py` | Main loop: loads example, sets up symbolic variables, runs simulation |
| `Estimator.py` | State estimation: Kalman filter, EKF, steady-state Kalman, MHE, Luenberger |
| `Utilities.py` | Plant/model definitions, RK4 integration, system matrix conversion, plotting |
| `Control_Calc.py` | Builds the dynamic optimization (control) NLP problem |
| `Target_Calc.py` | Builds the steady-state target optimization problem |
| `Default_Values.py` | Boolean flags and default values for all configurable options |
| `SS_JAC_ID.py` | Linearization via steady-state Jacobian identification |

### Example Files (`Ex_*.py`)

Example files serve as **configuration modules** loaded by `MPC_code.py`. Each follows a consistent structure:
1. Simulation parameters (`Nsim`, `N`, `h`)
2. Symbolic variable declarations (`xp`, `x`, `u`, `y`, `d`)
3. Process parameters and dynamics functions
4. Model parameters and dynamics functions
5. State estimation configuration
6. Steady-state optimization setup
7. Dynamic optimization setup

## Dependencies

- **CasADi** - Symbolic optimization framework (core dependency)
- **NumPy** - Matrix operations
- **SciPy** - Linear algebra (`scipy.linalg`), optimization (`scipy.optimize`), ODE integration
- **Matplotlib** - Visualization and plotting
No `requirements.txt` exists. Install manually:
```bash
pip install casadi numpy scipy matplotlib
```

## How to Run

The main entry point is `MPC_code.py`. To select which example to run, edit line 25:

```python
ex_name = __import__('Ex_LMPC_WB')  # Change string to desired example file name
```

Then execute:
```bash
python MPC_code.py
```

This runs the full MPC simulation loop and generates plots.

## Python Version

**Python 3 only.** All Python 2 compatibility code (`__future__`, `builtins`, `past.utils`) has been removed.

## Key Conventions

### Naming

- **Variables follow control theory notation:**
  - `x` / `xp` - model/process state vectors
  - `u` - control input vector
  - `y` - measured output vector
  - `d` - disturbance vector
  - `N` - prediction horizon
  - `h` - time step
  - `Nsim` - simulation length
  - `nx`, `nu`, `ny`, `nd` - dimensions of respective vectors
- **Functions:**
  - `User_fxp_Cont` / `User_fxm_Cont` - user-defined continuous process/model dynamics
  - `User_fxp_Dis` / `User_fxm_Dis` - user-defined discrete process/model dynamics
  - `User_hyp` / `User_hym` - user-defined process/model output functions
  - `F` prefix for CasADi function objects (e.g., `Fx_model`, `Fy_model`)
- **Files:**
  - `Ex_*` prefix for example/configuration files
  - Core modules use descriptive CamelCase names

### Coding Style

- 4-space indentation
- CasADi `SX.sym()` for symbolic variable declarations
- Matrix operations via CasADi (`vertcat`, `horzcat`, `mtimes`) and NumPy
- IPOPT solver for optimization problems
- Docstrings use SUMMARY / SYNTAX / ARGUMENTS / OUTPUTS format

### Configuration Pattern

Configuration is done through boolean flags (mostly in `Default_Values.py` and overridden in example files):
- `estimating` - estimation-only mode
- `StateFeedback` - full state measurement available
- `Fp_nominal` - nominal plant = model
- `offree` - offset-free disturbance model (`'no'`, `'lin'`, `'nl'`)
- `QForm` / `QForm_ss` - quadratic form objectives
- `DUForm` - delta-input formulation
- `ContForm` - continuous objective integration
- `TermCons` - terminal constraint enabled
- Estimator selection: `kal`, `ekf`, `kalss`, `lue`, `mhe` (booleans)

## Testing and CI/CD

**No automated tests or CI/CD pipelines exist.** Validation is done by running the example files and inspecting plots/output. There is no test suite, linter configuration, or pre-commit hooks.

To manually verify behavior, run each example:
```bash
# Edit MPC_code.py line 25 to select example, then:
python MPC_code.py
```

## Architecture Notes

1. **MPC_code.py** is the orchestrator - it imports an example module dynamically, extracts dimensions, builds CasADi optimization problems, runs the simulation loop, and plots results.
2. **Example files** act as declarative configuration - they define symbolic variables, system matrices, dynamics functions, tuning parameters, and solver settings. They are not run directly.
3. **Estimator.py** implements a unified estimation interface supporting multiple algorithms selected via boolean flags.
4. **The solver** uses CasADi's `nlpsol` with IPOPT backend for both steady-state target and dynamic control problems.
5. **No package structure** - all modules are imported directly from the root directory using star imports (`from Module import *`).

## Common Modification Patterns

### Adding a New Example

1. Copy an existing `Ex_*.py` file as a template
2. Define symbolic variables (`xp`, `x`, `u`, `y`, `d`) with appropriate dimensions
3. Define process and model dynamics functions
4. Set estimation, target calculation, and control parameters
5. Override any `Default_Values.py` flags as needed
6. Update `MPC_code.py` line 25 to import the new example

### Changing Estimator

Set exactly one estimator boolean to `True` in the example file and provide its tuning parameters:
- `kal = True` for Kalman filter (requires `Q_kal`, `R_kal`)
- `ekf = True` for Extended Kalman filter
- `kalss = True` for steady-state Kalman filter
- `lue = True` for Luenberger observer
- `mhe = True` for Moving Horizon Estimation

### Changing Control Strategy

- **Linear MPC:** Use linear system matrices (`A`, `B`, `C`) in example
- **Nonlinear MPC:** Define nonlinear `User_fxp_Cont`/`User_fxm_Cont` functions using CasADi symbolics
- **Economic MPC:** Define custom objective via `Jstage` (stage cost function)
- **Discrete-time:** Use `User_fxp_Dis`/`User_fxm_Dis` instead of continuous dynamics
