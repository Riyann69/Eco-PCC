# Eco-PCC: AI-Based Ecological Predictive Cruise Control for Connected EVs

> Optimizing Energy Efficiency in Connected Electric Vehicles through Deep Learning (LSTM) and Model Predictive Control (MPC)

**Riyan Wankhede (23BCE9287), Nishita (23BCE8235)**  
Department of Computer Science and Engineering, VIT-AP University

**Result:** in a 60-second variable highway simulation, both LSTM-based controllers use about **13 % less energy** than a
reactive bang-bang baseline, and LSTM-PID also gives **4× smoother torque**.

---

## Overview

Standard cruise control systems react to speed errors after they occur, wasting energy through unnecessary acceleration/braking cycles and missing regenerative braking opportunities. Eco-PCC addresses this by integrating a stacked LSTM neural network that predicts upcoming traffic speed into two advanced controllers, enabling proactive torque management and measurable energy savings over a reactive baseline.

---



## System Architecture

```
Vehicle Speed → LSTM Predictor → MPC / PID Controller → EV Torque → EV Dynamics
                                           ↑
                                  Battery SOC Feedback
```

---



## Results


| Controller         | Energy (Wh) | SOC Drop (%) | RMSE (m/s) | Torque Smoothness (Nm) | Energy Saving |
| ------------------ | ----------- | ------------ | ---------- | ---------------------- | ------------- |
| Bang-Bang Baseline | 219.1       | 0.329        | 4.077      | 30.57                  | —             |
| LSTM-PID           | 190.0       | 0.282        | 4.009      | **7.34**               | **+13.3%**    |
| LSTM-MPC           | 190.7       | 0.283        | 4.078      | 24.94                  | **+13.0%**    |


- LSTM-PID achieves the best torque smoothness — **4× smoother** than bang-bang
- Both LSTM controllers save ~13% energy over the reactive baseline
- Validated over a 60-second variable highway profile (vref = 20 + 8sin(t/10) + 3sin(t/3))

![Controller Comparison](results/controller_comparison.png)

---



## Technical Details



### LSTM Speed Predictor

- Architecture: 2-layer stacked (128→64 units), BatchNorm + Dropout (p=0.2)
- Input: 100-step velocity history → Output: 20-step velocity forecast
- Training: 10,000 synthetic sequences across 4 driving scenarios (highway, urban, smooth, variable)
- 80/20 train/val split — scalers fit on training data only (no leakage)
- Validation RMSE: **0.423 m/s** | MAPE: **1.76%** after 50 epochs



### EV Digital Twin

- Mass: 2500 kg | Battery: 48 kWh | Wheel radius: 0.36 m
- Drag coefficient: 0.28 | Frontal area: 2.5 m²
- Motor efficiency: 92% (motoring) | Regenerative efficiency: 75%
- Torque bounds: +400 Nm (drive) / −200 Nm (regen)



### LSTM-PID Controller

- Classical PID (kp=120, ki=8, kd=25) augmented with deceleration-anticipation feedforward
- Feedforward applied **only for predicted slowdowns** — no pre-emptive acceleration
- Feedforward asymmetry principle: coasting early saves energy; early motoring wastes it



### LSTM-MPC Controller

- Receding horizon N=10 steps via SLSQP solver (SciPy)
- Cost function: `w_track × error² + w_energy × (P/P_ref)² + w_smooth × ΔTorque²`
- Power normalized by P_ref = 10 kW to prevent energy term dominance
- LSTM forecast used directly as MPC reference trajectory
- Regen power not penalized — optimizer encouraged to maximize regen

---



## Key Design Decisions


| Decision                             | Why                                                                              |
| ------------------------------------ | -------------------------------------------------------------------------------- |
| Deceleration-only feedforward in PID | Pre-emptive acceleration wastes energy; only slowdowns benefit from early action |
| Normalized energy cost `(P/P_ref)²`  | Raw `P²` (~5×10⁸) dominates tracking cost at any reasonable weight               |
| Scalers fit on train split only      | Prevents data leakage from validation set into model training                    |
| Velocity warm-start `v(0) = vref(0)` | Avoids cold-start transient dominating 60s simulation results                    |


---



## Setup

```bash
git clone https://github.com/Riyann69/Eco-PCC.git
cd Eco-PCC
pip install -r requirements.txt
jupyter notebook Eco-Cruise_Control_for_EVs.ipynb
```

Run all cells top to bottom. The MPC simulation takes approximately 2–3 minutes.

---



## Repository Structure

```
Eco-PCC/
├── Eco-Cruise_Control_for_EVs.ipynb   # Main notebook — data, LSTM, controllers, simulation
├── requirements.txt                   # Python dependencies
├── README.md
├── results/
│   └── controller_comparison.png      # 6-panel simulation output
├── report/
│   ├── abstract.md                      
│   ├── Research_Paper.pdf             # IEEE-format paper
│   └── Eco-PCC-Slides.pdf             # Presentation slides
└── reference/
    └── Zhou_et_al_2021_PACC.pdf       # Base reference paper
```

---



## Paper

*"AI-Based Ecological Predictive Cruise Control for Connected Electric Vehicles Using LSTM-Enhanced PID and Model Predictive Control"*  
Riyan Wankhede (23BCE9287), Nishita (23BCE8235)  
VIT-AP University, 2026

---



## References

- Hochreiter, S. & Schmidhuber, J. (1997). Long Short-Term Memory. *Neural Computation*, 9(8), 1735–1780.
- Rawlings, J. B., Mayne, D. Q., & Diehl, M. (2019). *Model Predictive Control: Theory, Computation, and Design*. Nob Hill Publishing.
- Vahidi, A. & Sciarretta, A. (2018). Energy Saving Potentials of Connected and Automated Vehicles. *Transportation Research Part C*, 95, 822–843.
- SciPy Documentation — SLSQP solver: [https://docs.scipy.org/doc/scipy/](https://docs.scipy.org/doc/scipy/)
- TensorFlow/Keras Documentation: [https://www.tensorflow.org/api_docs/](https://www.tensorflow.org/api_docs/)

---



## License

MIT