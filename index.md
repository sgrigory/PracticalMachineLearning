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
## glm 0.4880513 2.678048e-01 2.113195e-01 2.602001e-03 3.022239e-02
## gbm 0.9999525 4.242473e-05 4.863914e-06 1.815484e-07 1.168302e-08
## rf  0.9960000 4.000000e-03 0.000000e+00 0.000000e+00 0.000000e+00
## svm 0.9542269 2.597133e-02 6.230547e-03 3.155364e-03 1.041584e-02
```

We then average these probabilities with the weights proportional to the overall accuracy of each model on the test set. The final result is produced by choosing the outcome with maximum probability for each point in the test set. This procedure is repeated for each of the six participants.

##Cross validation

Every fit is accompanied by 10-fold cross-validation, repeated 5 times. The parameters of cross-validation are
passed to the *predict* method through the *trControl* parameter. The cross-validated accuracy was calculated for each participant and for each model:

```
##        adelmo  carlitos   charles    eurico    jeremy     pedro      mean
## glm 0.7881956 0.8860678 0.9218440 0.9147824 0.8321322 0.8794561 0.8704130
## gbm 0.9900344 0.9917017 0.9959923 0.9930905 0.9915917 0.9873455 0.9916260
## rf  0.9867180 0.9953934 0.9935885 0.9935458 0.9877967 0.9906448 0.9912812
## svm 0.7667846 0.8362501 0.8505015 0.9138794 0.8384411 0.8062734 0.8353550
```
We expect the accuracy of our final outcome, averaged over the four models, to be close to that of the most efficient model here, i.e. GBM. On the test sample we have the accuracy is 0.9940517.

Here is how the confusion matrix on the test set looks like:
![](index_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

We can also look at the relative importance of variables. For example, in the most efficient model, GBM and for the first participant the most important variables are:

```
## gbm variable importance
## 
##   only 20 most important variables shown (out of 52)
## 
##                     Overall
## roll_belt           100.000
## pitch_belt           56.416
## yaw_belt             49.020
## yaw_arm              25.355
## magnet_dumbbell_z    13.696
## magnet_forearm_x     11.714
## magnet_belt_x        11.448
## magnet_arm_z          7.494
## magnet_belt_z         4.831
## total_accel_forearm   4.670
## magnet_dumbbell_y     4.388
## roll_arm              3.875
## accel_belt_z          2.262
## magnet_belt_y         2.013
## pitch_arm             1.984
## accel_belt_x          1.820
## accel_arm_y           1.681
## magnet_arm_y          1.598
## total_accel_arm       1.228
## roll_dumbbell         1.152
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



We have found that our prediction's accuracy on the quiz dataset is 100%.


