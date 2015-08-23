##Getting and Cleaning Data
##Course 3 - Data Science
##Course Project
##Submitted 082315 by Curtis Davis

##loading libraries
library(dplyr)
library(data.table)

Loaded libraries necessary for data table manipulation

##identify the zip file
fileZip <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileZip, destfile="data.zip", method="wininet")
dateDownloaded <- date()

I download the file every time to insure that the data is available.  It may take longer, but it will be accurate.

##Unzip the file 
unzip("data.zip")

Unzipped into my working directory

##Creating R objects for test directory
X_test <- read.table("./UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./UCI HAR Dataset/test/y_test.txt")
S_test <- read.table("./UCI HAR Dataset/test/subject_test.txt")

Grouping the observations, the participants, and the tests into R objects

##Creating R objects for train directory
X_train <- read.table("./UCI HAR Dataset/train/X_train.txt")
y_train <- read.table("./UCI HAR Dataset/train/y_train.txt")
S_train <- read.table("./UCI HAR Dataset/train/subject_train.txt")

Grouping the observations, the participants, and the tests into R objects

##Labels
A_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")

R object containing activity names

##Getting Column labels
X_names <- read.table("./UCI HAR Dataset/features.txt")

Column labels

##Merge column labels with data
names(X_train) <- X_names$V2
names(X_test) <- X_names$V2

Assign labels to R objects

##Binding data frames with required columns
testDF <- data.frame(cbind(S_test,y_test,X_test))
trainDF <- data.frame(cbind(S_train,y_train,X_train))

Binding columns that indicate observations, participants, and associated tests

##Binding rows together to produce big data set
resultsDF <- data.frame(rbind(testDF, trainDF))

Combining test and train data

##Renaming Activity Rows
resultsDF$V1.1 <- A_labels$V2[resultsDF$V1.1]

Renaming rows to more descriptive names

##Find Mean and STD columns
colMean <- subset(X_names, grepl("\\mean\\>", X_names$V2))
colSTD <-subset(X_names, grepl("std()", X_names$V2))
colMeanSTD <- data.frame(rbind(colMean, colSTD))

Subset to find columns only with "mean" and "std" in the name

##Ordering by column number and position
colMeanSTD <- colMeanSTD[order(colMeanSTD$V1),]

##Applying offset for first two columns
colMeanSTD$V1 <- colMeanSTD$V1+2

Needed to move the column number down two to offset for the participant and test columns

##Summary data set - extracted Mean and STD columns
summaryData <- subset(resultsDF, select = (colMeanSTD$V1))
subjectNumber <- resultsDF$V1
activityName <- resultsDF$V1.1
summaryData <- cbind(subjectNumber,activityName,summaryData)

Putting the result back together

##Assigning descriptive variable names
names(summaryData) <- gsub("Acc", "Accelerator", names(summaryData))
names(summaryData) <- gsub("Mag", "Magnitude", names(summaryData))
names(summaryData) <- gsub("Gyro", "Gyroscope", names(summaryData))
names(summaryData) <- gsub("^t", "time", names(summaryData))
names(summaryData) <- gsub("^f", "frequency", names(summaryData))

Changed the names to be more descriptive in order to comply with the tidy data set requirements

##Arranging rows
summaryData <- arrange(summaryData, subjectNumber, activityName)

Sorting rows to create order

##Setting up tidyData
Tidy.dt <- data.table(summaryData)

Changed to a data table to use special commands

##This takes the mean of every column broken down by participants and activities
TidyData <- Tidy.dt[, lapply(.SD, mean), by = 'subjectNumber,activityName']

Used lapply with the columns of Tidy.dt to group by activity and participant

##Write to a text file for uploading
write.table(TidyData, file = "Tidy.txt", row.names = FALSE)

Writing to the text file as a final step.
