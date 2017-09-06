if(!require(tidyverse)) install.packages('tidyverse')

# If UIC HAR Dataset doesn't exist in /Coursera/Getting & Cleaning Data
# then download and unzip
# else print('already exists')
if(!'UCI HAR Dataset' %in% list.files('~/Coursera/Getting & Cleaning Data')){
  # download file
  download.file('https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip',
                path.expand('~/Coursera/Getting & Cleaning Data/UCI HAR Dataset.zip'))
  # unzip
  #$$$ unzip('C:/Users/Minty/Documents/Coursera/Getting & Cleaning Data/UCI HAR Dataset.zip')
  unzip('~/Coursera/Getting & Cleaning Data/UCI HAR Dataset.zip')
}else{
  print('Already exists')
}

# set working directory to UIC HAR Dataset folder
#$$$ setwd('~/Coursera/Getting & Cleaning Data/UCI HAR Dataset')
setwd('~/Coursera/Getting & Cleaning Data/UCI HAR Dataset')

# 1. Merge the training and the test sets to create one data set.

  # read all 3 train files (subject_train.txt, X_train.txt, y_train.txt)
  train_files <- list.files('./train', pattern = '*.txt',
                            full.names = TRUE)
  train_files <- lapply(train_files, read_table, col_names = FALSE)
  
  # read all 3 test files (subject_test.txt, X_test.txt, y_test.txt)
  test_files <- list.files('./test', pattern = '*.txt',
                            full.names = TRUE)
  test_files <- lapply(test_files, read_table, col_names = FALSE)
  
  # read 'features.txt': List of all features
  features <- read_table('./features.txt', col_names = FALSE)
  # clean features names
    # remove ()
    features$X1 <- gsub('\\()', '', features$X1)
    # replace special characters with underscore
    features$X1 <- gsub('),| |,|-|\\(', '_', features$X1)
    # remove parenthesis at end
    features$X1 <- gsub('\\)$', '', features$X1)
    # all lower case
    features$X1 <- tolower(features$X1)

wide_data <- rbind(train_files[[2]], test_files[[2]])
colnames(wide_data) <- features$X1

# 2. Extract only the measurements on the mean and standard deviation for each measurement.
wide_data <- wide_data %>% select(matches('mean|std'))

# 3. Uses descriptive activity names to name the activities in the data set

  # read 'activity_labels.txt': Links the class labels with their activity name
  activity_labels <- read_table('./activity_labels.txt',
                                col_names =  FALSE)
  
  # activity ids to activity names
  activity <- left_join(
    bind_rows(train_files[[3]], test_files[[3]]), 
    activity_labels, 
    by = 'X1')
  
  # merge volunteer and activity columns to mean and std columns
  wide_data <- bind_cols(
    bind_rows(train_files[[1]], test_files[[1]]), #volunteer IDs
    activity[2], #activity names
    wide_data) #mean & std features columns
  
# 4. Appropriately labels the data set with descriptive variable names.
wide_data <- rename(wide_data, volunteer_id = X1, activity_name = X2)

# 5. From the data set in step 4, creates a second, independent tidy data set 
# with the average of each variable for each activity and each subject.
long_data <- gather(wide_data, features, value, -volunteer_id, -activity_name)
tidy_avg <- long_data %>% 
  group_by(volunteer_id, activity_name, features) %>% 
  summarise(average = mean(value))

# write final table
write.table(tidy_avg, 
            paste0(dirname(getwd()), '/tidy_avg.txt'), 
            row.names = FALSE)
