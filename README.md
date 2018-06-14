# DSI Project 4 - Predicting West Nile Virus in Chicago

## Table of Contents

1. Background
2. Data Cleaning
3. Exploratory Data Analysis
4. Modeling
5. Conclusions

## Background

The goal of this project was to predict the occurrence of West Nile Virus (WNV) in Chicago, IL. We obtained data from the City of Chicago hosted as part of a Kaggle competition [here.](https:/www.kaggle.com/c/predict-west-nile-virus/) The data was obtained in 3 separate files as described below: 

1) A dataset containing location data for traps placed around Chicago to monitor mosquito populations. Samples from these traps were collected by city workers and tested for the presence of WNV.
2) A dataset containing information about the weather occurring in the months when the mosquito data was being collected.
3) A dataset indicating the dates and locations where pesticides were sprayed by the city to eliminate adult mosquitos. 

Our goal was to perform analysis on the provided data and create machine learning models that predict the presence of West Nile Virus in mosquito samples. We used data from 2007, 2009, 2011, and 2013 to train the models; model performance was assessed using data collected in 2008, 2010, 2012, and 2014.

## Data Cleaning

### Trap Data

The training dataset contained no missing values. We collapsed the number of mosquito species by combining species who were infrequent into a category named "Culex Other." (The infrequent species accounted for only 3% of all observations in the training set.) We chose to impute the most common species in the Chicago region, C. pipiens, for any observations marked as 'Unspecified' based on rationale presented [here](http://www.ajtmh.org/content/journals/10.4269/ajtmh.2009.80.268#html_fulltext) by Hamer et. al.

### Weather Data

Null values in the weather dataset were encoded as "M" or "-". For features where the occurence of "M"s/"-"s was low (<5%), we dropped the missing entries (`Tavg`, `WetBulb`, `Depth`). When the number of missing values for a feature was high, we dropped the feature column (`Depart`, `Sunrise`, `Sunset`). A couple of columns required more consideration; for `PrecipTotal`, the measure of the total precipitation for one day in inches, some values were encoded as "T"s, representing trace amounts of rain or snow (stated in documentation). We imputed a value of 0.01" for these trace entries.  The `CodeSum` column showed the weather conditions for a given day; because some entries in this column contained multiple codes each describing a separate weather condition, we parsed the feature into unique features for each individual code.
 
### Spray Data

The `Time` column of the Spray dataset had many null values (over 500), so we chose to drop the column. We removed instances of duplicate entries attirbuting them to a data entry or collection error. We then developed a function that created a variable `spray_nearby` which indicated whether individual traps were within 1/8th of a mile of a sprayed zone.

### Data Merging

In order to merge the weather data and the trap data, we engineered a feature `station` which encoded for the closest weather station to a trap. We then merged the two datasets using the `date` and `station` columns. We then merged the combined weather/trap dataset with our engineered spray variable using `date` and `trap`.

## Exploratory Data Analysis

We created a number of plots to visually inspect our data, found in the EDA notebook. By graphing the cases of West Nile Virus against the latitudes where traps were placed, we determined that there were more cases of the West Nile Virus on the north side of the city. Inspection of weather data plotted against the presence of WNV suggested that mosquitos with WNV were more likely to be trapped on days with higher average temperatures and lower humidities. 

## Modeling

We tested several variations of machine learning models in an effort to maximize the receiver operating characteristic area under the curve (ROC AUC). The ROC AUC is a measure of the ability of a classifier to distinguish between two or more classes; a classifier that randomly guesses (for binary classification) will achieve a ROC AUC of 0.5; one that classifies perfectly will score 1.0. The table below lists a selection of the tested models and their corresponding ROC AUC scores:

| Model                               | ROC AUC (Train) | ROC AUC (Validation) | ROC AUC (Kaggle) |
|-------------------------------------|-----------------|----------------------|------------------|
| Logistic Regression                 | 0.754           | 0.744                | 0.63             |
| Logistic Regression (PCA)           | 0.679           | 0.692                |                  |
| Decision Tree                       | 0.982           | 0.693                |                  |
| Random Forest                       | 0.976           | 0.821                |                  |
| AdaBoost (Logistic Regression, PCA) | 0.741           | 0.739                |                  |
| Bagging (Decision Tree)             | 0.992           | 0.782                |                  |
| k-Nearest Neighbors                 | 0.929           | 0.620                |                  |

Our main modeling strategy was to vary the size of the feature space of the model inputs. We tested models using the entire feature space, a selection of features that we believed we were most predictive of WNV presence, and using a feature space of principal components generated by principal component analysis (PCA). Overall, reducing the number of features (through either manual feature selection or PCA) resulted in the greatest increases in ROC AUC.

## Conclusions

Our best performing model was a logistic regression model using 7 features created by principal component analysis (ROC AUC = 0.63). This model not only resulted in the highest ROC AUC when predicting WNV presence for the test data but also was incredibly fast to fit owing to its low feature count. 


