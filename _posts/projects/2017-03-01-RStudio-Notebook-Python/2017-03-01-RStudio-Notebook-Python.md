---
layout: page-fullwidth
title: "Running Python in an RStudio Notebook"
#subheadline: ""
meta_teaser: "Running Python in an RStudio Notebook"
teaser: "A quick project exploring using python within an Rstudio notebook."
breadcrumb: true
comments: true
meta: true
header:
    title: ""
    image_fullwidth: RStudio.png
    background-color: "#262930"
    caption: Running Python in an RStudio Notebook
image:
    thumb:  RStudio.png
    homepage: RStudio.png
    caption: Running Python in an RStudio Notebook
categories:
    - projects
tags:
    - r
    - python
    - classification
---

This is an [R Markdown](http://rmarkdown.rstudio.com) Notebook written in RStudio - except it's running python.

I was interested in running a python version of RStudio notebook as it's a neat and easy way to combine text and code to create a flowing document.
It's also easily transferable to [markdown](https://en.wikipedia.org/wiki/Markdown) which in turn makes it easy to add static pages with [jekyll](https://jekyllrb.com/) to a site like [github pages](https://pages.github.com/).

To get it running was fairly easy, Rstudio has done all the heavy lifting, I just needed to use the right version of python. I'm using anaconda so I prepend the location of my anaconda python.exe that to my path and it's done.

{% highlight r %}
python.path = "C:\\Users\\markp\\Anaconda2"
Sys.setenv(PATH = paste(python.path, Sys.getenv("PATH"), sep=";"))
{% endhighlight %}
To test it out let's run a simple project, create html and markdown documents and publish it.

## Project

I will use the [Smartphone-Based Recognition of Human Activities and
 Postural Transitions Data Set ](http://archive.ics.uci.edu/ml/datasets/Smartphone-Based+Recognition+of+Human+Activities+and+Postural+Transitions#) from the [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/).
 
This dataset containing movement measurements for 30 volunteers performing 6 different tasks. The idea is to train a model to associate different movements to each task, and then classify the remaining data according to which tasks our classification model assigns them.

## Problems

The original idea was to use chunks of code and write some explanatory text between them, but because of the way which rstudio runs python (it does a separate system call for each chunk) it's not possible to save variables between chunks.

This means that the RStudio/python combination doesn't do what I want, but I'll run the entire script as one piece and get an output below for completeness.

## Quick Code Summary

- Import libraries
- Read and combine the data into a [tidy](vita.had.co.nz/papers/tidy-data.pdf) dataset
- Train a [random forest](https://en.wikipedia.org/wiki/Random_forest) on the training data
- Classify the test data
- Output a [confusion matrix](https://en.wikipedia.org/wiki/Confusion_matrix) of the results


{% highlight python %}
import pandas as pd
import numpy as np
from sklearn.cross_validation import train_test_split
from sklearn.metrics import confusion_matrix
from sklearn.metrics import roc_curve
from sklearn.metrics import auc
from ggplot import *

base_folder = 'C:\Users\markp\OneDrive\Documents\Data Science\Projects\smartphone_movement_python\UCI HAR Dataset\\'

a_labels = pd.read_csv(
                        base_folder + 'activity_labels.txt',
                        header=None, 
                        delim_whitespace=True,
                        names=('ID','Activity'))

features = pd.read_csv(
                        base_folder + 'features.txt',
                        header=None, 
                        delimiter=r"\s+",
                        names=('ID','Sensor')
                        )

# train
trainSub = pd.read_csv(
                        base_folder + 'train\subject_train.txt',
                        header=None, 
                        names=['SubjectID'])

trainX = pd.read_csv(
                     base_folder + 'train/X_train.txt', 
                     sep='\s+', 
                     header=None)

trainY = pd.read_csv(
                    base_folder + 'train/y_train.txt',
                    sep=' ',
                    header=None)
trainY.columns = ['ActivityID']

# test
testSub = pd.read_csv(
                        base_folder + 'test\subject_test.txt',
                        header=None, 
                        names=['SubjectID'])

testX = pd.read_csv(
                     base_folder + 'test/X_test.txt', 
                     sep='\s+', 
                     header=None)

testY = pd.read_csv(
                    base_folder + 'test/y_test.txt',
                    sep=' ',
                    header=None)
testY.columns = ['ActivityID']

# combine
allSub = pd.concat([trainSub, testSub], ignore_index=True)
allX   = pd.concat([trainX, testX], ignore_index=True)
allY = trainY.append(testY, ignore_index=True) # alt method

# change column namnes
sensor_names = features.Sensor
allX.columns = sensor_names

# change activity ID for activity
allY = pd.merge(allY, a_labels, how='left', 
                left_on=['ActivityID'], right_on=['ID'])

allY = allY[['Activity']]

all = pd.concat([allSub, allX, allY], axis=1)

# save as csv for later use
all.to_csv(base_folder + "FullData_tidy.csv")

# split into train/test
xtrain, xtest, ytrain, ytest = train_test_split(all.drop('Activity', axis=1),
                                                all[['Activity']],
                                                train_size = 0.7,
                                                random_state=42
                                                )

# random forest
from sklearn.ensemble import RandomForestClassifier

# use a random forest with 3 cores
rfc = RandomForestClassifier(n_jobs=3)

rfc.fit(xtrain, ytrain.Activity)

# get predictions for our test data
predictions = rfc.predict(xtest)

# Show a confusion matrix
cm = confusion_matrix(ytest, predictions)

print(cm)
{% endhighlight %}




{% highlight text %}
[[597   0   0   0   0   0]
 [  0 538  25   0   0   0]
 [  0  52 495   0   0   0]
 [  0   0   0 533   2   4]
 [  0   0   0   2 407  11]
 [  0   0   0   9  10 405]]
{% endhighlight %}

## Conclusions

Looking at the confusion matrix above (showing predicted class along the top and actual class down the side), we can see that the random forest model works pretty well. It assigns the correct class label the majority of the time.

We could maybe improve this by trying some parameter tuning or using other algorithm. Maybe naive bayes, svm, neural networks or perhaps AdaBoost.

But the main aim of this project was to try an RStudio notebook with python, which certainly works but with the limitation of not being able to store variables between chunks, thus breaking up the flow of the document.

Another option I'd be interested in trying for combining python and text is [pweave](http://mpastell.com/pweave/). And an option I'd like to try which combines R and python (as well as many other data science related languages) is [beaker](http://beakernotebook.com/).  



.





