# CodeBook.md
This is the Code book of: 'MeanBySubjectsActivities.txt'.

## General
This code book is based on features_info.txt and readme.txt. Both are part of a zip file that can be downloaded from: [https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip](http://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip, "UCI HAR Dataset"). For more information, please see [1].

'MeanBySubjectsActivities.txt' consists of 180 observations of three variables.

These variables are:

* Subjects;
* Activities and
* SignalMean.

The number 180 is derived from 30 subjects, with six activities. And every activity has one measure.

### Subjects
Name of variable: Subjects  
Short for: subject  
Number of values: 30  
Values: 1:30  
Unit:  none (dimensionless)  
Remarks: none  

### Activities
Name of variable: Activities  
Short for: subject, a person who took part of the trial mentioned in **General**  
Number of values: 6  
Values:  

* WALKING
* WALKING_UPSTAIRS
* WALKING_DOWNSTAIRS
* SITTING
* STANDING
* LAYING

Unit: none (dimensionless)  
Remarks: none  

### SignalMean
Name of variable: SignalMean  
Short for: Mean of Signal  
Number of values: 1  
Values: Ranging from -0.7534767 up to and including  0.1549415  
Unit: none (dimensionless)  
Remarks: SignalMean is the mean of a set of signals belonging to the subject and the activity in this observation. These signals are a subset of the training and test data set mentioned in **General**. Which subset is documented in README.md which is located in the same directory as this CodeBook.md file.  

## Citation
[1] Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. Human Activity Recognition on Smartphones using a Multiclass Hardware-Friendly Support Vector Machine. International Workshop of Ambient Assisted Living (IWAAL 2012). Vitoria-Gasteiz, Spain. Dec 2012.
