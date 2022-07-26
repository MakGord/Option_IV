# Stock Price Probabilty Plot

Project Goal: 
The goal of this project is to create a model that estimates a potential direction and magnitutde of the price movement for a given stock for a given time interval based on the historical data for the given periods. 

Model Inputs: 
- Stock Ticker
- Start Date of the historical interval
- End Date of the histroical interval (Today's Date by default)
- Desired Time Frame (1 day, 1 week, 1 month)
- Price Resolution (Rounding factor)

**Model Outputs:** 
- A csv format table representing a count of Open-High-Low-Close (OHLC) occurances and their total sum for each historical price point. 
- A chart that combines a bar plot of sum of OHLC occurances at each price point (as per table above) and a normal distribution plot calculated with the latest available historical price (as mu) and historical standard deviation (as sigma). 

**Code:** 
1. Getting Historical Data and Extracting Variables
  * Import the historical data as a Pandas DataFrame using YFinance extension.
  ```
  def GetStockData(ticker,start_date,end_date,interval):
    df = yf.download(ticker, start_date, end_date,interval=interval).dropna()
    return(df)
  ```
  * Extract the last available closing stock price from the DataFrame.
  
   ```
  def Last_Price(df):
    Last_Price=df.Close[len(df.Close)-1]
    return(Last_Price) 
   ```
  * Calculate Log Return for each closing period using Numpy extension.
  * Calculate Standard Deviation for the Log Return using Numpy extension. 
  
   ```
  def Get_STD(df):
    df['log_R'] = np.log(df.Close) - np.log(df.Close.shift(1))
    STD=np.std(df.log_R)
    return(STD)
  ```
  
  * Round the DataFrame to the base.
  ```
  def Round(df,base):
      df = base*round(df/base)
      return(df)
  ```

# Conda environment with environment.yml

[![Binder](http://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/MakGord/Stock_Price_Probability_Plot/main?filepath=index.ipynb)

 ```
 ```

A Binder-compatible repo with an `environment.yml` file.

Access this Binder by clicking the blue badge above or at the following URL:

https://mybinder.org/v2/gh/MakGord/Stock_Price_Probability_Plot/main




## Notes
The `environment.yml` file should list all Python libraries on which your notebooks
depend, specified as though they were created using the following `conda` commands:

```
conda activate example-environment
conda env export --from-history -f environment.yml
```



Note that the only libraries available to you will be the ones specified in
the `environment.yml`, so be sure to include everything that you need! 

Also note that if you skip the `--from-history`, conda may include OS-specific
packages in `environment.yml`, which you would have to manually prune from
`environment.yml`.  For example, confirmed macOS-specific packages that should
be removed are:

* libcxxabi=4.0.1
* appnope=0.1.0
* libgfortran=3.0.1
* libcxx=4.0.1
