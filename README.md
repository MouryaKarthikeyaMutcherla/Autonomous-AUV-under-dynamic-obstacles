# AUV Simulation Projects
### Underwater navigation, path planning, predictive control, and moving-obstacle avoidance

Welcome! This repository brings together two related AUV simulation versions:

1. **Demo / extended-dynamics version** — the original moving-obstacle simulation with an extended six-degree-of-freedom (6-DOF) dynamics model.
2. **AURORA / multi-controller version** — a reduced eight-state model with selectable controller modes and adjustable simulation conditions.

Both versions explore the same core idea: an Autonomous Underwater Vehicle (AUV) must move through a three-dimensional underwater environment, follow a reference route, and respond to obstacles and simulated disturbances. They are separate implementations, so their models, controls, run settings, and results should not be treated as interchangeable.

> **Important:** These are simulation projects, not tests on a physical AUV. Their results describe the behaviour of the implemented models. Real-world accuracy, formal stability, and guaranteed collision avoidance have not been established by the supplied simulations.

## Which version should I use?

| Version | Folder / ZIP | Model and control focus |
|---|---|---|
| **Demo / extended dynamics** | `AUV_PROJECT_ROBOTICS.zip` | Extended 12-state model including position, orientation, linear velocities, and angular rates; predictive control and moving-obstacle replanning. |
| **AURORA / multi-controller** | `AUV_Virtualistic_MultiController_Simulator.zip` | Reduced 8-state model; four selectable controller modes, adjustable environment conditions, and controller comparisons. |

Keep the two versions in separate folders or repositories. Each has its own code and assumptions.

---

## 1. Demo / extended-dynamics version

This version focuses on a more complete set of simulated vehicle motion variables and the interaction between predictive control and moving-obstacle avoidance.

### Main features

- VTK-based 3D underwater environment.
- Extended 12-state, six-degree-of-freedom dynamics model.
- Three-dimensional route planning around static obstacles.
- Prediction of moving-obstacle positions and online route replanning.
- Finite-horizon predictive control.
- Tube-style feedback processing and a predictive safety-shield stage.
- Generalized force/moment magnitude limits and rate-of-change limits.
- Fourth-order Runge–Kutta (RK4) integration.
- Mission logs and plots for state response, control inputs, tracking, and clearance.

### Model and assumptions

The extended state includes position, roll/pitch/yaw, body-frame linear velocities, and angular rates. The model includes damping, motion coupling, simulated current and bounded disturbances, and a difference between the nominal model and the simulated plant.

The directly commanded inputs remain:

- Surge (longitudinal) force — N
- Heave (vertical) force — N
- Yaw moment — N·m

Roll and pitch motion are modelled passively; there are no independent roll or pitch control inputs. Hydrodynamic and rotational parameters are assumed rather than identified from a specific physical AUV. Treat this as an extended simulation model, not a validated digital twin.

### Run on Windows

1. Extract `AUV_PROJECT_ROBOTICS.zip`.
2. Open the project's `upgraded` folder.
3. Install dependencies:

```powershell
py -m pip install -r requirements.txt
```

4. Start the simulator:

```powershell
py main.py
```

Or use `INSTALL_AUV_SIM.bat` once and then `RUN_AUV_SIM.bat`.

To run without the 3D window:

```powershell
py main.py --headless --seconds 150
```

Regenerate plots after a run:

```powershell
py plot_results.py
```

### Keyboard controls

| Key | Action |
|---|---|
| `SPACE` | Pause / resume |
| `R` | Restart mission |
| `1` | Overview camera |
| `2` | Follow-AUV camera |
| `3` | Free / trackball camera |
| `4` | Top camera |
| `S` | Save screenshot |
| `ESC` | Exit |

---

## 2. AURORA / multi-controller version

This version emphasizes controller choice and testing the same simulation under different configured conditions.

### Main features

- VTK-based underwater scene with static scenery and moving objects.
- Reduced eight-state 3D AUV dynamics model.
- Three-dimensional path planning and obstacle-aware replanning.
- Four selectable controller modes.
- Adjustable environment presets, current and disturbance levels.
- Optional noise on the state estimate supplied to the controller.
- Multiple camera views and live mission telemetry.
- CSV logging and controller-comparison output.

### Available controllers

| Mode | Description |
|---|---|
| **NMPC-style** (`nmpc`) | Evaluates a small set of candidate actions over a finite prediction horizon. This is a lightweight predictive search, not a general-purpose nonlinear optimizer. |
| **PID** (`pid`) | Conventional feedback-control baseline. |
| **LOS-PID** (`los-pid`) | Line-of-sight route guidance with PID-style control loops. |
| **Robust-PD** (`robust-pd`) | Damped PD-style control with a bounded disturbance-rejection term. It is a practical heuristic, not a formal robust-stability guarantee. |

All modes use the project's shared safety processing and input constraints. The safety logic is a simulation heuristic, not a formally verified collision-avoidance guarantee.

### Eight-state model

The state is:

```text
[X, Y, Z, psi, u, v, w, r]
```

It represents world position, yaw, body surge/sway/heave velocities, and yaw rate. It includes nonlinear damping and coupling, simulated current, bounded disturbances, nominal/plant parameter mismatch, and RK4 integration.

The directly commanded inputs are longitudinal (surge) force (N), vertical (heave) force (N), and yaw moment (N·m).

**This version is not a full 6-DOF model.** It does not include roll, pitch, or their angular-rate states, and it does not independently command lateral force or roll/pitch moments.

### Environment and noise options

- `calm` — lower current/disturbance scale and slower moving objects.
- `nominal` — configured baseline.
- `rough` — higher current/disturbance and faster moving objects.
- Optional position and heading noise on the controller-facing state estimate.
- Option to disable predictive awareness of moving obstacles; the objects remain in the scene.

The noise option does not create a complete noisy-perception model: the internal plant state, map, route planner, and clearance calculations still use simulator state.

### AURORA reference data

The loader can use the supplied AURORA-related navigation text as a contextual reference path. The trace is smoothed and mapped into the local scene envelope; it is not a metrically exact geographic reconstruction or ground-truth AUV dynamics. The supplied navigation text does not contain depth, so a fixed nominal depth is used for the reference display.

Check the mission summary to confirm which reference source was used for a run.

### Run on Windows

1. Extract the multi-controller ZIP.
2. Open the `AUV_VIRTUALISTIC_UPGRADE` folder.
3. Install dependencies:

```powershell
py -m pip install -r requirements.txt
```

4. Start the simulator:

```powershell
py main.py
```

Or run `INSTALL_AUV_SIM.bat` once and then `RUN_AUV_SIM.bat`.

### Keyboard controls

| Key | Action |
|---|---|
| `SPACE` | Pause / resume |
| `C` | Cycle controllers |
| `5` | NMPC-style |
| `6` | PID |
| `7` | LOS-PID |
| `8` | Robust-PD |
| `O` | Toggle moving-obstacle predictive awareness |
| `[` / `]` | Decrease / increase current and disturbance |
| `N` | Cycle controller-state noise presets |
| `1` | Overview camera |
| `2` | Follow-AUV camera |
| `3` | Free camera |
| `4` | Top camera |
| `S` | Save screenshot |
| `R` | Restart mission |
| `ESC` | Exit |

### Useful commands

```powershell
# Start the interactive simulator
py main.py

# Select a controller and scenario
py main.py --controller pid --scenario calm
py main.py --controller los-pid --scenario rough
py main.py --controller robust-pd --scenario nominal

# Run a short headless test
py main.py --headless --seconds 30 --controller nmpc

# Add noise to the controller-facing state estimate
py main.py --headless --seconds 30 --controller pid --position-noise 0.05 --heading-noise-deg 0.5

# Compare all four controllers
py main.py --headless --compare-controllers --seconds 24
```

On Windows, `RUN_CONTROLLER_COMPARISON.bat` starts the controller-comparison workflow.

---

## Results and generated files

Depending on the version and commands used, the `results/` folder may contain:

- `mission_log.csv` — recorded state, control inputs, tracking and clearance information.
- `mission_summary.txt` — settings and summary for a run.
- State and acceleration plots.
- Control input plots, including force/moment channels.
- Tracking-error, goal-distance, and obstacle-clearance plots.
- Controller comparison CSV files, per-controller logs, and a comparison chart.

The force/moment signals and acceleration signals are different quantities. Use N for force, N·m for moment, m/s² for linear acceleration, and rad/s² for angular acceleration.

To regenerate plots, run the plotting script provided in the version's project folder:

```powershell
py plot_results.py
```

## How to interpret results

A simulation run shows how the implemented model behaved under one set of settings. It does not establish physical AUV accuracy or guarantee collision avoidance. Clearance values are sampled simulation metrics, and a single run is not enough to rank controllers statistically.

For stronger evaluation:

1. Repeat each controller under the same initial conditions.
2. Vary obstacle trajectories, current, and disturbances.
3. Record the exact controller and scenario settings.
4. Compare goal error, tracking error, clearance, control effort, and collision samples across repeated runs.
5. Validate model parameters and navigation references against trusted data before making real-world claims.

## Suggested repository structure

If you keep both projects in one GitHub repository, a clear layout is:

```text
AUV-Simulation-Projects/
├── README.md
├── demo_extended_dynamics/
│   └── upgraded/
│       ├── main.py
│       ├── config.py
│       ├── requirements.txt
│       └── auv_sim/
└── aurora_multi_controller/
    └── AUV_VIRTUALISTIC_UPGRADE/
        ├── main.py
        ├── config.py
        ├── requirements.txt
        └── auv_sim/
```

If the projects are in separate repositories, use this README as the combined overview or copy the relevant section into each repository's `README.md`. Keep each version's source files and required data/configuration with the version that uses them.

## Final note

These projects are intended to make underwater navigation and control easier to explore. Run the models, change the settings, inspect the logs, and compare repeatable scenarios. Be clear about which version produced a result, and treat the outputs as simulation evidence—not experimental proof.
