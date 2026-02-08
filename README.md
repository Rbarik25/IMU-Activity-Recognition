# IMU-Based Human Activity Recognition using Nicla Vision

## Overview
This project implements an end-to-end **Human Activity Recognition (HAR)** system using wrist-mounted **IMU data** collected from the **Nicla Vision** board. Raw accelerometer and gyroscope signals are processed using a **sliding window–based feature extraction pipeline**, followed by **machine learning–based activity classification**. The trained model is deployed for **real-time inference** using live sensor data streaming from the device to a host PC.

The project follows a **TinyML-inspired workflow**, covering data collection, exploratory analysis, feature engineering, model development, evaluation, and real-time deployment.

---

## Activities Considered
The system recognizes the following activities:

- Sitting  
- Standing  
- Walking  
- Brisk Walking  
- Jogging  
- Cycling  
- Stair-Up  
- Stair-Down  
- Sit–Stand–Sit (Transitional)  
- Phone Interaction  
- Eating with Spoon  
- Pick and Place  

---

## Data Collection
- **Device:** Nicla Vision (wrist-mounted)  
- **Sensors:**  
  - Accelerometer (Ax, Ay, Az)  
  - Gyroscope (Gx, Gy, Gz)  
- **Sampling Frequency:** 50 Hz  
- **Duration per Activity:** ~3 minutes  
- **Samples per Activity:** ~9000  
- **Format:** Separate CSV file per activity  

Data collection was performed individually following prescribed motion constraints to ensure consistency and avoid data leakage.

---

## Windowing Strategy
- **Window Size:** 20 samples  
- **Overlap:** 50% (step size = 10 samples)  

**Rationale:**
- Preserves temporal continuity  
- Enables near real-time inference  
- Matches deployment constraints  

Each window produces **one training instance**.

---

## Feature Engineering

### Raw Features
For each window, raw IMU values from the **last sample** are included:
- Ax, Ay, Az, Gx, Gy, Gz  

### Time-Domain Features (per channel)
- Mean  
- Minimum  
- Maximum  
- Standard Deviation  
- Variance  
- Root Mean Square (RMS)  
- Peak-to-Peak  
- Median  
- Mean Absolute Deviation (MAD)  
- Signal Energy  

**Total Features per Window:** 66

---

## Exploratory Data Analysis
Exploratory analysis was conducted to understand signal characteristics and inter-activity differences using:
- Time-domain accelerometer vs gyroscope plots  
- Kernel Density Estimation (KDE) plots for all IMU channels  
- Box plots showing activity-wise signal spread  
- Feature variance and distribution analysis  

These analyses helped identify separable patterns as well as overlapping activity behaviors.

---

## Model Development
- **Model Used:** Decision Tree Classifier  

**Reasons for Selection:**
- Interpretable decision-making structure  
- Low computational complexity  
- Suitable for embedded and TinyML-style pipelines  

### Hyperparameter Exploration
Model depth and split criteria were tuned to balance bias–variance tradeoff and improve generalization.

---

## Model Evaluation
The trained model was evaluated using:
- Accuracy  
- Precision  
- Recall  
- F1-score  
- Confusion Matrix  

The evaluation showed strong performance across most activities, with expected confusion among activities with similar wrist motion patterns.

---

## Real-Time Deployment

### Deployment Pipeline
1. **Nicla Vision (OpenMV)**  
   - Streams live IMU data over UDP  

2. **Host PC (Python)**  
   - Buffers incoming IMU samples  
   - Applies identical windowing and feature extraction  
   - Loads trained model and label map  
   - Outputs real-time activity predictions  

### Live Prediction Behavior
- Predictions are generated once every window  
- Lightweight smoothing improves temporal stability  
- No retraining is performed during deployment  

---

## Misclassifications and Challenges

### Observed Misclassifications
- Standing vs Sit–Stand–Sit due to similar low-motion patterns  
- Stair-Up vs Stair-Down due to comparable vertical wrist movements  
- Walking vs Brisk Walking depending on pace variability  

### Challenges During Data Collection
- Variability in wrist orientation  
- Human inconsistency across repetitions  
- Transitional motion introducing noise  
- Wireless streaming jitter during live inference  

---

## Key Observations and Learnings
- Time-domain features are effective for short-window HAR  
- Transitional activities are inherently harder to classify  
- Offline evaluation metrics are more stable than live predictions  
- Prediction smoothing improves real-time usability  

---

## Project Structure
IMU-Activity-Recognition/
│
├── README.md
├── requirements.txt
│
├── data/
│ ├── Raw data/
| └── Featured data/
│
├── notebooks/
│ ├── 01_data_analysis.ipynb
│ ├── 02_feature_engineering.ipynb
│ └── 03_model_development_and_evaluation.ipynb
│
├── nicla/
| ├──  main.py # Runs on Nicla Vision (OpenMV)
| └── recieve.py # Runs o PC for data saving
├── deployment/
│ ├── activity_model.pkl
│ ├── label_map.json
│ └── live_prediction.py # Runs on PC for real-time inference
│
└── results/
  ├── Confusion_Matrix.jpg
  ├── Classification_Report.jpg
  ├── decision_Tree.jpg
  ├── important_features.png
  ├── KNE_plots/
  ├── box_plots/
  └── time_domain_signals/

---

## How to Run

### 1. Install Dependencies
```bash
pip install -r requirements.txt
 ### Data Collection
- Open main.py in OpenMV IDE
- Flash and run on Nicla Vision

---

### Model Training
Run notebooks in order:
1. 01_data_analysis.ipynb
2. 02_feature_engineering.ipynb
3. 03_model_development_and_evaluation.ipynb

### Real-Time Inference
python deployment/live_prediction.py

---

## Conclusion
This project demonstrates a complete IMU-based human activity recognition pipeline, integrating sensor data acquisition, feature engineering, interpretable machine learning, and real-time deployment. It highlights practical challenges and design considerations involved in building reliable edge-AI systems for wearable sensing applications.
