# Detecting & Treating Price Outliers

A few outliers can significantly skew our understanding of variables or influence a predictive model. Depending on the source and nature of the outliers, they could also hold valuable information. Detecting and treating them needs special care. Here's a clear, thorough, and useful series of posts on outliers ([Part 1](https://towardsdatascience.com/detecting-and-treating-outliers-in-python-part-1-4ece5098b755), [Part 2](https://towardsdatascience.com/detecting-and-treating-outliers-in-python-part-2-3a3319ec2c33), [Part 3](https://towardsdatascience.com/detecting-and-treating-outliers-in-python-part-3-dcb54abaf7b0)). Here's a great paper on best practices by [Aguinis et al](http://www.hermanaguinis.com/ORMoutliers.pdf). 

Today, let's work with Mike Powell's [global wine data set](https://data.world/markpowell/global-wine-points) that includes vintage, country, county, designation, province, title, variety, winery, points, and price per bottle for nearly 25,000 wines. Let's say a global winery's marketing team would like to understand and model wine points using price as a variable. Extreme price values could unduly influence the model so let's understand, detect, and handle them. 

#### **Univariate or Multivariate**

First let's check if outliers are univariate or multivariate. Univariate outliers are extreme values in the distribution of a specific variable, for example, a \$5,000 bottle. Multivariate outliers are unlikely combinations of values, for example, a \$5,000 bottle of a 2018 vintage with a low score of 83 points. For simplicity, let's focus on univariate price outliers here. If you're interested in multivariate outliers, check out applications of [Mahalanoubis distance](https://www.machinelearningplus.com/statistics/mahalanobis-distance/) and [minimum covariance distance](https://towardsdatascience.com/detecting-and-treating-outliers-in-python-part-2-3a3319ec2c33). 

#### **Source**

Next, let's understand the source of the outliers to see how we'd like to treat them. In our case, price outliers aren't due to human or measurement error, but are true extreme values not fully representative of the wine population. We don't want to keep them as is, but, since they hold valuable information, we don't want to simply delete them. Instead, we can recode them to moderate their impact. 

#### **Detect**

Let's plot distribution of key variables to understand them a bit more. For example, we'll see that price is heavily right-skewed, while vintage is heavily left-skewed. 

To detect outliers, we'll use 2 approaches - Tukey's method and Median Absolute Deviation (MAD). We avoid using the z-score method (studentized residuals) because:

- Distribution mean and standard deviation are sensitive to outliers  - Finding an outlier is dependent on other outliers as every observation affects the mean
- It assumes variable is normally distributed 

Externally studentized residuals removes the influence of each observation from the calculation, but the distribution is still sensitive to outliers and it assumes the variable is normally distributed. 


##### **Tukey's Method**

Tukey's method generates possible and probable outliers by constructing inner and outer fences based on IQR. The lower inner fence is 1.5 x IQR below Q1, while the upper inner fence is 1.5 x IQR above Q3. The lower outer fence is 3 x IQR below Q1, while the upper outer fence is 3 x IQR above Q3. Possible outliers lie between the inner and outer fence, while probable outliers lie outside the outer fence. We care about treating the probable outliers. Tukey's method is robust to outliers and doesn't require a normal distribution. Additionally, for right-skewed data, we can still apply Tukey's method on log transformed variables. 

##### **MAD**

MAD is a modified, more robust alternative to z-score. It replaces mean and standard deviation with median and median absolution deviation and is more robust to outliers and doesn't assume the variable is normally distributed. Outliers lie beyond the cutoff of 3. 

#### **Treat**

##### **Winsorize**

Winsorizing replaces all values beyond a k percentile with the k percentile value, which is less extreme than simply removing these values. Piggybacking off of the Tukey method, the k percentile will be set at the distribution's outer fence. 

The price variable's upper outer fence is 117, which is closest to the 97.5% (109), so let's set the upper bound for winsorization to 97.5%. For the lower bound, a price below zero doesn't make sense, so we'll set the lower bound to 0 and the data will only be right-tail winsorized.  


##### **Impute**

Similar to howe we've imputed missing values in previous posts, we can apply it to outliers by replacing outliers with NaN and imputing these new missing values with estimates based on the remaining data. Rather than simple imputation with mean or median, which obscures valuable information held by extreme values, let's apply iterative imputation. 

In sklearn's IterativeImputer, the outlier becomes a dependent variable in a prediction model and is predicted based on remaining non-missing non-outlier values. The default estimator is regularized linear regression BayesianRidge. Let's impute both Tukey's probable outliers and MAD outliers. 

Finally, we'll bring it all together with a statistical summary to compare how the mean and max of the original price compares to log price, winsorized, imputed Tukey's, and imputed MAD. 


