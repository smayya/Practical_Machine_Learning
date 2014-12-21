---
title: "Qualitative Activity Recognition of Weight Lifting Exercises"
author: "smayya"
date: "Sunday, December 21, 2014"
output: html_document
---
###Prepare Training Data

####1.0 Read training data set
```{r, eval=FALSE}
data_raw <- read.csv("pml-training.csv")
```
####1.1 Explore the dataset
```{r, eval=FALSE}
str(data_raw)
```
Clearly there are many columns with missing/NA data that need to be removed.

####1.2 Data cleanup

1.2.1 Select only numeric data
```{r, eval=FALSE}
numIndex <- sapply(data_raw, is.numeric)
data_clean1 <- data_raw[,numIndex]
```
1.2.2  Remove columns with NA
```{r, eval=FALSE}
data_clean2 <- data_clean1[ , colSums(is.na(data_clean1)) == 0]
```
1.2.3  Timestamps, user names are irrelevent to the problem and are removed
```{r, eval=FALSE}
training <- subset(data_clean2, select = -c(raw_timestamp_part_1, raw_timestamp_part_2,num_window, X))
```
1.2.4  Now add the outcome variable "classe"
```{r, eval=FALSE}
training$classe <- data_raw$classe
```

###Prepare Testing Data

####2.0 Read test data set

```{r, eval=FALSE}
data_raw <- read.csv("pml-testing.csv")
```
####2.1 Data cleanup

2.1.1 Select only numeric data

```{r, eval=FALSE}
numIndex <- sapply(data_raw, is.numeric)
data_clean1 <- data_raw[,numIndex]
```
2.1.2 Remove columns with NA
```{r, eval=FALSE}
data_clean2 <- data_clean1[ , colSums(is.na(data_clean1)) == 0]
```
2.1.3 Timestamps, user names, problem ID are irrelevent to the problem and are removed
```{r, eval=FALSE}
testing <- subset(data_clean2, select = -c(raw_timestamp_part_1, raw_timestamp_part_2,num_window, X, problem_id))
```
Now both training and test set have same predictors, 52 in number.


###3.0 Model Building

We choose random forests as they are known to perform very well.

We set the number of trees to 10 and we use 10 fold cross validation to set an appropriate value for mtry (number of variables sampled as candidates at each split).

```{r, eval=FALSE}
model <- train(classe ~ .,data = training, method="rf", importance = TRUE, proximity = FALSE, ntree = 10, keep.forest = TRUE, trControl=trainControl(method="cv",number=10,verboseIter=TRUE))
```

####3.1. The generated model:

```{r, eval=FALSE}
> model

Random Forest 

19622 samples
   52 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

No pre-processing
Resampling: Cross-Validated (10 fold) 

Summary of sample sizes: 17659, 17660, 17659, 17661, 17660, 17661, ... 

Resampling results across tuning parameters:

  mtry  Accuracy   Kappa      Accuracy SD  Kappa SD   
   2    0.9871565  0.9837526  0.002884864  0.003649500
  27    0.9919475  0.9898131  0.001609753  0.002036827
  52    0.9870555  0.9836242  0.001403482  0.001774887

Accuracy was used to select the optimal model using  the largest value.
The final value used for the model was mtry = 27.
```

####3.2 The final Model:
```{r, eval=FALSE}
> model$finalModel

Call:
 randomForest(x = x, y = y, ntree = 10, mtry = param$mtry, importance = TRUE,      proximity = FALSE, keep.forest = TRUE) 
               Type of random forest: classification
                     Number of trees: 10
No. of variables tried at each split: 27

        OOB estimate of  error rate: 2.39%
Confusion matrix:
     A    B    C    D    E class.error
A 5467   29   15    7    5  0.01013942
B   49 3620   45   27   19  0.03723404
C   11   55 3281   36    7  0.03215339
D    7   15   69 3075   14  0.03301887
E   10   15   11   18 3520  0.01510912
```


####3.2 Out-of-Sample Error Estimate

As can be seen, the estimated out of sample errors from the above model: 2.39%


###4.0 Applying the model on the test data

```{r, eval=FALSE}
predict(modelfit, newdata=testing)
 1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
 B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
Levels: A B C D E
```
