# Web Scraping 

### Premise
Often, the data we want is not available, or it is available on a website, but we cannot download it. American cities have done a very good job in recent years of developing "data portals" that allow for easy downloads of large amounts of data, and many major websites (Wikipedia, NYTimes, etc.) have developed APIs specifically for people wishing to download content from their sites. 

However, sometimes we want data that we can find on the web, but that is not pre-compiled for easy use, and is not available on a site with an API. This is where web scraping comes on. For our test case, we will look for census data about Johannesburg. [We know it exists](http://www.statssa.gov.za), but there is nowhere to download it directly. There is, however, one individual ([Adrian Firth](https://census2011.adrianfrith.com)) who built a website displaying all of this data in a clean and uniform way. Though it cannot be downloaded with one click, it can be scraped very simply, so that is what we are going to do. 

*Note: This website is remarkably well formatted and clean. Most sites you encounter will not be this well organized or simple. However, the same principles apply.*

There are a great many web scraping tutorials in the world, many of which are very good. This one is aimed at a true novice programmer, with some Python programming skill. You will not be writing code from scratch, but rather, modifying code as needed. For a more thorough introduction, I recommend [Chapter 11 of Automate the Boring Stuff](https://automatetheboringstuff.com/chapter11/) or the the [O'Reilly book, Web Scraping with Python](http://shop.oreilly.com/product/0636920034391.do). 

### Outcomes
By the end of this tutorial you will be able to:
* Determine if a website can be scraped
* Estimate how difficult a website will be to scrape
* Identify the key components of a website that are needed to scrape the site
* Modify Python code to scrape a website

### Downloads
I **highly** encourage you to download the [Anaconda](https://www.anaconda.com/download) distribution of Python 3 for this tutorial. This is both an excellent environment to learn Python in, and comes with all of the packages you will need. If you have used Python on your computer before, you may have some of the packages already. If you choose **not** to use Anaconda, you will need:

* [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) 
* [requests](http://docs.python-requests.org/en/master/user/install/)
* [pandas](https://pypi.org/project/pandas/)
* [numpy](http://www.numpy.org/)

This tutorial is written for use in an Jupyter Notebook, which is an interpreter that comes with Anaconda. You can use any interpreter you wish - the code will be the same, though the screenshots are from a Jupyter Notebook.

For web access, I recommend Google Chrome as it has some of the most robust and easily understood developer features. You can also use Firefox or Safari. Do NOT use Internet Explorer.

### General Idea
We have identified [a site](https://census2011.adrianfrith.com/place/798014056) that has the data we are interested in. We want the data in a format we can use (i.e., a csv, txt, xls, or other readable format). We also want the geometries that these numbers refer to. Looks like we can get both.

There are two flavors of web scraping: scraping static or interactive pages. We will start with static pages. For this, we will open a url, read the contents on the page, and parse that content. If this approach will dot fit your needs (i.e., you must click on something to activate something else), check out the [selenium](http://selenium-python.readthedocs.io/index.html) package. 

The bs4 package allows you to extract information from webpages by searching the HTML and CSS to find elements (tags, id's, classes, etc.). To make this work, we need to be able to *inspect* our websites. Inspecting a website means looking at the code the developer used to write it. Though there are best practices in web development, this code is often unique to the site you are scraping. However, you can be reasonably confident that most of the individual pages within a larger site will be uniformly constructed.


### Vocabulary

**List** An ordered set of values or strings ['house', 'apartment', 'condo', 'loft'] (*ordered here refers to the fact that 'house' is always in the same spot in the list.*)

**Dictionary** An unordered set of key:value pairs.  {'house': 2000, 'apartment': 800, 'condo': 800, 'loft':1200}
(*unordered here refers to the fact that every time you look at this list, the order of elements may change, though the "key" (i.e., 'house') is always associated with the "value" (i.e., 2000).*)

**String** Words and other units that Python can match, but DOES NOT EVALUATE i.e., 'house', 'apartment'

**DataFrame** Structure for data with rows and columns, similar to Excel


### General Practices

When writing any program it is essential to write out the steps. Even if the steps seem very basic, it will help to write out how you plan to approach the problem. The logic can get very confusing very quickly, and writing down each step in the most minute detail will help keep your program organized. [Flow charts and pseudocode](https://www.lynda.com/Programming-Foundations-tutorials/Writing-pseudocode/83603/90472-4.html) are invaluable tools in designing your program. It is much easier to think through the logic and then translate it into code than it is to do both at once. Often this pseudocode is written in iterations.

For our problem, we will:
1. Understand the organization of the website
2. Understand the organization of the data
3. Write a program that can 
    1. visit every site with data we want
    2. collect the data from that site
    3. handle errors and problems
    4. export the data to a csv file

### 1. Understand the Website

#### Organization
The first thing we need to do is get a sense of how the site is organized.

Visit https://census2011.adrianfrith.com/place/798 We see that this site has a city page then a list of links that take you to the 'main pages'. Let's look at a mainpage https://census2011.adrianfrith.com/place/798014. We see that the mainpages have their own subpages. Let's look at a subpage https://census2011.adrianfrith.com/place/798014056. This seems to be the smallest areal unit, so that is what we want. It is good practice to go through an make sure it is fairly uniform throughout by clicking on some other mainpages and their subpages. It looks like the data is organized in tables for each region. 

Since we are interested in using this data in a map, we are also interested in the shapefile associated with this data. There seems to be a link on the right to download the geometry for each location. This is great, because we know we can get this data and we know it exists. However, it is easier to deal with one problem at a time, so we will first focus on getting the data from the tables and then the geometries. 

So far, we know that we will have to:

  1. Start at the main page for Johannesburg 
  *for each link on the JHB page, follow it to the main page*
  
  2. Follow each link to each 'Main Place' 
  *for each link on the main page, follow it to the subpage*
  
  3. Follow each link to each 'Sub Place'
  *read the data on this page*
  
 
#### Categories
As I flip through the pages, I see that the categories are not always in the same order, and not every category appears on every page (i.e., the category, 'White' only appears on some pages, but not all). There are also two categories called 'Other', this could cause problems if we aren't careful.

The ordering issue means I need to match the name of the category to my list of pre-defined categories (that is, I can't just fill by position in the page since the first category isn't always the first thing to be read). This means I will have to use something unordered such as a dictionary (we would want to use a dictionary anyhow because it is a more reliable way to read the page). 

All of the categories that appear across all pages are on the main JHB page. Fantastic! That will be my master list.

#### Data
There seems to be raw numbers and percentages presented. I want the raw numbers, but not the percentages. The data also appears in 3 tables, we will preserve that structure when planning our web scraper.

#### Plan 
**Manual Tasks**
1. Make sure the subsites data is visible in the HTML
2. Make sure the links can be collected on the JHB page and the main pages

**Programmatic Tasks**
3. Make a list of all the features we will look for to make the columns of our dataframe (decide what to do with the 'Other' problem and *Not applicable*)
4. Set up our program and download all the packages we think we will need
5. Make a new dataframe that will hold all the data
6. Make a list of the links from the JHB site to each main place site

7. For each main place, make a list of the links to each subplace
8. Make a master list for Python to use to visit each subplace site
9. Visit each subplace and collect the data from the table
10. For each subplace,
    1. Read the data from the page
    2. Clean the data
    3. Store the data in a dictionary
    4. Pass the dictionary to a dataframe
    5. Add the new dataframe to the existing one
11. Save the dataframe as a csv
12. Check the csv
    1. If it worked, do a happy dance, you got your data!
    2. If it didn't work, debug

### Inspecting the Pages

**Step 1 Make sure the subsites data is visible in the HTML**
Go to the webpage you want to scrape. Right-click and select 'Inspect Element' 

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws1.png)

A box should appear on the right hand side with a lot of code. 

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws2.png)

There are a lot of tools here (at the bottom of this box) that could make our life easier. We won't use them now, but just know that they are there. 

Let's start with the JHB city site. We know that from here, we want to follow all of the links to other locations. 

If I move my cursor around in this box, it highlights different elements on the page. This indicates what part of the code on the right is making the page on the left. The code has little dropdown arrows. opening and closing those opens the containers that each thing sits in. Playing around like this, we see that the three grids are in three different containers. 
* Each container is called < table class="group">
* Each row is in a <tr class="datarow">
* The category is in the <td class = "namecell">
* The numbers are in the <td class = "datacell">
  
By right clicking on the title, I see that the name of this sub area is in the class attribute, 'topname'

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws3.png)

We now understand how our data is organized on our site.

**Step 2. Make sure the links can be collected on the JHB page and the main pages**
Now go back to the [JHB site](https://census2011.adrianfrith.com/place/798) Scroll to the list of mainplaces. Start clicking on them and see if there is a pattern to the numbers - if just the last 2 digits change (a common strategy), then we can just check pages incrementally. Unfortunately, there is not a pattern here, so we will have to figure something else out. 

Inspect the main place to see how the links are formated:

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws4.png)

We see it is in the `<td class="namecell">` in an `<a href   >` the href points us to the address extension, /place/798014 if we paste that into the browser after the https://census2011.adrianfrith.com/, we see that it takes us to the page we want. Perfect! We found the addresses. 
    
Let's double check that it's the same on the main page site by inspecting the element, and it is!

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws5.png)

**Step 3. Make a list of all the features we will look for**
There are Gender, Population, and Language tables on each page. 

Gender has 2 features: ['Male', 'Female']

Population has up to 5: ['Black African', 'Coloured', 'Indian or Asian', 'White', 'Other']

Language has up to 12: ['Sepedi', 'isiZulu', 'Xitsonga', 'Setswana', 'Sesotho', 'isiXhosa', 'Tshivenda', 'English', 'isiNdebele', 'Sign language', 'Afrikaans', 'SiSwati', 'Not Applicable', 'Other']


The order of this list does not matter because Python will match the strings rather than the index (position in the list).

**Open Python**

Find Anaconda on your computer (it might not be in your Applications, so you may have to search for it).

Launch a Jupyter Notebook
![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws6.png)

It will open in a browser window and you will see a list of all of your files. Open a New Python3 Notebook.

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws7.png)

A cell-by-cell interpreter will open.

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws8.png)

At this stage, I like to save my notebook - I can save it anywhere, it won't be reading any other files for now. It WILL be creating a file, so save it where ever you want your data to be saved. Word to the wise: Don't don't save it somewhere that iCloud will be involved, and avoid Dropbox if you can.

In the first cell, call all of the programs we will need
Type:

```import requests
from bs4 import BeautifulSoup
import pandas as pd
from collections import defaultdict
```

**Click `Shift` + `Return` to run the cell.** 

If nothing happens (rather, an asterisk appears in the brackets, and then the asterisk becomes a number), then it worked! 

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws9.png)

**Step 4. Set up our program and download all the packages we think we will need**

We just imported:
requests : a library for reading webpages
bs4 : a library for reading HTML
pandas : a library for data analysis - you need this to make dataframes
from the collections library, we imported defaultdict so we would be able to set the behavior of our dictionary (we need to use a special kind of dictionary)

In `NEXT CELL`, define the url to start from and set up the lists of categories we made earlier, and make a bunch of empty lists that we will fill.

```
url = 'https://census2011.adrianfrith.com'

genders=['Male', 'Female']
races=['Black African', 'Coloured', 'Indian or Asian', 'White', 'Other']
languages=['Sepedi', 'isiZulu', 'Xitsonga', 'Setswana', 'Sesotho', 'isiXhosa', 'Tshivenda', 'English', 'isiNdebele', 'Sign language', 'Afrikaans', 'SiSwati', 'Not Applicable', 'Other']

sites=[]
sitelist=[]
subsitelist=[]
```

**Step 5. Make a new dataframe that will hold all the data**

For now, we will actually make 3 dataframes that each have a locationID. We will use this locID to match them up later. 

In `NEXT CELL`, type

```locID = []

gender_name = []
gender_num = []
gender_df = pd.DataFrame(['locID', 'gender', 'count'])

race_name = []
race_num = []
race_df = pd.DataFrame(['locID', 'race', 'count'])

lang_name = []
lang_num = []
lang_df = pd.DataFrame(['locID', 'language', 'count'])
````

**Step 6. Make a list of the links from the JHB site to each main place site**

First visit the page and get the text, and then pass it to beautiful soup to make a page it can read

In `NEXT CELL`, type

page = requests.get("https://census2011.adrianfrith.com/place/798")
apage = BeautifulSoup(page.text, 'lxml')

You will get a warning about the default language, and how to specify a language so your program can be used across multiple environments. This will not affect anything, so just ignore it. 

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws10.png)

If you hate the warning, use this code instead to specify the markup:

apage = BeautifulSoup(page.text, 'lxml')


It would be great if we could just get all of the links on the page, but as it turns out, there are other links that we do NOT want to get. All of the links we do want are contained in tr tags.

Instead, we will find all of the tr tags on the page, and within the tr tags, find all the 'a' tags and collect these into a list. For every element in the list, we will find the contents of the href, and add those contents to our sitelist.

In `NEXT CELL`, type

`for tr in apage.find_all('tr'):
    aref = tr.find_all('a')
    for a in aref:
        sitelist.append(a.get('href'))`

This is the first time we've collected something from outside our program.
But, when we run it nothing happens, it would be great if we could see the result to see if it matches what we expected. Let's print it to make sure we have all the links we think we should have.

In `NEXT CELL`, type

print(sitelist)

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws11.png)

**Step 7. For each main place, make a list of the links to each subplace** 

**Step 8. Make a master list for Python to use to visit each subplace site**

Now let's use that list to go out and find all the subsites. Just like before, we will get the site, but this time, from the url, and read it into our program as a beautiful soup object. We will then find all of the 'tr' tags and make a list of all the a tags (name aref), and for each element in the aref list, find it's contents, and add the contents to the subsite list

In `NEXT CELL`, type

`for site in sitelist:
    bpage = requests.get(url+site)
    cpage = BeautifulSoup(bpage.text, 'lxml')
    for tr in cpage.find_all('tr'):
        aref = tr.find_all('a')
        for a in aref:
            subsitelist.append(a.get('href'))`
            
This will probably take a very long time to compile.

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws12.png)

To check to be sure everything works, we only need to go through part of our site, so let's take a slice of our list rather than the whole list. 

In `NEXT CELL`, type
`sitelist[:25]`

If a list of url suffixes appears, it worked. We have a list of all the extensions for the subsite locations, and can go visit them to get the data.


**Step 9. Visit each subplace and collect the data from the table**
Again, we will go out to the page location and visit it, though we will only visit the first 25 in the list to save time.

The next command is huge, so we will break it down before putting it into our Notebook. It is huge because we have bundled a bunch of instructions into one 'for-loop'. So for each page, we do a whole bunch of things before doing it again for the next page. This is NOT the most efficient way to do this, but it is easier to understand, and accomplishes our goals.


1. go out to all of the subsites ( which we make by concatenating the url+subsite) and read in the text. 
`for subsite in subsitelist[:25]:
    dpage = requests.get(url+subsite)
    mypage = BeautifulSoup(dpage.text, 'lxml')`

2. make a new list for the location name (that will get rewritten for every item in the loop), find the location name by selecting the CSS id, .topname. We only want the first instance of it (in case there are more), so we select the zeroth element (Python starts counting at zero). We will get the text from the element (the stuff between the opening and closing tags), and add it to the list of location names that we are making. 

**Step 10 For each subplace**

**1. Read the data from the page**

**2. Clean the data**

**3. Store the place names in a list**

**4. Store the data in a dictionary**

**5. Pass the dictionary to a dataframe**

**6. Add the list of locations to the dataframe**

**7. Add the new dataframe to the existing one**
    
Next, we will start a new list, and call it 'mylist' to collect all the items in the td tags that are within tr tags. A more elegant way to do this would be to select by the tags. 

We will make yet another list that will get rewritten now, to remove the percentages that we don't want. Again, this removes data based on position, not ideal. This should be rewritten, but it works for now. We want to remove every 3rd entry from the list, so we will make a new list with just the first 2 entries. 

Then we create a dictionary of lists. A dictionary is a key:value pair, so every key will be the category name, and every value will be the raw numbers. 

Now we need to put the list into a dictionary (to then get it into a Dataframe), and handle the exceptions. First, we want to get rid of anything called 'Other', and anything called 'Not applicable'. Though this is less than ideal, it allows us to proceed in getting the data with less trouble. So for all the pairs in the statlist, if they are 'Other', skip it, or if they are 'Not applicable', skip it. Strings are case sensitive, so 'Other' is not the same thing as 'other'. If it's not either of those, add the value to the dictionary under the given key.

Finally, we will start another dataframe and call it df1, we want df1 to be fresh every time we go to a new page. We will make the dataframe from the dictionary. The default value is to make the keys be the indexes, so it would be one column. I want every column to be a key to match the dataframe I set up in the first place. So I change the index and transpose it. 

Then I want to add the name of the place, so I turn my list into a Series, which is a special, ordered datatype that can be added to a dataframe. I then add it to my dataframe with a new column header, 'locID'

Finally, I append this new dataframe to the one I established at the beginning - the one *outside* of the for-loop, and fill in any missing values with 0

In `NEXT CELL`, type

`for subsite in subsitelist[:5]:
    dpage = requests.get(url+subsite)
    mypage = BeautifulSoup(dpage.text)
    
    locats=[]
    elem = mypage.select('.topname')
    locat=elem[0].getText()
    locats.append(locat)
    
    mylist = []
    for tr in mypage.find_all('tr'):
        tds = tr.find_all('td')
        for item in tds:
            mylist.append(item.getText())

    statlist = []
    num = len(mylist)//3
    for n in range(num):
        x = n*3
        y = 1+(n*3)
        statlist.append((mylist[x], mylist[y]))

    mydict = defaultdict(list)
    
    for k,v in statlist:
        if k == "Other":
            pass
        elif k == "Not applicable":
            pass
        else:
            mydict[k].append(v)
            
    df1 = pd.DataFrame.from_dict(mydict, orient='index').transpose()
    se = pd.Series(locats)
    df1['locID'] = se.values
    df=df.append(df1).fillna(value=0)`
    
Let's check if it worked by printing our df

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws14.png)

You can look at different sections by changing the slice of the list. Remember that Python counts from 0, so we are looking at entries [0, 1, 2, 3, 4] 

See what is in [5:15] or other areas. 

To actually scrape this site, we would remove the list slice completely, but that will really take a long time. 

**Step 11. Save the dataframe as a csv**

Finally, we save the whole dataframe to a csv on our computer. This will be saved in the same directory (folder) that your file is currently in.

In `NEXT CELL`, type

df.to_csv('jhb_census.csv')

Now if we run this, it will download all of the data to our csv file!!

**Step 12. Check the csv**

Open with Excel or another program

![blank](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/blob/master/img/ws15.png)


Our next task is to download the kml's. The code for that is in the data download. This is written as 2 programs - making a list of locations and downloading the data has been separated. It is VERY SLOW, downloading entire files is an involved process. This will be provided to you in the next tutorial. 

The complete code is available [on Github in the Data Folder](https://github.com/michellejm/ConflictUrbanism-InfraPolitics/tree/master/Data/04_Webscraping) both as an iPython Notebook and as a Python program. 

To complete this tutorial on your own, send Michelle (mam2518) your csv file of a SLICE of the data and tell me what slice you chose.
********************************************************

This tutorial was prepared by Michelle McSweeney for the Conflict Urbanism: InfraPolitics Course offered by the [Center for Spatial Research](http://c4sr.columbia.edu) at Columbia University in Fall 2017. 
