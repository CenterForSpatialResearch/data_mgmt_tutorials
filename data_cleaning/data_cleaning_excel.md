# Data Cleaning 

Data cleaning itself is a highly specific task. Each and every instance will be different because the data will be formatted in different ways and the project will be unique. Much of data cleaning is custom problem solving, transforming the data into the right format. This is a very basic introduction to data cleaning, and assumes only familiarity with Excel. However, data cleaning is a broad topic that is not easily pinned down, and you are likely to need to do a variety of different steps in order to employ it on your own dataset. This tutorial will walk through cleaning one dataset, highlighting some of the strategies and best practices.

We will use Excel for this task because the data is relatively small, straightforward, and clean. Most datasets made available by governments and large institutions are "clean", meaning that there are very few errors, duplicate entries, misspellings, alternative descriptions, etc. 

By the end of this tutorial you will be able to:

* Download an original dataset from NYC Open Data
* Modify that dataset to suit your needs
* Pivot a data table
* Strategically use exploratory visualizations to better understand a dataset

For this tutorial, we are going to take a publicly available dataset and map some of the datapoints. Most datasets will need some cleaning before they are ready to be visualized; there are usually some errors, missing information, duplicate information, organization issues, etc. Therefore, we will clean this dataset before we begin using it. 

## Part I : Making "wide" data "long

When we talk about data, we often mention that we are looking for data which is "long and skinny" by that we mean that repetition is ok so long as there is no data in the headers. For example, the following table has data in the headers, and without a label for the table, we don't know what the number represent. Generally, data that you want to use for any sort of mapping, visualization, or analysis needs to have descriptive headers and data in each column. It can have repetition so long as no row is completely the same. 

|  Week  |	Mon | Tues  | Wed   | Thurs  | Fri   | Sat  | Sun   |
|:------:|:----:|:-----:|:-----:|:------:|:-----:|:----:|:-----:|
| Week 1 |  0   | 5     | 5     | 0      | 8     |  0   |  15  |
| Week 2 |  0   | 6     | 4     | 0      | 8     |  0   |  16  |
| Week 3 |  0   | 6     | 4     | 0      | 8     |  0   |  18  |
 
If we want to make a visualization, this data would be better formatted as below:

| Week  |	Day  | Mileage  |
|:-----:|:------:|:--------:|
|  1    |  Mon   |   0      |
|  1    |  Tues  |   5      |
|  1    |  Wed   |   5      |
|  1    |  Thurs |   0      |
|  1    |  Fri   |   8      |
|  1    |  Sat   |   0      |
|  1    |  Sun   |   15     |
|  2    |  Mon   |   0      |
|  2    |  Tues  |   6      |
|  2    |  Wed   |   4      |
|  2    |  Thurs |   0      |
|  2    |  Fri   |   8      |
|  2    |  Sat   |   0      |
|  2    |  Sun   |   16     |
|  3    |  Mon   |   0      |
|  3    |  Tues  |   6      |
|  3    |  Wed   |   4      |
|  3    |  Thurs |   0      |
|  3    |  Fri   |   8      |
|  3    |  Sat   |   0      |
|  3    |  Sun   |   18     |

This is, of course, very hard on human eyes, we tend to be able to read the first version better. However, if you want to make a visualization, add to this data, or otherwise interact with it programmatically, the second option is much better. Maybe we wanted to add a link to the route for each day, or the number of miles we actually ran (versus what was prescribed), or the type of shoes we should wear, or if it is a tempo run, easy, etc. This would be impossible with the first table, but can all be added to the second table by adding new columns, and infinite number of rows detailing what should happen on each run. 

Unfortunately, performing this operation in Excel is exceedingly difficult. The best solution for Pivoting this data is to use Tableau. Tableau has a free online version and a free-for-educators desktop version. If you have a lot of data like this to transform, and the only thing you ever do in Tableau is transform this data, it will be worth signing up for it. 

But, for us, we are only working with this one dataset, and it is small, so we will do it manually. 

First, because we are going to manually move our numbers, we would rather copy and paste 7 rows 3 times than 3 rows 7 times. So let's start by transposing our data. First, select all of the cells with content in them and Copy them. 

![select cells](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc9.png)

Make another tab (at the bottom of the page), and select 'Paste Special'. A dialogue box should appear to specify which variety of Special you want to Paste. Select the 'Transpose' button. 

![transpose](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc10.png)

Now your data should look like this:

![transposed](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc11.png)


We need to repeat the days column three times. Since it's small, we will copy and paste them 3 times. However, if you needed to repeat each value a different number of times (or a large number of times), you would use the VLOOKUP strategy, [explained here](https://stackoverflow.com/questions/11841213/copy-value-n-times-in-excel).

Copy the list of days and paste it 2 times below the original list. 

![repeated cells](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc12.png)

Now we need to insert a column to the left of our weekdays for the weeks that each refers to. Right click on the column with the Days and select 'Insert'. Title this column 'Week' in the first data cell (A2), write this formula: `=$C$1`, and drag it into the next 7 cells (or until Sunday). The dollar signs tell Excel to hold the value constant and do not increment it. If we just did '=C1', then as we dragged it down, it would have filled from C2, C3, C4, etc. 

![formula](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc13.png)

Repeat this step with Weeks 2 & 3. 

Now highlight column A where you just pasted the weeks into. Copy and Paste Special, and select 'Values'. If we don't do this, when we delete the data in columns D and E, the values in the 'Week' column won't have anything to reference, and we will get an error. 

Copy and paste the numbers from each column into their respective locations, and delete the data in columns D and E. 

Finally, be sure to save your Workbook as an Excel file, and save the sheet you just created as a csv file. 

Now your data is ready to be used for visualization or have data added to it. In the next section, we will work with real data that we might use in a mapping program. 


## Part II: Cleaning and Original Dataset.

*Our questions:*
Community Centers in New York City offer a variety of services from classes to family support to immigration help and much more. This is an exploratory mapping exercise where we want to know how these services are situated in the city.


1. Go to [NY OpenData](https://nycopendata.socrata.com/). As you can see, there are a lot of datasets available categorized under a variety of topics. 

2. Use the search bar to find datasets that reference immigration. You will likely come to a list of Datasets.
	
3. Today we are going to work with: DYCD_after-school_programs__Neighborhood_Development_Area__NDA__Family_Support

![image](https://github.com/michellejm/ConflictUrbanism_LanguageJustice/blob/master/Images/opendata.tiff)

4. Spend a few minutes getting acquainted with the data.
	1. What does each column refer to?
	2. Are there any columns that are going to be irrelevant to our project?
	3. Is there georeferenced information in this dataset? What does it look like?
	4. What problems will we need to resolve?
	
5. Export the data and download it as a csv for Excel file. 
![image](https://github.com/michellejm/ConflictUrbanism_LanguageJustice/blob/master/Images/bluebutton.tiff) "CSV" stands for "comma separated values".
	
**Plan for cleaning**

1. Remove duplicates
2. Handle missing values
3. Conversion issues (i.e., numbers rendered as dates, etc.) 
4. Separate cells with multiple pieces of data, concatenate cells if needed
5. Trim leading and trailing white space

*Data from different sources will need different approaches, this is appropriate for this particular dataset.*

*Steps*
1. Prepare the workspace. When cleaning datasets, it is far too easy to make mistakes, forget where you are, and otherwise get confused. This step helps prevent that.
	1. Make four tabs (sheets) in your Excel Workbook - 'Original', 'Working', 'Final', 'Incomplete'
![four tabs](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc3.png)
	2. Don't change the Original tab
		1. Make a copy of the data and paste it into 'Working'
		2. Select all of the cells by clicking on the cell in the bottom right and pressing 'Command' + a
	3. Remove Duplicates (this step comes early because in the next step, we give every row a unique ID)
		1. Select the entire sheet by clicking on the box in the upper left corner.
		![select all](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc1.png)
		2. Select Data >> Remove Duplicates
		![remove duplicates](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc2.png)
		3. You will be prompted with a dialoge box, 'Accept' the duplicates if you think it is correct.
	4. Add an 'ID' column
		1. Insert a column to the left of column A and title it 'ID'
		2. Scroll down to see how many values we have. Looks like 464. Click on the first data cell (cell A2) and then  select Edit > Fill > Series. Make the following selections:
			* Series in Columns
			* Type Linear
			* Step value 1
			* Stop value 464
					![series options](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc4.png)
	5. At this point, I like to rename the columns into single words (i.e., 'Age Group' > 'AgeGroup'), make the column names bold and freeze the first row and first column.
		![freeze columns](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc5.png)
2. Missing Data
	1. Each Row that is missing an address needs to be removed from the data set since we cannot map it. We want to cut and paste these rows into the 'Incomplete' Tab. We won't be able to map these data without finding more information, though we don't want to lose them completely.
		1. Copy the header row from the Working Tab into the Incomplete Tab.
		2. Return to the Working Tab and Sort the sheet by addresses so all the missing ones are together. Sort by 'Location1'. The empty cells may appear at the bottom rather than the top.
		![sort sheet](https://github.com/CenterForSpatialResearch/data_mgmt_tutorials/blob/master/Images/dc6.png)
		3. Cut and paste the rows without addresses into the Incomplete Tab.
	2. Each Row that has a missing Age Group should stay in the dataset but **also** be added to the Incomplete tab. 
		1. Sort the sheet by Age Group and copy and paste these rows into the Incomplete Tab.
		2. Return to the Working tab and fill in the missing values with a null value (i.e., 'Unk' or 'NaN'). A simple way to do this is to type 'Unk' in the first cell and hover your mouse on the lower left corner to drag the same value the rest of the way down.
	3. Find and Replace (Under Edit >> Find and Replace) to fill any remaining empty cells. Leave the find box empty and 'Find entire cells only' and Replace with: Unk
	4. We can return to these datapoints later to recover the information.
	
3. Fix errors created by opening with Excel
	1. There are two dates (May 20 and Nov 14) that have been automatically created by Excel
	2. Change the datatype in this column to 'Text'
		1. Format > Cells > Data Type > Text
	3. Go back to the original data to find the correct values.
		1. (May 20 is Age 5-20; Nov 14 is Age 11-14)
	4. Enter the correct ranges.

4. Separate Addresses from the latitude/longitude. 
*Location contains both address information and latitude/longitude. Some sites have lat/long information separated out, but some don't. For those that don't have the lat/long easily available, we want to extract it from the address field.*
	1. First take a look at the cells with the location information in them. There are multiple lines per cell. We need the information in one line. We are going to use an Excel formula to collapse those lines, and will be a little clever about it because there is no direct way to do this. 
	2. Insert FOUR columns to the right of 'Location'.
	3. Type this formula into the first cell of the first new column (probably Column J: =SUBSTITUTE(I2,CHAR(13),"\")  *This means take cell I2, find Character #13 (the carriage return) and substitute the backslash. This was a bit of trial and error as it could have also been Character #10 (new line).* (obviously, if your Location1 column is in a different place for any reason, change I to the relevant column).
		2. Drag this formula to the bottom of the data to fill in all of the cells.
		3. Call this new column Location2
	3. Right now, all of these cells are filled with formulas, not actual values. You can change that by a special kind of Copy/Paste. Copy this column, and select Paste Special > Values into the same location. This turns those formulas, which are all referential into actual values that can be read in place. 
	4. Now we want to break apart those lines so each part of the address is in its own column.
	5. Go to Data > Text to column
		1. Select 'Delimited' 
		2. Select \
		3. Finish
5. Separate Latitude and Longitude
	1. Select the column with latitude and longitude in it
	2. Insert a column to the right
	3. Select text-to-columns
	4. Select 'Delimited', 'comma-separated', Finish
	5. Remove the parentheses
		1. Select the Latitude column
			1. Find and Replace the character, ( , with nothing
		2. Select the Longitude column
			1. Find and replace the character, ) , with nothing
	6. Rename the columns appropriately
		1. Location2, Street, City_State_Zip, Latitude, Longitude
5. Select all of the data in this sheet and copy it onto the Final Tab.
6. Save
	1. From the Final Tab, select Save As > csv, and rename it something easier to type. 
	2. Save the whole workbook as an Excel Spreadsheet. 


__________________________________________________________________________________________

Tutorial written by [Michelle McSweeney](www.michelleamcsweeney.com) for the the [Center for Spatial Research](http://c4sr.columbia.edu) at Columbia University.




	

