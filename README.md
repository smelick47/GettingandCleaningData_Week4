## GettingandCleaningData_Week4
Peer-graded Assignment: Getting and Cleaning Data Course Project

### Data overview
One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

## Preperation

Install and Load the related packages and keep their version for references

```
install.packages("data.table")
install.packages("dplyr")

library("data.table")
library("dplyr")

#1.12.2
packageVersion("data.table")
#0.8.1
packageVersion("dplyr")
```

Set the working directory , folder path, dowload the files and extract

```
#set the wording directory
setwd("C:/Melick/Bigdata/Assignments/4")


download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip" , destfile = "dataset.zip")

if(!dir.exists("Data")){dir.create("Data")}
unzip("dataset.zip",exdir = "Data")

varDataPath <- "Data/UCI HAR Dataset"
```

### Part 1
Merges the training and the test sets to create one data set.
```
## Load the file and convert to table
dt_activity<- fread(file = paste(varDataPath,"activity_labels.txt",sep = "/") , col.names = c("id" , "activity")) %>% tibble::as_tibble()

## Load the file and convert to table
dt_features<-fread(file = paste(varDataPath,"features.txt",sep = "/") , col.names = c("id" , "feature")) %>% tibble::as_tibble()

## Load the file text X and train X
dt_temp1 <-fread(file = paste(varDataPath,"test/X_test.txt",sep = "/") )
dt_temp2 <-fread(file = paste(varDataPath,"train/X_train.txt",sep = "/") )

##Merge two data set and convert to table
dt_x <- rbind(dt_temp1,dt_temp2) %>% tibble::as_tibble()

# Remove temp data set for performance
rm("dt_temp1")  ; rm("dt_temp2")

## Load the file text Y and  train Y
dt_temp1 <-fread(file = paste(varDataPath,"test/y_test.txt",sep = "/") )
dt_temp2 <-fread(file = paste(varDataPath,"train/y_train.txt",sep = "/") )

##Merge two data set and convert to table and Asign the column ID
dt_y<- rbind(dt_temp1,dt_temp2) %>% tibble::as_tibble() %>%rename("id" = V1)

# Remove temp data set for performance
rm("dt_temp1") ; rm("dt_temp2")


# Load the file text subject and train subject
dt_temp1 <-fread(file = paste(varDataPath,"test/subject_test.txt",sep = "/") )
dt_temp2 <-fread(file = paste(varDataPath,"train/subject_train.txt",sep = "/") )

##Merge two data set and convert to table and Asign the column ID
dt_subject<- rbind(dt_temp1,dt_temp2) %>% tibble::as_tibble() %>%rename("subjectid" = V1)

# Remove temp data set for performance
rm("dt_temp1") ; rm("dt_temp2")

```

### Part 2
Extracts only the measurements on the mean and standard deviation for each measurement.

```
#Extracts only the measurements on the mean and standard deviation for each measurement.
dt_meanstd <-dt_features[grep("mean\\(\\)|std\\(\\)",dt_features$feature),]
```

### Part 3
Uses descriptive activity names to name the activities in the data set
```
# Uses descriptive activity names to name the activities in the data set
dt_y<- left_join(dt_y,dt_activity , by = c("id"="id") ) 
```

### Part 4
Appropriately labels the data set with descriptive variable names.
```
#Appropriately labels the data set with descriptive variable names. 
# this is to assign column namea from features dataset
#  clean the columns names , to lowwer and remove unneccessary chars
names(dt_x) <- tolower(gsub("\\(\\)|,","",dt_features$feature))
```

### Part 5
creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```
## bind all dataset and remove activity ID
## creates a second, independent tidy data set with the average of each variable for each activity and each subject.
## make the 
dt_cleaned <- bind_cols(dt_subject,dt_y,dt_x) %>% select(-c("id"))%>%group_by(subjectid,activity)

dt_tidydata <-  summarise_all(dt_cleaned,list(mean))
```

### save the table
```
## select (dt_tidydata , 1:3)
write.table(dt_tidydata, file = "Tidy.txt", row.names = FALSE)

```
