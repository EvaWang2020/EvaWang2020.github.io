---
title: ""
published: true
---
![](https://static.wixstatic.com/media/456b92_393b8a71090444e290d75b40b1bfe794~mv2.jpg/v1/fill/w_740,h_416,al_c,q_90/456b92_393b8a71090444e290d75b40b1bfe794~mv2.webp)

There are different ways to scrape real estate data. No solution fits all. I tried two solutions: Beautiful Soup and Scrapy. Overall, I feel Beautiful Soup is earlier to learn while Scrapy is much faster.

-   **Beautiful Soup**
    

Beautiful Soup is a Python package that helps to parse HTML and XML documents. With the help of Selector Gadget (a Chrome plugin that allows us to obtain unique CSS selectors and XPath queries to extract targeted sections of a website), using Beautiful Soup to scrape web data is not very hard to start. Below is an example of scrapping the price, address, structure, and size of the Vancouver real estate lists and save into a data frame.

from bs4 import BeautifulSoup
import time
import pandas as pd
from selenium import webdriver
from webdriver\_manager.chrome import ChromeDriverManager

URL \= "https://www.findvancouverhouses.com/search/quick?city=Vancouver&page=1"
browser \= webdriver.Chrome(ChromeDriverManager().install())
browser.get(URL)
\# website ask to accept to cookie to be able to browse to the next page
button \= browser.find\_element\_by\_css\_selector("#accept-cookie-notification")
button.click()
time.sleep(2)

\# below is to ask to scrape only 3 pages. you can adjust
for i in range(0, 1):
\# ".old-school" is the css code to click to go to the next page
    button \= browser.find\_element\_by\_css\_selector(".old-school")
    button.click()     
\# if you see the error (ElementClickInterceptedException: element click intercepted:), increase the sleep time
 print("Count: ", str(i))
    time.sleep(10)
print("done loop")

content \= browser.find\_elements\_by\_css\_selector(".card--list-wrap")
listings \= \[\]

class Listing:
    price \= ""
    address \= ""
    structure\_size\=""
 def \_\_init\_\_(self, price, address, structure\_size):
 self.price \= price
 self.address \= address
 self.structure\_size \=  structure\_size

def extractText(data):
    text \= data.get\_attribute('innerHTML')
    beautifulSoupText \= BeautifulSoup(text, features\="lxml")
    cleanedText \= beautifulSoupText.get\_text()
    cleanedText \= re.sub(r"\[\\t\]\*", "", cleanedText)
    cleanedText \= re.sub(r"\\n\\n", "", cleanedText)
    cleanedText \= re.sub('\[ \]{2,}', '\*', cleanedText)
    cleanedText \= re.sub(r"\\n", " ", cleanedText)
 return cleanedText.strip()

for e in content:
    price \= e.find\_element\_by\_css\_selector(".card--list-header.pdq-old").text
    address \= e.find\_element\_by\_css\_selector(".full-address").text
    structure\_size \= e.find\_element\_by\_css\_selector("ul").text
    listing \= Listing(price, address, structure\_size)  \# to creat a subject of a class
    listings.append(listing)

dataSet \= {'price': \[\], 'address': \[\], 'Structure & Size':\[\] }
df \= pd.DataFrame(dataSet, columns\=\['price', 'address', 'Structure & Size' \])

for listing in listings:
    listingDictionary \= {
 "price": listing.price,
 "address": listing. address,
 "Structure & Size": listing.structure\_size
    }
    df \= df.append(listingDictionary, ignore\_index\=True)

print (df)

It took me around 20 seconds to scrape one page, while there are 167 pages of lists on Jul 4, 2021. Think about how long it will take to scrape the whole Vancouver list.

**Please notice**: website developers constantly change their CSS selectors to protect their website's data from crawlers/spiders and other security issues. Therefore, the CSS selectors in the above code for the price, address, etc., may change as time goes on.

-   **Scrapy**
    

I feel Scrapy ( a free and open-source web-crawling framework written in Python) is a bit harder to learn initially, but once you get the experience, you will love to use it.

Below is an example of scaping the same content like the above. The result is saved into a JSON file. Here we target to scrape the whole Vancouver list on Jul 4, 2021. It took 6 mins 5 seconds to finish. As there are 167 pages, it took 2.16 seconds on average to scrape one page, while Beautiful Soup took 20 seconds to scrape one page.

import scrapy
from scrapy.crawler import CrawlerProcess
from ratelimiter import RateLimiter
from scrapy.signalmanager import dispatcher
from scrapy import signals
import json
import os.path
from datetime import date

\# this mean to call 10 pages per 5 seconds. If scraping too fast, it could be blocked. Also, some lists might be skipped when scrapping too fast 
rate\_limiter \= RateLimiter(max\_calls\=10, period\=5)
results \= \[\]
date \= date.today()
\# change path1 text according to your need."xxxxxxx" has to be replaced
path1 \= "C://xxxxxxx\_"
path2 \= str(date)+"-1.txt"
file\_path \= path1+path2

if os.path.isfile(file\_path):
    os.remove(file\_path)
    file\_object \= open(file\_path, "w+")
    file\_object.close()
else:
    pass

\# create a class inherited from another class scrapy.Spider
class BlogSpider(scrapy.Spider):
    name \= 'blogspider'
\# this is the starting URL
    start\_urls \= \['https://www.findvancouverhouses.com/search/quick?city=Vancouver&page=1'\]

 def parse(self, response):
\# print(response.text)
\# reponse is the scraped result
 for title in response.css('.card--list-wrap'):
 yield {
\# depending on what you want to collect, the detail is different
 'price': title.css('.card--list-header::text').get().strip(),
 'address': title.css('.full-address').get().strip(),
 'Structure & size': title.css('.card--list stats').xpath('ul/li/text()').getall()
            }   \# yield is used to return the result

 for next\_page in response.css('a\[class="old-school"\]'):
 with rate\_limiter:
\# this is to ask to go through the pages
 yield response.follow(next\_page, self.parse)

\# below is a function to call after spider find content
def crawler\_results(signal, sender, item, response, spider):
    results.append(item)   \# item is the scraped result from yield

dispatcher.connect(crawler\_results, signal\=signals.item\_passed)
process \= CrawlerProcess({
 'USER\_AGENT': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10\_15\_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36'

})   \# a user agent is any software, acting on behalf of a user, which "retrieves, renders and facilitates end\-user interaction with Web content. Sometimes you need to change it to avoid banning from your scrapying websites

process.crawl(BlogSpider)
process.start()

json\_result \= {
 'listings': results
}
\# to create called sample .'a' means to append content
with open(file\_path, 'a') as outfile:
    outfile.write(json.dumps(json\_result) + '\\n')

When using Scrapy, often you get the error indicating your call is blocked. The common reason is that your code is asking too much content in a short period, so the website detects you and therefore blocks you. What you can do is that you limit the scrapping speed. Also, you might get the error "ReactorNotRestartable." There could be many reasons. I solved it by closing/restarting my Visual Studio.

It is important to verify whether you have scraped all the lists. If scraping too fast, some lists could be skipped. There is no error message or warning if you miss some, so you should verify the result.

One good thing about using Scrapy is that you can run multiple spiders simultaneously to speed up scraping. Here is a doc explaining how to run several spiders at the same time.

[https://docs.scrapy.org/en/latest/topics/practices.html](https://docs.scrapy.org/en/latest/topics/practices.html)

Notice: the article is for learning purposes. Please check the website's privacy and terms when practicing scraping. I encourage you to also check the legal issue related to web scraping to avoid unethical usage of scraped data.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI4MjE2Nzg1MSwtMTcxOTg4ODI5OF19
-->