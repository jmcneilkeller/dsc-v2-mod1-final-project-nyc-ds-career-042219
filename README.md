
# Module 1 Final Project - Jonathan Keller


## 2014/2015 King County Housing Data

## Project Questions
In this project I hope to answer the following questions:

1. Can we accurately predict housing prices in King County using available data?
2. Are housing prices affected by the time of year?
3. What variables are the best predictors?


## Data Cleaning

The first step was to check the basic information of the dataframe. It was discovered that 'waterfront', 'view' and 'yr_renovated' were missing observations. Also both the 'date' and 'sqft_basement' variables were reporting as being object instead of integers, or in the case of date, datetime objects. 'View' had a relatively insignificant number of null observations (63), but both 'waterfront' (2376) and 'yr_renovated' (3842) were significant so I determined not to remove any observations from those variables.

* 'waterfront' was a binary (on the waterfront or not). Since the majority of observations were not on the waterfront (0), I filled in the missing values with a 0.
* 'yr_renovated' also mostly filled 0 values (so not renovated) otherwise the renovation year. First I filled in missing values with 0 again. I considered resetting houses that hadn't been renovated in the last 30 years to 0 as well (since thats the average life of a renovation), but as renovations can last significantly longer based on materials/craftsmanship/type of work, I left those values intact. Instead I change the variable to a binary: 0 - has not been renovated or 1: has been renovated.
* 'view' is grading scale of the view from the property. Again, the majority of the properties were graded out as 0, so I filled missing values as such.

I then dealt with 'sqft_basement' and 'date':

* Looking closely through the data 'sqft_basement' had several '?' which would not show up as null values since the datatype of that column was an object. So I recast the column as numeric with coerce=True to change those '?' values to NaNs, then filled in missing values by taking the difference of 'sqft_living' and 'sqft_above'.
* 'date' I first cast as datetime objects ordered by year first in order to sort. From my reading I had seen that home prices tended to be higher in summer than in winter, so I binned the date range into four sections, roughly: Summer 2014(May to August), Winter 2014(Sept to Dec), Winter 2015(Jan to Apr) and Summer 2015 (functionally just May as the dataset only covers one year).


## Visualization/Further Correction

Plotting a KDE/histogram plot revealed a right skewedness to both the target variable ('price') and 'sqft_living'. This was corrected before modeling with a log transformation on both.

Before:

![sqft_kde_before](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/sqft_living_kde_hist.png)

![price_kde_before](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/price_kde_hist.png)

After:
![sqft_kde_after](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/sqft_living_log.png)
![price_kde_after](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/price_log.png)

Scatter plots of the independent variables revealed a linear but possibly exponential relationship between 'price' and most of the square footage variables, specifically 'sqft_living'. It also revealed most of the variables to be categorical. It also showed some outliers for 'sqft_living', 'bedrooms' and 'bathrooms'.

![scatter_1](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/scatter_1.png)
![scatter_2](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/scatter_2.png)
![scatter_3](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/scatter_3.png)
![scatter_4](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/scatter_4.png)
![scatter_5](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/scatter_5.png)

In response I:

* Binned 'grade', 'bathrooms', 'bedrooms' and 'condition'.
* I chose not to bin 'view' as most of the values were '0'.
* I created dummies for zipcode as each zipcode will presumably present unique features and aren't really ranges.
* Trimmed off the outliers in 'sqft_living', 'bedrooms' and 'bathrooms' by trimming the 99th quantile from each data set.

## Modeling

First, I split my data set into training and testing data. Checking the heatmap of my correlation, I determined that 'sqft_living', 'bathrooms' and 'grade' were the variables most highly correlated to the target variable.

![heatz](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/Heatmap_R2.png)

I decided to model those individually first. 'sqft_above' was also highly correlated, but as I already had 'sqft_living' and it was more highly correlated when checking variables with the .corr() function, I decided to skip that one. I also decided to model 'lat' as well as it was highly correlated to 'price' but no other variables except 'zipcode'.

After first trying combinations of the most highly correlated variables, along with date variables to test my hypothesis about seasons, I switched to modeling all independent variables. I removed variables that could be highly correlated until I got to my final model.

![Summary_header](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/final_model_header.png)
![Residuals](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/Residuals_hist.png)
![price_hist](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/Final_model_price_hist.png)
![QQ](https://github.com/jmcneilkeller/dsc-v2-mod1-final-project-nyc-ds-career-042219/blob/master/Final_Model_QQ.png)

## Questions Answered/Conclusions:

A reminder of my initial questions and the answers I came to:

1. Can we accurately predict housing prices in King County using available data?
* Partially. The model returned a relatively high R-squared values and a low mean squared error value on the testing data. However the actually plotting the data revealed a lack of normalization for more extreme ends of house prices. Likely this model would be fairly reliable for houses landing in between 25% quartile and 75% quartile, but probably less reliable below 25% and above 75%.

 2. Are housing prices affected by the time of year?
 * The answer here seems to be no. Adding in the seasons only resulted in a .02 increase in R-squared but also introduced additional multicollinearity.

 3. What variables are the best predictors?
 * In my final model, the only predictors I ended up utilizing were 'sqft_living' and 'zipcode' (separated into dummies).  
