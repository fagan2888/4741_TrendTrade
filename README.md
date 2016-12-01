# 4741 TrendTrade
This is Project for class ORIE 4741 by Tongcheng Li. 

# Final Report

In this project, we want to identify relations between Google Trend data and Stock's daily price and volume.
Specifically, we are interested in the following problems:

Given Daily prices (including: open price, close price, daily high price, daily low price) and volume of 500 S&P500 stocks.
We want to find how information of Google Trends contribute to the formation of prices in the following ways:

### (1) How does the Google Trend of stock abbreviated symbols (for example: AAPL for apple stock) indicate price change of the day, max price difference of the day and volume of the day.

This question is interesting because when Google Trend of a stock in spiked, it could mean some major events happened to the company, for example, Apple launched its new products.


# Step 1: Data Gathering

Our S&P500 price and volume daily data was from https://quantquote.com/historical-stock-data.

This data source claims it considers split and dividend adjustments, therefore this source is better than Yahoo finance data, which didn't include split and divident adjustments.

But for our Google Trend data because (1): Google doesn't have an Official API to query Google Trend, (2): Google's unofficial API (I tried one python library called pytrends) was rate limited in a blackbox fashion which limits the efficient and scalable Google Trend data gathering, I have devise some method to gather the data.

The way I use to gather data is simulate manual process of typing a URL in browser, and click some buttons to download the CSV, and move then rename the CSV to appropriate location with proper name. However, this is complicated by the following factors:

#### (1): The python library used (pyautogui) is not very robust in typing characters, so when the script types a uppercase character, sometimes the following characters can change from lowercase to uppercase, which is distorted by pyautogui library.

This is solved by the data verification process in our Data Cleanup session.

#### (2): The typing error can cause systematic problems, for example, if typing wants to make a uppercase character and if the next thing it want to type is enter, then it will likely to trigger hot key in Chrome, which is (Shift + Enter) to open a new window. Similarly, it can open a new tab.

Opening a new tab or window will systematically destroy our GUI automation to gather data because our data gathering is based on pixel wise operations, therefore we have to eliminate this type of problem.

In OS X environment I am using, in order to prevent new tabs, I used a chrome extension of xtab, which limits the number of tabs you can open. We set this limit to 1. In order to prevent new windows, I used a mac software called Keyboard Maestro, which changes the hotkey actions, to change (Shift + Enter)'s effect to go to Keyboard Maestro's official website, but this effect will be cancelled by new tab limit. Therefore new tab and new window problem is solved.

#### (3): Google seems to block IP if we search too much Google Trends. 

This is solved by using VPN at different locations and using incognito mode in browser. It is necessary to change the VPN every 2 hours (Maybe next step can be changing the VPN automatically).

#### (4): Google Trend has different granularity of output given different length of time range queried.

So Google returns daily granularity for Google Trend for date range queries less than 4 months, and the second level is weekly granularity, and the third level is monthly granularity. Therefore in our case, we query every 3 months.

#### (5): Network latency is hard to predict, therefore need redundant time between clicks.

### So Google Trend data we gathered is the 2010-2012's three year data for each abbreviated symbol. Each symbol requires 12 downloads since we are doing 3 month length query range every download.

# Step 2: Data Cleanup

For CSV files downloaded, we want to verify it is correct before using it since there exist possibility of typing the wrong query before download. First, we merge the CSV file of GoogleTrends to the CSV of prices and volume. So we create a csv file for every 3 months between Year [2010,2012]. As a cleanup, we throw away merge CSV files which have too few trade days or too much trade days, because that would mean something is wrong with it. In particular, we throw away CSV with <= 55 trade days and CSV with >=70 trade days, with the concern that normally there are 21 to 22 trade days per month, and other holidays should not affect every 3 month by more than -8 or +6 days.

### Then we define our problem concretely as follows: For each 3-month time frame, given complete Google Trend data and trade data(only happens on business days, which excludes weekends and holidays), try to model the correlation.

Then we notice the map from Trade entries to Trend entries is a one-to-one but not onto mapping. This is equivalent to saying, for each trade entry, there is a trend entry, but not the other way around.

# Step 3: Correlation Modeling on Complete S&P 500 Symbols
First attempt we try is using all 500 S&P stock symbols, with two perspectives:

### Perspective I:

(1): Using the current day Google Trend data to model trading information.

(2): Using the past 3-day arithmetic average to model trading information.

(3): Using the past 7-day arithmetic average to model trading information.

### Perspective II:

(1): Try to model Volume traded.

(2): Try to model Max Price Difference (Daily High Price - Daily Low Price) of the day.

(3): Try to model (Close price - Open price) of the day.

### Currently the criterion we will examine is for all 3-month windows combined, we look at the histogram of r (the correlation coefficient).

##### The histogram of r using 1 day Google Trend information with volume traded, with mean correlation = 0.097:
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/1dayTrend_Volume_Corr.png" height="240">

From this plot, even though the correlation information seems low, it is actually very high based on stock markets being very noisy. And this plot has some portion with almost 1 correlation which is interesting.

##### The histogram of r using 1 day Google Trend information with (max price of day - min price of day), with mean correlation = 0.0594.

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/1dayTrend_priceMaxMinDiff.png" height="240">

There is certainly a positive correlation between daily price range (can be think of close to volatility) and Google Trends which makes sense: When some big news happens, volatility rises, and on the other hand, people tend to search for stock symbol, which cause Google Trend to rise.

##### The histogram of r using 1 day Google Trend information with (close price of day - open price of day) with mean correlation -0.00513.

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/1dayTrend_priceCloseOpenDiff.png" height="240">

From this plot, we notice that change of price of the day is negatively correlated with Google Trend of the day, this can be explained by volatility is asymmetrical in terms of direction. That is, it is more volatile for price to decrease (when stock price plummeted overnight) but not as volatile for price to increase (when price increase steadily though slowly over 3 months due to Good Expectations).

### Then we examine over different time averages of Google Trend:
### 1-day vs. 3-day vs. 7-day Volatility Correlation: with r1 = 0.097, r3 = 0.101, r7 = 0.073.
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/1dayTrend_Volume_Corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/3dayArithmeticTrend_volume_corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/7dayArithmeticTrend_Volume_Corr.png" height="180">

We can see that Correlation of Trend with Volatility is less spiked as time range considered is longer. This is because information is dispersed. And it also seems 3-day average is most predictive, which can be think of either (1) appropriate time needed (eigen time) for information to propagate to Google Trend or (2) Saturday and Sunday's cumulative effect on Monday.

### 1-day vs. 3-day vs. 7-day (Max Daily Price - Min Daily Price) Correlation: with r1 = 0.0594, r3 = 0.056, r7 = 0.037.
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/1dayTrend_priceMaxMinDiff.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/3dayArithmeticTrend_priceMaxMinDiff_corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/7dayArithmetic_MaxMinPrice_Corr.png" height="180">

### 1-day vs. 3-day vs. 7-day (Daily Close Price - Daily Open Price) Correlation: with r1 = -0.00513, r3 = -0.0055, r7 = -0.0067
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/1dayTrend_priceCloseOpenDiff.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/3dayTrend_CloseOpenDiff_corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/7dayArithmetic_CloseOpenPriceDiff_Corr.png" height="180">

# Step 4: Correlation Modeling on a More Informative Subset of Stock Symbols

Notice that in our select of stock symbols, there are symbols such as 'a', which we don't really know if the person searching 'a' is looking for "Agilent Technologies" which has abbreviation "a" or just typing random things. 

### Therefore we limit the set of symbols we look at to smaller and more informative subset, namely symbols with length >= 3.

### Now the plots over different trade features:

### 1-day vs. 3-day vs. 7-day Volatility Correlation: with r1 = 0.11, r3 = 0.1143, r7 = 0.082.
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/1day_volume_corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/3day_Volume_Corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/7day_Volume_Corr.png" height="180">

We can see that Correlation of Trend with Volatility is less spiked as time range considered is longer. This is because information is dispersed. And it also seems 3-day average is most predictive, which can be think of either (1) appropriate time needed (eigen time) for information to propagate to Google Trend or (2) Saturday and Sunday's cumulative effect on Monday.

### 1-day vs. 3-day vs. 7-day (Max Daily Price - Min Daily Price) Correlation: with r1 = 0.068, r3 = 0.064, r7 = 0.0425.
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/1day_maxminprice_corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/3day_MaxMinPriceDiff_Corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/7day_MaxMinPriceDiff_Corr.png" height="180">

### 1-day vs. 3-day vs. 7-day (Daily Close Price - Daily Open Price) Correlation: with r1 = -0.0044, r3 = -0.0063, r7 = -0.0064
<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/1Day_CloseOpen_Corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/3day_CloseOpenPriceDiff_Corr.png" height="180"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/7day_CloseOpenPriceDiff_Corr.png" height="180">

### After this cleanup, our mean correlation is becoming larger in magnitude, which means filter out short name symbols will increase information in Google Trend signal.

# Step 5: Linear Regression (A Failed Attempt) 
First, we try to model the Volume Traded using past 3 days' Google Trend with Linear Regression.

Intuitively, overall Volume should be normalized because large cap stocks will be traded with larger daily volume therefore if we didn't normalize, the prediction will be biased toward large cap stocks. 

Therefore we attempt with the first kind of normalization for Y, which is dividing the original Y value's 3 month average.

So for example, the scatter plot of GoogleTrend of the day (X variable) and Normalized Volume Traded:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/LR_scatter_withoutNormX.png" height="240">

While using Ordinary Linear Regression, our fitted line looks like:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/LR_Line_withoutNormX.png" height="240">

Combine the scatter plot with Ordinary Linear Regression:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/LR_line_scatter_withoutNormX.png" height="240">

The weights for Linear Regression for Normalized Volume is: coefficient for 2 days before = -2.19 * 1e-3, coefficient for 1 day before = 3.73 * 1e-4, coefficient for current day is 2.99* 1e-3, intercept is 9.24*1e-1.

Notice that in the line plot, even though our X axis is only current day's google trend, the corresponding Y is the predicted Y that considers Google Trend 2 days before and Google Trend 1 day before.

Similarly, We build a Normalized Price for Max - Min price, called NormalizedMaxMinRatio = (Max price - Min price)/mean_over3month(Price).

This regression has regression weight -3.47 * 1e-5 for Google Trend 2 days before, weight -7.02 * 1e-6 for Google Trend 1 day before, weight 4.85 * 1e-5 for current day's Google Trend and intercept 2.3 * 1e-2.

For Normalized Price for Close - Open price, called NormalizedCloseOpenRatio = (Close price - Open price)/mean_over3month(Price).

This regression has regression weight -2.003 * 1e-6 for Google Trend 2 days before, weight 9.91 * 1e-7 for Google Trend 1 day before, weight -9.24 * 1e-7 for current day's Google Trend and intercept 2.77 * 1e-4.

This means simply using Linear Regression is not satisfactory, because it offer almost no predictive power in this case.

# Step 6: Linear Regression with Feature Engineering

Next, we define what really should be our objectives and use cases of Google Trend. 

Qualitatively the following holds: Google Trend of a stock's abbreviation, as a indicator, varies some amount by each day, and some proportion of the variation is correlated with the stock's volume or price change.

But this does not necessarily mean this qualitative statement can be easily converted to a quantitative statement. Consider the following 2 examples: 

Example 1: ABC, which stands for AmerisourceBergen Corp, a healthcare company, can also refer to English Alphabet in general (i.e. This 6 year old boy is just learning his ABCs). And when people are typing ABC, most of the cases are they are referring to the English Alphabet, therefore the Google Trend for ABC have a very small signal to noise ratio. 

Example 2: AAPL, which stands for Apple Inc, is not naturally a word. And at least as far as I know, AAPL does not have any connotations other than referring to the company Apple. Therefore the Google Trend for AAPL have a large signal to noise ratio.

Based on this observation, intuitively we want to "normalize" the signal to noise ratio for each company so that each abbreviation's Google Trend, after some feature transformation, should have similar impacts. 

We do this by the classical trick: First remove the mean, then devide by standard deviation. (Both mean and standard deviation is calculated using the 3 month horizen.)

Similarly, we have similar considerations for Y variables (Volume, (Max - Min) daily price,(Close - Open) daily price). For example, some small cap company can be very volatile but most large cap companies tend to be not very volatile (for most time).

We treat the Y values by also using Z-score as feature transformation (remove mean than devide by standard deviation).

As an experimental exploration, let's just focus on Volume traded for now.

The following plot gives us what the scattered points and line looks like:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/LR_Z_ScatterLine.png" height="240">

The following plot gives us what the regression line looks like (Notice that the regressed line, even though we only showed the x-axis with ) :

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/LR_Z_line.png" height="240">

Calculated over the entire dataset we have the weight vector for Y as Volume traded: weight = -0.00025 for Z-score of Google Trend two days before, weight = 0.023 for Z-score of Google Trend one day before, weight = 0.0534 for Google Trend of the current day and intercept weight = -0.00038.

The above plots are made using 1000 data points drawn randomly from the dataset.

Then we calculate the mean absolute error, which turns out to be 0.314, this looks moderately decent.

The scatter plot for the entire dataset looks like following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/Scatter_Z.png" height="240">

The following plot gives us the error distribution in terms of Z-score of the volume:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/error_Distribution_Zscore.png" height="240">

According to the error Distribution plot, the error distribution do have fat tail in the negative direction. And Z = Z_pred - Z_true = -4 does occur for few data points' predictions. I think this corresponds to Z_true = Z_pred + 4, which is underestimated Volume traded.

Error of 4 standard deviation is absolutely a phenomena. This corresponds to severely under-estimated volume traded which corresponds to black swan events (event that is totally unexpected, originally from the scenario when you suppose a group of swans in the lake are all white swans, until you see a black swan, which changes your previous hypothesis) but statistically black swan events happens more often than expected.

On the other hand, the mode of the error distribution is in Z = +0.3, this verifies what we already know about the ordinary linear regression use maximum likelihood estimation, which is sensitive to outliers.

With Feature Engineering, this Linear Regression Model tells us the following improvements:

(1): First of all, there is a common bias called future function bias in financial modeling, that is you use the information that can only be accessed in the future to model what you could know currently. Our model is prone to this bias because we use Google Trend of that day. To avoid this bias, we will try to use only information in 1 previous day or more.

(2): There is certainly some black swan events that make certain cases very volatile (So much more volume traded). One way to model such events are using quantile regression for top quantiles, modeling such events are meaningful, as they could, in theory, provide us profitable opportunities if we can foresee black-swan events because people are less rational and more error-prone in trading during black-swan (or totally unexpected) events.

(3): Similar to quantile regression, we could find companies that more the best fit for our approach. That is, for every company, we evaluate the effectiveness of our model, and we only apply the model to the most effective set of companies.

This corresponds to the real world case where our alpha-generating method has is conditional on the companies selected.

# Step 7: Future-Bias Free Linear Regression.

Doing the same thing for Volume using Z-score of Google Trend 3 days before, 2 days before and 1 day before, we have a Linear Regression for Z-score without the bias of looking into future. 

The mean absolute error of the regression is 0.307, which makes it slightly better than previous result of step 6. (Though I don't think there is a systematic cause behind this.) 

The weight of regressions is: weight for Google Trend Z-score 3 days before is 0.0025, weight for 2 days before is 0.0045, weight for 1 day before is 0.038, intercept is -0.0009.

The plot of scattered data points and regressed line is the following, the x-axis is only the Google Trend Z-score 1 day before while the predicted y consider all 3 days Google Trend information:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/FB_Free_LineScatter.png" height="240">

The plot for regression line is the following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/FB_Free_Line.png" height="240">

The error distribution is the following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/All500S%26Pplots/FB_Free_errorHist.png" height="240">

# Step 8: Quantile Regression

Then I try Quantile Regression to model the top Volume changes, quantile regression models the response variable (y) for a given quantile (q) conditioned on variable x. In my case, variable x is vector with 3 elements, namely: Z-score of Google Trend 3 days before, Z-score Google Trend 2 days before and Z-score Google Trend 1 day before. The Y variable is Z-score of the volume traded. 

The 10 quantile lines for quantile = {50,55,60,...,95} is plotted as follows:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/50_95_5_QuantileLines.png" height="240">

The picture is the ten regression lines, the higher up the line is, the higher up percentile it represents. For ease of visualization, the x-axis only included the 1 day before Google Trend Z-score. Though the predicted y for certain percentile included all 3 days' information.

The regression line within a subset of datapoints is the following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/50_95_5_QuantileScatter.png" height="240">

The regression line within all datapoints is the following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/50_95_5_bigScatter.png" height="240">

If we plot the quantile regression for quantile = {0.9,0.91,0.92,...,0.99}, the ten regression line is the following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/90_99_1_Line.png" height="240">

Within some subset of data points (Left) and all datapoints (Right), the regression line will be the following:

<img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/90_99_1_Scatter.png" height="240"> <img src="https://github.com/Tongcheng/4741_TrendTrade/blob/master/LongNameS%26P500/90_99_1_BigScatter.png" height="240">



