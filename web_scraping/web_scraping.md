# Web Scraping 

### Premise

Often, the data we want is not available, or it is available on a website, but we cannot download it. American cities have done a very good job in recent years of developing "data portals" that allow for easy downloads of large amounts of data, and many major websites (Wikipedia, NYTimes, etc.) have developed APIs specifically for people wishing to download content from their sites. 

However, sometimes we want data that we can find on the web, but that is not pre-compiled for easy use, and is not available on a site with an API. This is where web scraping comes on. For our test case, we will look for census data about Johannesburg. [We know it exists](http://www.statssa.gov.za), but there is nowhere to download it directly. There is, however, one individual ([Adrian Firth](https://census2011.adrianfrith.com)) who built a website displaying all of this data in a clean and uniform way. Though it cannot be downloaded with one click, it can be scraped very simply, so that is what we are going to do. 

*Note: This website is remarkably well formatted and clean. Most sites you encounter will not be this well organized or simple. However, the same principles apply.*

There are  many web scraping tutorials in the world, many of which are very good. This one is aimed at a novice programmer, with some Python programming skill. You should be familiar with importing packages and understand the basic mechanics of a for-loop and if-statement. You will not be writing code from scratch, but rather, modifying code as needed. For a more thorough introduction, I recommend [Chapter 11 of Automate the Boring Stuff](https://automatetheboringstuff.com/chapter11/) or the the [O'Reilly book, Web Scraping with Python](http://shop.oreilly.com/product/0636920034391.do). 

### Outcomes
By the end of this tutorial you will be able to:
* Determine if a website can be scraped
* Estimate how difficult a website will be to scrape
* Identify the key components of a website that are needed to scrape the site
* Modify Python code to make calls and requests.

### Downloads

I **highly** encourage you to download the [Anaconda](https://www.anaconda.com/download) distribution of Python 3 for this tutorial. This is both an excellent environment to learn Python in, and comes with all of the packages you will need. If you have used Python on your computer before, you may have some of the packages already. If you choose **not** to use Anaconda, you will need:

* [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) 
* [requests](http://docs.python-requests.org/en/master/user/install/)
* [pandas](https://pypi.org/project/pandas/)

This tutorial is written for use in an Jupyter Notebook, which is an interpreter that comes with Anaconda. You can use any interpreter you wish - the code will be the same, though the screenshots are from a Jupyter Notebook.

For web access, I recommend Google Chrome as it has some of the most robust and easily understood developer features. You can also use Firefox or Safari. Do NOT use Internet Explorer.

### General Idea

We have identified [a site](https://census2011.adrianfrith.com/place/798014056) that has the data we are interested in. We want the data in a format we can use (i.e., a csv, txt, xls, or other readable format). We also want the geometries that these numbers refer to. Looks like we can get both.

There are two flavors of web scraping: scraping static or interactive pages. We will start with static pages. For this, we will open a url, read the contents on the page, and parse that content. If this approach will dot fit your needs (i.e., you must click on something to activate something else), check out the [selenium](http://selenium-python.readthedocs.io/index.html) package.

The bs4 package allows you to extract information from webpages by searching the HTML and CSS to find elements (tags, id's, classes, etc.). To make this work, we need to be able to *inspect* our websites. Inspecting a website means looking at the code the developer used to write it. Though there are best practices in web development, this code is often unique to the site you are scraping. However, you can be reasonably confident that most of the individual pages within a larger site will be uniformly constructed.


### General Practices

When writing any program it is essential to write out the steps. Even if the steps seem very basic, it will help to write out how you plan to approach the problem. The logic can get very confusing very quickly, and writing down each step in the most minute detail will help keep your program organized. [Flow charts and pseudocode](https://www.lynda.com/Programming-Foundations-tutorials/Writing-pseudocode/83603/90472-4.html) are invaluable tools in designing your program. It is much easier to think through the logic and then translate it into code than it is to do both at once. Often this pseudocode is written in iterations.

For our problem, we will:
1. Understand the organization of the website
2. Write a program that can 
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

The ordering issue means  I can't just fill by position in the page since the first category isn't always the first thing to be read. One way to approach this is to make a separate dataframe for each location and then let pandas do the matching up at the end. 


#### Data

There seems to be raw numbers and percentages presented. I want the raw numbers, but not the percentages. The data also appears in 3 tables, we will preserve that structure when planning our web scraper.

#### Plan 

**Manual Tasks**
1. Find out where the links are on the page and figure out structure of the pages.
2. Make sure the links can be collected on the JHB page and the main pages

**Programmatic Tasks**
3. Set up our program and download all the packages we think we will need
4. Make a list of all the links we want to visit
5. Visit each subplace and collect the data from the tables
6. Reshape the data as needed
7. Save the dataframe to a csv



### Inspecting the Pages

**Step 1. Make sure the subsites data is visible in the HTML**
Go to the webpage you want to scrape (https://census2011.adrianfrith.com/place/798). Right-click and select 'Inspect Element' 

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws1.png)

A box should appear on the right hand side with a lot of code. 

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws2.png)

There are a lot of tools here (at the bottom of this box) that could make our life easier. We won't use them now, but just know that they are there. 

Let's start with the JHB city site. We know that from here, we want to follow all of the links to sublocations and the sub-sublocations. 

If you move you cursor around in this box, it will highlight different elements on the page. This indicates what part of the code on the right is making the page on the left. The code has little dropdown arrows. opening and closing those opens the containers that each thing sits in. Playing around like this, we see that the three tables are in three different containers. 
* Each container is called < table class = "group" >
* Each row is in a < tr class = "datarow" >
* The category is in the < td class = "namecell" >
* The numbers are in the **second** < td class = "datacell" >
  
By right clicking on the title, we see that the name of this sub area is in the class attribute, 'topname'

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws3.png)


**Step 2. Make sure the links can be collected programmatically**

Now go back to the [Johannesburg site](https://census2011.adrianfrith.com/place/798) Scroll to the list of mainplaces. Start clicking on them and see if there is a pattern to the numbers - if just the last 2 digits change (a common strategy), then we can just check pages incrementally. Unfortunately, it's not incremental, so we will have to first collect the extensions. 

Inspect the main place to see how the links are formated:

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws4.png)

We see it is in the `<td class="namecell">` in an `<a href   >` the href points us to the address extension, /place/798014 if we paste that into the browser after the https://census2011.adrianfrith.com/, we see that it takes us to the page we want. We found the addresses. 
    
Let's double check that it is the same on the main page site by inspecting the element.

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws5.png)

**Step 3. Set up the program**

**Open Python**

Find Anaconda on your computer (it might not be in your Applications, so you may have to search for it).

Launch a Jupyter Notebook
![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws6.png)

It will open in a browser window and you will see a list of all of your files. Open a New Python3 Notebook.

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws7.png)

A cell-by-cell interpreter will open.

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws8.png)

At this stage, we should name and save my notebook - we can save it anywhere, it won't be reading any other files for now. It WILL be creating a file, so save it where ever you want your data to be saved. If you can avoid it, don't don't save it somewhere that iCloud will be involved.

In the first cell, we will call all of the programs we will need

Type:

```import requests
from bs4 import BeautifulSoup
import pandas as pd
import csv
```

**Click `Shift` + `Return` to run the cell.** 

If nothing happens (rather, an asterisk appears in the brackets, and then the asterisk becomes a number), then it worked. 

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws9.png)

**Step 4. Set up our program and download all the packages we think we will need**

We just imported:

requests : a library for reading webpages
bs4 : a library for reading HTML
pandas : a library for data analysis - you need this to make dataframes
csv : a built-in package for reading and writing to csv's.

In `NEXT CELL`, define the url to start from and set up the empty sitelist lists.

```
url = 'https://census2011.adrianfrith.com'

sites=[]
sitelist=[]
subsitelist=[]
```

**Step 5. Make a list of the links we will want to visit**

First visit the page and get the text, and then pass it to beautiful soup to make a page it can read

In `NEXT CELL`, type

```
page = requests.get("https://census2011.adrianfrith.com/place/798")
apage = BeautifulSoup(page.text, 'lxml')
```

You may get a warning about the default language, and how to specify a language so your program can be used across multiple environments. This will not affect anything, so just ignore it. 

![dataframe](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws10.png)

To get rid of the warning, use this code instead to specify the markup:

`apage = BeautifulSoup(page.text, 'lxml')`


It would be great if we could just get all of the links on the page, but as it turns out, there are other links that we do NOT want to get. All of the links we do want are contained in 'tr' tags.

Instead, we will find all of the 'tr' tags on the page, and within the 'tr' tags, find all the 'a' tags and collect these into a list. For every element in the list, we will find the contents of the href, and add those contents to our sitelist.

In `NEXT CELL`, type

```for tr in apage.find_all('tr'):
    aref = tr.find_all('a')
    for a in aref:
        sitelist.append(a.get('href'))
```

This is the first time we've collected something from outside our program.
But, when we run it nothing happens, it would be great if we could see the result to see if it matches what we expected. Let's print it to make sure we have all the links we think we should have.

In `NEXT CELL`, type

`print(sitelist)`

![sitelist](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws11.png)


Now we will use that list to visit these sites and find the subsites. Just like before, we will save the url, but this time, we will loop through our list of sites, and read each site into our program as a beautiful soup object. We will then find all of the 'tr' tags and make a list of all the 'a; tags (name 'aref'), and for each element in the 'aref' list, find its contents, and add those to the subsite list.

In `NEXT CELL`, type

```for site in sitelist:
    bpage = requests.get(url+site)
    cpage = BeautifulSoup(bpage.text, 'lxml')
    for tr in cpage.find_all('tr'):
        aref = tr.find_all('a')
        for a in aref:
            subsitelist.append(a.get('href'))
```
            
This will probably take a long time to compile. If you need to stop it, use the Keyboard interrupt button

![stop button](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws12.png)

To check to be sure everything works, we only need to view part of our site, so let's take a slice of our list rather than the whole list. 

In `NEXT CELL`, type
`subsitelist[:25]`

If a list of url suffixes appears, it worked. We have a list of all the extensions for the site locations, and can go visit them to get the subsites.


**Step 6. Visit each subplace and collect the data from the table**

We will define a function to:
* visit a site
* set up an empty container structure for each table (lists and a dataframe)
* gather the location name
* gather the gender data
* gather the race data
* gather the language data
* input the gender, race, and language lists into the respective dataframes
* return the dataframes

1. define the function and go to a site (which we make by concatenating the url+subsite) and read in the text.  

```def scrape_census_data(subsite):
    dpage = requests.get(url+subsite)
    mypage = BeautifulSoup(dpage.text, 'lxml')
```

2. set up the container structure 

We need a name list for the categories, a number list for the counts, and a location list. Though the location will stay the same and therefore be very redundant, we will add it to the list every time so that we can use it as a key later and reduce a lot of the redundancy. This is not the only way to set up this structure, but is much more straightforward than some other options(thank you to Will Greary, a student in [Conflict Urbanism Fall 17](http://c4sr.columbia.edu/infra-politics) for pointing this out). 

```gender_name = []
    gender_num = []
    gender_loc = []
    gender_df = pd.DataFrame(columns=['locID', 'gender', 'count'])

    race_name = []
    race_num = []
    race_loc = []
    race_df = pd.DataFrame(columns=['locID', 'race', 'count'])

    lang_name = []
    lang_num = []
    lang_loc = []
    lang_df = pd.DataFrame(columns=['locID', 'language', 'count'])

```

3. get the location

This command tells our program to find the first (0th) instance of the css selector 'topname' (the leading period communicates )

```
locat = mypage.select('.topname')[0].getText()

```

Thinking about how we will use the data and what the data represents, it will be best to collect the gender, race, and language data separately since they are different types of data and 3 ways of categorizing a population. These tables appear in the same order on every page. Make a note of this as it may be helpful. 


4.  Get the gender data

This is the first table, so we will find all of the tables, and then find all of the 'td' tags. We don't want to collect the percentages - just the raw numbers, and we notice that all of the percentages have the % symbol in them. So, if a cell has a %, skip it.


``` gender_table = mypage.find_all("table")[0]
    for td in gender_table.find_all("td"):
        if '%' not in td.text:
            try:
                value = float(td.text)
                gender_num.append(value)
                gender_loc.append(locat)
            except:
                gender_name.append(td.text)
                
```

5. Get the race data

Same as the gender data, except go to the 2nd table in the list (position 1)

```
    # Get the second table about race
    race_table = mypage.find_all("table")[1]
    for td in race_table.find_all("td"):
        if '%' not in td.text:
            try:
                value = float(td.text)
                race_num.append(value)
                race_loc.append(locat)
            except:
                race_name.append(td.text)
```
6. Get the language data

This time, we need to do a little more error checking and handle exceptions. If we write our code the first way and just replicate it for the language table, we get an error saying "ValueError: Length of values does not match length of index" because some of our cells are empty. We can fix that by specifying that td.text should not be empty. 

If we just add that line, we get the error: "ValueError: invalid literal for int() with base 10: '27.34%'", meaning that the value being passed is still the percentage - not the raw value. We will handle this by only collecting location and names from the even positions and the values from the odd positions. So every row has 3 positions: position 0 in the list of td's is even and contains the name, and position 1 in the list of td's is odd, and contains the count, and position 2 is even and contains the percentage. We will use a counter and the modulo operator to determine "even-ness".

```
    language_table = mypage.find_all("table")[2]
    count = 0
    for td in language_table.find_all("td"):
        if (('%' not in td.text) and (td.text != "")):
            if count % 2 == 0:
                lang_loc.append(locat)
                lang_name.append(td.text)
                count += 1
            else:
                value = float(td.text)
                lang_num.append(value)
                count += 1

```

7. input the gender, race, and language lists into the respective dataframes and return the dataframes

```
    gender_df['locID'] = gender_loc
    gender_df['gender'] = gender_name
    gender_df['count'] = gender_num

    race_df['locID'] = race_loc
    race_df['race'] = race_name
    race_df['count'] = race_num
    
    lang_df['locID'] = lang_loc
    lang_df['language'] = lang_name
    lang_df['count'] = lang_num

    return gender_df, race_df, lang_df
```

Finally, we will call our new function for each subsite in our subsite list, but we will try it with a slice first just to make sure. 

```
for subsite in subsitelist[:10]:
    gender, race, language = scrape_census_data(subsite)
```

If you do not get an error, it worked! If you want to see what you have, go ahead and print any of the data frames. Now we are going to extend this to the entire list and collect our results in a series of lists, we will then fill our dataframes with these lists. 

Now, if you take away the slice and run it on the entire list of subsites, you will get an error something like this:"IndexError: list index out of range"

The problem is that some pages don't have any data on them, or don't have data for all the categories. We don't have a good way to deal with this, so we will omit these pages by just passing them over (as it turns out, this isn't actually a problem). We will embed the entire code block that reads the pages in a try/except structure.  

So, all together, the code is as follows:

``` def myscraper(subsite):
    dpage = requests.get(url+subsite)
    mypage = BeautifulSoup(dpage.text, 'lxml')
    
    gender_name = []
    gender_num = []
    gender_loc = []
    gender_df = pd.DataFrame(columns=['locID', 'gender', 'count'])

    race_name = []
    race_num = []
    race_loc = []
    race_df = pd.DataFrame(columns=['locID', 'race', 'count'])

    lang_name = []
    lang_num = []
    lang_loc = []
    lang_df = pd.DataFrame(columns=['locID', 'language', 'count'])

    locat = mypage.select('.topname')[0].getText()
    locat=elem[0].getText()
    
	try:
        
        # The tables are always in the same order: gender, race, language
        gender_table = mypage.find_all("table")[0]
        for td in gender_table.find_all("td"):
            if '%' not in td.text:
                try:
                    value = float(td.text)
                    gender_num.append(value)
                    gender_loc.append(locat)
                except:
                    gender_name.append(td.text)

        # Get the second table about race
        race_table = mypage.find_all("table")[1]
        for td in race_table.find_all("td"):
            if '%' not in td.text:
                try:
                    value = float(td.text)
                    race_num.append(value)
                    race_loc.append(locat)
                except:
                    race_name.append(td.text)

        language_table = mypage.find_all("table")[2]
        count = 0
        for td in language_table.find_all("td"):
            if (('%' not in td.text) and (td.text != "")):
                if count % 2 == 0:
                    lang_loc.append(locat)
                    lang_name.append(td.text)
                    count += 1
                else:
                    value = float(td.text)
                    lang_num.append(value)
                    count += 1

        gender_df['locID'] = gender_loc
        gender_df['gender'] = gender_name
        gender_df['count'] = gender_num

        race_df['locID'] = race_loc
        race_df['race'] = race_name
        race_df['count'] = race_num

        lang_df['locID'] = lang_loc
        lang_df['language'] = lang_name
        lang_df['count'] = lang_num
    
    except:
        pass
    
    return gender_df, race_df, lang_df
    
```

Now that we have our method of collecting the data from all of the sites, we can take it and fill our dataframes with the individual dataframes. 


```all_gender = []
all_race = []
all_lang = []

for subsite in subsitelist:
    gender, race, language = myscraper(subsite)
    all_gender.append(gender)
    all_race.append(race)
    all_lang.append(language)
    
gender_df = pd.concat(all_gender)
race_df = pd.concat(all_race)
language_df = pd.concat(all_lang)

```
This will take a very long time to run because it is going through all of the subsites.

**Step 6. Reshape as needed**

Now we will inspect what our gender dataframe looks like.

In `NEXT CELL`, type

```
gender_df.head()
```

This should show the first 5 rows of the table. If it only shows the headers, go back and check your code. While it looks like the data is there, we want to pivot it so that the locations are the uniqueID, and the categories (gender, race, language) are the headers. We want the values to fill in for each location/category pair. 

We will pivot the table, and assign the locID as the index, the columns as the category, and the values as the counts. Finally, if any cells are empty (i.e., Afrikaans does not appear in a subsite), we want to fill in a 0. 

In `NEXT CELL`, type

```gender = gender_df.pivot_table(index='locID', columns='gender', values='count').fillna(0)
race = race_df.pivot_table(index='locID', columns='race', values='count').fillna(0)
language = lang_df.pivot_table(index='locID', columns='language', values='count').fillna(0)
```


**Step 11. Save the dataframe as a csv**

Finally, we save the whole dataframe to a csv on our computer. This will be saved in the same directory (folder) that your file is currently in. I've made a subfoldered entitled, 'csvs' to case these files in.

In `NEXT CELL`, type

```gender.to_csv('csvs/jhb_gender.csv')
race.to_csv('csvs/jhb_race.csv')
language.to_csv('csvs/jhb_lang.csv')
```

Now if we run this, it will download all of the data to our csv file. (You may have to **run this cell a second time** because of an issue with saving to csv and Jupyter notebooks, sometimes the output of the first run is empty even though the .to_csv function should create and fill the file in one step.)

**Step 12. Check the csv**

Open with Excel or another program that can read csv's. 

![blank](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/ws15.png)

### Download Shape Files

Our next task is to download the kml's, this is actually much more straightforward now that we have our subsiteslist. 

If we open a few of the links to the shapefiles and inspect the url, we see that the url is just the same as the data site, but with '/kml' appended to the end. We will utilize this fact to visit every site using the same logic as we did before. 

Secondly, the only thing at this address is the kml itself. The urllib.request library has a function, urlretrieve to deal with cases such as this, where we will visit the site, and retrieve whatever file exists at that location. So first thing, we will import the urlretrieve library from urllib.request. 

**Note** that the way our extensions are formatted, there is a /place/ in the extension. This is read as a folder location. The easiest way to deal with this is to make a 'place' folder wherever you want these files to be saved, and all the files will be saved there.  Once we download the kml file, we will need to add the .kml extension to it. We can do all of this in the urlretrieve command. 

However, running this will take some time, so try it with a slice first. Here we'll use the first 5 links.

In `NEXT CELL`, type

``` from urllib.request import urlretrieve
for subsite in subsitelist[:5]:
	mylink = url+subsite+"/kml"
	urlretrieve(mylink, "kmls/" + subsite + ".kml")
```
 

Remove the slice and run this cell to retrieve all of the kml files. 

That is it. From here, you can take this data to a GIS program such as QGIS or ArcGIS, combine the shapefiles to reconstruct Johannesburg and join the data based on the locID column. 

********************************************************


Tutorial written by [Michelle McSweeney](www.michelleamcsweeney.com) for the the [Center for Spatial Research](http://c4sr.columbia.edu) at Columbia University.