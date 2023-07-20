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
        
```
