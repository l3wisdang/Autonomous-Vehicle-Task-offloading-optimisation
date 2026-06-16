# Autonomous Vehicle Task Offloading: A Prescriptive Analytics Approach

![Hero](images/hero.png)

**8.2% lower daily cost than the industry-standard heuristic, using the same hardware**
Built with Python · Gurobi · pandas · NumPy · matplotlib

📄 [Read the full report (PDF)](AV_Task_Offloading_Report.pdf)

---

## 📌 Key Results

- **The integer linear program cut daily processing cost from SGD 2.2575 to SGD 2.0732 (8.2%)** versus the standard Greedy-Local rule, without buying any extra hardware. The saving comes entirely from smarter offloading decisions, not a bigger budget.
- **In the latency model, the ILP achieved a total latency of 12,748 versus 24,522 for Greedy-Local and 74,745 for Greedy-Cloud**, roughly halving the best heuristic while still respecting every safety constraint.
- **The optimiser beat the heuristic in all seven tested scenarios**, with savings of 1.3% to 8.5% across priority and traffic shifts, showing the advantage is robust to changing workload composition rather than overfitted to one case.
- **At fleet scale this compounds to roughly SGD 675,000 over ten years for 1,000 vehicles**, achieved through software logic alone with no change to the vehicle's physical architecture.

---

## 🧠 Why This Project

Autonomous vehicles constantly run tasks that compete for limited onboard compute: pedestrian detection has to happen in milliseconds, while scene understanding can wait. Each task can either run locally (fast, but the hardware is expensive) or be offloaded to the cloud (cheap hardware, but network latency and pay-per-use cost). Most systems decide this with fixed rules. I wanted to test whether treating it as a formal optimisation problem actually does better, and by how much.

What drew me to it was that it is a clean, real example of the gap between a heuristic and a true optimiser. A greedy rule makes each decision in isolation; an integer linear program looks at all 800 tasks at once and finds the assignment that minimises total cost subject to the constraints. Seeing the ILP consistently outperform the heuristic, especially when resources were tight, was a concrete demonstration of why prescriptive analytics is worth the extra modelling effort. The same way of thinking, formulate the objective, define the constraints, let the solver find the global optimum, is exactly what underpins portfolio optimisation and hedging problems I care about in finance.

---

## 📦 Technologies

| Tool | Purpose |
|---|---|
| Python | Core language |
| Gurobi (gurobipy) | Integer linear programming solver |
| pandas / NumPy | Data preparation and the VEINS telemetry dataset |
| matplotlib | Sensitivity charts and result visualisation |

---

## ⚙️ Method in Brief

The data is a vehicular simulation dataset (VEINS OMNeT++), filtered to a single vehicle and stratified-sampled down to 800 representative tasks as a one-day workload. Each task carries a size, available local CPU, network latency, signal strength, priority and weather condition. A binary `force_local` safety flag is set whenever the signal is weak or weather is severe, which applied to about 11% of tasks.

I built two integer linear programs, both solved with Gurobi in under a second. The first minimises total latency by choosing, for each task, local or cloud execution subject to a safety constraint and a local CPU capacity limit. The second is the headline model: it minimises real money cost, splitting hardware capital expense (a one-time cost amortised per day) from cloud operating expense (pay-per-task), and adds a latency service-level-agreement constraint with a small violation budget. Both are benchmarked against Greedy-Local and Greedy-Cloud heuristics, and a buffer sensitivity analysis traces the capex-versus-opex trade-off as onboard hardware power changes.

The full formulation, assumptions, scenario tables, feasibility analysis and limitations are in the [report](AV_Task_Offloading_Report.pdf).

![Latency model](images/latency_model.png)

---

## 📚 What I Learned

**Global optimisation beats local rules precisely when resources are scarce.** The gap between the ILP and the greedy baseline was widest at low CPU buffers, the exact high-load conditions a real vehicle faces most often. When capacity was generous the two converged, because there is little to optimise. The model adds the most value where it matters most.

**Splitting capex from opex changes the whole problem.** Once onboard hardware is treated as a sunk one-time cost with near-zero marginal cost, the solver is really just minimising cloud spend under constraints. Framing the cost structure correctly was as important as the optimisation itself.

**A constraint is only as good as the data that exercises it.** The safety rule was correctly formulated but triggered almost entirely by weak signal; severe weather never appeared in the sample. It taught me to be honest about what a model has and has not actually been stress-tested against, which is now a section in the report rather than something glossed over.

---

## 💬 How Can It Be Improved?

- **Rolling-horizon scheduling.** The current model is a static single-period batch over 800 tasks. Real tasks arrive sequentially, so a production version should re-optimise over short look-ahead windows.
- **Priority-weighted objective.** The dataset has a task priority field that does not yet enter the objective. Weighting high-priority tasks would let the optimiser explicitly protect safety-critical work.
- **Compound stress testing.** Validate the safety constraint under simultaneous poor signal, bad weather and low battery, the worst-case conditions currently under-represented in the data.
- **Broader sensitivity scope.** Vary network latency, battery thresholds and signal cutoffs, not just the CPU buffer, to build a fuller robustness profile.

---

## 🔌 Running the Project

```bash
pip install gurobipy pandas numpy matplotlib
jupyter notebook av_task_offloading_ilp.ipynb
```

Gurobi requires a licence; a free academic licence covers a problem of this size. The notebook runs end to end: data prep, both ILP formulations, the heuristic baselines, scenario comparison and the sensitivity charts.

---

## 🙏 Acknowledgments

- Group project for BC2411 Prescriptive Analytics, Nanyang Technological University. Team members: Lee Pei Wen, Pahwa Ronak, Lewis Dang, Sheng Xiaxi.
- Qayyum, T. (2025). [Vehicular Simulation Dataset (VEINS OMNeT++)](https://www.kaggle.com/datasets/ranatariq09/vehicular-simulation-dataset-veins-omnet). Kaggle.
- Viktor, P. and Kiss, G. (2026). [Sensors in Self-Driving Vehicles: A Detailed Literature Review and New Trends](https://doi.org/10.3390/s26072153). Sensors, 26(7), 2153.
- Optimisation solved with [Gurobi](https://www.gurobi.com/).
