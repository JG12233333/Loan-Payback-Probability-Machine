# Loan-Payback-Probability-Machine
A loan risk model that scores 600K+ borrowers on likelihood of repayment. Helps lenders focus on safer approvals and reduce losses without rejecting good customers. Strong out-of-sample performance on held-out validation.

# Start to Finish 
The goal of this project was to predict the probability that a borrower will pay back their loan. It is a workflow that:

A. Takes the loan application data (income, credit, loan terms, borrower attributtes)
B. Trains a model to estimate the repayment probability 
C. Evaluates whether those probabilities are useful and well-behaved
D. Outputs scored predictions for new applications 

The performance of this tool is going to all depend on how you configure the training: Mainly, this mean the time_limit and presets(medium_quality, high_quality, best_quality). More time and higher quality is obviously going to generate stronger models; there is no single fixed outcome. 

# The pipeline 
The model is suppose to read application features and return a repayment probability for each row. Those scores can be used for ranking, reporting, or downstream decision rules. Just remember that the core task is probability estimation, it is not a built in yes/no rule. 

The pipeline itself works through: 

A. Loading training data (with known outcomes) and test data (features only) 
B. Trains an ensemble model with AutoGluon, optimized for ROC AUC.
C. Saves the model to ag_models/ for reuse 
D. Writes a file with the ID and the predicted loan_pay_back probability

# Important notes
time_limit = a cap on total training time in seconds
presets = model depth controls and ensemble strength
ROC = Reciever Operating Characteristic Curve\
AUC = Area Under Curve

# Important instructions 
You must extract and unzip the files, they are quite large... 

# Why did this project depend its success on the ROC AUC? 
The project, once again, predicts the probability that a borrower will repay their loan. Success is going to depend on whether those probabilities rank borrowers correctly: repayers should score higher than defaulters. ROC AUC was the most useful metric because it evaluates that ranking solution without forcing a strict yes/no cutoff, stays meaningful when most loans are repaid, and mathces both the model output (probabilities) and the validation approach (deciles, seperation, calibration). Model training was optimized for the ROC AUC so the ensemble was selected for the same standard used to judge the final scores. 

# How can we determine whether or not the model is the correct fit? 
The case that the model works rests on four layers of evidence. Distribution checks show the pipeline is stable and discriminative. ROC AUC on held-out data shows probabilities rank outcomes correctly. Log loss and calibration show those values mean something as probabilities. Decile separation makes that ranking visible — low-scored borrowers repay less often than high-scored ones. All of this is configuration-dependent; change training time or preset and re-run the layers. The decile plot is the first place to look when something fails: it turns abstract metrics into a pattern anyone can read.

