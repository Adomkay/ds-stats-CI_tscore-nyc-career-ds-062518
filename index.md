
## Confidence Intervals with t-score

### SWBAT: 

1. Understand the concept of a confidence interval and be able to construct one for a mean

3. Demonstrate how to use the t-distribution for constructing intervals for small sample sizes

4. Express a correct interpretation of confidence intervals. 

### Introduction:

In the previous lab we saw that if we have the standard deviation for the population, we can use use z-score to calculate our confidence interval using mean of sample means. 

If, on the other hand, standard deviation of the population is not known (which is mostly the case), you have to use the standard deviation of your sample as a stand in when creating confidence intervals. Also, as we saw earlier, sample standard deviation may not match the population parameter the interval will have more error when you don't know the population standard deviation. To account for this error, we use what's known as a t-critical value instead of the z-critical value in such cases.

The t-critical value is drawn from what's known as a t-distribution
> A t-distribution closely resembles the normal distribution but gets wider and wider as the sample size falls. A t distribution tends to look like a z distribution as sample size increases

![](http://ci.columbia.edu/ci/premba_test/c0331/images/s7/6317178747.gif)

The t-distribution is available in scipy.stats with the nickname "t" so we can get t-critical values with `stats.t.ppf()`.

Let's take a new, smaller sample and then create a confidence interval without the population standard deviation, using the t-distribution:


```python
# Import the necessary libraries

import numpy as np
import pandas as pd
import scipy.stats as stats
import matplotlib.pyplot as plt
import random
import math
```

Let's investigate point estimates by generating a population of random age data collected a two different locations and then drawing a sample from it to estimate the mean:


```python
np.random.seed(20)
population_ages1 = np.random.normal(20, 4, 10000) 
population_ages2 = np.random.normal(22, 3, 10000) 
population_ages = np.concatenate((population_ages1, population_ages2))

pop_ages = pd.DataFrame(population_ages)
pop_ages.hist(bins=100,range=(5,33),figsize=(9,9))
pop_ages.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>20000.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>21.008578</td>
    </tr>
    <tr>
      <th>std</th>
      <td>3.671277</td>
    </tr>
    <tr>
      <th>min</th>
      <td>4.784588</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>18.662256</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>21.163276</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>23.489438</td>
    </tr>
    <tr>
      <th>max</th>
      <td>36.140116</td>
    </tr>
  </tbody>
</table>
</div>




![png](output_4_1.png)


Let's take a new, smaller sample (<30) and calculate how much sample mean differs from population mean.


```python
np.random.seed(23)

sample_size = 25
sample = np.random.choice(a= population_ages, size = sample_size)
sample_mean = sample.mean()
print ("Sample Mean:", sample_mean)
print ("Mean Difference:", population_ages.mean() - sample_mean)

# Sample Mean: 19.870788629471857
# Mean Difference: 1.1377888781920937
```

    Sample Mean: 19.870788629471857
    Mean Difference: 1.1377888781920937
    

We can see that sample mean differs from population mean by 1.13 years. We can calculate a confidence interval without the population standard deviation, using the t-distribution using `stats.t.ppf(q, df)` function. This function takes in a value for confidence level required (q) with "degree of freedom" (df) .

> degrees of freedom = sample_size -1.


```python
# Cal culate the t-critical value for 95% confidence level for sample taken above. 
t_critical = stats.t.ppf(q = 0.975, df=sample_size-1)  # Get the t-critical value*
print("t-critical value:")                  # Check the t-critical value
print(t_critical)     

# t-critical value:
# 2.0638985616280205
```

    t-critical value:
    2.0638985616280205
    

Calculate the confidence interval of the sample by sigma and calculating margin of error as:
> **sigma = sample_std/√n**

> **Margin of Error = t-critical-value * sigma**

and finally the confidence interval can be calculated as : 

> **Confidence interval = (sample_mean + margin of error, sample_mean - margin of error)**


```python
# Calculate the sample standard deviation
sample_stdev = sample.std()    # Get the sample standard deviation

# Calculate sigma using the formula described above to get population standard deviation estimate
sigma = sample_stdev/math.sqrt(sample_size)  

# Calculate margin of error using t_critical and sigma
margin_of_error = t_critical * sigma

# Calculate the confidence intervals using calculated margin of error 
confidence_interval = (sample_mean - margin_of_error,
                       sample_mean + margin_of_error)  



print("Confidence interval:")
print(confidence_interval)

# Confidence interval:
# (18.4609156900928, 21.280661568850913)
```

    Confidence interval:
    (18.4609156900928, 21.280661568850913)
    

We can verify our calculations by using the Python function stats.t.interval():


```python
stats.t.interval(alpha = 0.95,              # Confidence level
                 df= 24,                    # Degrees of freedom
                 loc = sample_mean,         # Sample mean
                 scale = sigma)             # Standard deviation estimate
```




    (18.4609156900928, 21.280661568850913)



We can see that the calculated confidence interval includes the population mean calculated above.

Lets run the code multiple times to see how often our estimated confidence interval covers the population mean value:

**Write a function using code above that takes in sample data and returns confidence intervals**




```python
# Function to take in sample data and calculate the confidence interval
def conf_interval(sample):
    '''
    Input:  sample 
    Output: Confidence interval
    '''
    n = len(sample)
    x_hat = sample.mean()
    # Calculate the z-critical value using stats.norm.ppf()
    # Note that we use stats.t.ppf with q = 0.975 to get the desired t-critical value 
    # instead of q = 0.95 because the distribution has two tails.

    t = stats.t.ppf(q = 0.975, df=24)  #  t-critical value for 95% confidence
    
    sigma = sample_stdev/math.sqrt(sample_size) 

    # Calculate the margin of error using formula given above
    moe = t * sigma

    # Calculate the confidence interval by applying margin of error to sample mean 
    # (mean - margin of error, mean+ margin of error)
    conf = (x_hat - moe, x_hat + moe)
    
    return conf
```

**Call the function 25 times taking different samples at each iteration and calculating sample mean and confidence intervals**


```python
# set random seed for reproducability
np.random.seed(12)

# Select the sample size 
sample_size = 25

# Initialize lists to store interval and mean values
intervals = []
sample_means = []

# Run a for loop for sampling 25 times and calculate + store confidence interval and sample mean values in lists initialised above

for sample in range(25):
    # Take a random sample of chosen size 
    sample = np.random.choice(a= population_ages, size = sample_size)
    
    # Calculate confidence_interval from function above
    confidence_interval = conf_interval(sample)    

    # Calculate the sample mean 
    sample_mean = sample.mean()
    
    # Calculate and append sample means and conf intervals for each iteration
    sample_means.append(sample_mean)
    intervals.append(confidence_interval)


```

**Plot the confidence intervals along with sample means and population mean**


```python
# Plot the confidence intervals with sample and population means
plt.figure(figsize=(15,9))

# Draw the mean and confidence interval for each sample
plt.errorbar(x=np.arange(0.1, 25, 1), 
             y=sample_means, 
             yerr=[(top-bot)/2 for top,bot in intervals],
             fmt='o')

# Draw the population mean 
plt.hlines(xmin=0, xmax=25,
           y=population_ages.mean(), 
           linewidth=2.0,
           color="red")
```




    <matplotlib.collections.LineCollection at 0x146369490b8>




![png](output_18_1.png)


Just like the last lab, all but one of the 95% confidence intervals overlap the red line marking the true mean. This is to be expected: since a 95% confidence interval captures the true mean 95% of the time, we'd expect our interval to miss the true mean 5% of the time.

## Summary and Conclusion

In this lab we learnt how to use confidence intervals when population standard deviation is not known, and the sample size is small (<30) . We also saw how to construct them from random samples. The lesson differentiates between the use cases for z-score and t-distribution. We also saw how t value can be used to define the confidence interval based on confidence level. 
