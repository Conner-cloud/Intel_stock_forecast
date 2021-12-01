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

<img src="Images/Plot_logvolume.png" width="500"  >
