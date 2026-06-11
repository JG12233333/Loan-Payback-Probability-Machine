
# Loan-Payback-Probability-Machine
A loan risk model that scores 600K+ borrowers on likelihood of repayment. Helps lenders focus on safer approvals and reduce losses without rejecting good customers. Strong out-of-sample performance on held-out validation.

# Start to Finish 
The goal of this project was to predict the probability that a borrower will pay back their loan. It is a workflow that:

A. Takes the loan application data (income, credit, loan terms, borrower attributtes)
B. Trains a model to estimate the repayment probability 
C. Evaluates whether those probabilities are useful and well-behaved
D. Outputs scored predictions for new applications 

The performance of this tool is going to all depend on how you configure the training, and what your data architecture looks like: Mainly, this means the time_limit and presets(medium_quality, high_quality, best_quality). More time and higher quality is obviously going to generate stronger models; there is no single fixed outcome. 

# The pipeline 
The model is suppose to read application features and return a repayment probability for each row. Those scores can be used for ranking, reporting, or downstream decision rules. Just remember that the core task is probability estimation, it is not a built in yes/no rule. 

The pipeline itself works through: 

A. Loading training data (with known outcomes) and test data (features only) 
B. Trains an ensemble model with AutoGluon, optimized for ROC AUC.
C. Saves the model to ag_models/ for reuse 
D. Writes a file with the ID and the predicted loan_pay_back probability

# Instructions
loanrisk.zip is where the project, and its datasets are at, please make sure to extract the files to view the project. Furthermore, when it comes to RAM and CPU, please be sure, after you've ran the main notebook, to delete the ag_models folder, otherwise it will take forever to load the project if you plan on extensively reusing it, and regenerating models. If anyone has suggestions to make on how this project could be best suited for their needs, please email jgoetz3@live.spcollege.edu. 

# Important notes
time_limit = a cap on total training time in seconds
presets = model depth controls and ensemble strength
ROC = Reciever Operating Characteristic Curve\
AUC = Area Under Curve

# Why did this project depend its success on the ROC AUC? 
The project, once again, predicts the probability that a borrower will repay their loan. Success is going to depend on whether those probabilities rank borrowers correctly: repayers should score higher than defaulters. ROC AUC was the most useful metric because it evaluates that ranking solution without forcing a strict yes/no cutoff, stays meaningful when most loans are repaid, and matches both the model output (probabilities) and the validation approach (deciles, seperation, calibration). Model training was optimized for the ROC AUC so the ensemble was selected for the same standard used to judge the final scores. 

# How can we determine whether or not the model is the correct fit? 
The case that the model works rests on four layers of evidence. Distribution checks show the pipeline is stable and discriminative. ROC AUC on held-out data shows probabilities rank outcomes correctly. Log loss and calibration show those values mean something as probabilities. Decile separation makes that ranking visible meaning low-scored borrowers repay less often than high-scored ones. All of this is configuration-dependent; change training time or preset and re-run the layers. The decile plot is the first place to look when something fails: it turns abstract metrics into a pattern anyone can read.

# Imagery and Explanation: 
## Decile Validation Plot
<img width="889" height="490" alt="image" src="https://github.com/user-attachments/assets/09ad2a46-ebba-498f-90e7-ceae6983a2f5" />

This plot showcases whether or not the predicted repayment probabilities rank borrowers correctly. Borrowers are sorted by the model's predicted repayment probability and split into 10 equal groups (deciles): 
A. Decile 1 = lowest 10% of scores (model sees them as high risk) 
B. Decile 10 = highest 10% of scores (models sees them as low risk) 

For each group, the bar height is the actual payback rate (the share who really did payback). The red dashed line at 80% is the overall payback rate across everyone in the validation set. 

To better undersand the measurements: 
A. Decile 1 had an actual payback rate of 2%, which claims that the lowest scored borrowers almost never paid back their loans. 
B. Decile 2 had an actual payback rate of 50%, which is still below average.
C. Decile 3 had an actual payback rate of 77%, which is slightly below the 80% baseline.
D. Decile 4-6 had an acutal payback rate of 87% - 95%, which indicates above average claims.
E. Decile 7-8 had an actual payback rate of 98% - 99%, which means there was a very high payback rate in this decile.
F. Decile 9-10 had an actual payback rate of 100%, in which the model claims that this is the highest-scored group in the entirety of the dataset. 

## The policy simulation plot

<img width="889" height="489" alt="image" src="https://github.com/user-attachments/assets/75e22c6b-4af6-45b7-8dd8-750b4e21db8b" />

This plot is suppose to show what happens if you use the model's repayment probabilities to remove the lowest-scored borrowers from the pool, and what you gain versus what you give up on. 

The plot sorts borrowers by predicted repayment probability (lowest=riskiest). The x-axis is what share of applicants you are suppose to remove, starting from the riskiest:
A. 5% declined = drop the bottom 5% by score 
B. 10% declined = drop the bottom 10% by score
C. ...Up to 50% declined

The blue line is suppose to indicate the defaults avoided of all actual defaults in the data, and what % fell in the declined (riskiest) group. 
The red line is suppose to indicate the good borrwers lost of all the borrowers who actually paid back and what % fell in the declined group.

The main takeaway here is that its a tradeoff curve: A stricter cutoff = more defaults caught, but also more good borrowers removed. 

What the lines do
Blue (defaults avoided) rises quickly, then levels off:

~5% declined → ~25% of defaults avoided
~10% declined → ~49% of defaults avoided
~20% declined → ~74% of defaults avoided
~50% declined → ~96% of defaults avoided
Defaults concentrate in the low-probability tail — declining that tail removes a large share of bad outcomes early.

Red (good borrowers lost) stays low at first, then climbs:

~10% declined → almost 0% good borrowers lost
~15% declined → ~3% good borrowers lost
~30% declined → ~16% good borrowers lost
~50% declined → ~38% good borrowers lost
The model’s riskiest slice is mostly defaulters, not repayers.

## The score seperatation plot

<img width="889" height="490" alt="image" src="https://github.com/user-attachments/assets/89f36eb0-c2b0-422b-a31b-df0b6787921a" />

What the chart shows:
Defaulters concentrate at low predicted repayment probabilities (peak near 0), while repayers concentrate at high probabilities (peak near 1), showing the model assigns clearly different scores to the two groups.

For further reference, think of it like this: 
The score separation plot compares two groups: borrowers who defaulted (red) and borrowers who paid back (blue). Each person’s position on the x-axis is the model’s predicted probability of repayment.

Defaulters are concentrated on the left side of the chart. Many receive very low repayment probabilities, shown by a sharp peak near 0.0, which means the model often identifies them as high risk. A thinner tail extends toward the right, showing that some defaulters still receive moderate or high scores.

Repayers show the opposite pattern on the right. Their distribution builds strongly from about 0.8 upward and peaks near 1.0, meaning the model frequently assigns them very high repayment probabilities. There is very little blue below 0.8, so borrowers who actually paid back rarely receive low scores.

Where the two distributions overlap tells you how well the model separates the groups. On the left (roughly 0–0.3), the chart is mostly red, so the model correctly flags high default risk. On the right (0.8–1.0), it is mostly blue, so it correctly identifies likely repayers. In the middle, where the colors blend, both groups appear together—these are harder cases where the model is less certain. The main area of error is the small red bump on the right: defaulters who were scored as likely to pay back. Even so, most of the separation happens at the extremes.

This chart matters because it is a direct visual check that the repayment probabilities separate the two outcomes. A weak model would show both curves piled on top of each other around the overall payback rate (~0.8). Instead, defaulters cluster at low probabilities and repayers cluster at high ones. That supports the same conclusion as ROC AUC and the decile plot: the model’s scores carry real signal, not arbitrary numbers.

## The ROC curve on held-out validation data:

<img width="590" height="590" alt="image" src="https://github.com/user-attachments/assets/bb88561f-2651-48dc-8bd4-d1bafd2a4931" />

The ROC curve evaluates the model on held-out validation data by testing every possible probability cutoff. The teal line rises steeply toward the top-left, meaning the model achieves a high true positive rate while keeping the false positive rate low. An AUC of 0.933 indicates strong ranking performance, thus repayers consistently receive higher predicted repayment probabilities than defaulters, which is far above random guessing (0.50). This supports the same conclusion as the decile and separation plots: the model’s probability scores carry real discriminative signal.

## The model calibration curve

<img width="590" height="590" alt="image" src="https://github.com/user-attachments/assets/6e8da4d4-7a09-4cf9-8c1e-dde9da20681c" />


The calibration curve tests whether predicted repayment probabilities match observed outcomes on held-out validation data. The model’s line follows the diagonal almost perfectly: when the model assigns a given probability of payback, the actual repayment rate in that group is close to the same value. That means the scores are not only useful for ranking borrowers, as shown by the ROC curve and decile plot, but also trustworthy as probability estimates. A lender or analyst using these numbers directly for example, estimating expected repayment at a given score can treat them as reflecting reality rather than arbitrary labels.

# Data Information: 

This is a tabular binary classification dataset for predicting whether a borrower repays their loan: the target is loan_paid_back, a float where 1.0 means repaid and 0.0 means not repaid, scored with ROC AUC on roughly 593,994 training rows and 254,569 test rows linked by integer id (train ids 0–593,993, test ids 593,994–848,562), with no missing values in the raw CSVs and a moderately imbalanced training target of about 79.9% repaid (474,494) versus 20.1% not repaid (119,500). Each row has 11 predictors: five numeric features (annual_income in USD, roughly $15k–$100k+; debt_to_income_ratio, e.g. 0.084 = 8.4%, roughly 0.04–0.37; credit_score, roughly 500–750+; loan_amount in USD, roughly $900–$30k+; and interest_rate in percent, roughly 9%–16%+, all non-negative) and six categorical features (gender: Female, Male, Other; marital_status: Single, Married, Divorced, Widowed; education_level: High School, Bachelor's, Master's, PhD, Other; employment_status: Employed, Self-employed, Unemployed, Retired, Student; loan_purpose: Debt consolidation, Home, Education, Vacation, Car, Medical, Business, Other; and grade_subgrade, 30 Lending Club–style ordered risk grades A1–A5 through F1–F5 where A is lowest risk and F is highest, often treated as ordinal or high-cardinality categorical). Test rows match train except loan_paid_back is omitted; for example, id 0 with annual_income 29367.99, debt_to_income_ratio 0.084, credit_score 736, loan_amount 2528.42, interest_rate 13.67, Female, Single, High School, Self-employed, Other, C3, and target 1.0. Submissions should output probabilities in [0, 1] for each test id rather than hard labels; tree models such as CatBoost, LightGBM, and XGBoost often work well with minimal encoding, though the data is synthetic and may not fully mirror real-world lending patterns.









