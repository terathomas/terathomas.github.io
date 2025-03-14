---
layout: default
---
# Welcome

Hello, I am Tera Thomas. My background in academic and clinical research has launched me into the fascinating world of data analysis. I have a deep rooted passion for healthcare innovation and patient care quality. 

I welcome you to my digital portfolio, where you can explore some of my work with health care data analytics. 
The work you will see below was conducted through SQL (PgAdmin), Excel, and Tableau. 

# Sample Patient Population Analyzing: Patients, Encounters, Conditions and Immunizations

Below you will explore a sample population provided by "Data Wizardry", retrieved from Data.CMS.gov.
The patient population was filtered using SQL, and analyzed through Excel. The visuals you will 
see were created in Tableau.

This analysis covered:

**21,670** individual patient encounters

**10** different payer groups

**7** different encounter classes

**4** different joined databases (patients, encounters, conditions, immunizations)

![SamplePop](https://terathomas.github.io/images/SamplePop.jpg)


### The Sample Population Excel File

_please note, all patient information is synthetic_

![SamplePopExcel](https://terathomas.github.io/images/SamplePopExcel.jpg)

This is a 21,670 row & 24 column data set analyzing
various population metrics.

-----------------------
-----------------------

# Medicare Clinical Laboratory Fee Schedule Private Payer Rates and Volumes 

Below you will explore a sample analysis using data provided by Data.CMS.gov.

My goal when analyzing this data was to find insightful trends and patterns
that would be relevant for payer groups or billing offices within a
hospital system. 

The images provided were taken from a presentation I created to represent the 
data findings (see the full presentation [here](https://terathomas.github.io/downloads/MedicareSample.pdf)). A similar presentation could be used for hospital administrators or health care providers within the clinic environment.

Here are excerpts from the presentation that I found to be the most insightful:

![TopOrdered](https://terathomas.github.io/images/TopOrdered.jpg)

This chart represents the top 15 most frequently ordered laboratory codes used within this sample. 

![TopRevenue](https://terathomas.github.io/images/TopRevenue.jpg)

This chart represents the top 15 highest earning codes within this sample.
Note, some of the most frequently ordered codes are *not* the highest earning (as shown by the green and orange highlighting).

![Var](https://terathomas.github.io/images/Var.jpg)
 
 This plot demonstrates wide variability demonstrated for the most
 frequently ordered code, G0483. 

 This result demonstrates an urgent need to examine the reason behind the
 extreme outliers within the sample. 

![VarExcludeOutlier](https://terathomas.github.io/images/VarExcludeOutlier.jpg)

This is the same plot as previous, but with the outlier removed. 

Though the new price variability still seems extreme, there were _several_ orders near this new maximal value of $12,098.

You can see how it reduced the average price and standard deviation for this order. 


[Medicare Lab Payer Project SQL Code](./sqlcode.html)

[Link to citations](./citations.html)
