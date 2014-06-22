GetCleanDataRepo
================

Project for Getting and Cleaning Data 
================

README for "run_analysis.R"
===


The following highlights how the "run_analysis.R" script works. 

The "run_analysis.R" script includes a brief description of its scope and task. Note that these are only comments behind "##".

```
#### Run Analysis ####
## This program collects, transforms, and cleans the UCI HAR Dataset.

## The following instructions were performed for the project:
## 1. Merges the training and the test sets to create one data set.
## 2. Extracts only the measurements on the mean and standard deviation for each measurement. 
## 3. Uses descriptive activity names to name the activities in the data set
## 4. Appropriately labels the data set with descriptive activity names. 
## 5. Creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```

Since the reshape2 package is required for melt and dcast, which are used later in the script, the package is loaded. Note that to reproduce this process the reshape2 package must be installed.

```
## Load requires packages
require(reshape2)
```

The working directory is set and objects to help in locating data folders and files are created. Note that to reproduce this process in a different computer the mainwd object must be changed accordingly.

```
## Set wd and set object to help in loading files
## Note this assumes the .zip has been donwloaded to wd. 
mainwd <- "C:/Users/Nahir/Documents/CleaningData"
setwd(mainwd)

rawdata <- "UCI HAR Dataset"
train <- "train"
test <- "test"
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
```

The .zip files is downloaded to the computer. 

```
## Originally donwloaded on 4/26/2014
download.file(url, file.path(mainwd,paste(rawdata,".zip",sep="")))

unzip(file.path(mainwd,paste(rawdata,".zip",sep="")))
```

The following section shows the reading of the different data tables.

```
## Load test files
filename <- file.path(mainwd,rawdata,test,"X_test.txt") 
x_test <- read.table(filename)

filename <- file.path(mainwd,rawdata,test,"y_test.txt") 
y_test <- read.table(filename)

filename <- file.path(mainwd,rawdata,test,"subject_test.txt") 
subject_test <- read.table(filename)

## Load train files
filename <- file.path(mainwd,rawdata,train,"X_train.txt") 
x_train <- read.table(filename)

filename <- file.path(mainwd,rawdata,train,"y_train.txt") 
y_train <- read.table(filename)

filename <- file.path(mainwd,rawdata,train,"subject_train.txt") 
subject_train <- read.table(filename)

## Load features
filename <- file.path(mainwd,rawdata,"features.txt") 
features <- read.table(filename)

## activity_labels
filename <- file.path(mainwd,rawdata,"activity_labels.txt") 
activity_labels <- read.table(filename)
```

The following section shows how the column names of the x_ data tables are set according to the "features.txt" data table. After this, the variables which to not include "mean()" or "std()" are excluded from the x_ data tables. Note that fixed is set equal to TRUE, indicating that pattern is a string to be matched as is.

Also, the column names for y_ data table is set to "activity" while the column names for subject_ is set to "subject"

```
## assign colnames, in the case of x_test and x_train also subset desired columns, those with "mean()" and "std()"
colnames(x_test) <- features[,c("V2")]
colnames(x_train) <- features[,c("V2")]

## Note that this relates to:
## 2. Extracts only the measurements on the mean and standard deviation for each measurement. 
x_test <- x_test[,(grepl("mean()", colnames(x_test),fixed = TRUE) | grepl("std()", colnames(x_test),fixed = TRUE))]
x_train <- x_train[,(grepl("mean()", colnames(x_train),fixed = TRUE) | grepl("std()", colnames(x_train),fixed = TRUE))]

colnames(y_test) <- "activity"
colnames(y_train) <- "activity"

colnames(subject_test) <- "subject"
colnames(subject_train) <- "subject"
```

The following section highlights the merging process which is performed using column bind, at he _test and _train level, while row bind is used to combines test and train together.

```
## Merge test data
testdb <- cbind(y_test,subject_test,x_test)

## Merge train data
traindb <- cbind(y_train,subject_train,x_train)

## Merge test and train data, note that this, and the prior merges relate to:
## 1. Merges the training and the test sets to create one data set.
db <- rbind(testdb,traindb)
```

The following section highlights how factors are assigned to activity.

```
## assign factors to activity, this relates to:
## 3. Uses descriptive activity names to name the activities in the data set
## 4. Appropriately labels the data set with descriptive activity names. 
db$activity <- factor(db$activity,levels = 1:6,labels = activity_labels$V2)

```

The following section highlights how the data is transformed using melt and dcast from the reshape2 package.

```
## Second tidy data set, this relates to:
## 5.Creates a second, independent tidy data set with the average of each variable for each activity and each subject. 
dbmelted <- melt(db,id.vars = c("activity","subject"))
dbcasted <- dcast(dbmelted, activity + subject ~ variable,mean)
```


The last section shows how the output is created. Note that the output is named "tidydataoutput.txt" but formatted like a .csv.

```
## Write Results
tidydataoutput <- dbcasted
write.csv(tidydataoutput, file = "tidydataoutput.txt",row.names = FALSE)
```
