# covid-19
An analysis of John Hopkins University COVID-19 time series data.

The information below is the Jupyter notebook, contained in this repository, converted to markdown.

---
## Analysis of Coronavirus (COVID-19) Pandemic Data 

The purpose of the notebook is to evaluate the coronavirus (COVID-19) data provided by the Center for Systems Science and Engineering (CSSE) at Johns Hopkins University (JHU).  This workbook only evaluates the time series dataset provided by JHU-CSSE.

As of March 17, 2020 this notebook is still a work in progress.  Most disappointing is the updates from the JHU dataset for U.S. county reporting seem to have been discontinued. All the datasets from JHU seem to lag behind state reporting.

#### Data Source: 
CSSE-JHU COVID-19 data repository https://github.com/CSSEGISandData/COVID-19  

---

#### Python Libraries


```python
import pandas as pd
import matplotlib as mpl
import matplotlib.pyplot as plt
from matplotlib import pyplot as plt

import requests
import numpy as np
%matplotlib inline
import seaborn as sns
```

    /Users/mark/anaconda3/envs/sandbox2/lib/python3.7/site-packages/statsmodels/tools/_testing.py:19: FutureWarning: pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead.
      import pandas.util.testing as tm


*** The warning message above is the result of the seaborn package needing to be updated.


```python
#plt.style.use('ggplot')
plt.style.use('fivethirtyeight')
```

#### Load csv data into dataframes


```python
# confirmed cases data
confirm_url = 'https://github.com/CSSEGISandData/COVID-19/raw/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Confirmed.csv'

# recovered cases data
recover_url = 'https://github.com/CSSEGISandData/COVID-19/raw/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Recovered.csv'

# death cases data
deaths_url = 'https://github.com/CSSEGISandData/COVID-19/raw/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Deaths.csv'

# create dataframes
confirm_df = pd.read_csv(confirm_url)
recover_df = pd.read_csv(recover_url)
deaths_df = pd.read_csv(deaths_url)

```


```python
state_abbr_fips_df = pd.read_csv('state_abbr_fips.csv', header=None, dtype=object, names=['NAME','state_fips','state_abbr'])
state_abbr_fips_df.head(15)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>state_fips</th>
      <th>state_abbr</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alabama</td>
      <td>01</td>
      <td>AL</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alaska</td>
      <td>02</td>
      <td>AK</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Arizona</td>
      <td>04</td>
      <td>AZ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Arkansas</td>
      <td>05</td>
      <td>AR</td>
    </tr>
    <tr>
      <th>4</th>
      <td>California</td>
      <td>06</td>
      <td>CA</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Colorado</td>
      <td>08</td>
      <td>CO</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Connecticut</td>
      <td>09</td>
      <td>CT</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Delaware</td>
      <td>10</td>
      <td>DE</td>
    </tr>
    <tr>
      <th>8</th>
      <td>District of Columbia</td>
      <td>11</td>
      <td>DC</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Florida</td>
      <td>12</td>
      <td>FL</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Georgia</td>
      <td>13</td>
      <td>GA</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Hawaii</td>
      <td>15</td>
      <td>HI</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Idaho</td>
      <td>16</td>
      <td>ID</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Illinois</td>
      <td>17</td>
      <td>IL</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Indiana</td>
      <td>18</td>
      <td>IN</td>
    </tr>
  </tbody>
</table>
</div>



#### Set dataframe window viewing size


```python
pd.set_option('display.max_rows', 1000)
```


```python
confirm_df.head(15)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Province/State</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Thailand</td>
      <td>15.0000</td>
      <td>101.0000</td>
      <td>2</td>
      <td>3</td>
      <td>5</td>
      <td>7</td>
      <td>8</td>
      <td>8</td>
      <td>...</td>
      <td>50</td>
      <td>50</td>
      <td>50</td>
      <td>53</td>
      <td>59</td>
      <td>70</td>
      <td>75</td>
      <td>82</td>
      <td>114</td>
      <td>147</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>Japan</td>
      <td>36.0000</td>
      <td>138.0000</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
      <td>4</td>
      <td>...</td>
      <td>461</td>
      <td>502</td>
      <td>511</td>
      <td>581</td>
      <td>639</td>
      <td>639</td>
      <td>701</td>
      <td>773</td>
      <td>839</td>
      <td>825</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>Singapore</td>
      <td>1.2833</td>
      <td>103.8333</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>4</td>
      <td>5</td>
      <td>...</td>
      <td>138</td>
      <td>150</td>
      <td>150</td>
      <td>160</td>
      <td>178</td>
      <td>178</td>
      <td>200</td>
      <td>212</td>
      <td>226</td>
      <td>243</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>Nepal</td>
      <td>28.1667</td>
      <td>84.2500</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>Malaysia</td>
      <td>2.5000</td>
      <td>112.5000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>4</td>
      <td>...</td>
      <td>93</td>
      <td>99</td>
      <td>117</td>
      <td>129</td>
      <td>149</td>
      <td>149</td>
      <td>197</td>
      <td>238</td>
      <td>428</td>
      <td>566</td>
    </tr>
    <tr>
      <th>5</th>
      <td>British Columbia</td>
      <td>Canada</td>
      <td>49.2827</td>
      <td>-123.1207</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>21</td>
      <td>27</td>
      <td>32</td>
      <td>32</td>
      <td>39</td>
      <td>46</td>
      <td>64</td>
      <td>64</td>
      <td>73</td>
      <td>103</td>
    </tr>
    <tr>
      <th>6</th>
      <td>New South Wales</td>
      <td>Australia</td>
      <td>-33.8688</td>
      <td>151.2093</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>...</td>
      <td>28</td>
      <td>38</td>
      <td>48</td>
      <td>55</td>
      <td>65</td>
      <td>65</td>
      <td>92</td>
      <td>112</td>
      <td>134</td>
      <td>171</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Victoria</td>
      <td>Australia</td>
      <td>-37.8136</td>
      <td>144.9631</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>11</td>
      <td>11</td>
      <td>15</td>
      <td>18</td>
      <td>21</td>
      <td>21</td>
      <td>36</td>
      <td>49</td>
      <td>57</td>
      <td>71</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Queensland</td>
      <td>Australia</td>
      <td>-28.0167</td>
      <td>153.4000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>13</td>
      <td>15</td>
      <td>15</td>
      <td>18</td>
      <td>20</td>
      <td>20</td>
      <td>35</td>
      <td>46</td>
      <td>61</td>
      <td>68</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NaN</td>
      <td>Cambodia</td>
      <td>11.5500</td>
      <td>104.9167</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>5</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
    </tr>
    <tr>
      <th>10</th>
      <td>NaN</td>
      <td>Sri Lanka</td>
      <td>7.0000</td>
      <td>81.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>10</td>
      <td>18</td>
      <td>28</td>
    </tr>
    <tr>
      <th>11</th>
      <td>NaN</td>
      <td>Germany</td>
      <td>51.0000</td>
      <td>9.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>799</td>
      <td>1040</td>
      <td>1176</td>
      <td>1457</td>
      <td>1908</td>
      <td>2078</td>
      <td>3675</td>
      <td>4585</td>
      <td>5795</td>
      <td>7272</td>
    </tr>
    <tr>
      <th>12</th>
      <td>NaN</td>
      <td>Finland</td>
      <td>64.0000</td>
      <td>26.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>15</td>
      <td>23</td>
      <td>30</td>
      <td>40</td>
      <td>59</td>
      <td>59</td>
      <td>155</td>
      <td>225</td>
      <td>244</td>
      <td>277</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>United Arab Emirates</td>
      <td>24.0000</td>
      <td>54.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>45</td>
      <td>45</td>
      <td>45</td>
      <td>74</td>
      <td>74</td>
      <td>85</td>
      <td>85</td>
      <td>85</td>
      <td>98</td>
      <td>98</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Philippines</td>
      <td>13.0000</td>
      <td>122.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>6</td>
      <td>10</td>
      <td>20</td>
      <td>33</td>
      <td>49</td>
      <td>52</td>
      <td>64</td>
      <td>111</td>
      <td>140</td>
      <td>142</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 59 columns</p>
</div>




```python
recover_df.head(15)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Province/State</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Thailand</td>
      <td>15.0000</td>
      <td>101.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>...</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>33</td>
      <td>34</td>
      <td>34</td>
      <td>35</td>
      <td>35</td>
      <td>35</td>
      <td>35</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>Japan</td>
      <td>36.0000</td>
      <td>138.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>76</td>
      <td>76</td>
      <td>76</td>
      <td>101</td>
      <td>118</td>
      <td>118</td>
      <td>118</td>
      <td>118</td>
      <td>118</td>
      <td>144</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>Singapore</td>
      <td>1.2833</td>
      <td>103.8333</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>78</td>
      <td>78</td>
      <td>78</td>
      <td>78</td>
      <td>96</td>
      <td>96</td>
      <td>97</td>
      <td>105</td>
      <td>105</td>
      <td>109</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>Nepal</td>
      <td>28.1667</td>
      <td>84.2500</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>Malaysia</td>
      <td>2.5000</td>
      <td>112.5000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>23</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>35</td>
      <td>42</td>
      <td>42</td>
    </tr>
    <tr>
      <th>5</th>
      <td>British Columbia</td>
      <td>Canada</td>
      <td>49.2827</td>
      <td>-123.1207</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>New South Wales</td>
      <td>Australia</td>
      <td>-33.8688</td>
      <td>151.2093</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Victoria</td>
      <td>Australia</td>
      <td>-37.8136</td>
      <td>144.9631</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Queensland</td>
      <td>Australia</td>
      <td>-28.0167</td>
      <td>153.4000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NaN</td>
      <td>Cambodia</td>
      <td>11.5500</td>
      <td>104.9167</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>NaN</td>
      <td>Sri Lanka</td>
      <td>7.0000</td>
      <td>81.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>NaN</td>
      <td>Germany</td>
      <td>51.0000</td>
      <td>9.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>25</td>
      <td>25</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>67</td>
    </tr>
    <tr>
      <th>12</th>
      <td>NaN</td>
      <td>Finland</td>
      <td>64.0000</td>
      <td>26.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>10</td>
      <td>10</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>United Arab Emirates</td>
      <td>24.0000</td>
      <td>54.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>12</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>23</td>
      <td>23</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Philippines</td>
      <td>13.0000</td>
      <td>122.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 59 columns</p>
</div>




```python
deaths_df.head(15)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Province/State</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Thailand</td>
      <td>15.0000</td>
      <td>101.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>Japan</td>
      <td>36.0000</td>
      <td>138.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>6</td>
      <td>6</td>
      <td>10</td>
      <td>10</td>
      <td>15</td>
      <td>16</td>
      <td>19</td>
      <td>22</td>
      <td>22</td>
      <td>27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>Singapore</td>
      <td>1.2833</td>
      <td>103.8333</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>Nepal</td>
      <td>28.1667</td>
      <td>84.2500</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>Malaysia</td>
      <td>2.5000</td>
      <td>112.5000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>British Columbia</td>
      <td>Canada</td>
      <td>49.2827</td>
      <td>-123.1207</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>New South Wales</td>
      <td>Australia</td>
      <td>-33.8688</td>
      <td>151.2093</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Victoria</td>
      <td>Australia</td>
      <td>-37.8136</td>
      <td>144.9631</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Queensland</td>
      <td>Australia</td>
      <td>-28.0167</td>
      <td>153.4000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NaN</td>
      <td>Cambodia</td>
      <td>11.5500</td>
      <td>104.9167</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>NaN</td>
      <td>Sri Lanka</td>
      <td>7.0000</td>
      <td>81.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>NaN</td>
      <td>Germany</td>
      <td>51.0000</td>
      <td>9.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>7</td>
      <td>9</td>
      <td>11</td>
      <td>17</td>
    </tr>
    <tr>
      <th>12</th>
      <td>NaN</td>
      <td>Finland</td>
      <td>64.0000</td>
      <td>26.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>United Arab Emirates</td>
      <td>24.0000</td>
      <td>54.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Philippines</td>
      <td>13.0000</td>
      <td>122.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>5</td>
      <td>8</td>
      <td>11</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 59 columns</p>
</div>



### US & Individual State dataframe setup

#### Create a dataframe without null values and does not contain state localities


```python
confirm_US_df = confirm_df[~confirm_df['Province/State'].isnull()].copy()

confirm_US_df = confirm_US_df[~confirm_US_df['Province/State'].str.contains(',') & (confirm_US_df['Country/Region']=='US')].sort_values('Province/State').copy()

confirm_US_df = confirm_US_df.rename(columns={'Province/State': 'state'})
confirm_US_df_timeline = confirm_US_df.copy()
confirm_US_df_timeline.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>417</th>
      <td>Alabama</td>
      <td>US</td>
      <td>32.3182</td>
      <td>-86.9023</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>6</td>
      <td>12</td>
      <td>29</td>
    </tr>
    <tr>
      <th>141</th>
      <td>Alaska</td>
      <td>US</td>
      <td>61.3707</td>
      <td>-152.4044</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>118</th>
      <td>Arizona</td>
      <td>US</td>
      <td>33.7298</td>
      <td>-111.4312</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>6</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>12</td>
      <td>13</td>
      <td>18</td>
    </tr>
    <tr>
      <th>142</th>
      <td>Arkansas</td>
      <td>US</td>
      <td>34.9697</td>
      <td>-92.3731</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>6</td>
      <td>6</td>
      <td>12</td>
      <td>16</td>
      <td>22</td>
    </tr>
    <tr>
      <th>100</th>
      <td>California</td>
      <td>US</td>
      <td>36.1162</td>
      <td>-119.6816</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>144</td>
      <td>177</td>
      <td>221</td>
      <td>282</td>
      <td>340</td>
      <td>426</td>
      <td>557</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 59 columns</p>
</div>



#### Simplify the datafram containing the US confirmed cases


```python
num_columns = len(confirm_US_df.columns)

confirm_US_df['confirmed'] = confirm_US_df[confirm_US_df.columns[4: num_columns]].max(axis=1).astype(int)
confirm_US_df = confirm_US_df[['state','Lat','Long','confirmed']].copy()
#sort descending by total count
confirm_US_df[['state','confirmed']].sort_values(by= ['confirmed'], ascending=False).style.hide_index()
```

<table id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1" ><thead>    <tr>        <th class="col_heading level0 col0" >state</th>        <th class="col_heading level0 col1" >confirmed</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row0_col0" class="data row0 col0" >New York</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row0_col1" class="data row0 col1" >967</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row1_col0" class="data row1 col0" >Washington</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row1_col1" class="data row1 col1" >904</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row2_col0" class="data row2 col0" >California</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row2_col1" class="data row2 col1" >557</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row3_col0" class="data row3 col0" >Massachusetts</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row3_col1" class="data row3 col1" >197</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row4_col0" class="data row4 col0" >New Jersey</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row4_col1" class="data row4 col1" >178</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row5_col0" class="data row5 col0" >Colorado</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row5_col1" class="data row5 col1" >160</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row6_col0" class="data row6 col0" >Florida</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row6_col1" class="data row6 col1" >155</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row7_col0" class="data row7 col0" >Louisiana</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row7_col1" class="data row7 col1" >136</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row8_col0" class="data row8 col0" >Georgia</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row8_col1" class="data row8 col1" >121</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row9_col0" class="data row9 col0" >Illinois</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row9_col1" class="data row9 col1" >105</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row10_col0" class="data row10 col0" >Texas</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row10_col1" class="data row10 col1" >85</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row11_col0" class="data row11 col0" >Pennsylvania</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row11_col1" class="data row11 col1" >77</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row12_col0" class="data row12 col0" >Minnesota</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row12_col1" class="data row12 col1" >54</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row13_col0" class="data row13 col0" >Michigan</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row13_col1" class="data row13 col1" >53</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row14_col0" class="data row14 col0" >Tennessee</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row14_col1" class="data row14 col1" >52</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row15_col0" class="data row15 col0" >Ohio</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row15_col1" class="data row15 col1" >50</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row16_col0" class="data row16 col0" >Virginia</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row16_col1" class="data row16 col1" >49</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row17_col0" class="data row17 col0" >Wisconsin</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row17_col1" class="data row17 col1" >47</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row18_col0" class="data row18 col0" >Diamond Princess</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row18_col1" class="data row18 col1" >47</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row19_col0" class="data row19 col0" >Nevada</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row19_col1" class="data row19 col1" >45</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row20_col0" class="data row20 col0" >Maryland</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row20_col1" class="data row20 col1" >41</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row21_col0" class="data row21 col0" >Utah</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row21_col1" class="data row21 col1" >39</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row22_col0" class="data row22 col0" >Oregon</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row22_col1" class="data row22 col1" >39</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row23_col0" class="data row23 col0" >North Carolina</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row23_col1" class="data row23 col1" >38</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row24_col0" class="data row24 col0" >South Carolina</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row24_col1" class="data row24 col1" >33</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row25_col0" class="data row25 col0" >Connecticut</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row25_col1" class="data row25 col1" >30</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row26_col0" class="data row26 col0" >Alabama</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row26_col1" class="data row26 col1" >29</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row27_col0" class="data row27 col0" >Indiana</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row27_col1" class="data row27 col1" >25</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row28_col0" class="data row28 col0" >Iowa</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row28_col1" class="data row28 col1" >23</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row29_col0" class="data row29 col0" >Arkansas</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row29_col1" class="data row29 col1" >22</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row30_col0" class="data row30 col0" >District of Columbia</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row30_col1" class="data row30 col1" >22</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row31_col0" class="data row31 col0" >Rhode Island</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row31_col1" class="data row31 col1" >21</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row32_col0" class="data row32 col0" >Kentucky</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row32_col1" class="data row32 col1" >21</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row33_col0" class="data row33 col0" >Grand Princess</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row33_col1" class="data row33 col1" >21</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row34_col0" class="data row34 col0" >Nebraska</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row34_col1" class="data row34 col1" >18</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row35_col0" class="data row35 col0" >Arizona</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row35_col1" class="data row35 col1" >18</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row36_col0" class="data row36 col0" >Maine</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row36_col1" class="data row36 col1" >17</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row37_col0" class="data row37 col0" >New Hampshire</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row37_col1" class="data row37 col1" >17</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row38_col0" class="data row38 col0" >New Mexico</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row38_col1" class="data row38 col1" >17</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row39_col0" class="data row39 col0" >Mississippi</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row39_col1" class="data row39 col1" >13</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row40_col0" class="data row40 col0" >Vermont</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row40_col1" class="data row40 col1" >12</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row41_col0" class="data row41 col0" >Kansas</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row41_col1" class="data row41 col1" >11</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row42_col0" class="data row42 col0" >Oklahoma</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row42_col1" class="data row42 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row43_col0" class="data row43 col0" >South Dakota</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row43_col1" class="data row43 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row44_col0" class="data row44 col0" >Delaware</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row44_col1" class="data row44 col1" >8</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row45_col0" class="data row45 col0" >Montana</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row45_col1" class="data row45 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row46_col0" class="data row46 col0" >Hawaii</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row46_col1" class="data row46 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row47_col0" class="data row47 col0" >Missouri</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row47_col1" class="data row47 col1" >6</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row48_col0" class="data row48 col0" >Puerto Rico</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row48_col1" class="data row48 col1" >5</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row49_col0" class="data row49 col0" >Idaho</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row49_col1" class="data row49 col1" >5</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row50_col0" class="data row50 col0" >Guam</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row50_col1" class="data row50 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row51_col0" class="data row51 col0" >Wyoming</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row51_col1" class="data row51 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row52_col0" class="data row52 col0" >North Dakota</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row52_col1" class="data row52 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row53_col0" class="data row53 col0" >Virgin Islands</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row53_col1" class="data row53 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row54_col0" class="data row54 col0" >Alaska</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row54_col1" class="data row54 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row55_col0" class="data row55 col0" >West Virginia</td>
                        <td id="T_ed5d74ee_686b_11ea_b34d_acbc328f6cc1row55_col1" class="data row55 col1" >0</td>
            </tr>
    </tbody></table>




```python
confirm_US_df['confirmed'].sum()
```




    4633



#### Create a dataframe without null values and does not contain state localities


```python
recover_US_df = recover_df[~recover_df['Province/State'].isnull()].copy()

recover_US_df = recover_US_df[~recover_US_df['Province/State'].str.contains(',') & (recover_US_df['Country/Region']=='US')].sort_values('Province/State').copy()

recover_US_df = recover_US_df.rename(columns={'Province/State': 'state'})
recover_US_df_timeline = recover_US_df.copy()
recover_US_df_timeline.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>417</th>
      <td>Alabama</td>
      <td>US</td>
      <td>32.3182</td>
      <td>-86.9023</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>141</th>
      <td>Alaska</td>
      <td>US</td>
      <td>61.3707</td>
      <td>-152.4044</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>118</th>
      <td>Arizona</td>
      <td>US</td>
      <td>33.7298</td>
      <td>-111.4312</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>142</th>
      <td>Arkansas</td>
      <td>US</td>
      <td>34.9697</td>
      <td>-92.3731</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>100</th>
      <td>California</td>
      <td>US</td>
      <td>36.1162</td>
      <td>-119.6816</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 59 columns</p>
</div>



#### Simplify the dataframe containing the US `recovered` cases


```python
num_columns = len(recover_US_df.columns)

recover_US_df['recovered'] = recover_US_df[recover_US_df.columns[4: num_columns]].max(axis=1).astype(int)
recover_US_df = recover_US_df[['state','Lat','Long','recovered']].copy()
#sort descending by total count
recover_US_df[['state','recovered']].sort_values(by= ['recovered'], ascending=False).style.hide_index()
```

<table id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1" ><thead>    <tr>        <th class="col_heading level0 col0" >state</th>        <th class="col_heading level0 col1" >recovered</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row0_col0" class="data row0 col0" >California</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row0_col1" class="data row0 col1" >6</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row1_col0" class="data row1 col0" >Maryland</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row1_col1" class="data row1 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row2_col0" class="data row2 col0" >Illinois</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row2_col1" class="data row2 col1" >2</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row3_col0" class="data row3 col0" >Arizona</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row3_col1" class="data row3 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row4_col0" class="data row4 col0" >Wisconsin</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row4_col1" class="data row4 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row5_col0" class="data row5 col0" >New Jersey</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row5_col1" class="data row5 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row6_col0" class="data row6 col0" >Washington</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row6_col1" class="data row6 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row7_col0" class="data row7 col0" >Kentucky</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row7_col1" class="data row7 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row8_col0" class="data row8 col0" >Massachusetts</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row8_col1" class="data row8 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row9_col0" class="data row9 col0" >Alabama</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row9_col1" class="data row9 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row10_col0" class="data row10 col0" >Oklahoma</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row10_col1" class="data row10 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row11_col0" class="data row11 col0" >Ohio</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row11_col1" class="data row11 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row12_col0" class="data row12 col0" >North Carolina</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row12_col1" class="data row12 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row13_col0" class="data row13 col0" >North Dakota</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row13_col1" class="data row13 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row14_col0" class="data row14 col0" >Pennsylvania</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row14_col1" class="data row14 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row15_col0" class="data row15 col0" >New York</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row15_col1" class="data row15 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row16_col0" class="data row16 col0" >New Mexico</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row16_col1" class="data row16 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row17_col0" class="data row17 col0" >Oregon</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row17_col1" class="data row17 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row18_col0" class="data row18 col0" >Rhode Island</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row18_col1" class="data row18 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row19_col0" class="data row19 col0" >Puerto Rico</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row19_col1" class="data row19 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row20_col0" class="data row20 col0" >Nevada</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row20_col1" class="data row20 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row21_col0" class="data row21 col0" >South Carolina</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row21_col1" class="data row21 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row22_col0" class="data row22 col0" >South Dakota</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row22_col1" class="data row22 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row23_col0" class="data row23 col0" >Tennessee</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row23_col1" class="data row23 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row24_col0" class="data row24 col0" >Texas</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row24_col1" class="data row24 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row25_col0" class="data row25 col0" >Utah</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row25_col1" class="data row25 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row26_col0" class="data row26 col0" >Vermont</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row26_col1" class="data row26 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row27_col0" class="data row27 col0" >Virgin Islands</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row27_col1" class="data row27 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row28_col0" class="data row28 col0" >Virginia</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row28_col1" class="data row28 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row29_col0" class="data row29 col0" >West Virginia</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row29_col1" class="data row29 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row30_col0" class="data row30 col0" >New Hampshire</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row30_col1" class="data row30 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row31_col0" class="data row31 col0" >Missouri</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row31_col1" class="data row31 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row32_col0" class="data row32 col0" >Nebraska</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row32_col1" class="data row32 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row33_col0" class="data row33 col0" >Montana</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row33_col1" class="data row33 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row34_col0" class="data row34 col0" >Arkansas</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row34_col1" class="data row34 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row35_col0" class="data row35 col0" >Colorado</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row35_col1" class="data row35 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row36_col0" class="data row36 col0" >Connecticut</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row36_col1" class="data row36 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row37_col0" class="data row37 col0" >Delaware</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row37_col1" class="data row37 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row38_col0" class="data row38 col0" >Diamond Princess</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row38_col1" class="data row38 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row39_col0" class="data row39 col0" >District of Columbia</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row39_col1" class="data row39 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row40_col0" class="data row40 col0" >Florida</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row40_col1" class="data row40 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row41_col0" class="data row41 col0" >Georgia</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row41_col1" class="data row41 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row42_col0" class="data row42 col0" >Grand Princess</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row42_col1" class="data row42 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row43_col0" class="data row43 col0" >Guam</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row43_col1" class="data row43 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row44_col0" class="data row44 col0" >Hawaii</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row44_col1" class="data row44 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row45_col0" class="data row45 col0" >Idaho</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row45_col1" class="data row45 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row46_col0" class="data row46 col0" >Indiana</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row46_col1" class="data row46 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row47_col0" class="data row47 col0" >Iowa</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row47_col1" class="data row47 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row48_col0" class="data row48 col0" >Kansas</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row48_col1" class="data row48 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row49_col0" class="data row49 col0" >Louisiana</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row49_col1" class="data row49 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row50_col0" class="data row50 col0" >Maine</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row50_col1" class="data row50 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row51_col0" class="data row51 col0" >Michigan</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row51_col1" class="data row51 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row52_col0" class="data row52 col0" >Minnesota</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row52_col1" class="data row52 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row53_col0" class="data row53 col0" >Mississippi</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row53_col1" class="data row53 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row54_col0" class="data row54 col0" >Alaska</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row54_col1" class="data row54 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row55_col0" class="data row55 col0" >Wyoming</td>
                        <td id="T_ed70271a_686b_11ea_b34d_acbc328f6cc1row55_col1" class="data row55 col1" >0</td>
            </tr>
    </tbody></table>




```python
recover_US_df['recovered'].sum()
```




    17



#### Create a dataframe without null values and does not contain state localities


```python
deaths_US_df = deaths_df[~deaths_df['Province/State'].isnull()].copy()

deaths_US_df = deaths_US_df[~deaths_US_df['Province/State'].str.contains(',') & (deaths_US_df['Country/Region']=='US')].sort_values('Province/State').copy()

deaths_US_df = deaths_US_df.rename(columns={'Province/State': 'state'})
deaths_US_df_timeline = deaths_US_df.copy()
deaths_US_df_timeline.head()
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>417</th>
      <td>Alabama</td>
      <td>US</td>
      <td>32.3182</td>
      <td>-86.9023</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>141</th>
      <td>Alaska</td>
      <td>US</td>
      <td>61.3707</td>
      <td>-152.4044</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>118</th>
      <td>Arizona</td>
      <td>US</td>
      <td>33.7298</td>
      <td>-111.4312</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>142</th>
      <td>Arkansas</td>
      <td>US</td>
      <td>34.9697</td>
      <td>-92.3731</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>100</th>
      <td>California</td>
      <td>US</td>
      <td>36.1162</td>
      <td>-119.6816</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
      <td>4</td>
      <td>5</td>
      <td>6</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 59 columns</p>
</div>



#### Simplify the datafram containing the US `deaths` cases


```python
num_columns = len(deaths_US_df.columns)

deaths_US_df['deaths'] = deaths_US_df[deaths_US_df.columns[4: num_columns]].max(axis=1).astype(int)
deaths_US_df = deaths_US_df[['state','Lat','Long','deaths']].copy()
#sort descending by total count
deaths_US_df[['state','deaths']].sort_values(by= ['deaths'], ascending=False).style.hide_index()
```

<table id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1" ><thead>    <tr>        <th class="col_heading level0 col0" >state</th>        <th class="col_heading level0 col1" >deaths</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row0_col0" class="data row0 col0" >Washington</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row0_col1" class="data row0 col1" >48</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row1_col0" class="data row1 col0" >New York</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row1_col1" class="data row1 col1" >10</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row2_col0" class="data row2 col0" >California</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row2_col1" class="data row2 col1" >7</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row3_col0" class="data row3 col0" >Florida</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row3_col1" class="data row3 col1" >5</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row4_col0" class="data row4 col0" >Louisiana</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row4_col1" class="data row4 col1" >3</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row5_col0" class="data row5 col0" >New Jersey</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row5_col1" class="data row5 col1" >2</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row6_col0" class="data row6 col0" >South Carolina</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row6_col1" class="data row6 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row7_col0" class="data row7 col0" >South Dakota</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row7_col1" class="data row7 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row8_col0" class="data row8 col0" >Oregon</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row8_col1" class="data row8 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row9_col0" class="data row9 col0" >Kentucky</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row9_col1" class="data row9 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row10_col0" class="data row10 col0" >Kansas</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row10_col1" class="data row10 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row11_col0" class="data row11 col0" >Indiana</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row11_col1" class="data row11 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row12_col0" class="data row12 col0" >Nevada</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row12_col1" class="data row12 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row13_col0" class="data row13 col0" >Georgia</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row13_col1" class="data row13 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row14_col0" class="data row14 col0" >Virginia</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row14_col1" class="data row14 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row15_col0" class="data row15 col0" >Colorado</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row15_col1" class="data row15 col1" >1</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row16_col0" class="data row16 col0" >Texas</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row16_col1" class="data row16 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row17_col0" class="data row17 col0" >West Virginia</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row17_col1" class="data row17 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row18_col0" class="data row18 col0" >New Mexico</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row18_col1" class="data row18 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row19_col0" class="data row19 col0" >Wisconsin</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row19_col1" class="data row19 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row20_col0" class="data row20 col0" >North Carolina</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row20_col1" class="data row20 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row21_col0" class="data row21 col0" >North Dakota</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row21_col1" class="data row21 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row22_col0" class="data row22 col0" >Ohio</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row22_col1" class="data row22 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row23_col0" class="data row23 col0" >Oklahoma</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row23_col1" class="data row23 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row24_col0" class="data row24 col0" >Puerto Rico</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row24_col1" class="data row24 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row25_col0" class="data row25 col0" >Pennsylvania</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row25_col1" class="data row25 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row26_col0" class="data row26 col0" >Tennessee</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row26_col1" class="data row26 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row27_col0" class="data row27 col0" >Virgin Islands</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row27_col1" class="data row27 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row28_col0" class="data row28 col0" >Rhode Island</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row28_col1" class="data row28 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row29_col0" class="data row29 col0" >Vermont</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row29_col1" class="data row29 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row30_col0" class="data row30 col0" >Utah</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row30_col1" class="data row30 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row31_col0" class="data row31 col0" >Alabama</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row31_col1" class="data row31 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row32_col0" class="data row32 col0" >Missouri</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row32_col1" class="data row32 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row33_col0" class="data row33 col0" >New Hampshire</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row33_col1" class="data row33 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row34_col0" class="data row34 col0" >Idaho</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row34_col1" class="data row34 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row35_col0" class="data row35 col0" >Arizona</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row35_col1" class="data row35 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row36_col0" class="data row36 col0" >Arkansas</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row36_col1" class="data row36 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row37_col0" class="data row37 col0" >Connecticut</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row37_col1" class="data row37 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row38_col0" class="data row38 col0" >Delaware</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row38_col1" class="data row38 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row39_col0" class="data row39 col0" >Diamond Princess</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row39_col1" class="data row39 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row40_col0" class="data row40 col0" >District of Columbia</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row40_col1" class="data row40 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row41_col0" class="data row41 col0" >Grand Princess</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row41_col1" class="data row41 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row42_col0" class="data row42 col0" >Guam</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row42_col1" class="data row42 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row43_col0" class="data row43 col0" >Hawaii</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row43_col1" class="data row43 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row44_col0" class="data row44 col0" >Illinois</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row44_col1" class="data row44 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row45_col0" class="data row45 col0" >Nebraska</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row45_col1" class="data row45 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row46_col0" class="data row46 col0" >Iowa</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row46_col1" class="data row46 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row47_col0" class="data row47 col0" >Maine</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row47_col1" class="data row47 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row48_col0" class="data row48 col0" >Maryland</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row48_col1" class="data row48 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row49_col0" class="data row49 col0" >Massachusetts</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row49_col1" class="data row49 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row50_col0" class="data row50 col0" >Michigan</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row50_col1" class="data row50 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row51_col0" class="data row51 col0" >Minnesota</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row51_col1" class="data row51 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row52_col0" class="data row52 col0" >Mississippi</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row52_col1" class="data row52 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row53_col0" class="data row53 col0" >Alaska</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row53_col1" class="data row53 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row54_col0" class="data row54 col0" >Montana</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row54_col1" class="data row54 col1" >0</td>
            </tr>
            <tr>
                                <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row55_col0" class="data row55 col0" >Wyoming</td>
                        <td id="T_ed88e020_686b_11ea_b34d_acbc328f6cc1row55_col1" class="data row55 col1" >0</td>
            </tr>
    </tbody></table>




```python
deaths_US_df['deaths'].sum()
```




    85




```python
states_df = pd.merge(confirm_US_df, recover_US_df, on='state')
#states_df

states_df = pd.merge(states_df, deaths_US_df, on='state')
states_df = states_df[['state','Lat','Long','confirmed','recovered','deaths']]
#states_df
```

#### Create a dataframe of current fips codes for counties and states


```python
county_fips_url = 'https://api.census.gov/data/2010/dec/sf1?get=NAME&for=county:*'
state_fips_url = 'https://api.census.gov/data/2010/dec/sf1?get=NAME&for=state:*'

# create county with fips dataframe
r = requests.get(county_fips_url)
county_fips_df = pd.DataFrame(r.json())
#convert the first row to the header
new_header_county = county_fips_df.iloc[0] #grab the first row for the header
county_fips_df = county_fips_df[1:] #take the data less the header row
county_fips_df.columns = new_header_county #set the header row as the df header
print(len(county_fips_df))
# create state with fips dataframe
r = requests.get(state_fips_url)
state_fips_df = pd.DataFrame(r.json())
#convert the first row to the header
new_header_state = state_fips_df.iloc[0] #grab the first row for the header
state_fips_df = state_fips_df[1:] #take the data less the header row
state_fips_df.columns = new_header_state #set the header row as the df header
state_fips_df=pd.merge(state_fips_df, state_abbr_fips_df, on='NAME')
print(len(state_fips_df))


fips_df = pd.merge(county_fips_df, state_fips_df, on='state')

fips_df = fips_df.rename(columns={'NAME_x': 'full_county', 'county': 'county_fips'})

fips_df['county']=fips_df.full_county.str.split(",",expand=True)[0]

fips_df['county_short']=fips_df.county.str.split(" County",expand=True)[0]


fips_df=fips_df[['state_fips','county_fips','state','county','county_short','full_county','state_abbr']].sort_values(by=['state','county']).copy()
fips_df.reset_index(inplace = True,drop=True) 
print(len(fips_df))

fips_df

```

    3221
    51
    3143


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state_fips</th>
      <th>county_fips</th>
      <th>state</th>
      <th>county</th>
      <th>county_short</th>
      <th>full_county</th>
      <th>state_abbr</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01</td>
      <td>001</td>
      <td>01</td>
      <td>Autauga County</td>
      <td>Autauga</td>
      <td>Autauga County, Alabama</td>
      <td>AL</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01</td>
      <td>003</td>
      <td>01</td>
      <td>Baldwin County</td>
      <td>Baldwin</td>
      <td>Baldwin County, Alabama</td>
      <td>AL</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01</td>
      <td>005</td>
      <td>01</td>
      <td>Barbour County</td>
      <td>Barbour</td>
      <td>Barbour County, Alabama</td>
      <td>AL</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01</td>
      <td>007</td>
      <td>01</td>
      <td>Bibb County</td>
      <td>Bibb</td>
      <td>Bibb County, Alabama</td>
      <td>AL</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01</td>
      <td>009</td>
      <td>01</td>
      <td>Blount County</td>
      <td>Blount</td>
      <td>Blount County, Alabama</td>
      <td>AL</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3138</th>
      <td>56</td>
      <td>037</td>
      <td>56</td>
      <td>Sweetwater County</td>
      <td>Sweetwater</td>
      <td>Sweetwater County, Wyoming</td>
      <td>WY</td>
    </tr>
    <tr>
      <th>3139</th>
      <td>56</td>
      <td>039</td>
      <td>56</td>
      <td>Teton County</td>
      <td>Teton</td>
      <td>Teton County, Wyoming</td>
      <td>WY</td>
    </tr>
    <tr>
      <th>3140</th>
      <td>56</td>
      <td>041</td>
      <td>56</td>
      <td>Uinta County</td>
      <td>Uinta</td>
      <td>Uinta County, Wyoming</td>
      <td>WY</td>
    </tr>
    <tr>
      <th>3141</th>
      <td>56</td>
      <td>043</td>
      <td>56</td>
      <td>Washakie County</td>
      <td>Washakie</td>
      <td>Washakie County, Wyoming</td>
      <td>WY</td>
    </tr>
    <tr>
      <th>3142</th>
      <td>56</td>
      <td>045</td>
      <td>56</td>
      <td>Weston County</td>
      <td>Weston</td>
      <td>Weston County, Wyoming</td>
      <td>WY</td>
    </tr>
  </tbody>
</table>
<p>3143 rows × 7 columns</p>
</div>




```python
states_df = pd.merge(confirm_US_df, recover_US_df, how='left',left_on='state',right_on='state')
#states_df

states_df = pd.merge(states_df, deaths_US_df, how='left',left_on='state',right_on='state')
states_df = states_df[['state','Lat','Long','confirmed','recovered','deaths']]
print(len(states_df))
#states_df
```

    56



```python
states_df = pd.merge(states_df,state_fips_df, left_on='state', right_on='NAME')
states_df = states_df.rename(columns={'state_x': 'state'})
states_df = states_df[['state','state_fips','state_abbr','Lat','Long','confirmed','recovered','deaths']]
print(len(states_df))
states_df.sort_values(by='confirmed',ascending=False).head()
```

    51


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>state_fips</th>
      <th>state_abbr</th>
      <th>Lat</th>
      <th>Long</th>
      <th>confirmed</th>
      <th>recovered</th>
      <th>deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>32</th>
      <td>New York</td>
      <td>36</td>
      <td>NY</td>
      <td>42.1657</td>
      <td>-74.9481</td>
      <td>967</td>
      <td>0</td>
      <td>10</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Washington</td>
      <td>53</td>
      <td>WA</td>
      <td>47.4009</td>
      <td>-121.4905</td>
      <td>904</td>
      <td>1</td>
      <td>48</td>
    </tr>
    <tr>
      <th>4</th>
      <td>California</td>
      <td>06</td>
      <td>CA</td>
      <td>36.1162</td>
      <td>-119.6816</td>
      <td>557</td>
      <td>6</td>
      <td>7</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Massachusetts</td>
      <td>25</td>
      <td>MA</td>
      <td>42.2302</td>
      <td>-71.5301</td>
      <td>197</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>New Jersey</td>
      <td>34</td>
      <td>NJ</td>
      <td>40.2989</td>
      <td>-74.5210</td>
      <td>178</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
confirm_counties_df = confirm_df[~confirm_df['Province/State'].isnull()].copy()
confirm_counties_df = confirm_counties_df[confirm_counties_df['Province/State'].str.contains(',') & (confirm_counties_df['Country/Region']=='US')].sort_values('Province/State').copy()
confirm_counties_df = confirm_counties_df.rename(columns={'Province/State': 'county_state'})
confirm_counties_df_timeline = confirm_counties_df.copy()
print(len(confirm_counties_df))
confirm_counties_df_timeline.head()
```

    191



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county_state</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>242</th>
      <td>Adams, IN</td>
      <td>US</td>
      <td>39.8522</td>
      <td>-77.2865</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>362</th>
      <td>Alachua, FL</td>
      <td>US</td>
      <td>29.7938</td>
      <td>-82.4944</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>282</th>
      <td>Alameda County, CA</td>
      <td>US</td>
      <td>37.6017</td>
      <td>-121.7195</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>384</th>
      <td>Anoka, MN</td>
      <td>US</td>
      <td>45.3293</td>
      <td>-93.2197</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Arapahoe, CO</td>
      <td>US</td>
      <td>39.6203</td>
      <td>-104.3326</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 59 columns</p>
</div>




```python
num_columns = len(confirm_counties_df.columns)
confirm_counties_df['confirmed'] = confirm_counties_df[confirm_counties_df.columns[4: num_columns]].max(axis=1).astype(int)
confirm_counties_df=confirm_counties_df[['county_state','Lat','Long','confirmed']].copy()
split_county_state=confirm_counties_df.county_state.str.split(", ",expand=True)
confirm_counties_df['county']=split_county_state[0]
confirm_counties_df['state']=split_county_state[1]
confirm_counties_df=confirm_counties_df[['county_state','county','state','Lat','Long','confirmed']]
print(len(confirm_counties_df))

#confirm_counties_df.sort_values(by='confirmed',ascending=False)
```

    191



```python
recover_counties_df = recover_df[~recover_df['Province/State'].isnull()].copy()
recover_counties_df = recover_counties_df[recover_counties_df['Province/State'].str.contains(',') & (recover_counties_df['Country/Region']=='US')].sort_values('Province/State').copy()
recover_counties_df = recover_counties_df.rename(columns={'Province/State': 'county_state'})
recover_counties_df_timeline = recover_counties_df.copy()
print(len(recover_counties_df))

recover_counties_df_timeline.head()
```

    191


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county_state</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>3/7/20</th>
      <th>3/8/20</th>
      <th>3/9/20</th>
      <th>3/10/20</th>
      <th>3/11/20</th>
      <th>3/12/20</th>
      <th>3/13/20</th>
      <th>3/14/20</th>
      <th>3/15/20</th>
      <th>3/16/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>242</th>
      <td>Adams, IN</td>
      <td>US</td>
      <td>39.8522</td>
      <td>-77.2865</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>362</th>
      <td>Alachua, FL</td>
      <td>US</td>
      <td>29.7938</td>
      <td>-82.4944</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>282</th>
      <td>Alameda County, CA</td>
      <td>US</td>
      <td>37.6017</td>
      <td>-121.7195</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>384</th>
      <td>Anoka, MN</td>
      <td>US</td>
      <td>45.3293</td>
      <td>-93.2197</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Arapahoe, CO</td>
      <td>US</td>
      <td>39.6203</td>
      <td>-104.3326</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 59 columns</p>
</div>




```python
num_columns = len(recover_counties_df.columns)
recover_counties_df['recovered'] = recover_counties_df[recover_counties_df.columns[4: num_columns]].max(axis=1).astype(int)
recover_counties_df=recover_counties_df[['county_state','Lat','Long','recovered']].copy()
split_county_state=recover_counties_df.county_state.str.split(", ",expand=True)
recover_counties_df['county']=split_county_state[0]
recover_counties_df['state']=split_county_state[1]
recover_counties_df=recover_counties_df[['county_state','county','state','Lat','Long','recovered']]
print(len(recover_counties_df))

#recover_counties_df.sort_values(by='recovered',ascending=False)
```

    191



```python

```


```python
deaths_counties_df = deaths_df[~deaths_df['Province/State'].isnull()].copy()
deaths_counties_df = deaths_counties_df[deaths_counties_df['Province/State'].str.contains(',') & (deaths_counties_df['Country/Region']=='US')].sort_values('Province/State').copy()
deaths_counties_df = deaths_counties_df.rename(columns={'Province/State': 'county_state'})
deaths_counties_df = deaths_counties_df.copy()
print(len(deaths_counties_df))

#deaths_counties_df.head()
```

    191



```python
num_columns = len(deaths_counties_df.columns)
deaths_counties_df['deaths'] = deaths_counties_df[deaths_counties_df.columns[4: num_columns]].max(axis=1).astype(int)
deaths_counties_df=deaths_counties_df[['county_state','Lat','Long','deaths']].copy()
split_county_state=deaths_counties_df.county_state.str.split(", ",expand=True)
deaths_counties_df['county']=split_county_state[0]
deaths_counties_df['state']=split_county_state[1]

deaths_counties_df=deaths_counties_df[['county_state','county','state','Lat','Long','deaths']]
print(len(deaths_counties_df))
#deaths_counties_df.sort_values(by='deaths',ascending=False)
```

    191



```python
counties_df = pd.merge(confirm_counties_df, recover_counties_df, how='left', on=['state','county'])
counties_df = pd.merge(counties_df, deaths_counties_df, on=['state','county'])
counties_df = counties_df[['county_state','county','state','Lat','Long','confirmed','recovered','deaths']]


counties_df['county_short']=counties_df.county.str.split(" County",expand=True)[0]
print(len(counties_df))

#counties_df
```

    191



```python
counties_df = pd.merge(counties_df,fips_df, how='left', left_on=['county_short','state'], right_on=['county_short','state_abbr'])
#new_df = pd.merge(A_df, B_df,  how='left', left_on=['A_c1','c2'], right_on = ['B_c1','c2'])
print(len(counties_df))
#counties_df
```

    191



```python
counties_df = counties_df.rename(columns={'state_x': 'state','county_x':'county'})
counties_df = counties_df[['county_state','county','state','state_fips','county_fips','county_short','full_county','Lat','Long','confirmed','recovered','deaths']]
counties_df = counties_df.sort_values(by=['confirmed','state_fips','county_fips'],ascending=False)

counties_df.head(15)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county_state</th>
      <th>county</th>
      <th>state</th>
      <th>state_fips</th>
      <th>county_fips</th>
      <th>county_short</th>
      <th>full_county</th>
      <th>Lat</th>
      <th>Long</th>
      <th>confirmed</th>
      <th>recovered</th>
      <th>deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>186</th>
      <td>Westchester County, NY</td>
      <td>Westchester County</td>
      <td>NY</td>
      <td>36</td>
      <td>119</td>
      <td>Westchester</td>
      <td>Westchester County, New York</td>
      <td>41.1220</td>
      <td>-73.7949</td>
      <td>98</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>83</th>
      <td>King County, WA</td>
      <td>King County</td>
      <td>WA</td>
      <td>53</td>
      <td>033</td>
      <td>King</td>
      <td>King County, Washington</td>
      <td>47.6062</td>
      <td>-122.3321</td>
      <td>83</td>
      <td>1</td>
      <td>17</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Santa Clara County, CA</td>
      <td>Santa Clara County</td>
      <td>CA</td>
      <td>06</td>
      <td>085</td>
      <td>Santa Clara</td>
      <td>Santa Clara County, California</td>
      <td>37.3541</td>
      <td>-121.9552</td>
      <td>38</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Snohomish County, WA</td>
      <td>Snohomish County</td>
      <td>WA</td>
      <td>53</td>
      <td>061</td>
      <td>Snohomish</td>
      <td>Snohomish County, Washington</td>
      <td>48.0330</td>
      <td>-121.8339</td>
      <td>31</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>115</th>
      <td>New York County, NY</td>
      <td>New York County</td>
      <td>NY</td>
      <td>36</td>
      <td>061</td>
      <td>New York</td>
      <td>New York County, New York</td>
      <td>40.7128</td>
      <td>-74.0060</td>
      <td>19</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Nassau County, NY</td>
      <td>Nassau County</td>
      <td>NY</td>
      <td>36</td>
      <td>059</td>
      <td>Nassau</td>
      <td>Nassau County, New York</td>
      <td>40.6546</td>
      <td>-73.5594</td>
      <td>17</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>93</th>
      <td>Los Angeles, CA</td>
      <td>Los Angeles</td>
      <td>CA</td>
      <td>06</td>
      <td>037</td>
      <td>Los Angeles</td>
      <td>Los Angeles County, California</td>
      <td>34.0522</td>
      <td>-118.2437</td>
      <td>14</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>146</th>
      <td>San Francisco County, CA</td>
      <td>San Francisco County</td>
      <td>CA</td>
      <td>06</td>
      <td>075</td>
      <td>San Francisco</td>
      <td>San Francisco County, California</td>
      <td>37.7749</td>
      <td>-122.4194</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Contra Costa County, CA</td>
      <td>Contra Costa County</td>
      <td>CA</td>
      <td>06</td>
      <td>013</td>
      <td>Contra Costa</td>
      <td>Contra Costa County, California</td>
      <td>37.8534</td>
      <td>-121.9018</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>180</th>
      <td>Washington County, OR</td>
      <td>Washington County</td>
      <td>OR</td>
      <td>41</td>
      <td>067</td>
      <td>Washington</td>
      <td>Washington County, Oregon</td>
      <td>45.5470</td>
      <td>-123.1386</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Suffolk County, MA</td>
      <td>Suffolk County</td>
      <td>MA</td>
      <td>25</td>
      <td>025</td>
      <td>Suffolk</td>
      <td>Suffolk County, Massachusetts</td>
      <td>42.3601</td>
      <td>-71.0589</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>102</th>
      <td>Middlesex County, MA</td>
      <td>Middlesex County</td>
      <td>MA</td>
      <td>25</td>
      <td>017</td>
      <td>Middlesex</td>
      <td>Middlesex County, Massachusetts</td>
      <td>42.4672</td>
      <td>-71.2874</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Cook County, IL</td>
      <td>Cook County</td>
      <td>IL</td>
      <td>17</td>
      <td>031</td>
      <td>Cook</td>
      <td>Cook County, Illinois</td>
      <td>41.7377</td>
      <td>-87.6976</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>133</th>
      <td>Placer County, CA</td>
      <td>Placer County</td>
      <td>CA</td>
      <td>06</td>
      <td>061</td>
      <td>Placer</td>
      <td>Placer County, California</td>
      <td>39.0916</td>
      <td>-120.8039</td>
      <td>7</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>64</th>
      <td>Harris County, TX</td>
      <td>Harris County</td>
      <td>TX</td>
      <td>48</td>
      <td>201</td>
      <td>Harris</td>
      <td>Harris County, Texas</td>
      <td>29.7752</td>
      <td>-95.3103</td>
      <td>6</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
counties_df[counties_df.state=='NY']
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county_state</th>
      <th>county</th>
      <th>state</th>
      <th>state_fips</th>
      <th>county_fips</th>
      <th>county_short</th>
      <th>full_county</th>
      <th>Lat</th>
      <th>Long</th>
      <th>confirmed</th>
      <th>recovered</th>
      <th>deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>186</th>
      <td>Westchester County, NY</td>
      <td>Westchester County</td>
      <td>NY</td>
      <td>36</td>
      <td>119</td>
      <td>Westchester</td>
      <td>Westchester County, New York</td>
      <td>41.1220</td>
      <td>-73.7949</td>
      <td>98</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>115</th>
      <td>New York County, NY</td>
      <td>New York County</td>
      <td>NY</td>
      <td>36</td>
      <td>061</td>
      <td>New York</td>
      <td>New York County, New York</td>
      <td>40.7128</td>
      <td>-74.0060</td>
      <td>19</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Nassau County, NY</td>
      <td>Nassau County</td>
      <td>NY</td>
      <td>36</td>
      <td>059</td>
      <td>Nassau</td>
      <td>Nassau County, New York</td>
      <td>40.6546</td>
      <td>-73.5594</td>
      <td>17</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>142</th>
      <td>Rockland County, NY</td>
      <td>Rockland County</td>
      <td>NY</td>
      <td>36</td>
      <td>087</td>
      <td>Rockland</td>
      <td>Rockland County, New York</td>
      <td>41.1489</td>
      <td>-73.9830</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Saratoga County, NY</td>
      <td>Saratoga County</td>
      <td>NY</td>
      <td>36</td>
      <td>091</td>
      <td>Saratoga</td>
      <td>Saratoga County, New York</td>
      <td>43.0324</td>
      <td>-73.9360</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>174</th>
      <td>Ulster County, NY</td>
      <td>Ulster County</td>
      <td>NY</td>
      <td>36</td>
      <td>111</td>
      <td>Ulster</td>
      <td>Ulster County, New York</td>
      <td>41.8586</td>
      <td>-74.3118</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Suffolk County, NY</td>
      <td>Suffolk County</td>
      <td>NY</td>
      <td>36</td>
      <td>103</td>
      <td>Suffolk</td>
      <td>Suffolk County, New York</td>
      <td>40.9849</td>
      <td>-72.6151</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
counties_df[counties_df.state=='KY']
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>county_state</th>
      <th>county</th>
      <th>state</th>
      <th>state_fips</th>
      <th>county_fips</th>
      <th>county_short</th>
      <th>full_county</th>
      <th>Lat</th>
      <th>Long</th>
      <th>confirmed</th>
      <th>recovered</th>
      <th>deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>65</th>
      <td>Harrison County, KY</td>
      <td>Harrison County</td>
      <td>KY</td>
      <td>21</td>
      <td>097</td>
      <td>Harrison</td>
      <td>Harrison County, Kentucky</td>
      <td>38.4333</td>
      <td>-84.3542</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>74</th>
      <td>Jefferson County, KY</td>
      <td>Jefferson County</td>
      <td>KY</td>
      <td>21</td>
      <td>111</td>
      <td>Jefferson</td>
      <td>Jefferson County, Kentucky</td>
      <td>38.1938</td>
      <td>-85.6435</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Fayette County, KY</td>
      <td>Fayette County</td>
      <td>KY</td>
      <td>21</td>
      <td>067</td>
      <td>Fayette</td>
      <td>Fayette County, Kentucky</td>
      <td>38.0606</td>
      <td>-84.4803</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
states_df_plot=states_df[['state','confirmed']].sort_values(by=['confirmed'],ascending=False).copy().head(10)

states_df_plot.reset_index(drop=True, inplace=True)
states_df_plot.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>confirmed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>New York</td>
      <td>967</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Washington</td>
      <td>904</td>
    </tr>
    <tr>
      <th>2</th>
      <td>California</td>
      <td>557</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Massachusetts</td>
      <td>197</td>
    </tr>
    <tr>
      <th>4</th>
      <td>New Jersey</td>
      <td>178</td>
    </tr>
  </tbody>
</table>
</div>




```python
states_df_plot=states_df_plot.set_index('state').copy()
states_df_plot[['confirmed']].head(10).plot(kind='barh').invert_yaxis()
```


![png](output_47_0.png)



```python
states_df_plot
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>confirmed</th>
    </tr>
    <tr>
      <th>state</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>New York</th>
      <td>967</td>
    </tr>
    <tr>
      <th>Washington</th>
      <td>904</td>
    </tr>
    <tr>
      <th>California</th>
      <td>557</td>
    </tr>
    <tr>
      <th>Massachusetts</th>
      <td>197</td>
    </tr>
    <tr>
      <th>New Jersey</th>
      <td>178</td>
    </tr>
    <tr>
      <th>Colorado</th>
      <td>160</td>
    </tr>
    <tr>
      <th>Florida</th>
      <td>155</td>
    </tr>
    <tr>
      <th>Louisiana</th>
      <td>136</td>
    </tr>
    <tr>
      <th>Georgia</th>
      <td>121</td>
    </tr>
    <tr>
      <th>Illinois</th>
      <td>105</td>
    </tr>
  </tbody>
</table>
</div>




```python
states_df_plot.columns
```




    Index(['confirmed'], dtype='object')




```python
%matplotlib --list
```

    Available matplotlib backends: ['tk', 'gtk', 'gtk3', 'wx', 'qt4', 'qt5', 'qt', 'osx', 'nbagg', 'notebook', 'agg', 'svg', 'pdf', 'ps', 'inline', 'ipympl', 'widget']



```python
fig = plt.figure(figsize=(20,20))
plt.rcParams["axes.labelsize"] = 15

```


    <Figure size 1440x1440 with 0 Axes>



```python
states_df_plot2=sns.catplot(x=list(states_df_plot.confirmed),y=list(states_df_plot.index),kind='bar',data=states_df_plot,edgecolor='whitesmoke',linewidth=.5,color='blue')
states_df_plot2.set(xlabel='confirmed cases',ylabel='')
for bar_index, bar_attributes in enumerate(states_df_plot2.ax.patches):
    bar_width = bar_attributes.get_width() # x coordinate of text
    states_df_plot2.ax.text(
        bar_width+35, bar_index, '{}'.format(int(bar_width)),
        ha='center', va='center',size=14, color='red')
plt.show;

```

    /Users/mark/anaconda3/envs/sandbox2/lib/python3.7/site-packages/matplotlib/tight_layout.py:181: UserWarning: Tight layout not applied. The bottom and top margins cannot be made large enough to accommodate all axes decorations. 
      warnings.warn('Tight layout not applied. '



![png](output_52_1.png)



```python

```
