---
title: "Measuring Risk in NYC"
author: "Andy Areitio, Jorge Bravo, Claudia Kampbel and Pedro B Franco "
output:
  html_document:
    css: ../../AnalyticsStyles/default.css
    theme: paper
    toc: yes
    toc_float:
      collapsed: no
      smooth_scroll: yes
  pdf_document:
    includes:
      in_header: ../../AnalyticsStyles/default.sty
---

# Project Objective:

We intend to analyze and identify patterns regarding car accidents in New York City ("NYC"). In order to acheive this objective we have obtained from the New York Police Department ("NYPD") open source data with almost two years of accidents. The table has 29 columns containing details on each accident reported to the NYPD in these two years.

Source of data:
(https://data.cityofnewyork.us/Public-Safety/NYPD-Motor-Vehicle-Collisions/h9gi-nx95)
 
Discalimer: Please note that in order to run this project on a local computer, you will have do download the files in the GITHUB repository and extract the CSV file from the NYPD_Motor_Vehicle_Collisions_2.csv.zip

Please name the file: data/NYPD_Motor_Vehicle_Collisions_2.CSV

Thank you!




# What is this for?

In Data Analytics we often have very large data (many observations - "rows in a flat file"), which are however similar to each other hence we may want to organize them in a few clusters with similar observations within each cluster. For example, in the case of customer data, even though we may have data from millions of customers, these customers may only belong to a few segments: customers are similar within each segment but different across segments. We may often want to analyze each segment separately, as they may behave differently (e.g. different market segments may have different product preferences and behavioral patterns).

In such situations, to identify segments in the data one can use statistical techniques broadly called **Clustering** techniques. Based on how we define "similarities" and "differences" between data observations (e.g. customers or assets), which can also be defined mathematically using **distance metrics**, one can find different segmentation solutions. A key ingredient of clustering and segmentation is exactly the definition of these distance metrics (between observations), which need to be defined creatively based on contextual knowledge and not only using "black box" mathematical equations and techniques. 

> Clustering techniques are used to group data/observations in a few segments so that data within any segment are similar while data across segments are different. Defining what we mean when we say "similar" or "different" observations is a key part of cluster analysis which often requires a lot of contextual knowledge and creativity beyond what statistical tools can provide.

Cluster analysis is used in a variety of applications. For example it can be used to identify consumer segments, or competitive sets of products, or groups of assets whose prices co-move, or for geo-demographic segmentation, etc. In general it is often necessary to split our data into segments and perform any subsequent analysis within each segment in order to develop (potentially more refined) segment-specific insights. This may be the case even if there are no intuitively "natural" segments in our data. 

# Clustering and Segmentation using an Example

In this note we discuss a process for clustering and segmentation using a simple dataset that describes attitudes of people to shopping in a shopping mall. As this is a small dataset, one could also "manually" explore the data to find "visually" customer segments - which may be feasible for this small dataset, although clustering is in general a very difficult problem even when the data is very small.  

Before reading further, do try to think what segments one could define using this example data. As always, you will see that even in this relatively simple case it is not as obvious what the segments should be, and you will most likely disagree with your colleagues about them: the goal afternall is to let the numbers and statistics help us be more *objective and statistically correct*.

## The "Business Decision"

The management team of a large shopping mall would like to understand the types of people who are, or could be, visiting their mall. They have good reasons to believe that there are a few different market segments, and they are considering designing and positioning the shopping mall services better in order to attract mainly a few profitable market segments, or to differentiate their services  (e.g. invitations to events, discounts, etc) across market segments. 

## The Data

To make these decisions, the management team run a market research survey of a few potential customers. In this case this was a small survey to only a few people, where each person answered six attitudinal questions and a question regarding how often they visit the mall, all on a scale 1-7, as well as one question regarding their household income:

Name        | Description                                   | Scale
-----------:|:----------------------------------------------|:-----
V1          | Shopping is fun                               | 1-7
V2          | Shopping is bad for your budget               | 1-7
V3          | I combine shopping with eating out            | 1-7
V4          | I try to get the best buys while shopping     | 1-7
V5          | I don't care about shopping                   | 1-7
V6          | You can save lot of money by comparing prices | 1-7
Income      | The household income of the respondent        | Dollars
Mall.Visits | How often they visit the mall                 | 1-7


```r
# let's make the data into data.matrix classes so that we can easier visualize them
#----ProjectData = data.matrix(ProjectData)
```

#---Forty people responded to these 6 questions. Here are the responses for the first  people:


```r
#----knitr::kable(round(head(ProjectData, max_data_report), 2))
```

We will see some descriptive statistics of the data later, when we get into the statistical analysis.

How can the company segment these 963539 people? Are there really segments in this market? Let's see **a** process for clustering and segmentation, the goal of this report. 

## A Process for Clustering and Segmentation

As always: 

> It is important to remember that Data Analytics Projects require a delicate balance between experimentation, intuition, but also following (once a while) a process to avoid getting fooled by randomness and "finding results and patterns" that are mainly driven by our own biases and not by the facts/data themselves.

There is *not one* process for clustering and segmentation. However, we have to start somewhere, so we will use the following process:

# Clustering and Segmentation in 9 steps

1. Confirm data is metric
2. Scale the  data
3. Select Segmentation Variables
4. Define similarity measure
5. Visualize Pair-wise Distances 
6. Method and Number of Segments
7. Profile and interpret the segments 
8. Robustness Analysis

Let's follow these steps.

## Step 1: Confirm data is metric

While one can cluster data even if they are not metric, many of the statistical methods available for clustering require that the data are so: this means not only that all data are numbers, but also that the numbers have an actual numerical meaning, that is, 1 is less than 2, which is less than 3 etc. The main reason for this is that one needs to define distances between observations (see step 4 below), and often ("black box" mathematical) distances (e.g. the "Euclideal distance") are defined only with metric data. 

However, one could potentially define distances also for non-metric data. For example, if our data are names of people, one could simply define the distance between two people to be 0 when these people have the same name and 1 otherwise - one can easily think of generalizations. This is why, although most of the statistical methods available (which we will also use below) require that the data is metric, this is not necessary as long as we are willing to "intervene in the clustering methods manually, e.g. to define the distance metrics between our observations manually". We will show a simple example of such a manual intervention below. It is possible (e.g. in this report). 

> In general, a "best practice" for segmentation is to creatively define distance metrics between our observations. 

In our case the data are metric, so we continue to the next step. Before doing so, we see the descriptive statistics of our data to get, as always, a better understanding of the data. 
Our data have the following descriptive statistics: 


```r
#----knitr::kable(round(my_summary(ProjectData),2))
```

> Note that one should spend a lot of time getting a feeling of the data based on simple summary statistics and visualizations: good data analytics require that we understand our data very well.

## Step 2: Scale the  data

This is an optional step. Note that for this data, while 6 of the "survey" data are on a similar scale, namely 1-7, there is one variable that is about 2 orders of magnitude larger: the Income variable. 

Having some variables with a very different range/scale can often create problems: **most of the "results" may be driven by a few large values**, more so that we would like. To avoid such issues, one has to consider whether or not to **standardize the data** by making some of the initial raw attributes have, for example,  mean  0 and standard deviation 1 (e.g. `scaledIncome` `=` `(Income-mean(Income))` `/` `sd(Income)`), or scaling them between 0 and 1 (e.g. `scaledIncome` `=` `(Income-min(Income))` `/` `(max(Income)-min(Income))`). Here is for example the R code for the first approach, if we want to standardize all attributes:


```r
#----ProjectData_scaled=apply(ProjectData,2, function(r) 
#----{if (sd(r)!=0) res=(r-mean(r))/sd(r) else res=0*r; res})
```

Notice now the summary statistics of the scaled dataset:


```r
#----"knitr::kable(round(my_summary(ProjectData_scaled),2))""
```

As expected all variables have mean 0 and standard deviation 1. 

While this is typically a necessary step, one has to always do it with care: some times you may want your analytics findings to be driven mainly by a few attributes that take large values; other times having attributes with different scales may imply something about those attributes. In many such cases one may choose to skip step 2 for some of the raw attributes.  

## Step 3: Select Segmentation Variables

The decision about which variables to use for clustering is a **critically important decision** that will have a big impact on the clustering solution. So we need to think carefully about the variables we will choose for clustering. Good exploratory research that gives us a good sense of what variables may distinguish people or products or assets or regions is critical. Clearly this is a step where a lot of contextual knowledge, creativity, and experimentation/iterations are needed. 

Moreover, we often use only a few of the data attributes for segmentation (the **segmentation attributes**) and use some of the remaining ones (the **profiling attributes**) only to profile the clusters, as discussed in Step 8. For example, in market research and market segmentation, one may use attitudinal data for segmentation (to segment the customers based on their needs and attitudes towards the products/services) and then demographic and behavioral data for profiling the segments found. 

In our case, we can use the 6 attitudinal questions for segmentation, and the remaining 2 (Income and Mall.Visits) for profiling later. 

## Step 4: Define similarity measure

Remember that the goal of clustering and segmentation is to group observations based on how similar they are. It is therefore **crucial** that we have a good undestanding of what makes two observations (e.g. customers, products, companies, assets, investments, etc) "similar". 

> If the user does not have a good understanding of what makes two observations (e.g. customers, products, companies, assets, investments, etc) "similar", no statistical method will be able to discover the answer to this question. 

Most statistical methods for clustering and segmentation use common mathematical measures of distance. Typical measures are, for example, the **Euclidean distance** or the **Manhattan distance** (see `help(dist)` in R for more examples). 

> There are literally thousands of rigorous mathematical definitions of distance between observations/vectors! Moreover, as noted above, the user may manually define such distance metrics, as we show for example below - note however, that in doing so one has to make sure that the defined distances are indeed "valid" ones (in a mathematical sense, a topic beyond the scope of this note).

In our case we explore two distance metrics: the commonly used **Euclidean distance** as well as a simple one we define manually. 

The Euclidean distance between two observations (in our case, customers) is simply the square root of the average of the square difference between the attributes of the two observations (in our case, customers). For example, the distance of the first customer in our data from customers 2- "r max_data_report" (summarized above), using their responses to the 6 attitudinal questions is:


```r
#----euclidean_pairwise <- as.matrix(dist(head(ProjectData_segment, max_data_report), method="euclidean"))
#----euclidean_pairwise <- euclidean_pairwise*lower.tri(euclidean_pairwise) + euclidean_pairwise*diag(euclidean_pairwise) + 10e10*upper.tri(euclidean_pairwise)
#----euclidean_pairwise[euclidean_pairwise==10e10] <- NA

#----knitr::kable(round(euclidean_pairwise))
```

Notice for example that if we use, say, the Manhattan distance metric, these distances change as follows:


```r
#----manhattan_pairwise <- as.matrix(dist(head(ProjectData_segment, max_data_report), method="manhattan"))
#----manhattan_pairwise <- manhattan_pairwise*lower.tri(manhattan_pairwise) + manhattan_pairwise*diag(manhattan_pairwise) + 10e10*upper.tri(manhattan_pairwise)
#----manhattan_pairwise[manhattan_pairwise==10e10] <- NA

#----"knitr::kable(manhattan_pairwise)"
```

Let's now define our own distance metric, as an example. Let's say that the management team of the company believes that two customers are similar if they do not differ in their ratings of the attitudinal questions by more than 2 points. We can manually assign a distance of 1 for every question for which two customers gave an answer that differs by more than 2 points, and 0 otherwise. It is easy to write this distance function in R:


```r
#----My_Distance_function<-function(x,y){sum(abs(x-y)>2)}
```

Here is how the pairwise distances between the respondents now look like.


```r
#----Manual_Pairwise=apply(head(ProjectData_segment,max_data_report),1,function(i) apply(head(ProjectData_segment,max_data_report),1,function(j) My_Distance_function(i,j) ))
#----Manual_Pairwise <- Manual_Pairwise * lower.tri(Manual_Pairwise) + Manual_Pairwise * diag(Manual_Pairwise) + 10e10*upper.tri(Manual_Pairwise)
#----Manual_Pairwise[Manual_Pairwise == 10e10] <- NA

#----knitr::kable(Manual_Pairwise, col.names= 1:ncol(Manual_Pairwise))
```

In general a lot of creative thinking and exploration should be spent in this step, and as always one may need to come back to this step even after finishing the complete segmentation process - multiple times. 

## Step 5: Visualize Pair-wise Distances 

Having defined what we mean "two observations are similar", the next step is to get a first understanding of the data through visualizing for example individual attributes as well the pairwise distances (using various distance metrics) between the observations. If there are indeed multiple segments in our data, some of these plots should show "mountains and valleys", with the mountains being potential segments. 

For example, in our case we can see the histogram of, say, the first 2 variables:


```r
#----do.call(grid.arrange, lapply(1:2, function(n) {
  #----qplot(ProjectData_segment[, n], xlab=paste("Histogram of Variable", n), ylab="Frequency", binwidth=1)
#----}))
```

or the histogram of all pairwise distances for the "r distance_used" distance:


```r
#----Pairwise_Distances <- dist(ProjectData_segment, method = distance_used) 
#----qplot(as.vector(Pairwise_Distances), xlab="Histogram of all pairwise Distances between observtions", ylab="Frequency", binwidth=1)
```

















