### Getting and Cleaning Data Course Projectless 

The source and explanations of the data is located at:

  http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

The data for the project:

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

The run_analysis.R uses the tidyverse package. Tidyverse contains multiple packages used in the analysis: dplyr, readr, tidyr. The following line installs tidyverse if it does not exist
```
if(!require(tidyverse)) install.packages('tidyverse')
```

The run_analysis.R first checks if the UCI HAR Dataset exist in the directory. If it does not then it will download and unzip the file. If the UCI HAR Datase does exist, "Already exists" is printed out.
```
if(!'UCI HAR Dataset' %in% list.files('~/Coursera/Getting & Cleaning Data')){
  # download file
  download.file('https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip',
                path.expand('~/Coursera/Getting & Cleaning Data/UCI HAR Dataset.zip'))
  # unzip
  unzip('~/Coursera/Getting & Cleaning Data/UCI HAR Dataset.zip')
}else{
  print('Already exists')
}
```
1) The training and test data are merged. These are the X_train.txt and Y_train.txt files. 
```
  # read all 3 train files (subject_train.txt, X_train.txt, y_train.txt)
  train_files <- list.files('./train', pattern = '*.txt',
                            full.names = TRUE)
  train_files <- lapply(train_files, read_table, col_names = FALSE)
  
  # read all 3 test files (subject_test.txt, X_test.txt, y_test.txt)
  test_files <- list.files('./test', pattern = '*.txt',
                            full.names = TRUE)
  test_files <- lapply(test_files, read_table, col_names = FALSE)
```
The features.txt is cleared of special characters and assigned as the column names of the X_tain + X_test
```
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
```
The rowbind and rename stores into dataframe called wide_data as the data is in the wide format:
```
wide_data <- rbind(train_files[[2]], test_files[[2]])
colnames(wide_data) <- features$X1
```
(Alternatively dplyr::bind_rows)

2) Only columns with names containing mean and std are kept
```
wide_data <- wide_data %>% select(matches('mean|std'))
```
3) Volunteer ID and activity name are merged to the front of the wide_data
```
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
```
4) Volunteer ID and activity name are properly named
```
wide_data <- rename(wide_data, volunteer_id = X1, activity_name = X2)
```

5) The wide_data is converted to long format with tidyr::gather (alternative reshape2::melt) 
```
long_data <- gather(wide_data, features, value, -volunteer_id, -activity_name)
```
The wide data is converted to a tidy data of averages called tidy_avg: volunteer | activity | feature | average 
```
tidy_avg <- long_data %>% 
  group_by(volunteer_id, activity_name, features) %>% 
  summarise(average = mean(value))
```

The resulting tidy_avg can be printed with
```
write.table(tidy_avg, 
            paste0(dirname(getwd()), '/tidy_avg.txt'), 
            row.names = FALSE)
```
