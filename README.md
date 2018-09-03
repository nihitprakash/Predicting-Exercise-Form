# PracML_Coursera
For the Practical Machine Learning Coursera Course
---
title: "Practical Maching Learning-Course Project"
author: "Nihit Prakash"
date: "09/02/2018"
output: html_document
---

##Background
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways

##Analysis Walk Through
The training and testing data was imported and converted to dataframe variables.
The first point explored was the distribution of our output variable. The distribution seems a bit skewed towards "A", but generally uniform. There is no strong class imbalance. As stated before, we have 5 different levels of outputs here: A through E. For multi-output classfication problems, tree based models will generally work well Hence, the analysis from this point on will be performed keeping that in mind. 

The classe column(the output variable) was then separated from the training dataset.

From further exploration of the dataset, it was seen that there were some columns unnecessary for our analysis like, names, the timestamp and v1 columns. These were removed. It was also straighaway observed that multiple predictors had NA's. Therefore, full NA columns were removed, and then any columns which had >95% missing values. NearZeroVariance predictors were also removed.

A corelation plot was built at this point to observe the correlations between our predictors. There were some predictors that were highly correlated with each other (postively and negatively). Principal Component Analysis could be performed at this point, to remove correlations and reduce number of predictors. However, since we have decided to use Tree-based models anyway, we don't need to perform this step, since Tree-based models usually take care of collinearity. 

Now, with our data munging complete, at this point we recombine the classe column with the training dataset. We then split out the training dataset into two sets: **training_new** and **validation** (80:20). We will use the training_new dataset for training our models, and the validation dataset to test our model to determine our out-of-sample accuracy. The model with the best accuracy here will be used to predict on the Test Dataset.  

Our first model will be a single decision tree, the **CART Model**. We use 10 fold cross validation to avoid any overfitting. From predicting on our validation set, we get an out-of-sample accuracy on the Validation set of **49.3%**

Our second model will be a **Random Forest Model**. With the same 10 fold cross validation, we observe a significant increase in out-of-sample accuracy to **99.8%**

Our third model will be a **Gradient Boosing Model**. Here, we get an accuracy of **98.93%**. Pretty good, but our Random Forest Model still perfomed better. 

Thus, the Random Forest Model was selected for predicting on the Test Set. 


Importing Packages
```{r Packages}

library(dplyr)
library(stringr)

library(ggplot2)
library(pROC)

library(caret)
library(data.table)
library(knitr)
library(e1071)
library(DMwR)
library(corrplot)
library(rattle)

```


```{r Importing Data}

training <- fread("C:/Users/33805/Desktop/Nihit/Coursera/Practical ML/pml-training.csv")

testing <- fread("C:/Users/33805/Desktop/Nihit/Coursera/Practical ML/pml-testing.csv")

```


```{r Initial exploration of Dataset}

str(training)

#converting to dataframes
training <- as.data.frame(training)
testing <- as.data.frame(testing)

#observing distribution of output variable, classe
table(training$classe)

#separating out the classe column
classe <- training["classe"]
training <- select(training, -classe)

#removing non-numeric columns
training <- select_if(training, is.numeric)

#further removing non essential columns (like v1, timestamp etc.)
training <- select(training, -(1:3))



```


```{r Dealing with Missing Values for training set}

#removing complete NA columns
fullna <- names(training[colSums(!is.na(training))==0])

training <- select(training, -one_of(fullna))

#removing columns with more than 95% NA's
mostna <- names(training[colSums(is.na(training))>(0.95*nrow(training))])

training <- select(training, -one_of(mostna))

#removing near zero variance columns
nzvar <- nearZeroVar(training, names=TRUE)

training <- select(training, -one_of(nzvar))

#testing for any more missing values
which(is.na(training))
#no further missing values

```


```{r Dealing with Missing Values for testing set}

#removing complete NA columns
fullna <- names(testing[colSums(!is.na(testing))==0])

testing <- select(testing, -one_of(fullna))

#removing columns with more than 95% NA's
mostna <- names(testing[colSums(is.na(testing))>(0.95*nrow(testing))])

testing <- select(testing, -one_of(mostna))

#removing near zero variance columns
nzvar <- nearZeroVar(testing, names=TRUE)

testing <- select(testing, -one_of(nzvar))

#testing for any more missing values
which(is.na(testing))
#no further missing values

```


```{r Exploring Correlations in the Training Dataset}

correlation <- cor(training)

corrplot(correlation, order="hclust")

#joining the classe column back to the training set
training <- cbind(training, classe)
```


```{r Splitting training further into training_new and validation datasets}

set.seed(1243)
intrain <- createDataPartition(training$classe, p=0.80, list=FALSE)

training_new <- training[intrain,]
validation <- training[-intrain,]

```


```{r CART Model}

#defining Cross Validation Parameters
trainctr <- trainControl(method = "cv", number = 10)

#training CART model
cartmodel <- train(as.factor(classe)~.,training_new ,method="rpart", trControl = trainctr)

#printing the CART model
print(cartmodel$finalModel)
fancyRpartPlot(cartmodel$finalModel)

#predicting on the Validation Set
predres_cart <- predict(cartmodel, validation)
confmat_cart <- confusionMatrix(predres_cart,as.factor(validation$classe))

#CART model has accuracy of 49.3%


```


```{r Random Forest Model}

#defining Cross Validation Parameters
trainctr <- trainControl(method = "cv", number = 10)

#training CART model
rfmodel <- train(as.factor(classe)~.,training_new ,method="rf", trControl = trainctr)

print(rfmodel)
print(rfmodel$finalModel)
varImp(rfmodel)

predres_rf <- predict(rfmodel, validation)
confmat_rf <- confusionMatrix(predres_rf,as.factor(validation$classe))

#Random Forest Model has accuracy of 99.8%

```


```{r Gradient Boosting Model}

#defining Cross Validation Parameters
trainctr <- trainControl(method = "cv", number = 5)

#training CART model
boostmodel <- train(as.factor(classe)~.,training_new ,method="gbm", trControl = trainctr, verbose = FALSE)

print(boostmodel)
print(boostmodel$finalModel)

predres_boost <- predict(boostmodel, validation)
confmat_boost <- confusionMatrix(predres_boost,as.factor(validation$classe))

#Boosting Model has accuracy of 98.93%
```


```{r Final Result}

#predicting on Test Set using the Random Forest Model
predres_rf_test <- predict(rfmodel, testing)

#generating the Final Result dataframe
finalresult <- cbind(testing["problem_id"], predres_rf_test)

```

