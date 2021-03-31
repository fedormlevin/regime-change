# regime-change
# Summary
I am forecasting US recessions based on macroeconomic indicators from 1984-2020
# Results
Logistic Regression model trained on 1984-2009 data forecasted 2020 recession 1 month in advance of the official data from FRED
<br>
![image info](./regime_change.png)
<br>
## Macroeconomic indicators
After running the model for the first time and extracting features importance I selected the following indicators picking the ones with low correlation:
<br>
| Bucket                          | Indicator  | Details                                           |
|---------------------------------|------------|---------------------------------------------------|
| Bond Market and Monetary Policy | T1YFFM     | 1-Year Treasury C Minus FEDFUNDS                  |
| Bond Market and Monetary Policy | TERMSPREAD | Term Spread (10-year T-bill minus 3-month T-bill) |
| Bond Market and Monetary Policy | T10YFFM    | 10-Year Treasury C Minus FEDFUNDS                 |
| Bond Market and Monetary Policy | T5YFFM     | 5-Year Treasury C Minus FEDFUNDS                  |
| Real Estate Market              | HOUST      | Housing Starts: Total New Privately Owned         |
| Employment                      | AWOTMAN    | Avg Weekly Overtime Hours : Manufacturing         |
| Employment                      | ICSA       | Initial Claims, Monthly                           |
| Employment                      | UNRATE     | Unemployment Rate                                 |
| Economic growth                 | SP500      | Index Growth                                      |
| Foreign Exchange                | EXUSUK     | U.S. / U.K. Foreign Exchange Rate                 |
<br>

![image info](./feat_imp.png)
### Feature extraction and resampling
```python
recession = mitk.recession_nber() #monthly
T10YFFM_d = mitk.tenyear_cm_minus_fedfunds() #monthly
TB3SMFFM_d = mitk.three_month_tbill_minus_fedfunds() #monthly
PAYEMS_d = mitk.all_employee() #monthly
JTSJOL_d = mitk.job_open() #monthly
T5YFFM_d = mitk.five_year_cm_minus_fedfund() #monthly
T1YFFM_d = mitk.one_year_cm_minus_fedfund() #monthly
EXUSUK_d = mitk.us_uk_fx() #monthly
HOUSTW_d = mitk.housing_stats_west_census() #monthly
TB3MS_d = mitk.three_month_trs_secondary() #monthly
USCONS_d = mitk.employee_constr() #monthly
AWOTMAN_d = mitk.overtime_manuf() #monthly
HOUST_d = mitk.housing_owened() #monthly
DMANEMP_d = mitk.employee_durable() #monthly
SRVPRD_d = mitk.employee_service() #monthly
ICSA_d = mitk.initial_claims() #daily
ICSA_d = ICSA_d.resample('M').mean() #resampling daily to monthly
UNRATE_d = mitk.unemployment_rate_us() #monthly
sp500 = fltk.hist_returns(['^GSPC']) #daily
sp500_m = (1+sp500).resample('M').prod()-1 #resampling daily to monthly
sp500_m = sp500_m.rename(columns={'^GSPC': 'SP500_M'}) #changing column name
termspread = mitk.spread_tenyr_threemon_tr() #daily
termspread = termspread.resample('M').mean() #resampling daily to monthly
gdp = mitk.gdp_us() #quaterly
gdp = gdp.resample('M').mean() #resampling quaterly to monthly
gdp = gdp.interpolate(method='nearest') #interpolating gaps
gdp = gdp.pct_change() #percent change
```
Data is pulled from FRED using their API. All functions are located in macro_indicators_toolkit.py file.
Sample function:
```python
def three_month_trs_secondary():
    """
    3-Month Treasury Bill: Secondary Market Rate, Monthly
    """
    threems = fred.get_series('TB3MS')
    threems.dropna(inplace=True)
    threems = threems.rename('TB3MS')
    return threems
```
