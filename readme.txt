library(reshape2)               ## Loading the required libraries
library(plyr)

train_x<-read.table("./UCI HAR Dataset/train/x_train.txt" )           ##Loading the training and test databases
train_y<-read.table("./UCI HAR Dataset/train/y_train.txt")
train_sub<-read.table("./UCI HAR Dataset/train/subject_train.txt")

test_x<-read.table("./UCI HAR Dataset/test/x_test.txt")
test_y<-read.table("./UCI HAR Dataset/test/y_test.txt")
test_sub<-read.table("./UCI HAR Dataset/test/subject_test.txt")

feat<-read.table("./UCI HAR Dataset/features.txt")                     ## Loading the activity names and features details
act<-read.table("./UCI HAR Dataset/activity_labels.txt")

feat$V2<-tolower(as.character(feat$V2))               ## Reworking the feature names to make them easily understandable
feat$V2<-sub("acc","accelerometer",feat$V2)           ## Replacing acc with accelerometer
feat$V2<-sub("gyro","gyroscope",feat$V2)              ## Replacing gyro with gyroscope
feat$V2<-sub("mag","magnitude",feat$V2)               ## Replacing mag with magnitude
feat$V2<-sub("\\(\\)","",feat$V2)                     ## Taking out the () brackets
feat$V2<-sub("^t","time",feat$V2)                     ## Replacing starting t with time   
feat$V2<-sub("^f","frequency",feat$V2)                 ## Replacing starting f with frequency
feat$V2<-sub("^","Mean of ",feat$V2)                    ## As the means will be calculated in the tidy dataset, I add this to the beginning of all features
## One might continue to rework the feature labels, but I found these labels descriptive enough 

train<-cbind(train_sub,train_y,train_x)               ## Putting together the subject details with the training variables
test<-cbind(test_sub,test_y,test_x)                   ## Putting together the subject details with the test variables
full<-rbind(train,test)                               ## Putting together the training and test data

colnames(full)<-c("subject","activity",feat$V2)       ## Labeling the columns 

meandata<-subset(full,select=grep("mean",colnames(full)))    ## selecting the mean and standard deviation data
stddata<-subset(full,select=grep("std",colnames(full)))

subact<-subset(full,select=c(1,2))                    ## I take the first two columns (subject and activity) from the previous dataset
mestd<-cbind(subact,meandata,stddata)                 ## So I could join them with the meand and standard deviation data

mestd$activity<- factor(mestd$activity, levels=act[,1], labels=act[,2]) ## I convert the activity column into a factor
meltdata<-melt(mestd,id=c("subject","activity"))      ## I melt the dataset using the subject and activity columns as id variables      
tidydata<-dcast(meltdata, subject + activity ~ variable, mean) ## I cast the dataset by to get the mean by subject and activity

write.table(tidydata,file="tidydata.txt")               ## Writing out the tidydata as .txt file
