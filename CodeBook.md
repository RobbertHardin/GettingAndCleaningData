## 
## run_analysis.R
## 

## 
# Download the data sets
## 

fileURL = "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
fileDest = "./UCIHAR.zip"
if ( file.exists( fileDest )  ) {
    print( "File already exists !" )
} else {
    download.file(fileURL, destfile = fileDest, mode = "wb")
    downloadedAt <- Sys.time()
    print( paste0( "Downloaded file at ", downloadedAt ) )
}

## 
# Move the relevant data sets to the working directory
##

# Activity related files
file.copy("./UCI HAR Dataset/activity_labels.txt", "./activity_labels.txt")
file.copy("./UCI HAR Dataset/train/y_train.txt", "./y_train.txt")
file.copy("./UCI HAR Dataset/test/y_test.txt", "./y_test.txt")

# Sensor related files
file.copy("./UCI HAR Dataset/features.txt", "./features.txt")
file.copy("./UCI HAR Dataset/train/X_train.txt", "./X_train.txt")
file.copy("./UCI HAR Dataset/test/X_test.txt", "./X_test.txt")

# Subject related files
file.copy("./UCI HAR Dataset/train/subject_train.txt", "./subject_train.txt")
file.copy("./UCI HAR Dataset/test/subject_test.txt", "./subject_test.txt")

# Verify all files are present
dir(pattern = ".txt")


## 
# Read data files
##

# Activity related files
activityLabels <- read.table("./activity_labels.txt", as.is = TRUE)
activityTrain  <- read.table("./y_train.txt", as.is = TRUE)
activityTest   <- read.table("./y_test.txt", as.is = TRUE)

# Sensor related files
features    <- read.table("./features.txt", as.is = TRUE)
sensorTrain <- read.table("./X_train.txt", as.is = TRUE)
sensorTest  <- read.table("./X_test.txt", as.is = TRUE)

# Subject related files
subjectTrain <- read.table("./subject_train.txt", as.is = TRUE)
subjectTest  <- read.table("./subject_test.txt", as.is = TRUE)


## 
# Explore files
##

allFiles <- c("activityLabels", "activityTrain", "activityTest", "features", "featureTrain", "featureTest", "subjectTrain", "subjectTest")
sapply(allFiles, class)
dim(activityLabels)
dim(activityTrain)
dim(activityTest)
dim(features)
dim(sensorTrain)
dim(sensorTest)
dim(subjectTrain)
dim(subjectTest)

##
# Merge training and test data
## 

allActivityData <- rbind(activityTrain, activityTest)
allSensorData   <- rbind(sensorTrain, sensorTest)
allSubjectData  <- rbind(subjectTrain, subjectTest)

# Clean up: remove training and test data
rm(list = "allFiles")
rm(list = cbind("activityTrain", "activityTest", "sensorTrain", "sensorTest", "subjectTrain", "subjectTest"))

## 
# Extract only mean and standard deviation from allSensorData
##

# Grep only features containing mean() or std(). Ignore meanFreq and angle !
indexMeanStd <- grep("mean[(]|std[(]", features[,2])
# Restict allSensorData to these variable
allSensorData <- allSensorData[, indexMeanStd]

##
# Change activity from number to text
##

allActivityDataText <- as.data.frame(sapply(allActivityData, function(act){activityLabels[act, 2]}))
rm(list = "allActivityData")

## 
# Apply descriptive names to vars
## 

# Replacing names of allSubjectData
names(allSubjectData) <- "Subjects"

# Replacing names of allActivityDataText
names(allActivityDataText) <- "Activities"


# Replacing names of allSensorData
names(allSensorData) <- features[indexMeanStd, 2]
# Note there are mistakes in the names
# Correcting BodyBody --> Body
names(allSensorData) <- gsub("BodyBody", "Body", names(allSensorData))
# Make names more readable
# Remove ()
names(allSensorData) <- gsub("[(][)]", "", names(allSensorData))
# Replace mean by Mean
names(allSensorData) <- gsub("mean", "Mean", names(allSensorData))
# Replace std by StD
names(allSensorData) <- gsub("std", "StD", names(allSensorData))
# Remove -
names(allSensorData) <- gsub("-", "", names(allSensorData))


##
# Merge all data sets and clean up
##

allData <- cbind(allSubjectData, allActivityDataText, allSensorData)
rm(list = cbind("allSubjectData", "allActivityDataText", "allSensorData"))

## 
# Create tidy data set of allData
## 

# (Re)Install package tidyr and add it to the library
install.packages("tidyr")
library(tidyr)

# (Re)Install package dplyr and add it to the library
install.packages("dplyr")
library(dplyr)

allDataTidied <- gather(allData, feature, signal, -(Subjects:Activities))
allDataTidiedBySubjectsActivities <- group_by(allDataTidied, Subjects, Activities)
MeanBySubjectsActivities <- summarise(allDataTidiedBySubjectsActivities, SignalMean = mean(signal))
write.table(MeanBySubjectsActivities, file = "MeanBySubjectsActivities.txt", row.name = FALSE)
