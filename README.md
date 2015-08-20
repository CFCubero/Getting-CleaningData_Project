# Getting-CleaningData_Project
# Description of the different parts of the script that creates a tiny dataset using the information from the following link:
# https://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

## The script follows an easy process convert the raw information into a tiny dataset
## Steps:  

* 1: Download and store raw information (easy)

if (file.exists("./data")){
    setwd("./data")
} else {
    dir.create("./data")
    setwd("./data")
}
url<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
file<-"data.zip"
download.file(url,file, method="libcurl")
unzip(file, exdir = ".")

*2. Read Train and Test information and all related files and create a dataset with this information

train<-read.table("./UCI HAR Dataset/train/X_train.txt",sep="",head=FALSE)
train<-cbind(train,read.table("./UCI HAR Dataset/train/y_train.txt",head=FALSE))
train<-cbind(train,read.table("./UCI HAR Dataset/train/subject_train.txt",head=FALSE))

test<-read.table("./UCI HAR Dataset/test/X_test.txt",sep="",head=FALSE)
test<-cbind(test,read.table("./UCI HAR Dataset/test/y_test.txt",head=FALSE))
test<-cbind(test,read.table("./UCI HAR Dataset/test/subject_test.txt",head=FALSE))

#### To get together the two datasets (train and test) we use bind function
dataset<-rbind(train,test)

*3. Read file with features for each column, filter 'mean' and 'std' features and name columns of the dataset.

features<-read.table("./UCI HAR Dataset/features.txt",sep="",head=FALSE,col.names=c("id","feature"),colClasses=c("numeric","character"))
activity_labels<-read.table("./UCI HAR Dataset/activity_labels.txt",sep="",head=FALSE,col.names=c("activity_id","activity_name"),colClasses=c("numeric","character"))

#### Once we read the features, we filter the ones that are related with 'std' or 'mean' using the 'grep' function
#### We use this information to name the columns of the dataset too

selected_features<-features[grepl('std|mean',features$feature),]
dataset<-dataset[,c(selected_features[,1],562,563)]
names(dataset)<-c(selected_features[,2],"activity_id","subject_id")

*4. Include a column with the information of the activity labels and reorder the columns so that dimensions are first

#### This is very easy with 'merge'

dataset<-merge(dataset,activity_labels)
dataset=dataset[,c(1,ncol(dataset)-1,ncol(dataset),2:(ncol(dataset)-2))]

*5. Creation of a tidy dataset with the average of the features for each subject and activity

#### There are several ways to group the features, but I use the feature 'aggregate' since it can aggregate several columns at the same time

tidy_dataset<-aggregate(.~dataset$activity_id+dataset$activity_name+dataset$subject_id,data=dataset[,4:ncol(dataset)],FUN="mean",na.rm=TRUE)
names(tidy_dataset)[1:3]<-c("activity_id","activity_name","subject_id")

*6. Save the tidy dataset and remove all unzipped and downloaded files (easy)

write.table(tidy_dataset, file = "tidyDataSet.txt", sep=" ",row.name=FALSE)
file.remove(file)
unlink("./UCI HAR Dataset",recursive=TRUE)

#### :) :) :)