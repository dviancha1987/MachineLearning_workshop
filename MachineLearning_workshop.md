---
title: "Machine Learning Workshop"
author: "Daniel Viancha"
date: "Sunday, May 21, 2017"
output: html_document
---

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: <http://groupware.les.inf.puc-rio.br/har> (see the section on the Weight Lifting Exercise Dataset).

## Loading Packages and data

We need to install the packages that are goint to be used in this worshop, those are:

```{r}
library(caret)
library(randomForest)
library(rpart)
library(rpart.plot)
library(RColorBrewer)
library(rattle)
library(e1071)
```

## Loading data

```{r}
# Read the Training CSV file into R & replace missing values & excel division error strings #DIV/0! with 'NA'
train_data <- read.csv("C:/Users/Daniel/Desktop/Coursera/Machine_Learning/pml-training.csv", na.strings=c("NA","#DIV/0!",""), header=TRUE)

# Read the Testing CSV file into R & replace missing values & excel division error strings #DIV/0! with 'NA'
test_data <- read.csv("C:/Users/Daniel/Desktop/Coursera/Machine_Learning/pml-testing.csv", na.strings=c("NA","#DIV/0!",""), header=TRUE)

# Take a look at the Training data classe variable
summary(train_data$classe)

```

##Partitioning the data for Cross-validation

The training data is split into two data sets, one for training the model and one for testing the performance of our model. The data is partitioned by the classe variable, which is the varible we will be predicting. The data is split into 60% for training and 40% for testing.

```{r}
inTrain <- createDataPartition(y=train_data$classe, p = 0.60, list=FALSE)
training <- train_data[inTrain,]
testing <- train_data[-inTrain,]

dim(training); dim(testing)
```

## Data Processing

Drop the first 7 variables because these are made up of metadata that would cause the model to perform poorly.

```{r}
training <- training[,-c(1:7)]
```

There are a lot of variables where most of the values are ‘NA’. Drop variables that have 60% or more of the values as ‘NA’.

```{r}
training_clean <- training
for(i in 1:length(training)) {
  if( sum( is.na( training[, i] ) ) /nrow(training) >= .6) {
    for(j in 1:length(training_clean)) {
      if( length( grep(names(training[i]), names(training_clean)[j]) ) == 1)  {
        training_clean <- training_clean[ , -j]
      }   
    } 
  }
}

# Set the new cleaned up dataset back to the old dataset name
training <- training_clean
```

Transform the test_data dataset

```{r}
# Get the column names in the training dataset
columns <- colnames(training)
# Drop the class variable
columns2 <- colnames(training[, -53])
# Subset the test data on the variables that are in the training data set
test_data <- test_data[columns2]
dim(test_data)
```

## CROSS VALIDATION: Prediction with random forest

A Random Forest model is built on the training set. Then the results are evaluated on the test set
```{r}
set.seed(54321)
modFit <- randomForest(classe ~ ., data=training)
prediction <- predict(modFit, testing)
cm <- confusionMatrix(prediction, testing$classe)
print(cm)
```

```{r}
overall.accuracy <- round(cm$overall['Accuracy'] * 100, 2)
sam.err <- round(1 - cm$overall['Accuracy'],2)
```

The model is 99.39% accurate on the testing data partitioned from the training data. The expected out of sample error is roughly 0.01%.

```{r}
plot(modFit)
```

In the above figure, error rates of the model are plotted over 500 trees. The error rate is less than 0.04 for all 5 classe.

## CROSS VALIDATION: Prediction with a desicion tree

```{r}
set.seed(54321)
modFit2 <- rpart(classe ~ ., data=training, method="class")
prediction2 <- predict(modFit2, testing, type="class")
cm2 <- confusionMatrix(prediction2, testing$classe)
print(cm2)
```

```{r}
overall.accuracy2 <- round(cm2$overall['Accuracy'] * 100, 2)
sam.err2 <- round(1 - cm2$overall['Accuracy'],2)
``` 

The model is 75.29% accurate on the testing data partitioned from the training data. The expected out of sample error is roughly 0.26%.

Plot the decision tree model

```{r}
fancyRpartPlot(modFit2)
``` 

## Conclusion

There are many different machine learning algorithms. I chose to compare a Random Forest and Decision Tree model. For this data, the Random Forest proved to be a more accurate way to predict the manner in which the exercise was done.
