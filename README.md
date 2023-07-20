# yelp_DS
Yelp Data Investigation

Yelp provides an easily accessible and vast repository of business review data. In this coursework, the open-source Yelp dataset is investigated to see whether the distribution of the frequencies of reviews can be used to predict the failure of a business. 

The hypotheses are:
• Failed and successful business reviews have the same frequency at the beginning of business life • As negative reviews accumulate, failed businesses receive fewer and fewer reviews
• The decay of review frequency can be used to predict if a business will fail


The dataset is taken from: https://www.yelp.com/dataset 


# Data Loading and Cleaning 

Data was loaded into Python using the Pandas read json() method. The raw data contained 7,000,000 reviews relat- ing to 150,000 businesses. After removing any rows with any null values from both datasets and selecting only those businesses in Arizona, 7793 businesses remain. Reviews for businesses that do not appear in the business dataset were removed, leaving 400,000 relevant reviews. The distribution of total reviews over the lifetime of the dataset is heavily left skewed with mean equal to 51.24 and standard deviation of 100.06. Businesses that have significantly greater than the mean number of reviews are outliers and any with a review count greater than three standard deviations from the mean were removed. The final dataset contains 303,806 reviews for 7621 unique businesses, of which 6324 are open.

The rolling average in a 7 day period is used as a measure of review frequency and the lifetime of a busi- ness is defined as the timespan between the first and last reviews. An accurate mean business review rating was calculated from the reviews dataset and used in the analysis.

```python
import pandas as pd
import json

file = '**DIRECTORY**/yelp_academic_dataset_business.json'

business_data = pd.DataFrame()

# important to chunk as the files are large
with pd.read_json(file, lines=True, chunksize=100000) as reader:
    reader
    for chunk in reader:
        business_data = pd.concat([business_data, pd.DataFrame(chunk)], ignore_index = True)

file = '**DIRECTORY**/yelp_academic_dataset_review.json'

review_data = pd.DataFrame()

with pd.read_json(file, lines=True, chunksize=100000) as reader:
    reader
    for chunk in reader:
        review_data = pd.concat([review_data, pd.DataFrame(chunk)], ignore_index = True)

business_data = business_data.dropna()
review_data = review_data.dropna()

print("bdat len: ", len(business_data))
print("rdat len: ", len(review_data))

review_data = review_data.set_index('date')
review_data = review_data.sort_index()

# Only investigating Arizona
business_data = business_data[business_data['state']=='AZ']

# make sure only businesses that exist have reviews
review_data = review_data[review_data['business_id'].isin(pd.Series(business_data['business_id']))]

print(len(review_data))

```

# 2 Investigation

Exploratory data analysis was undertaken by indexing the review dataset by the date column and counting the number of reviews in each rolling seven day period. One significant issue was a large number of seven day periods with no reviews in them leading to an overestimation of the rolling seven day review count. Each individual business had review counts of 0 added to the dataset for each week over their respective lifespan, giving an accurate average rolling weekly review count.

First step is to remove outliers:
```python
review_count = pd.DataFrame(review_data.groupby('business_id')['stars'].count()).reset_index()

stand = review_count['stars'].std()
mu = review_count['stars'].mean()

review_count[review_count['stars'] > (mu + 3*stand)]['business_id']

# remove all business data with star values outside 3 SD from the mean.
review_data = review_data[review_data['business_id'].isin(review_count[review_count['stars'] <= (mu + 3*stand)]['business_id'])]
business_data = business_data[business_data['business_id'].isin(review_count[review_count['stars'] <= (mu + 3*stand)]['business_id'])]

print(len(review_data), len(business_data))
```

Perform data manipulation to input blank weeks where necessary:
```python
import numpy as np
import time

average = []
business = []
iter = 0
begin = time.time()
for i in business_id:
    business.append(i)
    
    start = np.min(review_data[review_data['business_id'] == str(i)].index)
    end = np.max(review_data[review_data['business_id'] == str(i)].index)
    df_Date=pd.date_range(start=start, end=end, freq='w')
    df_Date = pd.DataFrame(df_Date, columns = ['date'])
    df_Date['stars'] = 0
    
    #get the 
    average_rolling = review_data[review_data['business_id'] == str(i)]['stars'].rolling('7d').count()
    average_rolling = pd.DataFrame(average_rolling).reset_index()
    
    #concat the df on the date column
    df_Date = pd.concat([df_Date, average_rolling], ignore_index = True)
    df_Date = df_Date.groupby('date')['stars'].sum()
    
    average.append(np.mean(df_Date))
    
    #average_rolling.append(review_data[review_data['business_id'] == str(i)]['stars'].rolling('7d').count().mean())
    iter += 1
    if iter % 100 == 0:
        print(iter, "elapsed time: ", time.time() - begin)
    

        #print(len(business), len(average))

#print(business)
#print(average)
```
