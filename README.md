# datacleaning_project

## Read the data

### First the test data
subject_test <- read.table("test/subject_test.txt")  
X_test <- read.table("test/X_test.txt")  
y_test <- read.table("test/y_test.txt")  

### repeat the same for training data
subject_train <- read.table("train/subject_train.txt")  
X_train <- read.table("train/X_train.txt")  
y_train <- read.table("train/y_train.txt")  

## Question #1 

### Column bind test and training respectively
By running dimension on subject_test, X_test and y_test, where each one has 2947, and 
based on descriptions in readme, we assume that they all represent same observations
and are sorted the same way (this is an assumption we are making), so cbind these
have the order - subject, activity_label(this is currently an integer but we can map later), 
observation (each observation is in the features.txt file, which we'll map later)

test_data <- cbind(subject_test, y_test, X_test)  
train_data <- cbind(subject_train, y_train, X_train)  

### now merge the test and training data with rbind  
data <- rbind(test_data, train_data)  

### Test the results
We can quickly check that rbind worked correctly by checking the row nums and make sure they add.
here are the results.
nrow(test_data)  
[1] 2947  
nrow(train_data)  
[1] 7352  
nrow(data)  
[1] 10299  

## Question #4 

Note, I am doing #4 out of order since I think it makes it easier to grep, once we add col names
let's add the right column names. This is #4 from assignment that I'm doing before #2. 
In the cbind we first added subject and acitivity Id, then all the other 
obersvations attributes, which are captions in features.txt

### first get features.txt data
features <- read.table("features.txt")

### now add column names to data
names(data) <- c("sample_id", "activity_num", as.character(features[,2]))

## Question #2 

### get all column names with mean or std. 

Here we are no include Mean or Std (with) capitals, just since I'm making the assumption they are not relevant this could easily be done with ignore.case=TRUE parameter in the grep comment

mean_std_colucmns <- grep("mean|std", names(data))  

### now column 1 and column are the sample ID and activity ID, so let's get those to
relevant_columns <- c(1, 2, mean_std_columns)

### get the subset.I think we could use dplyr here as well
relevant_data <- data[,relevant_columns]

## Question #3 

### first get activity data
activity <- read.table("activity_labels.txt")

### then create a new variable for relevant data

Get descriptive variable by looking up the index (1st column) of actvitiy data frame  
relevant_data$activity_num <- activity[relevant_data$activity_num,2]

## Question #5 

### first get long form data by collapsing all observation attributes into a single variable

you end up getting 4 columns and 10299 (original observations) * 79 observation variables
data_melt <- melt(relevant_data, id=c("sample_id", "activity_num"))

### now group by sample_id (subject_id) and activity
grouped_data <- group_by(data_melt, sample_id, activity_num, variable)

### summarize by mean
Note, I'm creating a new variable each time, just to make sure I don't corrupt the data  
tidy_data <- summarize(grouped_data, value=mean(value, na.rm=TRUE))
