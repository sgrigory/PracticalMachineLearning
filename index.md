# Practical Machine Learning Course Assignment: Human Activity Recognition
Grigory Sizov  
6/11/2016  



## General description
Six participants of an experiment are asked to perform an excercise in 5 different ways (the *class* variable below): the correct one, denoted by "A" and 4 incorrect, denoted by "B"-"E".
Our goal is to construct a model which predicts the manner an excercise is preformed based on data from sensores fixed on users' arms, forearms, belts, and dumbbells.
Since the participants can be different in their physical characteristics and the manner of performing the excercises, we fit the model separately for each participant.




# Have a look at the data
We are provided with two files: "pml-training.csv", which will be used for training and testing, and "pml-testing.csv"
which will be used for predictions for the final quiz.
The quiz file contains 60 non-NA columns, of which 52 are the data from the sensors. Based on them we will be making our predictions.





To see what is going on let us plot a couple of variables against time:
![](index_files/figure-html/plot0-1.png)<!-- -->![](index_files/figure-html/plot0-2.png)<!-- -->

Here different color mark different classes of mistakes. Plots of most variables look like oscillations, but with range depending on the class. This means that the probability density of variables is different for different classes. Let us plot it: 
![](index_files/figure-html/plot2-1.png)<!-- -->

Indeed, we can see that, for example, large values of *roll_belt* almos always mean class "E", whereas values around 110 only have non-zero probabiliy for class "A". Hope our model will capture those patterns.













## Description of the fit
We split the data loaded from "pml-training.csv" into the training and testing sets: 70% for training and %30% for testing.
Then we fit the data on the training set with 4 different models: 

- multinomial regression, 
- generalized boosted regression, 
- random forest
- support vector model.

To this end, we use the *predict* function the *caret* package with "glm","gbm","rf", and "svmRadial" methods.
When called with an option 'type="prob"', it outputs the probabilities of different outcomes. We get a table of probabilities which looks like this: 





```
##             A            B            C            D            E
## glm 0.4345499 0.0408990129 5.049803e-01 1.904614e-05 1.955168e-02
## gbm 0.9998910 0.0000633604 4.534001e-05 1.411628e-07 1.116122e-07
## rf  0.9820000 0.0120000000 2.000000e-03 2.000000e-03 2.000000e-03
## svm 0.8406303 0.0825647063 1.722071e-02 1.013409e-02 4.945015e-02
```

We then average these probabilities with the weights proportional to the overall accuracy of each model on the test set. The final result is produced by choosing the outcome with maximum probability for each point in the test set. This procedure is repeated for each of the six participants.

##Cross validation

Every fit is accompanied by 10-fold cross-validation, repeated 5 times. The parameters of cross-validation are
passed to the *predict* method through the *trControl* parameter. The cross-validated accuracy was calculated for each participant and for each model:

```
##        adelmo  carlitos   charles    eurico    jeremy     pedro      mean
## glm 0.7959587 0.8920061 0.9254037 0.9297164 0.8485715 0.8566064 0.8747105
## gbm 0.9900535 0.9940992 0.9935516 0.9948449 0.9920302 0.9901171 0.9924494
## rf  0.9889521 0.9936479 0.9943561 0.9948467 0.9861577 0.9895568 0.9912529
## svm 0.7495444 0.8325694 0.8448115 0.9226831 0.8502416 0.8175960 0.8362410
```
We expect the accuracy of our final outcome, averaged over the four models, to be close to that of the most efficient model here, i.e. GBM. On the test sample we have the accuracy is 0.9932019.

Here is how the confusion matrix on the test set looks like:
![](index_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

We can also look at the relative importance of variables. For example, in the most efficient model, GBM and for the first participant the most important variables are:

```
## gbm variable importance
## 
##   only 20 most important variables shown (out of 52)
## 
##                      Overall
## roll_belt           100.0000
## pitch_belt           64.3756
## yaw_belt             61.7785
## yaw_arm              26.2486
## magnet_belt_x        15.2542
## magnet_dumbbell_z    11.7459
## magnet_forearm_x     11.7130
## magnet_arm_z         11.4785
## roll_arm              7.1000
## magnet_belt_z         6.7509
## magnet_dumbbell_y     6.6102
## accel_arm_y           3.7316
## total_accel_forearm   2.9366
## pitch_arm             2.3901
## magnet_dumbbell_x     2.1898
## magnet_belt_y         1.9755
## magnet_arm_y          1.3873
## roll_dumbbell         1.2925
## accel_belt_x          1.0273
## gyros_belt_z          0.8501
```
One can of course reduce the number of variables by dropping the factors lower in the list, but we did not do that because the cross-validated performance of the model seems to be good as it is.
# Predictions for the quiz dataset

After we have fitted and cross-validated our model, let us apply it to 20 data points from "pml-testing.csv".

As before, for each data point we will apply four models and average the probabilities. The output value is the one which maximizes the probability.
In this way we have obtained the following predictions for the quiz:

```
##  [1] "B" "A" "B" "A" "A" "E" "D" "B" "A" "A" "B" "C" "B" "A" "E" "E" "A"
## [18] "B" "B" "B"
```

In fact, the data points from the quiz set are a part of an open database available at

<http://groupware.les.inf.puc-rio.br/static/WLE/WearableComputing_weight_lifting_exercises_biceps_curl_variations.csv>

We have found that our prediction's accuracy on the quiz dataset is 1.


