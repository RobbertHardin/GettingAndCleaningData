# README.md
This is the Read me file.

## Data set
The data set is located at [https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip](https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip "Location of the data set"). It is based on the following publication [1].
I downloaded it with the following R-code:

```{r}
if ( file.exists( fileDest )  ) {   
    print( "File already exists !" )   
} else {   
    download.file( fileURL, destfile = fileDest, mode = "wb" )   
    downloadedAt <- Sys.time()   
    print( paste0( "Downloaded file at ", downloadedAt ) )   
} 
```

Please note the argument `mode = "wb"`. This is to make sure the zip-file is downloaded as a binary file.   
Next I unziped it by hand in the working directory of R-Studio. Resulting in a new folder `UCI HAR Dataset` containing the following folders and files:

```
~\activity_labels.txt
~\features.txt
~\features_info.txt
~\README.txt
~\test\
~\test\Inertial Signals\
~\test\Inertial Signals\body_acc_x_test.txt
~\test\Inertial Signals\body_acc_y_test.txt
~\test\Inertial Signals\body_acc_z_test.txt
~\test\Inertial Signals\body_gyro_x_test.txt
~\test\Inertial Signals\body_gyro_y_test.txt
~\test\Inertial Signals\body_gyro_z_test.txt
~\test\Inertial Signals\total_acc_x_test.txt
~\test\Inertial Signals\total_acc_y_test.txt
~\test\Inertial Signals\total_acc_z_test.txt
~\test\subject_test.txt
~\test\X_test.txt
~\test\y_test.txt
~\train\
~\train\Inertial Signals\
~\train\Inertial Signals\body_acc_x_train.txt
~\train\Inertial Signals\body_acc_y_train.txt
~\train\Inertial Signals\body_acc_z_train.txt
~\train\Inertial Signals\body_gyro_x_train.txt
~\train\Inertial Signals\body_gyro_y_train.txt
~\train\Inertial Signals\body_gyro_z_train.txt
~\train\Inertial Signals\total_acc_x_train.txt
~\train\Inertial Signals\total_acc_y_train.txt
~\train\Inertial Signals\total_acc_z_train.txt
~\train\subject_train.txt
~\train\X_train.txt
~\train\y_train.txt
```

### Getting a feel for the data
To get a feel for the data, I read the `README.txt`, the other text files in the same directory (`activity_labels.txt`, `features.txt` and `features_info.txt`) and I glimpsed at the data files. Not all files are relevant for the Course Project [2]. The relevant *data* files are the test and the training files with sensor data (`X_test.txt` and `X_train.txt`), activity data (`y_test.txt` and `y_train.txt`) and subject data (`subject_test.txt` and `subject_train.txt`). To know what you're looking at you need *label*  files (`activity_labels.txt`), a file with variable names (`features.txt`) and a codebook explaining the variable names (`features_info.txt`).

You can visualize their dependences in the following table: 

subjects      | activity_labels  | features
------------- | ---------------- | ---------
subject_train | y_train          | X_train
subject_test  | y_test           | X_test

## 1. Merge the training and the test sets to create one data set
After identifying the relevant files, I moved them to my working directory that also contains `run_analysis.R`. This was necessary because the Course Projects states *The code should have a file run_analysis.R in the main directory that can be run as long as the Samsung data is in your working directory.* It also ensured me all relevant files were at the same directory.  

```{r}
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
```  

Now that I verified all relevant files are present I will load them into R...   

```{r}
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
```

... and explore them a bit more thorough. Using functions such as `dim`, `head`, `names` and so on. Please note I use names that are more descriptive than `X` and `y`. This should make life easier.

I use `rbind` to append the test data to the training data:

```{r}
allActivityData <- rbind(activityTrain, activityTest)
allSensorData   <- rbind(sensorTrain, sensorTest)
allSubjectData  <- rbind(subjectTrain, subjectTest)
```

I might have done it the other way around, as long as I am consequent. Appending activity data and subject data is not a problem, because they are both one column wide. The sensor data however is 561 columns wide and appending is only allowed when both sets not only have the same number of columns, but also in the same order. Which is the case.

There are several ways to merge the data sets. `merge()` is one of them. But I preferred `rbind()` because it doesn't sort or reorder the data. Before I merge all data into one set, I first have to do step 2 and 3.

## 2. Extracts only the measurements on the mean and standard deviation for each measurement
The sensor data have 561 variables. According to the codebook (`features_info.txt`), you can recognise the mean and the standard deviation by `mean()` and `std()`. There are some (13) variables with 'meanFreq' in their names. And there are some (7) variables that have a mean relating to angles. These variables are frequency and angle related, not signal related. The hard part is not including or excluding them, the hard part is deciding to include or exclude them. According to **my** interpretation of the course project, they need to be excluded. This leaves me with only those variables with mean() and std() in their names. Resulting in `33 + 33 = 66` variables.

I used `grep` to get the variables with mean() or std() in their names. This resulted in an index vector (`indexMeanStd`) which I used to restrict `allSensorData`.

```{r}
# Grep only features containing mean() or std(). Ignore meanFreq and angle !
indexMeanStd <- grep("mean[(]|std[(]", features[,2])

# Restict allSensorData to these variable
allSensorData <- allSensorData[, indexMeanStd]
```

## 3. Uses descriptive activity names to name the activities in the data set
`allActivityData` is a data frame with one column, containing numbers in stead of activities. I used `sapply` with an anonymous function to create a matrix with activities spelled out in stead of numbered. With `as.data.frame()` I converted this matrix into a data frame:
```{r}
allActivityDataText <- as.data.frame(sapply(allActivityData, function(act){activityLabels[act, 2]}))
```

## 4. Appropriately labels the data set with descriptive variable names
Let's recap what we have so far: 

* All test data is appended to the training data;    
* allSubjectData is a data frame with numbers indicating subjects;    
* allActivityDataText is a data frame with activities speled out;
* allSensorData is a data frame with sensor data restricted to the mean and std variables.

And all of them have variable names of the type `V1`, `V2`, ... Let's fix that.

`allSubjectData` and `allActivityDataText` are easy:
```{r}
# Replacing names of allSubjectData
names(allSubjectData) <- "Subjects"

# Replacing names of allActivityDataText
names(allActivityDataText) <- "Activities"
```
allSensorData is more complicated. I use my index vector (`indexMeanStd`) to get the names from `features`. Unfortunately there are errors in these names; sometimes it says 'BodyBody' instead of 'Body'. To increase readability I remove the parenthesis and the dashes. And I change 'mean' to 'Mean' and 'std' into 'StD':

```{r}
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
```

I could have used more descriptive names, but this would mean manually creating a vector with names. This would not only clarify the names, it would also complicate the creation of the code book. In short, the added value of *more* descriptive names is small and the necessary effort is big. So that's why I did it like this. I hope you understand.

Now I can finally put everything together in one data set:
```{r}
allData <- cbind(allSubjectData, allActivityDataText, allSensorData)

rm(list = cbind("allSubjectData", "allActivityDataText", "allSensorData"))
```

When you look at the code, you see I use `rm()` to remove unnecessary objects, creating a tidier environment. From here it's a small step to the final step a tidy data set.

## 5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject
This step starts with installing packages `tidyr` and `dplyr` and adding them to the library.

```{r}
# (Re)Install package tidyr and add it to the library
install.packages("tidyr")
library(tidyr)

# (Re)Install package dplyr and add it to the library
install.packages("dplyr")
library(dplyr)
```

There are roughly two kinds of tidy data sets, wide and narrow. Either will do. I choose narrow:

```{r}
allDataTidied <- gather(allData, feature, signal, -(Subjects:Activities))
```

Now I can use `group_by` to group by 'Subjects' and 'Activities'. And `summarise()` to summarise with `mean()`.

```{r}
allDataTidied <- gather(allData, feature, signal, -(Subjects:Activities))
allDataTidiedBySubjectsActivities <- group_by(allDataTidied, Subjects, Activities)
MeanBySubjectsActivities <- summarise(allDataTidiedBySubjectsActivities, SignalMean = mean(signal))
write.table(MeanBySubjectsActivities, file = "MeanBySubjectsActivities.txt", row.name = FALSE)
```

`MeanBySubjectsActivities` is the tidy data set. It consists of 180 observations, six activities for each of the 30  subjects. Every column contains only one variable. They are named 'Subjects', 'Activities' and 'SignalMean'. 'SignalMean' is the mean of all values belonging to a specific subject and a specific activity.

As requested I used `write.table()` with `row.name = FALSE ` to write 'MeanBySubjectsActivities' to a text file: `MeanBySubjectsActivities.txt`. Next I uploaded it to Github. When you have it in your working directory of R(Studio), you can read it using `View(read.table("./MeanBySubjectsActivities.txt", header = TRUE))`.


## Citations
[1] Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. Human Activity Recognition on Smartphones using a Multiclass Hardware-Friendly Support Vector Machine. International Workshop of Ambient Assisted Living (IWAAL 2012). Vitoria-Gasteiz, Spain. Dec 2012.

[2] Course Project: [https://class.coursera.org/getdata-012/human_grading/view/courses/973499/assessments/3](http://class.coursera.org/getdata-012/human_grading/view/courses/973499/assessments/3, "Course Project").
