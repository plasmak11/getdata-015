---
"README"
---

#################################################################
# 1) Get a list of features, activity labels, and rows that contain 'mean' or std' in feature list.
#################################################################
  features <- read.delim("features.txt",sep="",col.names=c("feature_id","feature"),header=FALSE)

# Activity labels i.e WALKING, LAYING, STANDING, etc
  activity_labels <- read.delim("activity_labels.txt",sep="",header=FALSE)

# Search the features table for the list of row #s that has 'mean' or 'std'
  meanstdCheck<-grep("mean()|std()",as.character(features[,2]))

#################################################################
# Goal: X_test will store feature measurements from "test" folder filtered by mean/std
#################################################################

# Read data from 'test' folder
  subject_test <- read.csv("./test/subject_test.txt",col.names="subject",header=FALSE)
  X_test<-read.delim("./test/X_test.txt",sep="",header=FALSE,as.is=TRUE,col.names=as.character(features[,2]))
  y_test<-read.delim("./test/y_test.txt",sep="",col.names="", header=FALSE)

# Filter out the rows that have 'mean()' or 'std()' 
  X_test <- X_test[,meanstdCheck]

# Bind two columns: 1) activityID (y_test) and 2) subjectID (subject_test). I want it as first columns so it's on the left.
  X_test <- cbind(y_test,X_test)
  X_test <- cbind(subject_test,X_test)

# Merging the activity labels, based on the activity ID and activity labels.
# Probably there's a better way to do this.
  X_test <- merge(activity_labels,X_test,by.x="V1",by.y="X")

#################################################################
# 2) X_test now stores all mean and std raw feature measurements
#################################################################

#################################################################
# Goal: X_train will store feature measurements from "train" folder
# filtered by mean/std
#################################################################

# Read data from 'train' folder
  subject_train <- read.csv("./train/subject_train.txt",col.names="subject",header=FALSE)
  X_train<-read.delim("./train/X_train.txt",sep="",header=FALSE,as.is=TRUE,col.names=as.character(features[,2]))
  y_train<-read.delim("./train/y_train.txt",sep="",col.names="", header=FALSE)
  X_train <- X_train[,meanstdCheck]

# Bind two columns: 1) activityID and 2) subjectID 
  X_train <- cbind(y_train,X_train)
  X_train <- cbind(subject_train,X_train)

# Merging the activity labels, based on the activity ID and activity labels.
  X_train <- merge(activity_labels,X_train,by.x="V1",by.y="X")

#################################################################
# 3) X_train now stores all mean and std raw feature measurements
#################################################################


#################################################################
# 4) combined_X will combine test and train data sets!
#################################################################

  combined_X <- rbind(X_test,X_train)
  colnames(combined_X)[2]<-"activity"
  combined_X$V1<-NULL
  combined_X$subject<-factor(combined_X$subject)

#################################################################
# 5) Melt and cast based on Subject and Activity
#################################################################

# Melting with columns "subject" and "activity", everything else is a variable
  Xmelt<-melt(combined_X,id=c("subject","activity"),measure.vars=names(combined_X)[3:length(combined_X)])

# Casting, grouped by subject and activity.
  tidyCombined <- dcast(Xmelt, subject+activity~variable, mean)

# A little extra to clean up those pesky periods (.) and extra fBody in the column names
  colnames(tidyCombined)<-sub("*[:punct:]*|fBody|","",as.character(names(tidyCombined)))
  tidyCombined