---
layout: single
permalink: /python-write-googlesheets/
title: "Read and write in Google Sheets using Python"
tags: python
comments: true
---
This post will show how to Read and write in Google sheets using Python  

## Install Python modules
First we need to install the following modules  
```
$ pip install install gspread oauth2client
```

## Creating a project and a service account to access the Google sheet  
Follow the next steps:  
1. Go to [Google Cloud API dashboard](https://console.cloud.google.com/apis/)  
2. On the write of **Google Cloud** click on the facing down arrow  
3. Click on **New project**  
4. Give the project a name and Click **Create**
5. Search for Google drive API and enable it  
6. Do the same for Google Sheets API  
7. Go to **Credentials**  (On the left side of the dashboard)  
8. Click on **CREATE CREDENTIALS** and choose Service account, give it a name and click on **CREATE AND CONTINUE** and **DONE**  
9. Under **Service Accounts** you'll see the new service account. On the right you'll see **actions** , click on the pencil ("Edit service account") and go to **Keys** , then **ADD KEY** -> **Create new key** and **CREATE**.  
10. Save the JSON file created in a secure place, we'll use it to access the Google sheet.  
11. Under the the Service account's Details tab copy the email Under the **Email** and use the **Share** to add this email as an editor so your app can write to the Google sheet (Choose other permission if you want to)  


## Use Python to write and read to a cell
The following code writes and reads from a sheet called **Sheet1** in a book **test100**  
The service account file is **mycreds.json**  

```
import gspread
from oauth2client.service_account import ServiceAccountCredentials

book = "test100"
sheet = "Sheet1"

write_creds = ServiceAccountCredentials.from_json_keyfile_name(
    'mycreds.json', ['https://www.googleapis.com/auth/drive'])

write_client = gspread.authorize(write_creds)
write_book = write_client.open(book)
write_sheet = write_book.worksheet(sheet)

write_sheet.update('A1', 'foo')
print(write_sheet.acell('A1').value)
```
## Links
[gspread documentation](https://docs.gspread.org/en/latest/)  
