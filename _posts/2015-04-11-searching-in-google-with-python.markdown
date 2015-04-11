---
layout: post
title: "Searching in Google With Python"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2015-04-11T16:48:18+02:00
---

In this post we will cover how to connect to Google Custom Search API for retrieving google's search results about a particular topic.

The first step is to create a project. In this context, a project is the google's side application. I mean, if we make an application for retriving data, we also need to register it in  developers site to be able to comsume google's services.

#### Building a Google's Project

In this section, we will cover how to build a project in the google's developer console.

* Go to: https://console.developers.google.com
* Click on "Create Project"
	![Create_Project](/images/20150411/en/step1.png)
* Look for "Custom Search API" in the API section and click on "Enable API"
	![Enable_API_1](/images/20150411/en/step2.png)
	![Enable_API_2](/images/20150411/en/step3.png)
* Go to "Credentials" and click on "Create New Key" and choose "Server Key"
	![Key_API_1](/images/20150411/en/step4.png)
* Before that, You have to copy "API key" field which will be the neccesary code to be able to connect google's custom search API and save it for later.
	![Key_API_2](/images/20150411/en/step5.png)

#### Getting a Google's Search Engine

One disadvantage of this method is that you also need to build a search engine. Google enables you to make your own search engine for looking any topic under certain URLs. The steps, to achieve that, are listed here:

* Go to: https://cse.google.com/cse/all and click on "New search engine" or "Add" button
	![Create_Engine](/images/20150411/en/2_step1.png)
* Insert URLs for building your search domain (those URLs which will be scanned)
	![Build_Engine](/images/20150411/en/2_step2.png)
* Once you have created the engine go to "Edit search engine" 
	![Edit_Engine_1](/images/20150411/en/2_step4.png)
and you will seee someting like this:
	![Edit_Engine_2](/images/20150411/en/2_step3.png)
Then, push "search engine ID" button for getting the engine ID (and save it for later)

#### Making our application

We will nedd some Python libraries to be successful, import them:

```{python}
import json
import pandas as pd
from collections import namedtuple
from pandas import DataFrame as df_
from apiclient.discovery import build
```

We will also code some useful classes for filling with the data returned by google:

```{python}
class QueryItem(object):
    def __init__(self,dic):
        self.__dict__.update(dic)        
    def __getitem__(self,name):
        return self.__dict__[name]       
        
class QueryResult(object):
    def __init__(self,dic):
        self.__dict__.update(dic)
        self.items = [QueryItem(i) for i in self.items if type(i)==dict]        
    def __getitem__(self,name):
        return self.__dict__[name]

Config = namedtuple('Config',['developerKey','cx'])
```

We will use `QueryResult` for the main content of data and `QueryItem` for each delivered result (each link in a the google response). We also need a class for handling the json configuration for the app, so we will use `Config` namedtuple which works like a class in practise.

The next step is the big deal... We make the body of our application:

```{python}
def searchGoogle(terms='lectures',configFile='customsearch.json',**kwargs):      
  config = Config(**json.load(open(configFile,'r')))    
  service = build("customsearch","v1",developerKey=config.developerKey)  
  res = service.cse().list(q=terms,cx=config.cx,**kwargs).execute()  
  return QueryResult(res)
```

In that function, we see the next parts:

* Loading json config. file where `developerKey` is our API key and `cx` our engine ID. (so we will need a *.json file like this)
	```{json}
	{
	    "developerKey": "AIza................................n4",
	    "cx":"034......................a-k"
	}
	```
* Make the custom search service `service = build("customsearch",...`
* Request for some terms `res = service.cse().list(q=terms,...`
* Return a well-formatted object `return QueryResult(res)`

In last term I give you a working sample of code:

```{python}
if __name__ == '__main__':
    iterator,maxIter = 0,3    
    data = []
    res  = searchGoogle('S&P 500')  
    data += [df_({k: [item[k] for item in res.items] for k in ['link','title','snippet']})]
    while res.queries.has_key('nextPage') and iterator < maxIter:
        res = searchGoogle('S&P 500',start=res.queries['nextPage'][0]['startIndex'])    
        data += [df_({k: [item[k] for item in res.items] for k in ['link','title','snippet']})]
        iterator+=1
    data = pd.concat(data).reset_index(drop=1)
```
