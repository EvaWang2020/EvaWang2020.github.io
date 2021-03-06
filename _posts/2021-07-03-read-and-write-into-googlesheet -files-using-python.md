---
title: Read and Write into Google Sheet Files using Python
published: true
---

![](https://static.wixstatic.com/media/456b92_e69b58d6f4714359bf82c77069f99fd3~mv2.png/v1/fill/w_330,h_298,al_c,lg_1,q_95/456b92_e69b58d6f4714359bf82c77069f99fd3~mv2.webp)

Google Sheet is an important product to save data. Many applications, such as Tableau and Power BI, support Google Sheet data connection. This article explains how to read and write into Google Sheet files using Python.

I used this article ([https://medium.com/analytics-vidhya/how-to-read-and-write-data-to-google-spreadsheet-using-python-ebf54d51a72c](https://medium.com/analytics-vidhya/how-to-read-and-write-data-to-google-spreadsheet-using-python-ebf54d51a72c) ) to guide me through building my first Python script to read and write into Google Sheet files. From it, I learned to install the required python libraries, find Google Sheet ID, enable Google Sheet API, and set up Credentials. Overall, the doc is good, but there are some issues. For example, the first python script does not work for me due to credential settings. Also, the JSON file needs to have the full path instead of only the file name.

![](https://static.wixstatic.com/media/456b92_0ca246e897b14a22959e5061126567f2~mv2.png/v1/fill/w_740,h_566,al_c,lg_1,q_95/456b92_0ca246e897b14a22959e5061126567f2~mv2.webp)

The creds authorization is complicated. I followed the steps, but the script does not run. I, therefore, chose to use the service account credentials, which is more straightforward. Before setting up the credential, I need to enable Google Sheet API. The doc ([https://developers.google.com/workspace/guides/create-project](https://developers.google.com/workspace/guides/create-project)) is used as a reference to create a project and enable the Google Sheet API. After this, I set up the service account credential from Google Cloud Platform. Inside the service account, I set up Key ID and downloaded a JSON file.

![](https://static.wixstatic.com/media/456b92_5cc5dd7b81e849d88f9828acf2d74886~mv2.png/v1/fill/w_329,h_302,al_c,lg_1,q_95/456b92_5cc5dd7b81e849d88f9828acf2d74886~mv2.webp)

![](https://static.wixstatic.com/media/456b92_74f81d7ab1e74901ad4b76931e9c7086~mv2.png/v1/fill/w_360,h_163,al_c,lg_1,q_95/456b92_74f81d7ab1e74901ad4b76931e9c7086~mv2.webp)

Finally, here is my sample script to allow both read and write into a Google Sheet file.

```python
from __future__ import print_function
import os.path
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google.oauth2 import service_account
import json
from datetime import datetime

# if you want this script to read Google Sheet files only, use : SCOPES = ['https://www.googleapis.com/auth/spreadsheets.readonly']
SCOPES = ['https://www.googleapis.com/auth/spreadsheets']

# the ID and range of a Google Sheet spreadsheet
# if there are more than one tabs in the speadsheet, specific the tab in front of the range 
SAMPLE_SPREADSHEET_ID = 'XXXXXXXXXXXX'
SAMPLE_RANGE_NAME = 'A1:AE400'

# create a funtion to transfer json file content into a format used for Google Sheet file udpdate
def convert_json_to_googlesheets(json_object):
    number_row = 1
    values = [["date", "realestate", "mortgagerate", "interestrate", "location"]]
# for row in json_object:
    for attribute, value in json_object.items():
        print(attribute)
        values.append([
        value["date"],
        value["realestate"],
        value["mortgagerate"],
        value["interestrate"],
        value["location"]
            ])
        number_row = number_row + 1

    body = {
'values': values,
    }
    return body

def main():
 # put full path of the json file downloaded during creating Key ID
    credentials = service_account.Credentials.from_service_account_file("C:\\XXXXXXXX.json", scopes=SCOPES)
    service = build('sheets', 'v4', credentials=credentials)

 # call the sheets API
    sheet = service.spreadsheets()
    result = sheet.values().get(spreadsheetId=SAMPLE_SPREADSHEET_ID,
 range=SAMPLE_RANGE_NAME).execute()
    values = result.get('values', [])
 # print(values)
 # delete the old content inside the spreadsheet
    request=sheet.values().clear(spreadsheetId=SAMPLE_SPREADSHEET_ID, range=SAMPLE_RANGE_NAME).execute()
 
 # copy content from a json file, transfer its format, and use it to update the speadsheet
    f=open("C://Users/17789/Documents/Mentorship/GoogleTrendBC/GoogleTrend_60_months.json",)
    json_object = json.load(f)   
    body = convert_json_to_googlesheets(json_object)
    res=sheet.values().update(spreadsheetId=SAMPLE_SPREADSHEET_ID,   valueInputOption='USER_ENTERED', body=body, range=SAMPLE_RANGE_NAME).execute()

if __name__ == '__main__':
    main()
 

```
