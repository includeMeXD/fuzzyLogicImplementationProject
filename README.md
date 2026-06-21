# Fuzzy Logic Implementation: Airline Passenger Satisfaction

This repository contains the final project for the **Data and Knowledge Architecture (DKA)** course. The project designs and implements a **Fuzzy Inference System (FIS)** from scratch to estimate airline passenger satisfaction based on key service ratings. It implements and compares two primary fuzzy inference methods: **Mamdani** and **Sugeno (Order-1)**.

## Project Overview

Passenger satisfaction is a key metric in the airline industry. Predicting satisfaction based on multiple customer service touchpoints often involves subjective and imprecise human feedback. This project models this decision-making process using Fuzzy Logic, mapping five continuous service quality inputs (0–5 rating scale) to a unified passenger satisfaction score (0–100 scale).

### Project Metadata
*   **Course:** Data and Knowledge Architecture (DKA) — Final Project (Tugas Besar)
*   **Authors:**
    *   Muhammad Alzidane Mahesa
    *   Carrol Estevao Lay
    *   Hafizh Azrial
*   **Source Code Repository:** [Introduction-To-AI-Final-Project](https://github.com/alzidaneMhs/Introduction-To-AI-Final-Project.git)

---

## Dataset & Variables

The project utilizes the popular [Airline Passenger Satisfaction Dataset](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction) from Kaggle. After pre-processing and dropping missing values, the training set consists of **103,904 rows**. For fuzzy inference evaluation, a representative random sample of **5,000 rows** is used.

### Inputs (Linguistic Variables)
Ratings are evaluated on a scale of **0 to 5** across five selected service dimensions:
1.  **Inflight Wi-Fi Service**
2.  **Food and Drink**
3.  **Seat Comfort**
4.  **Inflight Entertainment**
5.  **Online Boarding**

### Output (Satisfaction Score)
*   **Continuous Output:** A score of **0 to 100** representing passenger satisfaction.
*   **Classification Label:** The score is mapped to a binary label (threshold $\ge 50$ for `satisfied`, otherwise `neutral or dissatisfied`) to compare against the dataset's ground truth.

---

## Fuzzy Logic Design

### 1. Membership Functions (Linguistic Variables)

The membership functions (fuzzy partitions) for inputs and outputs are defined below:

*   **Input Variables (0 to 5 scale):**
    *   **Poor:** Trapezoidal, parameters $[0, 0, 1, 2.5]$
    *   **Average:** Triangular, parameters $[1, 2.5, 4]$
    *   **Good:** Trapezoidal, parameters $[2.5, 4, 5, 5]$
*   **Mamdani Output Variable (Satisfaction Score, 0 to 100 scale):**
    *   **Low:** Trapezoidal, parameters $[0, 0, 30, 50]$
    *   **Medium:** Triangular, parameters $[30, 50, 70]$
    *   **High:** Trapezoidal, parameters $[50, 70, 100, 100]$

*Both the triangular function (`trimf`) and trapezoidal function (`trapmf`) are implemented from scratch in pure Python.*

### 2. Rule Base (20 Rules)

The system integrates a total of **20 fuzzy rules** combining inputs using the **AND (minimum)** operator. The rules govern how different service ratings contribute to the output satisfaction class:

| Rule No. | Inflight Wi-Fi | Food & Drink | Seat Comfort | Inflight Ent. | Online Boarding | Consequent (Output Label) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | Good | Good | Good | Good | Good | **High** |
| **2** | Good | Good | Good | Average | Good | **High** |
| **3** | Good | Average | Good | Good | Average | **High** |
| **4** | Average | Good | Good | Good | Good | **High** |
| **5** | Good | Good | Average | Good | Average | **High** |
| **6** | Average | Average | Average | Average | Average | **Medium** |
| **7** | Average | Average | Good | Average | Average | **Medium** |
| **8** | Good | Poor | Average | Average | Average | **Medium** |
| **9** | Average | Good | Poor | Average | Good | **Medium** |
| **10** | Average | Average | Average | Good | Poor | **Medium** |
| **11** | Poor | Average | Average | Average | Average | **Medium** |
| **12** | Average | Poor | Average | Poor | Average | **Low** |
| **13** | Poor | Poor | Poor | Poor | Poor | **Low** |
| **14** | Poor | Poor | Average | Poor | Average | **Low** |
| **15** | Poor | Average | Poor | Average | Poor | **Low** |
| **16** | Average | Poor | Poor | Poor | Average | **Low** |
| **17** | Poor | Poor | Poor | Average | Poor | **Low** |
| **18** | Good | Poor | Poor | Poor | Poor | **Medium** |
| **19** | Average | Average | Poor | Poor | Poor | **Low** |
| **20** | Good | Good | Poor | Average | Poor | **Medium** |

---

## Inference & Defuzzification Implementations

### Mamdani FIS
1.  **Fuzzification:** Input ratings are mapped to membership degrees.
2.  **Fuzzy Inference:** Firing strengths ($\alpha$) are calculated using the minimum of membership values. The consequent output fuzzy sets are clipped using the calculated firing strength.
3.  **Aggregation:** All clipped output sets are combined using the maximum operator.
4.  **Defuzzification:** A crisp output score is computed using the **Centroid (Center of Gravity)** method, discretized over the range $[0, 100]$.

### Sugeno FIS (Order-1)
*   Shares the same fuzzification and rule base as Mamdani.
*   The consequent of each rule is a linear function $z = f(\text{inputs})$:
    *   **Low Consequent:** $z = 4 \times (\text{wifi} + \text{food} + \text{seat} + \text{entertainment} + \text{boarding})$
    *   **Medium Consequent:** $z = 4 \times (\text{wifi} + \text{food} + \text{seat} + \text{entertainment} + \text{boarding}) + 15$
    *   **High Consequent:** $z = 4 \times (\text{wifi} + \text{food} + \text{seat} + \text{entertainment} + \text{boarding}) + 30$ (Capped at 100)
*   **Weighted Defuzzification:** The final output is calculated as:
    $$z^* = \frac{\sum \alpha_i z_i}{\sum \alpha_i}$$

---

## Comparison and Evaluation

Running both models on the same 5,000 sample records yields the following performance metrics:

### 1. Mamdani vs. Sugeno Scores
*   **Mean Absolute Error (MAE):** $5.783$
*   **Mean Squared Error (MSE):** $108.332$
*   **Correlation Coefficient:** $0.911$
*   **Mean Difference (Mamdani - Sugeno):** $-5.75$

*The 0.911 correlation confirms strong agreement on overall rating trends. Mamdani scores are systematically lower on average due to the centroid defuzzification's tendency to pull outputs toward the center (smoothing effect).*

### 2. Classification Metrics (Fuzzy Label vs. Ground Truth)
By thresholding predicted satisfaction scores at $\ge 50$ and comparing with Kaggle's binary ground truth:

| Method | Accuracy | Precision | Recall | F1-Score |
| :--- | :---: | :---: | :---: | :---: |
| **Mamdani** | **0.587** | 0.514 | 0.987 | **0.676** |
| **Sugeno** | 0.560 | 0.498 | **0.989** | 0.662 |

*Both methods demonstrate exceptional Recall ($\approx 99\%$), rarely misclassifying satisfied passengers. The lower Precision suggests that passengers predicted to be satisfied often belong to the neutral/dissatisfied group in reality.*

---

## Repository Structure

*   `DKA_TUBES_Fuzzy_Airline_Satisfaction.ipynb`: Jupyter notebook containing the full exploratory data analysis, fuzzy logic implementation (Mamdani and Sugeno), calculations, and visualizations.
*   `TUBES_Fuzzy_Airline_Satisfaction_Report.pdf`: Comprehensive final project report.
*   `train.csv`: Training dataset (103k+ records).
*   `test.csv`: Testing dataset (30k+ records).
*   `README.md`: Project documentation.

---

## Setup and Usage

### Prerequisites
Make sure you have Python 3.8+ installed along with the following packages:
```bash
pip install numpy pandas matplotlib seaborn scikit-learn
```

### Running the Project
1.  Clone this repository:
    ```bash
    git clone https://github.com/alzidaneMhs/Introduction-To-AI-Final-Project.git
    cd Introduction-To-AI-Final-Project
    ```
2.  Open and run the Jupyter Notebook:
    ```bash
    jupyter notebook DKA_TUBES_Fuzzy_Airline_Satisfaction.ipynb
    ```
