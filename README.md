## **VERITAS CASSANDRA: PROJECT REPORT**
### **Auto Insurance Risk Modeling | Electronics Engineering Society (EES) IIT BHU**

---

### **PART A: DATA INVESTIGATION (Exploratory Data Analysis)**

#### **1. Feature Understanding and Metadata**
* **Scale of Data:** Training set comprises **148,463 records** with 59 features; test set contains **178,564 records**.
* **Automated Metadata Schema:** A custom script categorized features based on semantic naming conventions:
    * **ind:** Individual driver data.
    * **reg:** Regional/registration data.
    * **car:** Vehicle-specific attributes.
    * **calc:** Calculated statistics (later identified as noise).
* **Type Determination:** Automated logic classified features into Binary, Nominal, Ordinal, and Continuous types to dictate encoding strategies.

#### **2. Missing Value & Null-Pointer Analysis**
* **Implicit Null Detection:** Missing data was strategically encoded as **-1**. Columns like `ps_car_03_cat` and `ps_car_05_cat` showed high missingness.
* **Strategy:** Missingness was treated as a **distinct category**. In insurance, the absence of data often carries a predictive risk signal (Non-disclosure risk).

#### **3. Semantic Validation: Ordinal vs. Nominal Patterns**
* **Ordinal Evidence (`ps_ind_15`):** Showed a clear **Monotonic Trend**. As the feature value increased, the claim rate followed a predictable curve, justifying its treatment as a numerical scale.
* **Nominal Evidence (`ps_car_01_cat`):** Showed a **Stochastic Pattern**. Different IDs resulted in erratic claim rates, confirming the necessity of One-Hot or Target Encoding.

#### **4. Risk Trend Analysis & Class Imbalance**
* **Class Imbalance:** Typical of insurance data; Class 0 (No Claim) at **96.33%** and Class 1 (Claim Filed) at **3.67%**.
* **Kolmogorov-Smirnov (KS) Test:** Applied to continuous features to distinguish "claimants" from "non-claimants." `ps_car_13` (KS: 0.117) emerged as the strongest continuous driver.

---

### **PART B: STRUCTURAL DISCOVERY AND ADVANCED EXPLORATION**

#### **B.1: Representation Learning (DAE)**
* **Denoising Autoencoder (DAE):** Implemented a **512-128-512** architecture with **15% Swap Noise** to force the model to learn relationships rather than memorization.
* **RankGauss Normalization:** Continuous variables were mapped to a normal distribution to prevent gradient saturation in Deep Learning.
* **Manifold Discovery (UMAP):** Projecting the 128D bottleneck into 2D revealed "Islands" of drivers—high-risk micro-segments that were linearly separable.


#### **B.2: Relationship and Network Discovery**
* **HDBSCAN Clustering:** Identified densities representing specific behavioral profiles. 
    * **Community 25:** "Critical Risk" zone with a **14.7% claim probability** (4x the baseline).
    * **Safe Havens:** Communities with near 0% claim rates despite high density.
* **Risk Homophily:** A score of **0.0295** confirmed that high-risk profiles tend to cluster together in latent space.

#### **B.3: Interaction and Dependency Discovery**
* **Information Theory (MI Lift):** Identified 20 high-signal feature couplings. The pair `ps_reg_01 x ps_ind_03` showed massive predictive "Lift."
* **KDE Visualization:** Kernel Density plots revealed "Emergent Manifolds"—hot zones where claimant densities spike in 2D space.


#### **B.4: Statistical Structure and Data Generation**
* **Copula Modeling:** Strategy to decouple marginal distributions from dependency structures to capture "tail dependency" (e.g., young drivers + high-power cars).
* **Directed Acyclic Graphs (DAGs):** Proposed use of the PC Algorithm to map causal logic (e.g., does Geography cause Vehicle Choice?).
* **Missingness Mechanism:** Modeled as **MNAR (Missing Not At Random)** to uncover the "Incomplete Information" rules of the system.

---

### **PART C: PREDICTIVE MODELING**

#### **1. Feature Audit & Assembly**
* **Recursive Feature Elimination (RFE):** Pruned the set to **250 high-signal predictors**.
* **Phase 2 Dominance:** **93.3%** of the top 30 features were derived from Phase 2 (DAE Embeddings & Interactions).

#### **2. Diverse Base Learner Architecture**
* **CatBoost:** Handled categorical bias with symmetric trees.
* **LightGBM/XGBoost:** Tuned via **Optuna** for leaf-wise and histogram-based growth.
* **JahrerNet:** A custom 4-layer Neural Network to extract non-linear latent signals.

#### **3. Meta-Stacking & Results**
* **Ridge Regression Stacker:** Aggregated Out-of-Fold (OOF) predictions. By using Ridge (`fit_intercept=False`), the model learned optimal "Trust Weights" for each learner.
* **Final Gini Score: 0.26331**


#### **4. Performance Analysis & Remarks**
* **Ensemble Alpha:** The success was driven by the **Low Correlation** between the Neural Network ($\rho \approx 0.72$) and the Tree models ($\rho > 0.94$). The NN captured functional relationships the GBDTs missed.
* **Conclusion:** The transition from raw EDA to latent DAE embeddings provided the "Structural Advantage" required to solve the anonymized Veritas dataset.
