### Luxury Watch Resale Price Analysis

**Author**: Mina Andrawos

#### Executive summary
The luxury watch market has grown significantly as both a passion asset and an alternative investment. This project identifies the features and patterns that drive the resale price of luxury watches. Using exploratory data analysis (EDA) and a baseline regression model on more than 40,000 listings, it measures how brand, case material, movement, condition, box & papers and year of production affect secondary-market prices. **Brand is by far the strongest driver.** A simple Ridge Regression baseline explains about **80% of the variance** in (log) price, and its typical prediction is within **about 37%** of the actual price.

#### Rationale
Understanding the drivers of luxury watch prices is crucial for collectors, investors and secondary-market platforms. Knowing which combinations of features best predict resale prices helps stakeholders make better-informed buying, selling and investment decisions.

#### Research Question
What key combination of features (e.g. brand, movement, material, condition) most significantly affects the resale price of luxury watches?

#### Data Sources
Two Kaggle datasets, stored as CSV files in [`data/`](data/):
1. **Luxury Watch Listings** (`data/Watches-ds-1.csv`, ~285k listings): price, brand, model, reference, movement, condition and year of production. [Dataset link](https://www.kaggle.com/datasets/philmorekoung11/luxury-watch-listings)
2. **Watch Prices Dataset** (`data/watches-ds-2.csv`, ~45k listings): adds scope of delivery (box & papers) and has more complete case material and gender fields. [Dataset link](https://www.kaggle.com/datasets/beridzeg45/watch-prices-dataset)

#### Methodology
1. **Data cleaning**: parsed price text such as `$43,500` into numbers and dropped "Price on request" listings. Extracted 4-digit production years, merged two duplicate condition columns, filled missing categorical values with `Unknown`, and removed ~9,000 duplicate listings.
2. **Outlier analysis**: the standard IQR rule on raw price would have removed 68–99% of Patek Philippe, Audemars Piguet and Richard Mille listings, i.e. the luxury segment itself. Instead, prices below $50 (a tiny number of listings, including an implausible $1 listing) and values beyond 3 x IQR on the **log** scale were removed, which affected less than 0.3% of rows.
3. **Feature engineering**: created `has_box` and `has_papers` flags from free text, `watch_age` and a `year_missing` flag, grouped rare brands and case materials into `Other`, and used `log10(price)` as the target because price is heavily right-skewed.
4. **EDA**: histograms (linear vs log), count plots, boxplots by brand, condition, box & papers and case material, a price-by-decade trend and an age scatter plot, built with Seaborn and Matplotlib.
5. **Modeling**: a scikit-learn pipeline that one-hot encodes the categorical features, with missing years filled in using the training data only. Models: a `DummyRegressor` (median) benchmark and a **Ridge Regression** baseline, with an 80/20 train/test split.
6. **Evaluation**: **R² on log price** (primary metric; scale-free and directly comparable to the benchmark), **RMSE on log price** (penalizes large mispricings) and **median absolute percentage error** (a plain-language "how far off in %" measure). Feature impact was measured with permutation importance.

#### Results
| Model | R² (log price) | RMSE (log10) | Median abs % error | Median abs error |
|---|---|---|---|---|
| Dummy (median) | -0.01 | 0.70 | 82.7% | $1,437 |
| **Ridge Regression** | **0.80** | **0.31** | **37%** | **$609** |

- **Brand dominance**: brand is the most important feature by a wide margin. Shuffling it drops R² by about 1.0, making the model worse than the naive benchmark. Among 12 well-known brands, median prices range from about $500 (Tissot) to about $80,000 (Patek Philippe).
- **Material nuances (steel vs gold)**: precious-metal cases usually command a premium. However, I know from experience that some steel Rolex, Patek Philippe and Audemars Piguet watches can be very expensive in the resale market. The data showed Patek Philippe steel watches have a higher median resale price (about $90k) than their white gold (about $68k) and yellow gold (about $20k) watches. A possible explanation is demand for steel sports models like the Nautilus, but the dataset has no model names to test this.
- **Secondary drivers**: after brand, the most important features are case material and movement (precious metals and mechanical movements carry premiums over steel and quartz), followed by condition and original papers. Listings with neither box nor papers have the lowest median price.
- **Brand-mix effects**: some simple comparisons are misleading. Used (Very good) watches have a higher overall median than New ones largely because they come from pricier brands; within the same brand, New is priced higher for 76% of brands.
- **Year of production**: prices follow a non-linear pattern by production decade (Dataset 1), and 41% of years are missing in the modeling data (Dataset 2). As a straight-line `watch_age` feature it adds almost nothing once brand is known. This means age is not useful *in a linear form*, not that it doesn't matter.
- **Model performance**: the baseline reliably places a watch in the right price tier (R² = 0.80), but a typical ~37% error is not yet precise enough to price individual watches for investment decisions. It also under-predicts the most expensive watches and over-predicts the cheapest ones.

#### Next steps
- **Advanced modeling**: try tree-based models (Random Forest) with cross-validated hyperparameter tuning, which can capture brand x material interactions and the non-linear age effect.
- **Deeper feature engineering**: extract model lines and reference numbers (e.g. Nautilus, Daytona, Royal Oak) from Dataset 1, which likely explain much of the remaining error.
- **Text analysis**: apply NLP to listing titles and descriptions to capture limited editions, special dials and other value drivers.

#### Outline of project
- [Luxury Watch Analysis Notebook](Luxury_Watch_Analysis.ipynb): data cleaning, outlier analysis, feature engineering, EDA, modeling and evaluation.
