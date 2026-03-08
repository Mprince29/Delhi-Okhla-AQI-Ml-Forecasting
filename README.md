# Delhi AQI Predictor — PM2.5 Forecasting for Okhla Phase-2

I built this project to predict Delhi's air quality using 6 years of real government sensor data from the Central Pollution Control Board (CPCB). This is a deep-dive learning journey into Machine Learning, evolving from a high-performance baseline model to building neural networks from scratch and exploring state-of-the-art architectures.

---

## 🗓️ Revised 4-Week Roadmap — Bootcamp Pace (10-13 Hours/Week)

This project has evolved into a rigorous 4-week deep-learning bootcamp.

### Week 1 — Neural Network From Scratch
**Status: [In Progress]**
- **Neural Network Foundation**: Built a multi-layer neural network using only NumPy (no PyTorch/Keras).
- **Activation & Loss**: Implemented ReLU, Linear, and MSE loss from scratch.
- **Optimization**: Upgraded from vanilla Gradient Descent to the **ADAM Optimizer** for faster convergence.
- **Comparison**: Achieved a functional (though currently underperforming) baseline to compare against the Week 0 Random Forest.

### Week 2 — Fine-Tuning BERT (HuggingFace)
- **Concept**: Understanding Attention mechanisms and Transformers.
- **Task**: Load BERT, tokenize Indian air quality text data, and fine-tune for sentiment or classification tasks.
- **Optimization**: Experiment with freezing layers and comparing DistilBERT/RoBERTa results.

### Week 3 — Multi-Model Systems and Stacking
- **Ensemble Theory**: Bagging vs Boosting vs Stacking.
- **Implementation**: Combine Model 1 (Random Forest), Model 2 (XGBoost), and Model 3 (Scratch Neural Net) into a meta-learner (Stacking).
- **Goal**: Outperform the current R² 0.9501 baseline.

### Week 4 — Reinforcement Learning
- **New Paradigm**: Agent, Environment, State, Action, Reward.
- **Algorithm**: Build Q-Learning from scratch and move to OpenAI Gymnasium.
- **Advanced**: Implement a Deep Q-Network (DQN) connecting RL with the Neural Networks from Week 1.

---

## 📊 Current Results & Progress

| Model | MAE | R² | Status |
|---|---|---|---|
| **Random Forest (Baseline)** | **12.40** | **0.9501** | Production Ready |
| **NN Scratch (Vanilla GD)** | 58.64 | 0.3742 | Concept Verified |
| **NN Scratch (ADAM)** | *Evaluating* | *Evaluating* | In Training |

---

## Project Structure

```
delhi_okhla_project/
│
├── raw_data/                            # Downloaded .csv.gz files from OpenAQ S3
│
├── delhi_newdelhi_2020_2026_clean.csv   # Final cleaned dataset
├── okhla_pm25_best_model.pkl            # Week 0 Baseline (Random Forest)
├── model_features.json                  # Feature list for baseline
│
├── okhla_aqi_model.ipynb                # Week 0 — Data Pipeline & Baseline Models
├── neural_network_from_scratch.ipynb    # Week 1 — NumPy Neural Network + Adam
│
├── requirements.txt                     # Dependencies
└── README.md                            # This file
```

---

## 🚀 Getting Started

### 1. Setup Environment
```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
# For Week 1:
pip install numpy pandas matplotlib jupyter
```

### 2. Data Download
I use AWS CLI to download 6 years of Okhla sensor data from OpenAQ's public S3 bucket (region `us-east-1`). No API key required.

```bash
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2024/ ./raw_data/2024/ --recursive --no-sign-request
```

---

## 📌 The Sunday Rule
Articulating what was built is mandatory. Every week ends with a documentation session answering:
1. What did I build this week?
2. What concept was most confusing?
3. What happens if I remove part X?
4. How does this connect to the previous week?

---

## Data Attribution
Data sourced from **OpenAQ** via **CPCB** (Central Pollution Control Board). The Okhla Phase-2 station is operated by **DPCC**.
