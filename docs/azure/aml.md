# Machine Learning

Machine Learning aims to use patterns in data to make estimates. There
are four main components to the learning process:

- Data: specifically features (inputs) and labels (expected output)
- Model: makes estimates from features. 
- Objective: what the model is trying to achieve
- Optimiser: used to improve the performance of the model

Models consist of logic and parameters that are used to transform the 
input into an estimate. Models are chosen based on their logic 
rather than the parameter values. Parameters are discovered during 
the training process. Training is achieved using a cost (objective)
function and an optimiser function. The cost function is used
to calculate how well a model performed. The optimiser function is
used to change the models parameters to improve the cost performance. 
A gradient-descent optimiser tracks the decrease in cost
as parameter values are changed and identifying when the cost has
been brought to it's lowest value. 

Crucial to the optimiser, is the step-size: how much a parameter value
is adjusted for the next iteration. If the step-size is too small, 
the optimiser can settle for a local minimal cost. If the step-size is
too large, the minimal cost can be missed entirely. Finding the optimal 
step-size requires experimentation. 

Machine Learning training can be either supervised or unsupervised. 
Supervised learning assesses a model's performance by comparing its 
estimates to the correct answer (labels). Unsupervised training only uses 
features. Because unsupervised training does not have access to 
expected output, it is often used in circumstances where no single 
correct answer exists. 

Once a Model has been trained, it can be saved and deployed. 

Model training is dependant on the data used in training. Ideally, this 
should be representative, free from errors and missing data points. A 
representative dataset contains sufficient values (samples) to represent 
the whole population. Errors in measurement values can also skew training 
results. Where data points are absent, models can be chosen that 
handle missing data, or the missing data can be substituted with 
reasonable values, or the records with missing data can be removed. 
