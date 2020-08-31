



#  Data Brokers: To know them is to hate them, but how consistent are they?

See abstract for motivation, background, and details.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine. 

### Prerequisites
- Python 3
- Python Libraries
    - json, logging, os, time, re, html, matplotlib.pyplot, numpy, pandas, itertools, statistics
- Selenium and Firefox Webdriver
- Jupyter Notebooks

## Running the Code


### Summary

At the end of the notebook run `do_it_all(firstname, lastname)` You must **look at the browser window** that pops up or the program will not run. There are optional parameters for this function to determine the weights of the different parameters. I decided that choosing those was outside of the scope of this project since the choice of those variables really depends on a lot of factors so each component counting equally is the default. A lot more details can be found below or in the abstract, but this is all you should need to get things running and some of these details may be similar to information in the abstract. 


### Details

There are additional cells in the notebook that show test cases for various functions. To prevent the accidental running of cells when starting up the notebook the variable test has been set to false in the third cell of the notebook. To run these cells set the variable to True and run the cells as normal.

The code as a whole is split up into a few subsections:
1. data collection from data broker websites
2. analysis of the types of data included on the data broker website
3. programs for attempting to connect users between two data broker websites
4. all of the above combined


## 1. data collection from data broker websites
----
For each of the data broker websites, there are typically two functions, one to load the results for a user, and one to retrieve the data from the page and get it ready for analysis. The first function will not return a value, while the second function will return a formatted dictionary. Both of these programs need to take in a selenium web driver which can be created by running the code below
`driver = webdriver.Firefox()`


**Truthfinder**
`truth_finder(driver,"firstname","lastname")`
`t_dict = truth_cont(driver)`

**Spokeo**

`spokeo(driver,"firstname lastname")`
    `s_dict = spokeo_cont(driver)
`
notes: _You must be looking at the page as the selenium runs or the program does not work_ and records are collected from **all pages** for Spokeo, not just the first one. all other sites display all results on a single page

**infotracer**
`anywho(driver,"firstname lastname")`
    `a_dict = anywho_continue(driver)`

### Format of the Data
- Name: 
    - String, all lower case, periods removed
    - "Alexandra M. Nisenoff" -> "alexandra m nisenoff"
- Current Location:
    - string,  all lower case, commas removed (e.g. "Chicago, lL" -> "chicago il")
- All Locations
    - set of locations, formatted in the same way as Current Location, Current Location is included in the set
- Relatives:
    - set of strings formatted the same way as name category, additional version is stored with middle names removed
- Aliases:
    - set of strings with "name" included, formatted in the same way as the name, additional version is stored with middle names removed
- Age
    - int

## 2. analysis of the types of data included on the data broker website
----

The first function looks at the number of responses that are returned for any given search and also looks at how many are classified as exact name matches as defined by the exact_name function (examples of how matches are determined can be seen in the notebook)

This function takes in a list of dictionaries (of any length) that are formated consistently with dictionaries produced by the functions mentioned earlier, and a list (of the same length as the dictionary list) of labels for the dictionary, for labeling the bar graph that is produced by that function. An example of the usage can be seen below.

`comp_counts([s_dict,a_dict,t_dict],["Spokeo", "Infotracer","Truthfinder"],"firstname lastname")`

comp_counts takes inputs of a list of dictionaries and a list of labels (much like comp_counts). This function produces several graphs: a graph of the average number of pieces of data for each data type across sites (e.g. if there was only one result and that person had lived in 3 cities, it would return 3 for the Location Count column), that maximum number of pieces of data for each data type across sites, a histogram of the ages for the results, and a graph that shows the fraction of records that had a specific type of data across sites (e.g if there were 4 records on a site and 3 of them included an age, and the fouth did not, the bar would show .75) Below is shown how to run this function.

`comp_counts_all([s_dict,a_dict],["Spokeo", "infotracer"])`



## 3. programs for attempting to connect users between two data broker websites
----

This function finds the "similarity" between every combination of records from two sites. The bare minimum for running this function is two dictionaries and is fun as follows.

`df_all = table(s_dict,a_dict)`

The return type of the function is a data frame that holds the keys (from the input dictionaries) of the items being compared, the similarity scores for the individual record dictionary attributes, and the total similarity based on the individual record dictionary attributes and the coefficients that determine how much that attributes are weighted. 

### similarity metrics (coefficent name)

- Name (n_c): perfect matches get full credit, partial credit is given for other circumstances
- Current Location (c_c): identical matches get full credit, the correct state gets full credit minus a penalty for edit distance of the city, out of state matches get no credit
- All Locations (prev_liv_c): Jaccard similarity
- Relatives (rel_c): Jaccard similarity (max of original and middle name removed sets w/ penalty)
- Aliases (ak_c): Jaccard similarity (max of original and middle name removed sets w/ penalty)
- Age (ag_c): absolute difference

These coefficients can also be passed into the table function as optional parameters. The reason for this is that there is probably no right choice for these values. If you are trying to match searches where there are very few results you may want very different numbers than if you were trying to match a large number of results. This may also depend on what sites you are comparing as not all sites may provide the same categories of data. The choices of coefficients influence how the records are matched up and the biases introduced must be considered in any analysis of the data. Below is shown how to run the program with different coefficients.

`df_all = table(s_dict,a_dict,n_c=1,c_c=1,prev_liv_c=1,rel_c=2,ak_c=1,ag_c=.5)`

get_max_table just creates a data frame that only includes the rows that correspond to the best-predicted pairings between the results of the two data broker searches based on the similarity score from the table function. Cutoff sets a lower limit on the similarity score for a pair to be considered a match.

`max_table = get_max_table(df_all,cutoff = 0.5)`

### 4. all of the above combined
----
The do_it_all function just runs the 3 earlier sets of programs. Minimally it can be run by just inputting a first and last name (as shown below) or with any of the optional parameters from table and get_max_table. It doesn't matter if the names are in lowercase or not. There is also the option of specifying the index of the dictionaries to be compared 0 = Spokeo, 1 = infotracer, 2 = truthfinder. 

`do_it_all("Alexandra", "Nisenoff")`

## Authors

* **Alexandra Nisenoff**  - [anisenoff](https://github.com/anisenoff) for [CMSC 25900](https://www.classes.cs.uchicago.edu/archive/2020/spring/25900-1/#) Ethics, Fairness, Responsibility, and Privacy in Data Science (Spring 2020) final project
