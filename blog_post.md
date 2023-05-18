# Google Drive API

This tutorial will demonstrate how to use the Google API Python client to manipulate a Google Document. The API allows users to interact with and manipulate Google Drive and Google documents in Python. While this library is in maintenance mode, it is still being maintained. 

[Official Github documentation](https://github.com/googleapis/google-api-python-client)

## What to install and import

Open up a Github Codespaces to complete this tutorial. In the command line, pip install the following libraries:

```
$ pip install google-api-python-client bs4 requests google-auth google-auth-oauthlib google-auth-httplib2 aiohttp datetime
```

At the top of your .py file, add these lines to import the following libraries:

``` Python
import os
import requests
from bs4 import BeautifulSoup
from google.oauth2 import service_account
from googleapiclient.discovery import build
import aiohttp
import datetime
import logging
logging.basicConfig(level=logging.DEBUG)
```

These libraries will allow you to interact with the Google Drive API and manipulate a Google Document.

## Create a Google APIs Console project and create a Google service account

1. To interact authenticate the API, you will need to create a [Google Cloud](https://console.cloud.google.com/) project. Go to that website and sign up for a Google account or sign into an existone one.
2. Read through the [Managing Projects](http://developers.google.com/console/help/managing-projects) page and create a project in the Google API console.
3. Create a service account and download the associated .json file. There are various ways to authenticate the API, but this is the one I found to work for me. A service account is a type of Google account that is used to acces Google APIs on behalf of an application/service. It's associated with a unique email address that identifies the application. 
4. Input the contents of the .json file into your Github Codespaces Secrets.

## Authenticate the Google API in Codespaces

Add this code to your .py file:
   ```Python
   # Load the Google service account JSON secret from your Codespaces secret
    service_account_info = json.loads(os.environ['GOOGLE_SERVICE_SECRET'])

    # Create a credentials object from the service account info
    creds = service_account.Credentials.from_service_account_info(service_account_info)

    # Define the scopes that the application needs access to
            SCOPES = ['https://www.googleapis.com/auth/drive']

    # Set up the Drive API client
    drive_service = build('drive', 'v3', credentials=creds)
   ```

## Reading a Google Document

You cannot view the page source fo a Google Document like you can with a regular web page. To read in a Google Document and parse through it, you need to turn it into something that can actually be parsed. An easy way to do this is using a Python library called BeautifulSoup. 

Add this code to your .py file:
```Python
# Retrieve the content of the Google Doc in HTML format
file_id = '1wBtEXq3LsxVOjqMT0RoEoQxbJShfPu1H2M0gLsJOOY8' #this is the part of the URL link that is between "document/" and "/edit"
export_mime_type = 'text/html' #this line exports my Google Doc in html format
response = drive_service.files().export(fileId=file_id, mimeType=export_mime_type).execute() #this line sends the request to the Google Drive API

# Extract the HTML content from the API response
doc_content = response.decode('utf-8') #this line takes the file from the Google Drive API and decodes it to a UTF-8 string
```

## Manipulate the Google Document

Now that the Google Document is in HTML, we can parse through it and pull out just the stuff we want to look at.

For this next section of code, I used BeautifulSoup to search for all ```<span>``` tags in the HTML becuase those tags contained the style entries I wanted to capture.

Add this code to your .py file:
```Python
# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(doc_content, 'html.parser')

# Find the span tags in the HTML
span_tag = soup.find_all("span")

# Create a list of each style entry
clean_style_tips = []

for row in span_tag:
    clean_style_tips.append(row.text.strip())

```

Now, I've pulled out each style entry in my Google Document and created a list of the items. In future iterations of this project, it would be important to add a cleaning section to ensure the list is only capturing what you want and making sure there are no duplicates in the list.

BeautifulSoup does not produce perfect HTML code for a Google Document. Because the library generate the HTML code, there are no human-created class or id tags that make it easy to grab the information you want. I found that to ensure I had a clean list of style entries, I would've had to add a lot more BS4 manipulation and data cleaning to this script.

### Thanks for following along!