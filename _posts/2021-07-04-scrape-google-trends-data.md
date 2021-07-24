---
title: Scrape Google Trends Data
published: true
---
Google Trends is a website that how frequently a given search term is entered into Google’s search engine relative to the site’s total search volume over a given period of time. From Google Trends, you can find the search trend of certain keywords on google. You can also compare the search trends of up to five keywords. For example, below compare five keywords search trend.

![](https://static.wixstatic.com/media/456b92_a9ad12973d384973b0ac2c0dbe63c65d~mv2.png/v1/fill/w_740,h_443,al_c,q_95/456b92_a9ad12973d384973b0ac2c0dbe63c65d~mv2.webp)

However, the provided visualizations might not satisfy your particular needs. If you have the underlying data, you can make your own visualization. This article explain how to scrape the data using Python pytrends library.

Below is the code to scrape the search trend of keywords "dress" and "pants" from Jan 1, 2020 to Jan 10 2020 at British Columbia, Canada.

```python
from pytrends.request import TrendReq
import pandas as pd
import time
pytrend = TrendReq(hl='en-GB', tz=360)

keywords=['dress','pants']
dataset = []

for x in range(0,len(keywords)):
     keyword = [keywords[x]]
     pytrend.build_payload(
kw_list=keyword,
cat=0,
timeframe='2020-01-01 2020-01-10',
geo='CA-BC')
     data = pytrend.interest_over_time()
if not data.empty:
        data = data.drop(labels=['isPartial'],axis='columns')   # thhis 
        dataset.append(data)
 
result = pd.concat(dataset, axis=1)   # merge the dataset using first column as reference
print(result) 
```

The result is:

![](https://static.wixstatic.com/media/456b92_a9cad9b5113142aebec54294be40953a~mv2.png/v1/fill/w_200,h_208,al_c,q_95/456b92_a9cad9b5113142aebec54294be40953a~mv2.webp)

Do you understand what the result means? The numbers represents the search interest of each keyword relative to the highest point of the same keyword within the period at British Columbia, Canada. A value of 100 is the peak popularity for the term. A value of 50 means that the term is half as popular. A score of zero means there was not enough data for the term. Besides search the popularity change within one region.

We can also search the popularity of a term by region within a specific period. Below is an example to find the search trend for the keyword "yoga pants" from Jan 1, 2016 to Feb 2, 2021 in Canada.

```python
from pytrends.request import TrendReq
pytrend = TrendReq()
pytrend.build_payload(kw_list=['yoga pants'], timeframe='2016-01-01 2021-07-02', geo='CA')
df = pytrend.interest_by_region()
df.reset_index().plot(x='geoName', y='yoga pants', figsize=(8, 5), kind ='bar')
```
![](https://static.wixstatic.com/media/456b92_cf86036c97ea4726bb80b8effcbc7565~mv2.png/v1/fill/w_360,h_490,al_c,q_95/456b92_cf86036c97ea4726bb80b8effcbc7565~mv2.webp)

Credits:

[https://pypi.org/project/pytrends/](https://pypi.org/project/pytrends/)

[https://towardsdatascience.com/telling-stories-with-google-trends-using-pytrends-in-python-a11e5b8a177](https://towardsdatascience.com/telling-stories-with-google-trends-using-pytrends-in-python-a11e5b8a177)
