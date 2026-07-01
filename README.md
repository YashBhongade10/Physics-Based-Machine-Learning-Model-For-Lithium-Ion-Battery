# Physics-Based Machine Learning Model for a Lithium-Ion Battery

This project predicts how a lithium-ion battery **degrades over time**, how its capacity fades, how much lithium it loses, and how much active material it loses, using a mix of **physics simulation** and **machine learning**.

The idea in one line: build a physics model of the battery, calibrate it against real test data, and then use a small machine-learning correction layer to fix whatever the physics model still gets wrong.

---

## Why this project exists

Batteries degrade differently depending on temperature. A cell aged at 10°C degrades differently from one aged at 25°C or 40°C. Running real experiments at every possible temperature is expensive and slow, so this project asks:

> If we test a battery at only two temperatures, can we accurately predict how it will behave at a third, untested temperature?

The answer here is yes, by combining a physics-based battery model with a small, smart machine-learning correction.

---

## Dataset

Real experimental data from the **Kirkaldy et al. (2024) Experiment 5** dataset, using **LG INR21700-M50T** cylindrical cells (NMC811 cathode, graphite-silicon anode). Cells were aged at three temperatures:

- **10°C** (cold)
- **25°C** (room temperature)
- **40°C** (hot)

Each cell was tested in cycles, with a full performance check-up (called an RPT) every so often to measure its capacity and health.

---

## How the project works

**Step 1 — Simulate the battery with physics.**
A PyBaMM battery model (a well-known open-source battery simulator) is set up and calibrated to match the real 25°C and 40°C cells. This gives a physics-based prediction of how the battery ages.

**Step 2 — Check where the physics model is wrong.**
The simulated results are compared against the real measured results. The physics model is good, but not perfect, there's always a small gap between what it predicts and what actually happened.

**Step 3 — Learn that gap with a Gaussian Process.**
A Gaussian Process (a type of machine-learning model that works well with small amounts of data) is trained to learn this gap, using the two temperatures we have real data for (25°C and 40°C).

**Step 4 — Predict the untested temperature.**
The learned gap is used to predict what the physics model is *missing* at 10°C, a temperature the model was never calibrated on. Add that correction back to the physics prediction, and you get an accurate estimate of how the 10°C cell behaves, even though it was never used to train the correction.

**Step 5 — Check the answer.**
Since real 10°C data does exist, it's used only at the very end, to check how good the prediction was, not to help make the prediction. This keeps the test fair and honest.

---

## What the model predicts

Four measures of battery health are tracked:

| Quantity | Plain meaning |
|---|---|
| **SoH** (State of Health) | How much capacity is left, compared to when the battery was new |
| **LLI** (Loss of Lithium Inventory) | How much usable lithium has been lost |
| **LAM (negative electrode)** | How much of the anode material has stopped working |
| **LAM (positive electrode)** | How much of the cathode material has stopped working |

---

## Repository contents

| File | What it does |
|---|---|
| `Pybamm_simulation_for_Data_Generation.ipynb` | Builds and runs the PyBaMM physics model, calibrated to the real battery cells, and generates the simulated ageing data |
| `Exp_kirkaldy_degradation_analysis.ipynb` | Loads and analyses the real experimental data from the Kirkaldy dataset |
| `Sim_vs_Exp_plot_comparison_with_calibrated_cell.ipynb` | Compares the physics simulation against the real experimental results, with error charts and accuracy plots |
| `GP_Residual_learning.ipynb` | Trains the Gaussian Process correction model and predicts the untested (10°C) cell |
| `Calculation for Predicting_Cell_A (Explanation of GP)` | A step-by-step, hand-worked explanation of exactly how the Gaussian Process prediction is calculated, with real numbers |

---

## Tools used

- **PyBaMM** — open-source battery simulation library (physics model)
- **scikit-learn** — for the Gaussian Process machine-learning model
- **pandas / numpy** — data handling
- **matplotlib** — plots and figures

---

## Results

The physics model alone gets battery ageing mostly right but is noticeably off at temperatures it wasn't tuned for. Adding the small machine-learning correction layer significantly closes that gap, cutting the prediction error by roughly 3-4 times, while still keeping the results grounded in real battery physics rather than being a "black box" guess.

---

