# Intel stock forecast for the next 5 years using ARIMA

### Visualising and Cleaning the Data

```
data intel_stock;
infile '/home/u40846561/Projects/INTC.csv' dlm = ',' firstobs = 2;   
input Date anydtdte10. Volume;
format Date date10.;
logvolume = log(Volume);
run;

proc sgplot data = intel_stock;
series x = Date y = Volume/markers;	
xaxis values = (1 to 5000 by 1);	
run;
```

<img src="Images/Plot_volume.png" width="500"  >

Firstly, I imported the data and gave the date variable a format that the program could read. This is important such that when we plot our graphs they are easier to read as it will show the exact date for each point instead of some arbitary number. After this, I plotted the data on a time series plot to visualise the data. From this plot I saw that the variance increased over time quite dramatically. This meant I then took a new variable called "logvolume" which is the logarithm of the volume variable to make the variance constant through out the time series plot.

```
proc sgplot data = intel_stock;
series x = Date y = logvolume/markers;	
xaxis values = (1 to 5000 by 1);	
run;
```

<img src="Images/Plot_logvolume.png" width="500"  >

The plot shows a how the variance has been made constant by taking the logarithm of volume. We can also see two significant outliers that are resolved later. The plot however, shows that there is a non-zero mean.

### Building and optimising a ARIMA model

```
proc arima data = intel_stock plots = all;
identify var= logvolume(1);
estimate p = 5 q = 5;
outlier id=Date;
run;
```

Now that I have obtained a time series with constant variance I need to make it stationary and that I need the mean of the model to be zero. We can do this by taking the first-difference of the variable logvolume. I then obtained the ARIMA plots using the code above to check if taking the first-difference worked. Using trial and error I found the best model with the lowest AIC was an ARIMA(5,1,5) model; residual correlation analysis suggests that model is a good fit for the data with now significant residuals in either the ACF or PACF plots. However, the model could be improved by identifying and resolving outliers.

<img src="Images/trend_correlation_analysis_plots.png" width="500"  > <img src="Images/Residual_plots_outliers_ARIMA(5,1,5).png" width="500"  > 

The observation against logvolume(1) plot shows us the effect of taking the first-difference, we can clearly see that the mean is zero and thus we have obtained a stationary time series and can move onto building an ARIMA model. Firstly, I must identify and resolve any outliers; using ``` outlier id=Date; ``` we can identify any outliers by their date. Using this I identified 5 significant outliers:

<img src="Images/Outlier_table_1.png" width="500"  >

The code below attempts to resolve the outliers identified:
```
data intel_stock;
set intel_stock;
if _n_ = 1657 then AO = 1;
else AO = 0.0;
if _n_ = 5237 then AO = 1;
else AO = 0.0;
if _n_ = 2816 then AO = 1;
else AO = 0.0;
if _n_ >= 1720 then LS = 1;
else LS = 0.0;
if _n_ = 5267 then AO = 1;
else AO = 0.0;
run;
```

Code for the ARIMA plots:

```
proc arima data=intel_stock;
identify var=logvolume(1)
crosscorr=( AO(1) LS(1) );
estimate p = 5 q = 5 noint
input=( AO LS )
method=ml plot;
outlier id=Date;
run;
```

<img src="Images/Cross_correlation_plot.png" width="400"  > <img src="Images/Residual_plot.png" width="400"  >
<img src="Images/Residual_plots_NOoutliers_ARIMA(5,1,5).png" width="600"  > <img src="Images/Residual_normality_plots_NOoutliers.png" width="600"  >
