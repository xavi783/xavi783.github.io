---
layout: post
title: "Searching in Google With Python"
modified:
categories:
tags: [google, python, APIs, text processing]
image:
  feature:
date: 2015-04-11T16:48:18+02:00
comments: trues
---

In this post we will cover how to connect to Google Custom Search API for retrieving google's search results about a particular topic.

The first step is to create a project. In this context, a project is the google's side application. I mean, if we make an application for retriving data, we also need to register it in  developers site to be able to comsume google's services.

#### Building a Google's Project

In this section, we will cover how to build a project in the google's developer console.

* Go to: https://console.developers.google.com
* Click on "Create Project"
	<figure><img src="/images/20150411/en/step1.png" alt="Create_Project"></figure>
* Look for "Custom Search API" in the API section and click on "Enable API"
	<figure class="half">
		<img src="/images/20150411/en/step2.png" alt="Enable_API_1">
		<img src="/images/20150411/en/step3.png" alt="Enable_API_2">
	</figure>
* Go to "Credentials" and click on "Create New Key" and choose "Server Key"
	<figure><img src="/images/20150411/en/step4.png" alt="Key_API_1"></figure>
* Before that, You have to copy "API key" field which will be the neccesary code to be able to connect google's custom search API and save it for later.
	<figure><img src="/images/20150411/en/step5.png" alt="Key_API_2"></figure>

#### Getting a Google's Search Engine

One disadvantage of this method is that you also need to build a search engine. Google enables you to make your own search engine for looking any topic under certain URLs. The steps, to achieve that, are listed here:

* Go to: https://cse.google.com/cse/all and click on "New search engine" or "Add" button
	<figure><img src="/images/20150411/en/2_step1.png" alt="Create_Engine"></figure>
* Insert URLs for building your search domain (those URLs which will be scanned)
	<figure><img src="/images/20150411/en/2_step2.png" alt="Build_Engine"></figure>
* Once you have created the engine go to "Edit search engine" 
	<figure><img src="/images/20150411/en/2_step4.png" alt="Edit_Engine_1"></figure>
and you will seee someting like this:
	<figure><img src="/images/20150411/en/2_step3.png" alt="Edit_Engine_2"></figure>
Then, push "search engine ID" button for getting the engine ID (and save it for later)

#### Making our application

We will nedd some Python libraries to be successful, import them:

{% highlight python %}
import json
import pandas as pd
from collections import namedtuple
from pandas import DataFrame as df_
from apiclient.discovery import build
{% endhighlight %}

We will also code some useful classes for filling with the data returned by google:

{% highlight python %}
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
{% endhighlight %}

We will use `QueryResult` for the main content of data and `QueryItem` for each delivered result (each link in a the google response). We also need a class for handling the json configuration for the app, so we will use `Config` namedtuple which works like a class in practise.

The next step is the big deal... We make the body of our application:

{% highlight python %}
def searchGoogle(terms='lectures',configFile='customsearch.json',**kwargs):      
  config = Config(**json.load(open(configFile,'r')))    
  service = build("customsearch","v1",developerKey=config.developerKey)  
  res = service.cse().list(q=terms,cx=config.cx,**kwargs).execute()  
  return QueryResult(res)
{% endhighlight %}

In that function, we see the next parts:

* Loading json config. file where `developerKey` is our API key and `cx` our engine ID. (so we will need a *.json file like this)
	{% highlight json %}
	{
	    "developerKey": "AIza................................n4",
	    "cx":"034......................a-k"
	}
	{% endhighlight %}
* Make the custom search service `service = build("customsearch",...`
* Request for some terms `res = service.cse().list(q=terms,...`
* Return a well-formatted object `return QueryResult(res)`

In last term I give you a working sample of code:

{% highlight python %}
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
{% endhighlight %}
