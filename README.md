# Quandela Hackathon, Track 2: swaption forecasting with an MLP and a small quantum circuit

Short introductory hackathon organised by Quandela (less than a day), Track 2: swaption price prediction with quantum machine learning. I only worked on one piece of the problem: forecasting the next observation of the price surface, to compare a quantum circuit against classical baselines. The notebook was started during the hackathon and cleaned up afterwards.

**Short version: neither the MLP nor the quantum circuit beats the "nothing changes" baseline.**

## Data

Simulated prices provided by Quandela: they calibrated a model on real market data and generated the prices with a random number generator. 5000 dates, 224 columns (tenor × maturity). The file belongs to the organisers, so it's not in this repo: put your copy next to the notebook as `train_augmented.xlsx`.

## Method

- Chronological split: 3500 train / 500 validation / 1000 test.
- Standardisation + PCA fitted on the train set only, 4 components (99.97% of the variance), one per qubit.
- The models predict the next change of the 4 PCA coordinates. The surface is then rebuilt with the inverse PCA and compared to the real one.
- Models:
  - persistence: the next surface is the current one,
  - an MLP (4-32-4, ReLU, Adam, early stopping on validation),
  - a 4-qubit variational circuit (4 layers, 32 parameters, trained with COBYLA, simulated with Qiskit `Statevector`).
- The MLP and the circuit use the same 400 training pairs, because the circuit is slow to simulate.

## Results (test set)

| Model | MAE | RMSE | RMSE vs persistence |
|---|---|---|---|
| Persistence | 0.002946 | 0.003914 | n/a |
| MLP | 0.002976 | 0.003970 | -1.4% |
| Quantum circuit | 0.003149 | 0.004233 | -8.1% |

- COBYLA stopped at its 200-evaluation limit without converging, so part of the gap with the MLP may just be optimisation budget.
- Re-encoding the inputs at every layer (`reupload = True`) did not improve the validation error.
- The circuit is simulated on a classical machine. This shows no quantum advantage.
