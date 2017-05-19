---
layout: post
title: Webscraping for Data Collection I
date: '2017-05-17 20:18:39 -0400'
categories: tutorial
published: true
---

# Webscraping for Data Collection

I ~love~ using webscraping to grab data from content on websites. In my day job this is something I end up using a lot, getting data from different websites, putting it back together, and it's also something I get a lot of questions about. Webscraping is particularily useful when 1) the service is not intended to provide data or 2) there is no good public api. This simple tutorial will build into my "Women's Representation in History" as I show how to webscrape data from webpages at different levels of diffculty. 

I'm not an expert in this! There is always a better way to solve a problem, but here are some methods that are pretty quick that should get you going, and also get you on the path to scraping your own data from webpages.

I plan to do at least 3 tutorials with different levels of capabilities. 

1. Very simple (this one!) basic request and parsing of html
    1. How to follow a link, and how to recursively scrape
3. Creating a spider, doing crawls, sending scraped data back to a data base
4. Handling javascript rendered pages (and looking for "hidden" apis)

### Basic page request and html parsing
Most of the time I use `lxml` and `requests` for light webscraping. There are other libraries, but I find that being conversant in these two are usually enough to solve more that 90% of my webscraping needs (you can see how some of my data collection from pt1 of my Women's Representation in History was done using this library [here](https://github.com/sjfwagner/Blog-code-snippets-and-notebooks/blob/master/SYMIH_Scrapes.ipynb)). Preferences may vary! [BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) is very popular. There *are* other HTTP request libraries, like urllib2 etc, but I have never seen a HTTP requests library recommended over requests. 


```python
#the two imports that we'll need immediately
import requests
from lxml import html
```

For this simple tutorial we'll be scraping a page from wikipedia, on historians in history. 
You can see the page here, and I also recommend that you view the source, as that will give you a good idea of the structure of the page we'll be scraping. Note! There are specific packages for wikipedia -- so if you want you should use them, this is an example of how to get started on scraping.

We'll start [here](https://en.wikipedia.org/wiki/List_of_historians), with a list of historians identified by wikipedia and then use the same techniques to move on to [here](https://en.wikipedia.org/wiki/List_of_historians_by_area_of_study), a list of historians identified by wikipedia for each area of study. There will be overlap, but I believe each list may have some different names. 


```python
url = 'https://en.wikipedia.org/wiki/List_of_historians'
response = requests.get(url)
```

Generically html is just a structured document, whose tags allow the browser to figure out how to display information. CSS and Javascript modify this, but most webpages present simple html. 

We can use the structure of the document, the tags, the pull features out of this. Xpath is a structured language for querying xml documents (great [tutorial](http://www.w3schools.com/xsl/xpath_intro.asp) at w3 schools) this allows us to easily query the document for the information that we want, because, lucky for us, html is a type of xml document. Xpath looks at the tags (like "div" etc) and the associated properties to reference different bits of the relevant html. 

It's super useful at this point to load the source (right-click view source) get a generic idea of the kind of html you'll be combing through. Keep this up throughout to use as a reference. If you'd like you can also pull up your browser's developer tools from the settings, many of which will highlight the section of html you're in while your mouse moves around the page. They will also provide the exact Xpath to the portion of the source you have highlighted *but this is generally way too specific, and will not extend well to many pages*


```python
wiki_page = html.fromstring(response.content)
```

Quickly, the xpath features I find it most useful to know are these: 
- "//" no matter the structure match to this tag 
    -i.e. "//div" will match every div tag
- xpath queries build, so "//div//li" will find *any* list tag **inside** *any* div tag
    - //div/li will return list items just inside a div tag
- [@....] the @ allows you to grab information from the tag itself
    -//div[@id="content"] will select any div tag *which* has a id of 'content'
    -//a[@title] will select all a tags with a title defined
There are other useful functions that you can use to more intelligently use xpath, but this will get you pretty far.



```python
historians = [] # create a json format of our data
for i in wiki_page.xpath('//div[@id="mw-content-text"]//li/a[@title][position()=1]'): #this xpath grabs all the list items, with a title
    print i.text #print the historian's name #unnecessary
    print i.xpath('@href') #print the link #unneccessary
    
    historians.append({"name":i.text, "url": "https://en.wikipedia.org"+i.xpath('@href')[0] })#store the full link
```

    Herodotus
    ['/wiki/Herodotus']
    Thucydides
    ['/wiki/Thucydides']
    Xenophon
    ['/wiki/Xenophon']
    Ctesias
    ['/wiki/Ctesias']
    Theopompus
    ['/wiki/Theopompus']
    Eudemus of Rhodes
    ['/wiki/Eudemus_of_Rhodes']
    Berossus
    ['/wiki/Berossus']
    Ptolemy I Soter
    ['/wiki/Ptolemy_I_Soter']
    ... [more]


At this point I've already extracted a bunch of useful information from a webpage! Let's step through. 
1. create an empty list for the data I want to save, called historians
2. the xpath here leads to all the list elements, and allows us to cycle through each one
    1. for each element, we can apply xpath that applys only within that element
    2. We print, for fun, and to make sure we're doing what we intend. 
        1. The name is just saved in the text of the tag, and .text gets the text for us! 
        2. The link is in the attribute href, notably the @href (link) tag comes back inside a list with *one* element **and** it comes back as a stub
    3. Save the info to a dictionary that we'll keep in a list
        1. name is under 'name'
        2. we add the root to the stub, and we index at [0] because the tag returns a list w/ just one value


```python
historians 
```




    [{'name': 'Herodotus', 'url': 'https://en.wikipedia.org/wiki/Herodotus'},
     {'name': 'Thucydides', 'url': 'https://en.wikipedia.org/wiki/Thucydides'},
     {'name': 'Xenophon', 'url': 'https://en.wikipedia.org/wiki/Xenophon'},
     {'name': 'Ctesias', 'url': 'https://en.wikipedia.org/wiki/Ctesias'},
     {'name': 'Theopompus', 'url': 'https://en.wikipedia.org/wiki/Theopompus'},
     {'name': 'Eudemus of Rhodes',
      'url': 'https://en.wikipedia.org/wiki/Eudemus_of_Rhodes'},
     {'name': 'Berossus', 'url': 'https://en.wikipedia.org/wiki/Berossus'},
     {'name': 'Ptolemy I Soter',
      'url': 'https://en.wikipedia.org/wiki/Ptolemy_I_Soter'},
     {'name': 'Duris of Samos',
      'url': 'https://en.wikipedia.org/wiki/Duris_of_Samos'},
     ...]



At this point it's time to start loading individual pages, and loading content. Here I explore with what xpath work best to extract the type of information I want to extract. 

To do gender at this moment, I'm counting how often the words 'he' and 'she' come up, as wikipedia biographies infrequently refer to their subject by name after the few few sentences, and instead use the gendered pronoun to call back. If there are more ' he ' than ' she ', I presume the article is male and vice-versa. This is isn't perfect, but we're just doing a quick and dirty investigation.

I also need to extract some sort of date to the article. Unfortunately not all historians have exhaustive articles and there are structural differences to the html, based on how high profile the article is. I turned to regex to pull the year from the the bio, regex is a text matching language, allowing you to define a pattern and return all matches. In this case I'm looking for 2,3,4 or for digit number, and I'm looking for the first time it's said. Wikipedia generally structures it's articles (even stubs) "Sophie XXX (c. 1991)" so we're depending on that logic, and our knowledge of wikipedia. You can see how scraped data can get messy quickly, with edge cases mucking up the data. 


```python
import re
```


```python
req_historian = requests.get(historians[7]['url'])
#print req_historian.text
print historians[6]['url'] # easy clicking to check
historian_page = html.fromstring(req_historian.text)
text = historian_page.xpath('//div[@id="mw-content-text"]/p//text()')
text = "".join(text)
text.replace("\u",'') #clean up
text.replace("\n",'') 
print
print text
print "'he'  counts:", text.count(' he '), text.count(' He ')
print "'she' counts:",text.count(' she '), text.count(' She ')


#test this before putting it online
year = re.compile("\d{3,4}")#this regex grabs years
    
hist_year = year.findall(text)[0]
hist_year = hist_year
print hist_year
```

    https://en.wikipedia.org/wiki/Berossus
    
    Ptolemy I Soter I (Ancient Greek: Πτολεμαῖος Σωτήρ, Ptolemaĩos Sōtḗr, i.e. Ptolemy (pronounced /ˈtɒləmi/) the Savior), also known as Ptolemy Lagides[1] (c. 367 BC – 283/2 BC), was a Macedonian Greek[2][3][4][5][6] general under Alexander the Great, one of the three Diadochi who succeeded to his empire. Ptolemy became ruler of Egypt (323–283/2 BC) and founded a dynasty which ruled it for the next three centuries, turning Egypt into a Hellenistic kingdom and Alexandria into a center of Greek culture. He assimilated some aspects of Egyptian culture, however, assuming the traditional title pharaoh in 305/4 BC. The use of the title of pharaoh was often situational: pharaoh was used for an Egyptian audience, and Basileus for a Greek audience, ... [ more ]
    'he'  counts: 20 4
    'she' counts: 0 0
    367


Note, I went through many of these to make sure this assumption passed at least the common sense challenge. Now we've got almost everything down, it's just down to parsing it and saving the data! 


```python
len(historians)
```




    1158



I'm going to loop through and apply everything I figured out above to everything else, storing it in a dictionary objected that makes it easy to cast this to pandas and save it out. 


```python
for i in historians:
    req_historian = requests.get(i['url'])
    historian_page = html.fromstring(req_historian.text)
    text = historian_page.xpath('//div[@id="mw-content-text"]/p//text()')
    text = "".join(text)
    text.replace("\u",'')
    text.replace("\n",'')
    #print
   
    he = text.count(' he ') + text.count(' He ')
    she= text.count(' she ') + text.count(' She ')
    
    year = re.compile("\d{3,4}")#this regex grabs years
    
    hist_year = year.search(text)
    try: 
        hist_year = hist_year.group()
    except AttributeError: 
        pass
        
    
    gender = ""
    if he > she: 
        gender = "Male"
    if she > he: 
        gender = "Female"
        #print i['url']
    
    i['gender'] = gender
    try:
        i['birth_year'] = hist_year
    except:
        i['birth_year'] = None
```


```python
historians #all done! 
```




    [{'birth_year': u'484',
      'gender': 'Male',
      'name': 'Herodotus',
      'url': 'https://en.wikipedia.org/wiki/Herodotus'},
     {'birth_year': u'460',
      'gender': 'Male',
      'name': 'Thucydides',
      'url': 'https://en.wikipedia.org/wiki/Thucydides'},
     {'birth_year': u'430',
      'gender': 'Male',
      'name': 'Xenophon',
      'url': 'https://en.wikipedia.org/wiki/Xenophon'},
     {'birth_year': u'401',
      'gender': 'Male',
      'name': 'Ctesias',
      'url': 'https://en.wikipedia.org/wiki/Ctesias'},
     {'birth_year': u'380',
      'gender': 'Male',
      'name': 'Theopompus',
      'url': 'https://en.wikipedia.org/wiki/Theopompus'},
     ...]




```python
import pandas as pd
```


```python
historians_data = pd.DataFrame(historians)
```


```python
historians_data.to_csv("wikipedia_historians.csv",encoding = 'utf8')
```

Now we're all done! 
