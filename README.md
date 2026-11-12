# FicZon Lead Category Predictor

FicZon gets most of its leads from its website and other online channels, and the sales team sorts them into good and bad leads by hand. This project does two things: it digs into the lead data to see what drives sales success, and it trains a model that predicts whether a new lead is **High Potential** or **Low Potential**, so the team can call the best leads first.

## Dataset

7,422 lead records with 9 columns (created time, product, source, mobile, email, sales agent, location, delivery mode, status). The data was pulled from a MySQL database with SQLAlchemy.

## How I built the target

The `Status` column has 10 different values, so I grouped them into two classes:

- **High Potential:** Converted, Potential, Long Term, In Progress Positive
- **Low Potential:** Junk Lead, Not Responding, Just Enquiry, In Progress Negative, Lost

The 80 `Open` leads were removed because they have no outcome yet and shouldn't be counted as Low Potential. That leaves 7,340 leads: 2,849 High Potential (38.8%) and 4,491 Low Potential.

## Cleaning and feature engineering

- Missing values were stored as blank strings, so `isnull()` missed them. I found them with `(df == "")` and filled `Product_ID`, `Source`, `Sales_Agent` and `Location` with "Unknown".
- `Mobile` was blank in 24% of rows and `EMAIL` had many `#VALUE!` errors, so I turned them into two 0/1 features: `Has_Mobile` and `Email_Valid`.
- Pulled `Hour`, `Weekday`, `Month_No` and `Lead_Age_Days` out of the created timestamp.
- Grouped sources with fewer than 100 leads into "Other", then one-hot encoded the categorical columns (75 features in the end).
- Dropped `Status` from the features since it is what the target is built from (data leakage). The scaler was fitted on the training data only.
- 80/20 stratified split: 5,872 train and 1,468 test rows.

## What the data showed

| Insight | Number |
|---|---|
| Customer Referral leads that are High Potential | 89% (180 leads) |
| Bangalore leads that are High Potential | 61% (2,082 leads) |
| Sales-Agent-9 High Potential rate | 63% (879 leads) |
| Delivery Mode-5 vs Mode-4 | 25% vs 68% |
| Leads with a valid email vs a broken one | 45% vs 18% |

I always read each rate next to the number of leads, because small groups can show 100% just by chance.

## Model results

Test set results (always predicting Low Potential would give 61.2% accuracy):

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Logistic Regression | 70.5% | 59.1% | 78.1% | 67.3% |
| Linear SVM | 70.4% | 58.9% | 78.1% | 67.2% |
| Random Forest | 67.9% | 55.9% | 82.8% | 66.7% |
| Naive Bayes | 69.8% | 58.8% | 74.2% | 65.6% |
| Decision Tree | 65.8% | 54.0% | 81.1% | 64.8% |
| Gradient Boosting | 73.4% | 67.2% | 61.6% | 64.3% |
| KNN | 71.7% | 63.7% | 62.8% | 63.3% |

I also ran 5-fold cross-validation on the training data. Random Forest (66.8% F1), Linear SVM (66.3%) and Logistic Regression (66.1%) came out on top.

Logistic Regression had the best test F1, Random Forest found the most good leads (83% recall) and Gradient Boosting had the best accuracy and precision. Which one is "best" depends on whether the sales team would rather catch every good lead or avoid wasting calls. The scores are moderate because the data only describes the lead, not what happened on the sales calls.

## Business recommendations

- Put more effort into high-quality sources like customer referrals, and review weak ones like the US website.
- Look at why "Other Locations" and Delivery Mode-5 convert so poorly.
- Study what Sales-Agent-9 does differently and share it with the team.
- Ask for a mobile number and a valid email on the lead form.
- Use the model to score new leads and call High Potential ones first.

## Tech used
- Python
- Pandas, NumPy
- SQLAlchemy and PyMySQL
- Scikit-learn
- Seaborn, Matplotlib
- Jupyter Notebook

## Possible next steps
- Save the data as a CSV in the repo so anyone can run the notebook without a database connection
- Tune the models with GridSearchCV instead of hand-picked settings
- Add call and follow-up details, since lead details alone cap the accuracy
