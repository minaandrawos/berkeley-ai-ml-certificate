# Module 17: Practical Application III - Comparing Classifiers (Bank Marketing)

[![Jupyter Notebook](https://img.shields.io/badge/Jupyter%20Notebook-prompt__III.ipynb-orange?logo=jupyter)](prompt_III.ipynb)
[![Python 3.13](https://img.shields.io/badge/Python-3.13-blue.svg?logo=python)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.7-F7931E.svg?logo=scikit-learn)](https://scikit-learn.org/)

---

## 1. Project Summary

This project evaluates four classification algorithms—**Logistic Regression**, **K-Nearest Neighbors (KNN)**, **Decision Trees**, and **Support Vector Machines (SVM)**—to optimize bank direct telemarketing campaigns for long-term deposit subscriptions.

Using a real-world dataset from a Portuguese banking institution containing **41,188 customer contacts** collected between May 2008 and November 2010 (Moro, Cortez & Rita, 2014), we build predictive models to pre-screen customer lead lists before calls are made. The accompanying paper ([Moro, Laureano & Cortez, 2011](CRISP-DM-BANK.pdf)) describes the same bank's **17 marketing campaigns** over that period.

Because **88.7%** of contacts declined deposit offers (`y = "no"`), a basic model that blindly predicts "no" for everyone achieves 88.7% accuracy while missing 100% of potential subscribers. To ensure meaningful business evaluation, models are assessed using **Recall** (percentage of subscribers captured), **F1-Score** (campaign efficiency balance), and **Accuracy**.

---

## 2. Practical Findings

### Pre-Call Model Comparison Table (Without Call Duration, since it was recommended to not use Duration in the analysis)

**Default class weighting** — every model collapses toward predicting "no":

| Model | Test Accuracy | Test Precision | Test Recall | Test F1-Score | Best Hyperparameters |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Baseline (Dummy)** | 0.8874 | 0.0000 | 0.0000 | 0.0000 | Most Frequent (`y = "no"`) |
| **Tuned K-Nearest Neighbors** | 0.8945 | 0.5623 | **0.2852** | **0.3785** | `n_neighbors=5` |
| **Tuned Decision Tree** | 0.8965 | 0.5907 | 0.2644 | 0.3653 | `max_depth=10, criterion="gini"` |
| **Tuned Logistic Regression** | **0.9006** | 0.6907 | 0.2134 | 0.3260 | `C=1.0` |
| **Tuned Support Vector Machine** | 0.9005 | 0.7139 | 0.1954 | 0.3068 | `C=1.0, Linear` |

**With `class_weight='balanced'`** — the recommended configuration (KNN is not shown in this table since it could not be balanced):

| Model | Test Accuracy | Test Precision | Test Recall | Test F1-Score | Best Hyperparameters |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Balanced Decision Tree** *(recommended)* | **0.8550** | 0.4057 | 0.6185 | **0.4900** | `max_depth=5, criterion="gini"` |
| **Balanced Logistic Regression** | 0.8321 | 0.3627 | **0.6480** | 0.4651 | `C=1.0` |
| **Balanced Support Vector Machine** | 0.8314 | 0.3611 | 0.6451 | 0.4630 | `C=1.0, Linear` |

*All figures are deterministic and reproducible with `random_state=42`. Fit times vary between executions and are reported in the notebook itself.*

### Key Insights from the Data

1. **Fixing the Class Imbalance was an improvement**: With default weighting, recall scores are very low, highest observed was **~0.2852 recall** — the models were quietly optimizing toward the useless "predict no for everyone" strategy. Setting `class_weight='balanced'` **noticeably increased recall** for the three models that support it (3.0x for Logistic Regression, 3.3x for SVM, 2.3x for the Decision Tree) and raised F1 for all three.
2. **Recommended Model — Class-Balanced Decision Tree**: At **0.6185 recall**, **0.85 accuracy** and **0.4900 F1**, it is both the highest F1 scored model and a practical one — you walk through a handful of comparisons, and it exposes a clear set of decision rules for a compliance review. The logistic regression is also interpretable, and while it scored higher in recall, it scored lower in F1 and accuracy. *The accuracy in both is understandably less than unbalanced models, however unbalanced models are not realistic to use here due to very limited ability to detect true positives*
3. **Economic Conditions Dominate**: Overall employment level in the economy (`nr.employed` — **71.2%** of the recommended tree's splitting power) is a strong predictor, and the Logistic Regression assigns its largest coefficient to the employment variation rate (`emp.var.rate`, odds ratio **0.111**). Deposit subscription rates rise when the labour market weakens. *`emp.var.rate`, `euribor3m` and `nr.employed` are almost collinear (pairwise r of 0.91-0.97), so they are best read as one combined "economic climate" effect; `cons.conf.idx` sits outside that block and carries its own signal.*
4. **Prior Campaign Outcome — the Customer-Level Lever**: Clients who subscribed in an earlier campaign convert at **65.1%**, versus **14.2%** for those who previously declined and **8.8%** for those never contacted before. `poutcome_success` is the top customer-level feature in the recommended tree and the second strongest positive coefficient in the logistic regression (odds ratio **2.764**) — descriptive statistics, tree, and linear model all agree.
5. **High-Converting Customer Demographic Profiles** :
   - **Seniors & Retirees**: Customers **over 60** convert at **45.5%** and retirees at **25.2%** (compared to the 11.3% dataset average). *The over-60 band contains only 910 contacts, so this rate should be confirmed before large-scale targeting.*
   - **Students**: Students convert at **31.4%** (nearly 3x the dataset average), whereas blue-collar workers convert at **6.9%**.
6. **Communication medium Matters**: Reaching a client on a mobile raises the odds of subscription (odds ratio **1.394**) while a landline lowers them (odds ratio **0.754**); descriptively, mobile converts at 14.74% against 5.23% for landline.

---

## 3. Next Steps (technical)

1. **Widen the hyperparameter grids** — More combinations of hyperparameters can be tried with the CV grids
2. **Add cost-based evaluation** attaching real costs to wasted calls and real value to captured deposits, so the model optimizes expected profit rather than F1.

---

## 4. Recommendations for Business Leaders (Non Technical)

1. **Focus Calling Lists on High-Value Profiles**: Prioritize calling seniors, retirees, students, and customers who subscribed in a previous campaign, rather than reaching out to database contacts at random.
2. **Always Call Mobiles Where Available**: Mobile contacts convert at **14.74%** against **5.23%** on a landline.
3. **Time Marketing Campaigns With Market Conditions**: Increase telemarketing efforts when employment is contracting and money is cheap: subscriptions were successful at low `euribor3m` values.
4. **Run a Pilot Campaign**: Test the ranked calling list produced by this analysis with a small team of call center agents to measure the actual increase in deposit sign-ups before rolling it out full-scale.